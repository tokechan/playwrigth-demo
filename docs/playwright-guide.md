# Playwright 実務リファレンスガイド

このドキュメントは、Playwrightを実務で使用する際のリファレンスです。

---

## 目次

1. [プロジェクト構成](#プロジェクト構成)
2. [基本概念](#基本概念)
3. [テストの書き方](#テストの書き方)
4. [テストの実行方法](#テストの実行方法)
5. [UIモードの活用](#uiモードの活用)
6. [CI/CD（GitHub Actions）](#cicdgithub-actions)

---

## プロジェクト構成

```
.
├── package.json            # プロジェクト設定 + パッケージ管理
├── package-lock.json       # 依存関係の正確なバージョン記録（自動生成）
├── playwright.config.ts    # Playwrightの設定ファイル
├── tests/                  # テストファイル
│   └── example.spec.ts     # テストコード
├── playwright-report/      # テストレポート（自動生成）
└── test-results/           # テスト結果（自動生成）
```

### 各ファイルの役割

| ファイル | 役割 |
|---------|------|
| `package.json` | プロジェクトの設定、npmスクリプト、パッケージ管理 |
| `package-lock.json` | パッケージの依存関係と正確なバージョンを記録（手動編集しない） |
| `playwright.config.ts` | テスト対象ブラウザ、レポート形式、タイムアウトなどの設定 |

---

## 基本概念

### テストケース vs テストコード

| 用語 | 意味 | 例 |
|------|------|-----|
| **テストケース** | 何をテストするかの**仕様・定義**（言語化したもの） | 「トップページにアクセスしたらタイトルにPlaywrightが含まれること」 |
| **テストコード** | テストケースを**実装したコード** | `await expect(page).toHaveTitle(/Playwright/);` |

**開発の流れ:**

```
① テストケース（設計）
   「何をテストするか」を言語化
        ↓
② テストコード（実装）
   テストケースをコードにする
```

### フィクスチャ（Fixture）

**テストに必要な「準備」と「後片付け」を自動でやってくれる仕組み**

```typescript
test('has title', async ({ page }) => {
  // { page } を書くだけで、ブラウザとページが自動で用意される
  await page.goto('https://example.com');
});
```

#### フィクスチャの動作

```
Before Hooks  ← フィクスチャが自動で実行（準備）
  ├── Fixture "browser"  → ブラウザを起動
  ├── Fixture "context"  → コンテキストを作成
  └── Fixture "page"     → ページを作成
        ↓
テスト本体    ← エンジニアが書いたテストコード
        ↓
After Hooks   ← フィクスチャが自動で実行（後片付け）
```

#### 主なフィクスチャ

| フィクスチャ | 説明 |
|-------------|------|
| `page` | ブラウザのページ（タブ） |
| `browser` | ブラウザ本体 |
| `context` | ブラウザコンテキスト（独立した環境） |
| `request` | APIリクエスト用 |

---

## テストの書き方

### 基本構造

```typescript
import { test, expect } from '@playwright/test';

test('テスト名', async ({ page }) => {
  // 1. ページにアクセス
  await page.goto('https://example.com');
  
  // 2. 操作を実行
  await page.getByRole('link', { name: 'Click me' }).click();
  
  // 3. 結果を検証
  await expect(page).toHaveTitle(/Expected Title/);
});
```

### import文の説明

```typescript
import { test, expect } from '@playwright/test';
```

- `test`: テストケースを定義する**関数**
- `expect`: アサーション（検証）を行う**関数**

### 実際のテスト例

```typescript
import { test, expect } from '@playwright/test';

// テストケース1: タイトルの確認
test('has title', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  await expect(page).toHaveTitle(/Playwright/);
});

// テストケース2: リンククリック後の確認
test('get started link', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  await page.getByRole('link', { name: 'Get started' }).click();
  await expect(page.getByRole('heading', { name: 'Installation' })).toBeVisible();
});
```

---

## テストの実行方法

### npmスクリプト

```bash
# 全テストを実行
npm test

# UIモードで実行（開発時推奨）
npm run test:ui

# デバッグモードで実行
npm run test:debug

# テストレポートを表示
npm run test:report
```

### 直接コマンド

```bash
# 全テストを実行
npx playwright test

# 特定のブラウザで実行
npx playwright test --project=chromium
npx playwright test --project=firefox
npx playwright test --project=webkit

# 特定のテストファイルを実行
npx playwright test tests/example.spec.ts

# ヘッドモード（ブラウザを表示して実行）
npx playwright test --headed
```

### クロスブラウザテスト

`playwright.config.ts`で設定された3つのブラウザで自動実行：

- **Chromium**（Chrome/Edge相当）
- **Firefox**
- **WebKit**（Safari相当）

例: 2つのテスト × 3ブラウザ = **6つのテスト**が実行される

---

## UIモードの活用

### 起動方法

```bash
npm run test:ui
```

### UIモードの利点

| 機能 | 説明 |
|------|------|
| **時間軸ビュー** | 各アクションを時系列で確認できる |
| **スクリーンショット** | 各ステップの画面を確認できる |
| **トレースビューア** | テストの実行を動画のように再生できる |
| **インタラクティブ実行** | 特定のテストだけを選択して実行できる |
| **デバッグ支援** | どのステップで失敗したかが視覚的にわかる |

### 開発時の推奨ワークフロー

| シーン | コマンド |
|--------|---------|
| 開発中 | `npm run test:ui` を起動してUIで確認 |
| デバッグ時 | `npm run test:debug` でステップ実行 |
| CI前の最終確認 | `npm test` で全テスト実行 |

---

## CI/CD（GitHub Actions）

### 設定ファイル

`.github/workflows/playwright.yml`

### 実行タイミング

- `main` または `master` ブランチへの **push**
- `main` または `master` ブランチへの **Pull Request**

### 実行内容

1. Ubuntuランナーでセットアップ
2. Node.js LTS版をインストール
3. `npm ci` で依存関係をインストール
4. Playwrightブラウザをインストール
5. `npx playwright test` でテスト実行
6. テストレポートをアーティファクトとして保存（30日間）

### ポイント

- PRやmain/masterへのpush時に自動でテストが実行される
- テストが失敗すると、PRのチェックが失敗として表示される
- テストレポートはGitHub Actionsのページからダウンロード可能

---

## 参考リンク

- [Playwright公式ドキュメント](https://playwright.dev/)
- [Playwright API Reference](https://playwright.dev/docs/api/class-playwright)
- [Playwright Test Configuration](https://playwright.dev/docs/test-configuration)

---

_最終更新: 2025-11-29_

