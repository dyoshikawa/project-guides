# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

このリポジトリは、LLMに特定のプログラミング言語を使ったプロジェクト初期構築を依頼するためのガイド集です。各プログラミング言語ごとのプロジェクトセットアップに関する指示書・テンプレートを管理しています。

## リポジトリ構造

- `/typescript/` - TypeScriptプロジェクト用のガイド
  - `INDEX.md` - TypeScriptプロジェクトの構築ガイド（現在作成中）

## 開発時の注意事項

### ガイド作成時の原則

1. 各言語のガイドは `/<言語名>/INDEX.md` に配置する
2. ガイドには以下の項目を含める：
   - Runtime Manager（例：mise）
   - Package Manager（例：pnpm）
   - Format, Lint設定
   - Typecheck設定
   - Test設定
   - Bundle設定
   - Git hooks設定

### TypeScriptガイドの現在の指定事項

- **Runtime Manager**: mise
- **Package Manager**: pnpm
- **Git hooks**: simple-git-hooks + lint-staged

## このプロジェクトの性質

- 実際のアプリケーションコードではなく、ドキュメントプロジェクトである
- 各言語のベストプラクティスをテンプレート化することが目的
- ガイドは日本語で記述する