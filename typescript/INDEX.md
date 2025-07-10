# Typescript プロジェクトガイド

## .gitignore

gibo CLIを使います。

```bash
gibo dump macos node > .gitignore
```

以後、必要に応じてこの.gitignoreに記述を追加していきます。

## Runtime Manager

miseを使用してください。
プロジェクトルートに `.mise.toml` を作成してください。

```bash
mise use node@24 pnpm@latest
```

## Runtime

Node.jsを使用してください。バージョンは20以上を推奨します。

## Package Manager

pnpmを使用してください。

```json
// package.json
{
  "packageManager": "pnpm@10.12.2",
  "engines": {
    "node": ">=20.0.0"
  }
}
```

## package.jsonの設定

`type` を `module` にしてください。

```json
{
  "type": "module"
}
```

## Format

biomeを使用してください。

```json
// biome.json
{
  "$schema": "https://biomejs.dev/schemas/2.0.0/schema.json",
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "semicolons": "always",
      "quoteStyle": "double",
      "trailingCommas": "all",
      "arrowParentheses": "always"
    }
  },
  "linter": {
    "enabled": false
  },
  "files": {
    "ignoreUnknown": false,
    "includes": ["src/**/*", "tests/**/*", "*.js", "*.mjs"]
  }
}
```

```json
// package.json scripts
{
  "scripts": {
    "bcheck": "biome check src/",
    "bcheck:fix": "biome check --write src/"
  }
}
```

必要なdevDependencies:
- `@biomejs/biome`

## Lint

oxlint+eslintを使用してください。高速なoxlintをメインで使用しつつ、足りないルールはeslintで補う運用です。高速さを活かすために、常にoxlintを先に実行すべきです。

### oxlint

```json
// .oxlintrc.json
{
  "$schema": "./node_modules/oxlint/configuration_schema.json",

  "plugins": [
    "typescript",
    "import",
    "unicorn"
  ],

  "categories": {
    "correctness": "error",
    "suspicious": "warn",
    "style": "off",
    "pedantic": "off"
  },

  "rules": {
    "eqeqeq": "error",
    "import/no-cycle": "error",
    "no-console": "off",
    "typescript/no-explicit-any": "warn",
    "unicorn/prefer-node-protocol": "error",
    "unicorn/no-array-reduce": "off"
  },

  "env": {
    "browser": false,
    "node": true,
    "es2024": true
  },

  "ignorePatterns": ["dist/**", "coverage/**", "*.d.ts", "node_modules/**"],
  
  "overrides": [
    {
      "files": ["**/*.test.ts"],
      "rules": {
        "typescript/no-explicit-any": "off"
      }
    }
  ]
}
```

```json
// package.json scripts
{
  "scripts": {
    "oxlint": "oxlint . --max-warnings 0",
    "oxlint:fix": "oxlint . --fix --max-warnings 0"
  }
}
```

### eslint

```javascript
// eslint.config.js
import js from "@eslint/js";
import tseslint from "typescript-eslint";
import oxlint from "eslint-plugin-oxlint";
import noTypeAssertion from "eslint-plugin-no-type-assertion";

export default tseslint.config(
  js.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  ...tseslint.configs.stylisticTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        project: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
  },
  {
    plugins: {
      "no-type-assertion": noTypeAssertion,
    },
    rules: {
      "no-type-assertion/no-type-assertion": "error",
    },
  },

  // oxlintの設定の読み込みは最後に行うべき（oxlint公式の指示より）
  ...oxlint.buildFromOxlintConfigFile("./.oxlintrc.json"),
);
```

```json
// package.json scripts
{
  "scripts": {
    "eslint": "eslint . --max-warnings 0 --cache",
    "eslint:fix": "eslint . --fix --max-warnings 0 --cache",
    "lint": "pnpm run oxlint && pnpm run eslint",
    "lint:fix": "pnpm run oxlint:fix && pnpm run eslint:fix"
  }
}
```

必要なdevDependencies:
- `oxlint`
- `eslint`
- `@eslint/js`
- `typescript-eslint`
- `eslint-plugin-oxlint`
- `eslint-plugin-no-type-assertion`

## Typecheck

TypeScript コンパイラを使用します。tsgo (TypeScript Native Preview の CLI) も利用可能です。

```json
// tsconfig.json
{
  "extends": "@tsconfig/node24/tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": ".",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  },
  "include": ["**/*.ts"],
  "exclude": ["node_modules", "dist"]
}
```

```json
// package.json scripts
{
  "scripts": {
    "typecheck": "tsgo --noEmit"
  }
}
```

エディタ拡張やLSPなどの支援を受けたいのでtscもインストールしますが、CLIでは高速なtsgoを使用します。

必要なdevDependencies:
- `typescript`
- `@tsconfig/node24`
- `@typescript/native-preview` (tsgoを使う場合)

## Test

Vitestを使用します。

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      reporter: ['text', 'html'],
      exclude: ['node_modules/', 'dist/']
    }
  }
})
```

```json
// package.json scripts
{
  "scripts": {
    "test": "vitest run --silent",
    "test:coverage": "vitest run --coverage --silent",
    "test:watch": "vitest --silent"
  }
}
```

必要なdevDependencies:
- `vitest`
- `@vitest/coverage-v8`

## Bundle

tsupを使用してバンドルします。

```json
// package.json scripts
{
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts --clean"
  }
}
```

必要なdevDependencies:
- `tsup`

## Git hooks

simple-git-hooksとlint-stagedを使ってpre-commit hookを設定してください。

```json
// package.json
{
  "scripts": {
    "prepare": "pnpm exec simple-git-hooks"
  },
  "simple-git-hooks": {
    "pre-commit": "pnpm exec lint-staged"
  }
}
```

```javascript
// .lintstagedrc.js
export default {
  "*.{ts,js}": [
    "biome check --write",
    "oxlint --fix --max-warnings 0",
    "eslint --fix --max-warnings 0 --cache --no-warn-ignored",
  ],
  "*.test.ts": ["vitest run --silent=true"],
  "*.ts": [() => "tsgo --noEmit"],
  "**/*": ["secretlint", "cspell"],
  "package.json": [() => "sort-package-json"],
};
```

hookの初期化は`pnpm install`時に自動的に実行されます（prepareスクリプトにより）。

必要なdevDependencies:
- `simple-git-hooks`
- `lint-staged`
- `secretlint`
- `@secretlint/secretlint-rule-preset-recommend`
- `cspell`

## セキュリティチェック

### secretlint

コミット前に機密情報（APIキー、パスワード、トークンなど）が含まれていないかチェックします。

```json
// .secretlintrc.json
{
  "rules": [
    {
      "id": "@secretlint/secretlint-rule-preset-recommend"
    }
  ]
}
```

```json
// package.json scripts
{
  "scripts": {
    "secretlint": "secretlint \"**/*\""
  }
}
```

secretlintは`.lintstagedrc.js`で設定されているため、コミット時に自動的に実行されます。手動で実行する場合：

```bash
pnpm run secretlint
```

### cspell

コード内のスペルミスをチェックします。

```json
// cspell.json
{
  "version": "0.2",
  "language": "en",
  "words": [],
  "ignorePaths": [
    "node_modules",
    "dist",
    "build",
    "coverage",
    "*.log",
    ".git",
    ".pnpm-store",
    "pnpm-lock.yaml",
    "package-lock.json",
    "yarn.lock"
  ]
}
```

使い方:
1. 初回実行時、エラーとなった単語を確認
```bash
pnpm cspell "**/*.{md,ts,tsx,js,jsx,json}" --no-progress
```

2. プロジェクト固有の単語や技術用語をcspell.jsonのwordsに追加
```json
{
  "words": [
    "tsup",
    "vitest",
    "oxlint",
    // その他のプロジェクト固有の単語
  ]
}
```

```json
// package.json scripts
{
  "scripts": {
    "cspell": "cspell '**/*.{md,ts,tsx,js,jsx,json}' --no-progress"
  }
}
```

cspellは`.lintstagedrc.js`で設定されているため、コミット時に自動的に実行されます。

## その他の推奨ツール

### 開発ツール
- `tsx`: TypeScriptファイルの直接実行

```json
// package.json scripts
{
  "scripts": {
    "dev": "tsx src/index.ts"
  }
}
```

### package.jsonの整形
sort-package-jsonを使用してpackage.jsonのキーを一貫した順序に保ちます。

```json
// package.json scripts
{
  "scripts": {
    "sort": "sort-package-json"
  }
}
```

package.jsonを編集した後に実行:
```bash
pnpm run sort
```

必要なdevDependencies:
- `sort-package-json`

### 統合チェックコマンド

```json
// package.json scripts
{
  "scripts": {
    "check": "pnpm run bcheck && pnpm run lint && pnpm run typecheck",
    "fix": "pnpm run bcheck:fix && pnpm run lint:fix"
  }
}
```

## rulesync

AIツール（Claude、Cursor、Windsurf等）にプロジェクトのコンテキストを提供するツールです。

### インストール

```bash
pnpm add -D @dyoshikawa/rulesync
```

### 設定ファイル

#### .rulesync/overview.md

プロジェクトの概要とAIへの指示を記載します。

```markdown
---
root: true
targets: ["*"]
description: "Project overview and development philosophy"
globs: "src/**/*.ts"
---

# プロジェクト概要

[プロジェクトの説明をここに記載]

## 開発方針

- TypeScriptファーストの開発
- クリーンアーキテクチャの原則に従う
- テスト駆動開発を推奨

## AIへの指示

- 型安全性を重視したコードを生成すること
- エラーハンドリングを適切に行うこと
- 日本語のコメントを使用すること
```

#### .rulesync/.mcp.json

MCP (Model Context Protocol) サーバーの設定を記載します。

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "path/to/allowed/directory"]
    }
  }
}
```

### 使い方

```json
// package.json scripts
{
  "scripts": {
    "rulesync": "rulesync sync"
  }
}
```

設定を同期:
```bash
pnpm run rulesync
```

これにより、各AIツールの設定ファイルが自動生成されます：
- `.cursorrules` (Cursor用)
- `.windsurfrules` (Windsurf用)
- `CLAUDE.md` (Claude用)
- `.mcp/servers.json` (MCP設定)

### .gitignore

rulesyncが生成するファイルは.gitignoreに追加してください：

```
# rulesync generated files
.cursorrules
.windsurfrules
CLAUDE.md
.mcp/
```

## GitHub Actions

### CI ワークフロー

プルリクエストとmainブランチへのプッシュ時に、コード品質チェックとテストを実行します。

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  quality:
    name: Code Quality & Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 10.12.2

      - name: Install dependencies
        run: pnpm install

      - name: Run check
        run: pnpm check

      - name: Run secretlint
        run: pnpm secretlint

      - name: Run cspell
        run: pnpm cspell

      - name: Run tests
        run: pnpm test

      - name: Build project
        run: pnpm build
```

### Release ワークフロー（npmパッケージの場合）

npmパッケージとして公開する場合は、リリース時の自動公開ワークフローを設定します。

```yaml
# .github/workflows/release.yml
name: Release

on:
  release:
    types: [created]

jobs:
  publish:
    name: Publish to NPM
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          ref: main

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://registry.npmjs.org'

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 10.12.2

      - name: Install dependencies
        run: pnpm install

      - name: Run quality checks
        run: |
          pnpm check
          pnpm secretlint
          pnpm cspell
          pnpm test

      - name: Publish to npm
        run: pnpm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

必要なGitHub Secrets（npmパッケージの場合）:
- `NPM_TOKEN`: npmパブリッシュ用のアクセストークン