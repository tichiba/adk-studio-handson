# ADK Studio ハンズオンガイド

##  ハンズオンの概要

このハンズオンでは、Google Cloudが提供するAI Agent開発ツールキット「ADK (Agent Development Kit)」および、GUIツール「ADK Studio」を用いて、AIエージェントの開発、テスト、デプロイの一連の流れを体験します。

### このラボの内容
*   ADK Studioの基本的なセットアップ方法を理解する。
*   ADK Studioを使用して、GUIベースでAIエージェントを構築する方法を学ぶ。
*   作成したエージェントをテストし、動作を確認する。
*   （応用）A2A (Agent-to-Agent)連携の概念を理解し、ローカルでエージェントサーバーを起動する方法を学ぶ。
*   （応用）複数のエージェントを連携させる方法を学ぶ。

## **Part 1: ADK Studioのセットアップ**

### タスク 1: ハンズオン環境のセットアップ

まず、ハンズオンに必要なファイルをダウンロードし、セットアップスクリプトを実行します。

1.  Cloud Shellを起動し、ターミナルを開きます。

2.  `gcloud`コマンドで、使用するGoogle Cloudプロジェクトを設定します。`YOUR_PROJECT_ID`の部分はご自身のプロジェクトIDに置き換えてください。
    ```bash
    gcloud config set project YOUR_PROJECT_ID
    ```

3.  `gsutil`コマンドを使い、GCSからハンズオン用のスクリプトと資材をダウンロードします。
    ```bash
    gsutil -m cp -r gs://gcj-ai-agent-handson-part3/ ./
    ```

4.  ダウンロードした`setup.sh`スクリプトを実行します。このスクリプトは、ADKやその他必要なライブラリのインストール、環境設定を自動で行います。
    ```bash
    bash setup.sh
    ```
    スクリプトの完了には数分かかる場合があります。`Setup complete!`というメッセージが表示されれば成功です。

5.  **【重要】** `pnpm`コマンドをシステムに認識させるため、以下のコマンドを実行してシェルの設定を再読み込みしてください。
    ```bash
    source ~/.bashrc
    ```

## タスク 2: ADK Studioの起動

次に、セットアップしたADK Studioを起動します。

1.  ADK Studioのディレクトリに移動します。
    ```bash
    cd adk-studio
    ```

2.  以下のコマンドでADK Studioを起動します。
    ```bash
    pnpm dev
    ```

3.  起動ログに表示される`http://localhost:5173/`のURLにアクセスし、ADK Studioの画面を開きます。

## **Part 2: ResearchAgentの作成**

### タスク 3: RootAgentの作成

ADK StudioのGUIを使って、Web検索機能を持つ`ResearchAgent`を作成します。

1.  ADK Studio画面右上の `+ Add New` ボタンをクリックし、`+ Blank Agent` を選択します。

2.  中央に表示された `RootAgent` をクリックし、右側のPropertiesパネルに以下の情報を入力します。
    *   **description**: `あなたは高度なリサーチエージェントです。`
    *   **instruction**:
        ```
        あなたは高度なリサーチエージェントです。与えられた検索クエリに基づき、`GoogleSearchAgentTool`と`URLFetchAgentTool`を使用してウェブ検索,Webからのデータ抽出を実行してください。検索結果から関連性の高い情報を複数抽出し、それらを統合・要約して、クエリに対する包括的な「回答の草案」を生成してください。出典のURLも必ず含めてください。
        ```

## タスク 4: GoogleSearchツールの作成

1.  `GoogleSearchAgentTool` を作成します。
    *   左上の `+` ボタンから `Agent` をドラッグ＆ドロップでキャンバスに追加します。
    *   `RootAgent` の `uses tool` の口から、新しいAgentの入力口へ線を繋ぎます。
    *   新しいAgentをクリックし、Propertiesパネルを以下のように設定します。
        *   **Name**: `GoogleSearchAgentTool`
        *   **description**: `GoogleSearchToolを使用してウェブ検索を行い、結果とソースを詳細に出力するエージェントツールです。`
        *   **instruction**:
            ```
            あなたは優秀なリサーチアシスタントです。与えられたクエリに対して、Google SearchToolを使用してウェブ検索を実行してください。そして、結果から得られた情報を、要約したり元の情報を変更したりせずに、できるだけ詳細に報告してください。すべての情報について、出典のURLを明記してください。複数の出典からの情報を組み合わせる場合は、それぞれの情報がどの出典からのものかが明確にわかるように記述してください。
            ```
        *   **`Enable AgentTool` のチェックボックスをオンにします。**

2.  `GoogleSearchTool` を作成します。
    *   左上の `+` ボタンから `Tool` を追加します。
    *   `GoogleSearchAgentTool` から新しいToolへ線を繋ぎます。
    *   新しいToolのPropertiesを以下のように設定します。
        *   **Label**: `GoogleSearchTool`
        *   **Tool Type**: `Built-in Tool`
        *   **Built-in Tool**: `Google Search`

## タスク 5: URLFetchツールの作成と保存

1.  同様の手順で `URLFetchAgentTool` と `URLFetchTool` を作成します。
    *   **URLFetchAgentTool (Agent)**:
        *   `Enable AgentTool` をオンにすることを忘れないでください。
        *   **description**: `URLFetchToolを使用してウェブページへのアクセスを行い、結果とソースを詳細に出力するエージェントツールです。`
        *   **instruction**:
            ```
            あなたは優秀なリサーチアシスタントです。与えられたURL、またはURLFetchToolを使用して、そのURLのウェブページにアクセスしてください。そして、結果から得られた情報を、要約したり元の情報を変更したりせずに、できるだけ詳細に報告してください。すべての情報について、出典のURLを明記してください。複数の出典からの情報を組み合わせる場合は、それぞれの情報がどの出典からのものかが明確にわかるように記述してください。
            ```
    *   **URLFetchTool (Tool)**:
        *   **Label**: `URLFetchTool`
        *   **Tool Type**: `Built-in Tool`
        *   **Built-in Tool**: `URL Context`

2.  右上の `Save` ボタンを押し、プロジェクト名を `Research Agent` として保存します。

## タスク 6: ResearchAgentのテスト

作成したエージェントが正しく動作するか確認します。

1.  右上の `Generate Code` ボタンを押し、コードがエラーなく生成されること（緑色の文字が表示される）を確認します。

2.  `Preview` ボタンを押して、テスト用のWeb UIを開きます。

3.  チャット画面で、以下のような質問を試してみてください。
    *   `今週の東京の天気について教えて`
    *   `今日の日経平均の株価`
    *   `https://www.nomu.com/mansion_n/new/にアクセスして情報をまとめて`

エージェントがWebで情報を検索し、結果を要約して回答することを確認できれば成功です。

---

## **Part 3: (応用) A2A Agentの作成**

### タスク 7: A2Aエージェントの準備

ADK Studioで生成したコードを元に、A2A連携用のエージェントサーバーを準備します。

1.  Cloud ShellのEditorに戻ります。

2.  `sandbox/agents/research/agent.py` を開き、ADK Studioの `Generated Code` タブに表示されているPythonコードをすべてコピー＆ペーストします。

3.  `agent.py` の末尾に、以下の2行を追加します。
    ```python
    from google.adk.a2a.utils.agent_to_a2a import to_a2a
    a2a_app = to_a2a(root_agent, port=8888)
    ```

4.  新しいターミナルを開き、以下のコマンドを実行して、ローカルでA2Aサーバーを起動します。`YOUR_PROJECT_ID`はご自身のものに置き換えてください。
    ```bash
    cd sandbox
    export GOOGLE_GENAI_USE_VERTEXAI=TRUE
    export GOOGLE_CLOUD_PROJECT=YOUR_PROJECT_ID
    export GOOGLE_CLOUD_LOCATION=global
    uvicorn agents.research.agent:a2a_app --host localhost --port 8888
    ```

5.  さらに別のターミナルを開き、以下のコマンドでAgent Cardが取得できることを確認します。
    ```bash
    curl localhost:8888/.well-known/agent-card.json
    ```
    JSON形式の情報が表示されれば、サーバーは正常に動作しています。

## タスク 8: 関西弁エージェントの作成

A2Aサーバーとして起動した`ResearchAgent`をツールとして利用する、新しい「関西弁エージェント」を作成します。

1.  ADK Studioに戻り、`+ Add new` -> `Blank Agent` で新しいエージェントを作成します。

2.  `RootAgent` の設定を以下のように変更します。
    *   **Desctipiton**: `あなたは関西弁を喋るResearch Agent です。`
    *   **Instruction**:
        ```
        あなたは関西弁を喋るResearch Agent です。ResearchAgentToolを使いこなし、ユーザーから質問に対して、検索を行い、関西弁で受け答えをしてください。
        ```

3.  `ResearchAgentTool` を作成します。
    *   左上の `+` ボタンから `Agent` を追加し、`RootAgent`と接続します。
    *   新しいAgentのPropertiesを以下のように設定します。
        *   **Label**: `ResarchAgentTool`
        *   **Agent Type**: `Remote A2A Agent`
        *   **Description**: `クエリに基づいて検索/Webページへのアクセスを行うAgent`
        *   **Agent Card URL**: `http://localhost:8888/.well-known/agent-card.json`
        *   **`Enable AgentTool` のチェックボックスをオンにします。**

4.  プロジェクトをSaveし、`Generate Code` -> `Preview` で動作を確認します。先ほどと同じ質問をすると、今度は関西弁で回答が返ってくるはずです。

---

## お疲れ様でした！

以上で、ADK Studioハンズオンは終了です。エージェント開発の基本的な流れを掴むことができました。
さらに発展的なタスクとして、TemplateからAgentを作成したり、自分自身でオリジナルのAgentを作成したりすることに挑戦してみてください。
