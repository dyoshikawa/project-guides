# Typescript プロジェクトガイド

## Runtime Manager

miseを使用してください。
プロジェクトルートに `mise.toml` を作成してください。

## Runtime

Node.jsを使用してください。

## Package Manager

pnpmを使用してください。

## package.jsonの

- `type` を `module` にしてください。

## Format,Lint

## Typecheck

## Test

## Bundle

## Git hooks

- simple-git-hooks
- lint-staged

を使ってpre-commit hookを設定してください。

以下のように設定してください。

```
# package.json

"simple-git-hooks": {
  "pre-commit": "pnpm exec lint-staged"
}
```

```
# .lintstagedrc.js
```