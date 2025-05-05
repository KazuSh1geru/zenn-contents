---
title: "cline ガイドラインを精読"
emoji: "🦁"
type: "tech"
topics: ["cline", "llm", "ai"]
published: true
---

# cline ガイドラインを精読

Clineの公式ドキュメント「Improving Your Prompting Skills」をもとに、プロンプト設計から運用までのベストプラクティスを一通りまとめます。AIとの対話をより生産的にし、チームで標準化されたプロンプト運用を行うための知見を提供します。

## 1. はじめに

### 目的
このガイドは、Clineを利用するエンジニアやプロダクトオーナー、テックライターを対象に、  
- Custom Instructions の活用方法  
- 設定ファイル（.clinerules／.clineignore）の運用  
- 効果的なプロンプト設計  
- 運用フェーズのテスト＆改善サイクル  
といったポイントを解説し、具体的なサンプルや注意点を取り上げます。

### 想定読者
- Cline拡張を既に導入している開発者  
- AIアシスタント運用の効率化を目指すチームリーダー  
- プロンプトエンジニアリングに興味を持つエンジニア  

## 2. Custom Instructions の概要と設定方法

#### 定義と役割  
Custom Instructions は Cline のシステムプロンプトの末尾に自動で付与される追加指示です。AI のデフォルト動作をカスタマイズし、組織やチームのルール、専門性、トーンを常時一貫して適用できます。

#### VSCode 拡張での設定手順  
1. VSCode 左下の Cline アイコン（歯車）をクリックして設定ダイアログを開く  
2. 「Custom Instructions」フィールドに YAML またはプレーンテキストで指示を記述  
3. 記述後は「Done」をクリックして保存  
4. 保存すると、以降の全てのリクエストに自動的に適用される  

#### 効果的な記述例  
```yaml
# プロジェクトのコーディング規約適用
You are a senior engineer. Always follow our project’s coding style:
- インデントはスペース 2 つ
- 変数名は lowerCamelCase
- 関数には必ず docstring を付与

# エラーハンドリング方針
When an error occurs, log the stack trace and return a user-friendly message.
```

## 3. .clinerules ファイルの仕組みと運用コスト

#### 役割
.clinerules ファイルは、Custom Instructions をプロジェクト単位で定義・共有する設定ファイルです。チームのガイドラインやシステムメッセージを一元管理でき、VSCode 拡張を経由して自動的にプロンプトの末尾に追加されます。

#### コンテキストの蓄積方法
- ファイルに記述した内容はシステムコンテキストとして逐次送信され、各リクエストの最後に付与されます。  
- 複数行の YAML やテキストを記述可能で、大規模な指示やコードスタイルルールも管理できます。

#### トークンコストとのトレードオフ
- 長い指示を .clinerules に記述すると、その分システムコンテキストが大きくなり、トークン消費量とコストが増加します。  
- 運用コストを抑えるため、頻繁に変更しない定義のみを .clinerules に含め、動的な情報は外部ストレージや環境変数で管理することを推奨します。

#### 設定例
```yaml
# .clinerules
You are ChatGPT, an expert with domain knowledge in our product.
Always return code snippets formatted in Markdown.
```

## 4. .clineignore ファイルの使い方とベストプラクティス

#### 定義と役割  
.clineignore ファイルは、プロンプト生成時に不要なファイルやディレクトリを除外する設定ファイルです。特定のログや大容量ファイルをプロンプトに含めず、コンテキストを軽量化できます。

#### 設定方法  
1. プロジェクトルートに `.clineignore` ファイルを作成  
2. 除外したいパスを一行ずつ記述（glob パターン対応）  
3. 保存後、次回以降のリクエスト時に指定ファイルが除外される  

#### 運用ルール例  
```gitignore
# node_modules とビルド成果物を無視
node_modules/
dist/

# ログファイルを無視
logs/
*.log
```

## 5. プロンプト設計の基本原則

プロンプト設計の4つの基本原則を詳しく解説します。

### 1. 明確性（Clarity）
- 指示が曖昧だとモデルが意図しない解釈をする可能性があります。具体的かつ詳細に要件を記述しましょう。

### 2. 簡潔性（Conciseness）
- 冗長な説明はモデルの注意力を散漫にします。必要な情報に絞り、短くまとめることで性能が向上します。

### 3. コンテキスト提供（Context）
- 背景情報や前提条件を適切に提供すると、出力の一貫性と精度が向上します。例：入力データ形式や環境設定など。

### 4. 出力フォーマット指定（Output Format）
- モデルに期待する出力形式（JSON、Markdown、CSVなど）を明示的に示すことで、後続処理やパースが容易になります。

## 6. 高度なテクニック（Few-shot, Chain-of-Thought, System メッセージ）

以下のテクニックを使いこなすことで、より複雑なタスクでも高精度な出力が得られます。

### Few-shot Learning
- 数例の入力と期待される出力（ショット）を示すことで、モデルに具体的なスタイルやフォーマットを学習させます。  
- 適切な例を3〜5件程度用意し、代表的なケースを網羅すると効果が高まります。

### Chain-of-Thought プロンプト
- モデルに思考過程を生成させることで、複雑な推論タスクの精度を向上させます。  
- “Let’s think step by step” のように明示的に指示し、中間ステップを可視化します。

### System メッセージの活用
- ChatGPTやClineのシステムメッセージ機能を使い、会話の全体設計や振る舞いのガードレールを設定します。  
- ユーザー・アシスタントの役割や禁止事項を明示的に定義し、誤出力を防ぎます。

## 7. テスト＆改善サイクル

プロンプトの品質を維持するため、評価と改善を定期的に実施します。

1. 評価指標の設定  
   - 定量評価：応答速度、トークン消費量、正答率など  
   - 定性評価：出力の可読性、一貫性、ユーザー満足度調査など  

2. 誤出力パターンの検出  
   - ログのレビューやサンプル出力の比較で頻出エラーを洗い出す  
   - フォールバック回答や不正確な情報の傾向を分析  

3. 反復的チューニングフロー  
   - 課題を洗い出し、プロンプトを修正  
   - テスト環境で再評価し、改善効果を確認  
   - 成果をチームで共有し、ナレッジを蓄積

## 8. 実践例：Before／After

#### サンプルケース  
REST API からデータを取得する JavaScript 関数の生成例  

##### Before  
```text
User: "APIからユーザーデータを取得する関数を書いてください"
```

##### After  
```text
System: |
  You are a senior JavaScript engineer.  
  Write an async function named `fetchUserData(url)` that:
  - Fetch API を使って JSON を取得する  
  - HTTP ステータスコードが 200 以外の場合にはエラーを Throw  
  - try/catch でエラーを捕捉し、コンソールにスタックトレースをログ  
  - 関数には JSDoc コメントを付与  
  - コードはスペース 2 つインデントで記述  

User: "fetchUserData('https://api.example.com/users') の実装をお願いします。"
```

## 9. まとめと参考リンク

ここまでのポイントを振り返り、参考リソースを紹介します。

- **ポイント総括**  
  - Custom Instructions と設定ファイルの運用  
  - プロンプト設計の基本原則と高度テクニック  
  - テスト＆改善サイクルでプロンプト品質を維持  

- **参考リンク**  
  - Cline: Prompt Engineering Guide – https://docs.cline.bot/improving-your-prompting-skills/prompting  
  - Cline GitHub リポジトリ – https://github.com/cline-bot  
  - Zenn ガイドライン – https://zenn.dev
