# Typescript プロジェクトガイド

## Runtime Manager

miseを使用してください。
プロジェクトルートに `.mise.toml` を作成してください。

```toml
# .mise.toml
[tools]
node = "20"
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

oxlint+eslintを使用してください。

### oxlint

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
  oxlint.configs["flat/recommended"],
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
  }
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
    "typecheck": "tsc --noEmit",
    // または tsgo を使う場合
    "typecheck": "tsgo --noEmit"
  }
}
```

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
  "*.ts": [() => "tsc --noEmit", () => "pnpm test"],
  "**/*": ["secretlint", "cspell"],
};
```

hookの初期化:
```bash
pnpm exec simple-git-hooks
```

必要なdevDependencies:
- `simple-git-hooks`
- `lint-staged`
- `secretlint`
- `@secretlint/secretlint-rule-preset-recommend`
- `cspell`

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