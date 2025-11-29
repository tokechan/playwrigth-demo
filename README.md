# Playwright 仮説検証プロジェクト

このプロジェクトは、**Playwrightを初めて使うための仮説検証**を目的としたプロジェクトです。

## 概要

Playwrightの基本的な機能や使い方を学習・検証するために作成されました。実際のプロダクション環境での導入前に、Playwrightの動作確認やテストの書き方を理解することを目的としています。

## プロジェクト構成

```
.
├── playwright.config.ts    # Playwrightの設定ファイル
├── tests/                  # テストファイル
│   └── example.spec.ts     # サンプルテスト
└── package.json
```

## セットアップ

### 1. 依存関係のインストール

```bash
npm install
```

### 2. Playwrightブラウザのインストール

```bash
npx playwright install
```

## 使い方

### テストの実行

```bash
# すべてのテストを実行
npx playwright test

# UIモードで実行（推奨）
npx playwright test --ui

# 特定のブラウザで実行
npx playwright test --project=chromium

# デバッグモードで実行
npx playwright test --debug
```

### テストレポートの確認

```bash
# HTMLレポートを表示
npx playwright show-report
```

## 設定

`playwright.config.ts`で以下の設定が可能です：

- **テストディレクトリ**: `./tests`
- **並列実行**: 有効
- **リトライ**: CI環境では2回、ローカルでは0回
- **対応ブラウザ**: Chromium, Firefox, WebKit

## テストの書き方

サンプルテストは `tests/example.spec.ts` を参照してください。

基本的なテストの構造：

```typescript
import { test, expect } from '@playwright/test';

test('テスト名', async ({ page }) => {
  await page.goto('https://example.com');
  await expect(page).toHaveTitle(/Example/);
});
```

## 参考リンク

- [Playwright公式ドキュメント](https://playwright.dev/)
- [Playwright API Reference](https://playwright.dev/docs/api/class-playwright)


