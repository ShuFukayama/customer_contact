# メンテナンスガイド - 問い合わせ対応自動化AIエージェント

このガイドは、問い合わせ対応自動化AIエージェントをメンテナンスする開発者向けの技術情報を提供します。

## 目次

- [システムアーキテクチャ](#システムアーキテクチャ)
- [ファイル構成](#ファイル構成)
- [主要コンポーネントの説明](#主要コンポーネントの説明)
- [環境設定](#環境設定)
- [データベース管理](#データベース管理)
- [デプロイ方法](#デプロイ方法)
- [トラブルシューティング](#トラブルシューティング)
- [拡張方法](#拡張方法)

## システムアーキテクチャ

問い合わせ対応自動化AIエージェントは、以下のコンポーネントで構成されています：

1. **フロントエンド**: Streamlitを使用したWebインターフェース
2. **バックエンド**: Python + LangChainによる処理ロジック
3. **データストア**: ChromaDBを使用したベクトルデータベース
4. **外部API連携**: OpenAI API、Slack API、SerpAPI

システムの全体的なフローは以下の通りです：

1. ユーザーがチャットインターフェースから質問を入力
2. 入力はLangChainのチェーンで処理され、必要に応じてRAG（Retrieval Augmented Generation）が適用される
3. AIエージェント機能が有効な場合、複数のツールを使用して回答を生成・評価・改善
4. 問い合わせモードがONの場合、Slack APIを使用して担当者に通知
5. 生成された回答がユーザーに表示される

## ファイル構成

```
customer_contact/
├── main.py                # メインアプリケーションエントリーポイント
├── components.py          # UI表示用コンポーネント
├── constants.py           # 定数定義
├── context_tools.py       # 文脈理解強化と回答品質評価・改善ツール
├── initialize.py          # 初期化処理
├── utils.py               # ユーティリティ関数
├── requirements_windows.txt  # Windows用依存パッケージ
├── requirements_mac.txt   # Mac用依存パッケージ
├── .env                   # 環境変数（gitignoreに含める）
├── .streamlit/            # Streamlit設定
│   └── config.toml
├── data/                  # データファイル
│   ├── rag/               # RAG用データ
│   │   ├── company/       # 会社情報
│   │   ├── customer/      # 顧客情報
│   │   └── service/       # サービス情報
│   └── slack/             # Slack連携用データ
├── images/                # 画像ファイル
│   ├── ai_icon.jpg
│   └── user_icon.jpg
└── logs/                  # ログファイル
```

## 主要コンポーネントの説明

### main.py

アプリケーションのエントリーポイントです。Streamlitアプリの設定、初期化処理、UIの表示、チャット処理のフローを管理します。

### components.py

画面表示に特化した関数を定義しています。タイトル表示、サイドバー表示、AIメッセージの初期表示、会話ログの表示、フィードバックボタンの表示などの機能を提供します。

### constants.py

固定の文字列や数値などのデータを変数として一括管理するファイルです。アプリ名、アイコンパス、プロンプトテンプレート、エラーメッセージなどが定義されています。

### context_tools.py

AIエージェントの回答精度向上のためのツールを提供します：
1. 文脈理解強化ツール（Context Enhancement Tool）
2. 回答品質評価・改善ツール（Answer Quality Assessment Tool）

### initialize.py

最初の画面読み込み時にのみ実行される初期化処理が記述されたファイルです。セッション状態の初期化、ログ出力の設定、Agent Executorの作成などを行います。

### utils.py

画面表示以外の様々な関数を定義するファイルです。RAGチェーンの作成、ドキュメントの追加と処理、各種ドキュメントチェーンの実行関数、古い会話履歴の削除、AIエージェントまたはRAGチェーンの実行、Slackへの通知機能などを提供します。

## 環境設定

### 必要な環境変数

`.env`ファイルに以下の環境変数を設定する必要があります：

```
OPENAI_API_KEY=your_openai_api_key
SLACK_USER_TOKEN=your_slack_user_token
SERPAPI_API_KEY=your_serpapi_api_key
```

### 依存パッケージ

依存パッケージは`requirements_windows.txt`または`requirements_mac.txt`に記載されています。主な依存パッケージは以下の通りです：

- streamlit: Webインターフェース
- langchain: LLMとのインテグレーション
- openai: OpenAI APIクライアント
- chromadb: ベクトルデータベース
- tiktoken: トークン数計算
- slack_sdk: Slack API連携
- PyMuPDF, docx2txt: ドキュメント処理
- SudachiPy: 日本語形態素解析

## データベース管理

### RAGデータベース

RAGデータベースは以下のディレクトリに保存されます：

- `./.db_all`: すべてのデータを含むデータベース
- `./.db_company`: 会社情報のみを含むデータベース
- `./.db_service`: サービス情報のみを含むデータベース
- `./.db_customer`: 顧客情報のみを含むデータベース

### データの追加方法

新しいデータを追加する場合は、以下の手順に従います：

1. 対応するフォルダ（`data/rag/company/`, `data/rag/service/`, `data/rag/customer/`）にPDF、DOCX、またはTXTファイルを追加します。
2. データベースを再構築するために、既存のデータベースディレクトリ（`./.db_*`）を削除します。
3. アプリケーションを再起動すると、新しいデータベースが自動的に構築されます。

## デプロイ方法

### ローカル環境

1. 仮想環境を作成し、有効化します。
2. 必要なパッケージをインストールします。
3. `.env`ファイルを作成し、必要な環境変数を設定します。
4. `streamlit run main.py`コマンドでアプリケーションを起動します。

### サーバー環境

1. サーバーにコードをデプロイします。
2. 仮想環境を作成し、必要なパッケージをインストールします。
3. `.env`ファイルを作成し、必要な環境変数を設定します。
4. Streamlitをサービスとして実行するか、Docker化して運用します。

#### Dockerを使用する場合

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements_linux.txt .
RUN pip install -r requirements_linux.txt

COPY . .

EXPOSE 8501

CMD ["streamlit", "run", "main.py"]
```

## トラブルシューティング

### 一般的な問題と解決策

1. **ModuleNotFoundError**: 必要なパッケージがインストールされていない場合は、`pip install -r requirements_windows.txt`または`pip install -r requirements_mac.txt`を実行します。

2. **API認証エラー**: `.env`ファイルに正しいAPIキーが設定されているか確認します。

3. **メモリエラー**: 大きなドキュメントを処理する場合、メモリ不足が発生する可能性があります。`constants.py`の`CHUNK_SIZE`を小さくするか、処理するドキュメントを分割します。

4. **ログの確認**: エラーが発生した場合は、`logs/application.log`ファイルを確認して詳細なエラーメッセージを確認します。

### ログ解析

アプリケーションは`logs/application.log`にログを出力します。ログには以下の情報が含まれます：

- エラーメッセージ
- ユーザー入力
- AIの回答
- フィードバック情報

## 拡張方法

### 新しいツールの追加

新しいツールを追加するには、以下の手順に従います：

1. `utils.py`に新しいツール用の関数を追加します。
2. `initialize.py`の`initialize_agent_executor`関数内の`tools`リストに新しいツールを追加します。

```python
tools = [
    # 既存のツール
    Tool(
        name="new_tool_name",
        func=utils.new_tool_function,
        description="New tool description"
    )
]
```

### プロンプトの変更

プロンプトを変更するには、`constants.py`ファイル内の対応するプロンプトテンプレートを編集します。

### UI要素の追加

新しいUI要素を追加するには、`components.py`に新しい関数を追加し、`main.py`から呼び出します。