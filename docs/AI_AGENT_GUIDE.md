# AIエージェント向けガイド - 問い合わせ対応自動化AIエージェント

このガイドは、問い合わせ対応自動化AIエージェントをメンテナンスするAIエージェント（Roocode）向けのシステム解説です。

## 目次

- [システム概要](#システム概要)
- [コードアーキテクチャ](#コードアーキテクチャ)
- [主要なクラスと関数](#主要なクラスと関数)
- [データフロー](#データフロー)
- [プロンプトエンジニアリング](#プロンプトエンジニアリング)
- [拡張ポイント](#拡張ポイント)
- [最適化のヒント](#最適化のヒント)

## システム概要

問い合わせ対応自動化AIエージェントは、Streamlitを使用したWebアプリケーションで、OpenAIのLLM（大規模言語モデル）を活用して、ユーザーからの問い合わせに対して自動的に回答を生成します。システムはRAG（Retrieval Augmented Generation）技術を使用して、社内文書から関連情報を検索し、より正確な回答を生成します。

## コードアーキテクチャ

システムは以下のモジュールで構成されています：

```
main.py                # メインアプリケーションエントリーポイント
components.py          # UI表示用コンポーネント
constants.py           # 定数定義
context_tools.py       # 文脈理解強化と回答品質評価・改善ツール
initialize.py          # 初期化処理
utils.py               # ユーティリティ関数
```

### 依存関係

```
main.py
├── utils.py
├── initialize.py
├── components.py
└── constants.py

initialize.py
├── utils.py
└── constants.py

components.py
└── constants.py

utils.py
└── constants.py

context_tools.py
└── constants.py
```

## 主要なクラスと関数

### main.py

```python
# 主要な関数と処理フロー
st.set_page_config()  # Streamlitページの設定
initialize()  # 初期化処理
cn.display_app_title()  # タイトル表示
cn.display_sidebar()  # サイドバー表示
cn.display_initial_ai_message()  # AIメッセージの初期表示
cn.display_conversation_log()  # 会話ログの表示
chat_message = st.chat_input()  # チャット入力の受け付け
if chat_message:
    # チャット送信時の処理
    # 1. ユーザーメッセージの表示
    # 2. LLMからの回答取得 or 問い合わせ処理
    # 3. 古い会話履歴を削除
    # 4. LLMからの回答表示
    # 5. 会話ログへの追加
cn.display_feedback_button()  # ユーザーフィードバックのボタン表示
```

### components.py

```python
# 主要な関数
display_app_title()  # タイトル表示
display_sidebar()  # サイドバーの表示
display_initial_ai_message()  # AIメッセージの初期表示
display_conversation_log()  # 会話ログの一覧表示
display_after_feedback_message()  # ユーザーフィードバック後のメッセージ表示
display_llm_response()  # LLMからの回答表示
display_feedback_button()  # フィードバックボタンの表示
```

### initialize.py

```python
# 主要な関数
initialize()  # 画面読み込み時に実行する初期化処理
initialize_session_state()  # 初期化データの用意
initialize_session_id()  # セッションIDの作成
initialize_logger()  # ログ出力の設定
initialize_agent_executor()  # Agent Executorの作成
```

### utils.py

```python
# 主要な関数
build_error_message()  # エラーメッセージと管理者問い合わせテンプレートの連結
create_rag_chain()  # 引数として渡されたDB内を参照するRAGのChainを作成
add_docs()  # フォルダ内のファイル一覧を取得
run_company_doc_chain()  # 会社に関するデータ参照に特化したTool設定用の関数
run_service_doc_chain()  # サービスに関するデータ参照に特化したTool設定用の関数
run_customer_doc_chain()  # 顧客とのやり取りに関するデータ参照に特化したTool設定用の関数
delete_old_conversation_log()  # 古い会話履歴の削除
execute_agent_or_chain()  # AIエージェントもしくはAIエージェントなしのRAGのChainを実行
notice_slack()  # 問い合わせ内容のSlackへの通知
```

### context_tools.py

```python
# 主要な関数
enhance_context()  # ユーザーの質問と会話履歴から文脈を強化する
assess_and_improve_answer()  # 生成された回答の品質を評価し、必要に応じて改善する
process_with_enhanced_tools()  # 文脈理解強化ツールと回答品質評価・改善ツールを統合して使用する
```

## データフロー

1. **ユーザー入力処理**:
   ```
   ユーザー入力 -> main.py -> utils.execute_agent_or_chain() -> LLM処理 -> 回答生成 -> 表示
   ```

2. **RAG処理**:
   ```
   ユーザー入力 -> utils.create_rag_chain() -> ドキュメント検索 -> 関連情報抽出 -> LLM処理 -> 回答生成
   ```

3. **AIエージェント処理**:
   ```
   ユーザー入力 -> initialize.agent_executor -> ツール選択 -> ツール実行 -> 結果評価 -> 回答生成
   ```

4. **問い合わせモード処理**:
   ```
   ユーザー入力 -> utils.notice_slack() -> 担当者選定 -> Slack通知 -> サンクスメッセージ表示
   ```

5. **強化ツール処理**:
   ```
   ユーザー入力 -> context_tools.process_with_enhanced_tools() -> 文脈強化 -> ドキュメント検索 -> 回答生成 -> 品質評価 -> 回答改善
   ```

## プロンプトエンジニアリング

システムで使用されている主要なプロンプトテンプレートは以下の通りです：

### 1. 独立したテキスト生成プロンプト

```
会話履歴と最新の入力をもとに、会話履歴なしでも理解できる独立した入力テキストを生成してください。
```

### 2. 問い合わせ対応プロンプト

```
あなたは社内文書を基に、顧客からの問い合わせに対応するアシスタントです。
以下の条件に基づき、ユーザー入力に対して回答してください。

【条件】
1. ユーザー入力内容と以下の文脈との間に関連性がある場合のみ、以下の文脈に基づいて回答してください。
2. ユーザー入力内容と以下の文脈との関連性が明らかに低い場合、「回答に必要な情報が見つかりませんでした。弊社に関する質問・要望を、入力内容を変えて送信してください。」と回答してください。
3. 憶測で回答せず、あくまで以下の文脈を元に回答してください。
4. できる限り詳細に、マークダウン記法を使って回答してください。
5. マークダウン記法で回答する際にhタグの見出しを使う場合、最も大きい見出しをh3としてください。
6. 複雑な質問の場合、各項目についてそれぞれ詳細に回答してください。
7. 必要と判断した場合は、以下の文脈に基づかずとも、一般的な情報を回答してください。

{context}
```

### 3. 従業員選択プロンプト

```
# 命令
以下の「顧客からの問い合わせ」に対して、社内のどの従業員が対応するかを
判定する生成AIシステムを作ろうとしています。

以下の「従業員情報」は、問い合わせに対しての一人以上の対応者候補のデータです。
しかし、問い合わせ内容との関連性が薄い従業員情報が含まれている可能性があります。
以下の「条件」に従い、従業員情報の中から、問い合わせ内容との関連性が特に高いと思われる
従業員の「ID」をカンマ区切りで返してください。

# 顧客からの問い合わせ
{query}

# 条件
- 全ての従業員が、問い合わせ内容との関連性が高い（対応者候補である）と判断した場合は、
全ての従業員の従業員IDをカンマ区切りで返してください。ただし、関連性が低い（対応者候補に含めるべきでない）
と判断した場合は省いてください。
- 特に、「過去の問い合わせ対応履歴」と、「対応可能な問い合わせカテゴリ」、また「現在の主要業務」を元に判定を
行ってください。
- 一人も対応者候補がいない場合、空文字を返してください。
- 判定は厳しく行ってください。

# 従業員情報
{context}

# 出力フォーマット
{format_instruction}
```

### 4. Slack通知プロンプト

```
# 役割
具体的で分量の多いメッセージの作成と、指定のメンバーにメンションを当ててSlackへの送信を行うアシスタント

# 命令
Slackの「動作検証用」チャンネルで、メンバーIDが{slack_id_text}のメンバーに一度だけメンションを当て、生成したメッセージを送信してください。

# 送信先のチャンネル名
動作検証用

# メッセージの通知先
メンバーIDが{slack_id_text}のメンバー

# メッセージ通知（メンション付け）のルール
- メッセージ通知（メンション付け）は、メッセージの先頭で「一度だけ」行ってください。
- メンション付けの行は、メンションのみとしてください。

# メッセージの生成条件
- 各項目について、できる限り長い文章量で、具体的に生成してください。

- 「メッセージフォーマット」を使い、以下の各項目の文章を生成してください。
    - 【問い合わせ情報】の「カテゴリ」
    - 【問い合わせ情報】の「日時」
    - 【メンション先の選定理由】
    - 【回答・対応案とその根拠】

- 「顧客から弊社への問い合わせ内容」と「従業員情報と過去の問い合わせ対応履歴」を基に文章を生成してください。

- 【問い合わせ情報】の「カテゴリ」は、【問い合わせ情報】の「問い合わせ内容」を基に適切なものを生成してください。

- 【メンション先の選定理由】には、なぜこの従業員が問い合わせ内容に対応するのに適しているかを詳細に説明してください。従業員の専門知識、過去の類似案件の対応経験、現在の役職や担当業務などを具体的に記載してください。

- 【回答・対応案】について、以下の条件に従って生成してください。
    - 回答・対応案の内容と、それが良いと判断した根拠を、それぞれ3つずつ生成してください。

# 顧客から弊社への問い合わせ内容
{query}

# 従業員情報と過去の問い合わせ対応履歴
{context}

# メッセージフォーマット
こちらは顧客問い合わせに対しての「担当者割り振り」と「回答・対応案の提示」を自動で行うAIアシスタントです。
担当者は問い合わせ内容を確認し、対応してください。

================================================

【問い合わせ情報】
・問い合わせ内容: {query}
・カテゴリ: 
・問い合わせ者: 山田太郎
・日時: {now_datetime}

--------------------

【メンション先の選定理由】

--------------------

【回答・対応案】
＜1つ目＞
●内容:
●根拠:

＜2つ目＞
●内容: 
●根拠: 

＜3つ目＞
●内容: 
●根拠: 

--------------------

【参照資料】
・従業員情報.csv
・問い合わせ履歴.csv
```

### 5. 文脈理解強化プロンプト

```
あなたは会話の文脈を理解し、ユーザーの意図を正確に把握するAIアシスタントです。
以下の会話履歴と最新のユーザー質問を分析し、質問の意図と文脈を強化してください。

# 会話履歴
{history_text}

# 最新のユーザー質問
{query}

# 分析タスク
1. この質問は過去の会話を参照していますか？（「はい」または「いいえ」で回答）
2. ユーザーの質問意図は何ですか？（簡潔に説明）
3. 質問に含まれる重要なキーワードを5つ以内でリストアップしてください（カンマ区切り）
4. 過去の会話を考慮した上で、より明確で具体的な質問に言い換えてください

# 出力形式
過去の会話参照: [はい/いいえ]
質問意図: [意図の説明]
重要キーワード: [キーワード1, キーワード2, ...]
強化された質問: [言い換えられた質問]
```

### 6. 回答品質評価・改善プロンプト

```
あなたは回答の品質を評価し改善するAIアシスタントです。
以下の情報を基に、生成された回答を評価し、必要に応じて改善してください。

# ユーザーの質問
{query}

# 生成された回答
{generated_answer}

# 参照情報
{reference_text}

# 評価タスク
1. 正確性（1-5）: 回答は参照情報と一致していますか？
2. 網羅性（1-5）: 回答はユーザーの質問のすべての側面に対応していますか？
3. 一貫性（1-5）: 回答に矛盾する情報はありませんか？
4. 改善点: 回答の問題点や改善すべき点を具体的に挙げてください
5. 改善された回答: 上記の評価に基づいて、改善された回答を提供してください

# 出力形式
正確性スコア: [1-5]
網羅性スコア: [1-5]
一貫性スコア: [1-5]
改善点: [改善点の説明]
改善された回答: [改善された回答の全文]
```

## 拡張ポイント

システムを拡張する際の主要なポイントは以下の通りです：

### 1. 新しいツールの追加

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

### 2. プロンプトの変更

プロンプトを変更するには、`constants.py`ファイル内の対応するプロンプトテンプレートを編集します。例えば：

```python
SYSTEM_PROMPT_INQUIRY = """
    あなたは社内文書を基に、顧客からの問い合わせに対応するアシスタントです。
    # 新しい指示や条件をここに追加
    ...
"""
```

### 3. 新しい強化ツールの追加

新しい強化ツールを追加するには、`context_tools.py`に新しい関数を追加し、`process_with_enhanced_tools`関数を更新します：

```python
def new_enhancement_tool(query, other_params):
    # 新しいツールの実装
    return result

def process_with_enhanced_tools(query, chat_history, retriever, llm):
    # 既存の処理
    
    # 新しいツールを追加
    new_enhancement_result = new_enhancement_tool(query, other_params)
    
    # 結果を統合
    # ...
    
    return final_answer, retrieved_docs
```

## 最適化のヒント

### 1. パフォーマンス最適化

- **チャンクサイズの調整**: `constants.py`の`CHUNK_SIZE`と`CHUNK_OVERLAP`を調整して、RAGのパフォーマンスを最適化できます。
- **トークン数の管理**: `delete_old_conversation_log`関数は、会話履歴のトークン数が上限を超えないように管理しています。必要に応じて`MAX_ALLOWED_TOKENS`を調整できます。

### 2. 回答品質の向上

- **プロンプトの改善**: `constants.py`のプロンプトテンプレートを改善することで、回答の品質を向上させることができます。
- **強化ツールの調整**: `context_tools.py`の強化ツールのパラメータを調整することで、文脈理解と回答品質を向上させることができます。

### 3. エラーハンドリング

- **例外処理の追加**: 各関数に適切な例外処理を追加することで、システムの堅牢性を向上させることができます。
- **フォールバックメカニズム**: 強化ツールが失敗した場合のフォールバックメカニズムを実装することで、システムの信頼性を向上させることができます。

### 4. ログ解析

- **ログレベルの調整**: `initialize_logger`関数でログレベルを調整することで、より詳細なログを取得できます。
- **ログ分析**: ログを分析することで、システムのパフォーマンスとユーザーの行動パターンを理解し、改善点を特定できます。