# Gemini Enterprise ハンズオン QA 集

> 対象教材: [pottava/gemini-enterprise-handson](https://github.com/pottava/gemini-enterprise-handson) （[Wiki](https://github.com/pottava/gemini-enterprise-handson/wiki) / [agent.md](https://github.com/pottava/gemini-enterprise-handson/blob/main/agent.md)）
> 用途: ハッカソン当日の参加者対応・メンター用カンペ

---

## 1. 全般・概要

### Q. このハンズオンでは何をしますか？
A. Gemini Enterprise app を「管理者視点で構築し、ユーザー視点で利用する」一連の流れを体験します。具体的には以下の 5 セクション構成です。

1. アプリ作成（管理者視点）: サービス有効化、アプリ作成、認証設定
2. 基本機能の利用（ユーザー視点）: チャット検索、テキスト生成、グラフ作成、ファイルアップロード
3. ドキュメント連携: Google Drive 連携、内部ソース検索
4. エージェント作成: エージェント デザイナーによる自動生成・マルチエージェント
5. 終了（クリーンアップ）

さらに開発者向けの発展編として、ADK (Agent Development Kit) で作ったカスタム AI エージェントを Gemini Enterprise に登録する 12 ステップのチュートリアル（`agent.md`）があります。

### Q. 所要時間はどれくらいですか？
A. Wiki の基本編はセクションごとに進められます。発展編のカスタムエージェント登録（`agent.md`）は約 30 分が目安です（デプロイ待ち約 5 分を含む）。

### Q. プログラミング経験は必要ですか？
A. 基本編（セクション 1〜4）は GUI 操作が中心で、コーディングは不要です。発展編（`agent.md`）は Python / gcloud CLI を使うため、ターミナル操作の経験があると進めやすいです。

---

## 2. 前提条件・環境

### Q. 必要なものは何ですか？
A. 以下が必要です。

- Google Cloud プロジェクトへのアクセス権（Qwiklabs 環境を想定）
- Google Workspace アカウント（セクション 3 のドライブ・ドキュメント連携で使用）
- 発展編のみ: gcloud CLI、Python（パッケージ管理は `uv`）— Cloud Shell を使えばインストール不要

### Q. ブラウザの推奨設定はありますか？
A. **シークレット ウィンドウで開き直す**ことが推奨されています。普段使いの Google アカウントと Qwiklabs の演習用アカウントが混ざると、認証エラーや権限エラーの原因になります。また、手順中のリンクは「Ctrl（Mac は Cmd）キーを押しながらクリック」で別タブに開くと作業が捗ります。

### Q. 利用料金はかかりますか？
A. Gemini Enterprise は 30 日間の無料トライアルで開始できます。Qwiklabs 環境であれば演習用プロジェクトが提供されるため、個人課金は発生しません。

---

## 3. セクション 1: アプリ作成（管理者視点）

### Q. Gemini Enterprise はどこから有効化しますか？
A. Google Cloud コンソールの検索窓に「Gemini」と入力し、「Gemini Enterprise プロダクト」を選択 →「30 日間の無料トライアルを始める」→「続行して API を有効にする」の順に進みます。コンソールの表示言語を日本語に変更しておくと手順と一致します。

### Q. 1 つのプロジェクトに複数のアプリを作れますか？
A. はい。1 つのプロジェクトで複数アプリケーションを起動できるため、業務別・部門別にアプリを分けて配布する運用が可能です。

### Q. 認証方式は何を選びますか？
A. 「ID を設定する」から **Google Identity** を選択し、「Workforce Identity を確認する」をクリックします。

### Q. アシスタントの構成変更では何をしますか？
A. 左メニュー「構成」→「アシスタント」タブで **Enterprise web search** を選択して「Save and publish」。さらに「機能管理」で**エージェント デザイナーを有効にする**を ON にして「Save」します（セクション 4 で使うため、ここで忘れると後で戻ることになります）。

### Q. 他のメンバーにアプリを使わせるには？
A. 管理画面のユーザーライセンス設定でメンバーのメールアドレスを追加し、IAM 権限でアプリケーションへのアクセスを許可します。

---

## 4. セクション 2: 基本機能の利用（ユーザー視点）

### Q. ユーザーはどの URL からアクセスしますか？
A. 管理画面の「概要」ページにある「Gemini Enterprise ウェブ アプリの URL」をコピーし、ブラウザで開いて「使ってみる」をクリックします。

### Q. 回答の根拠（出典）は確認できますか？
A. はい。回答の「ソース」ボタンから引用元を確認できます。また、回答テキストを選択して「Gemini に相談」で部分的に深掘りすることもできます。

### Q. グラフ作成の結果表示が遅いのですが？
A. 正常です。グラフ表示時は Gemini が裏でソースコードを書いて実行した結果を表示しているため、テキスト回答より時間がかかることがあります。

### Q. ファイルから PDF を作るには？
A. 「新しいチャット」→「ファイルをアップロード」でファイルを渡し、「以下の内容に基づいて PDF を作成して:」のようにプロンプトで指示します。

---

## 5. セクション 3: ドキュメント連携（Google Drive）

### Q. Drive 連携に必要な管理者側の設定は？
A. 次の 2 つです。

1. **Google Drive API の有効化**（コンソールの API 有効化フローから）
2. **OAuth クライアントの作成**: アプリ名は任意（例: `gemini-enterprise`）、アプリケーションの種類は「**内部**」、サポートメール・連絡先は演習アカウントのメールアドレスを指定

### Q. OAuth クライアントのリダイレクト URI には何を設定しますか？
A. `https://vertexaisearch.cloud.google.com/oauth-redirect` を設定します。作成後に表示される**クライアント ID とクライアント シークレットは必ず控えて**ください（データストア作成時に入力します）。

### Q. データストアはどう作りますか？
A. Gemini Enterprise app の「データストアの作成」→「Google ドライブ」→ 範囲は「すべて」を選択 → 控えておいたクライアント ID / シークレットを入力 →「認証を確認する」→ 許可画面で「Allow」→ コネクタ名（例: `google-drive-connector`）を付けて「作成」します。

### Q. Drive 内の特定ファイルだけを検索対象にできますか？
A. はい。チャットの「新規」→「**内部ソースを検索**」を選び、「+」ボタン →「ドライブから追加」で対象ファイルを明示的に指定できます。

### Q. Gemini の回答を Google ドキュメントに貼ると崩れます
A. Markdown 形式のままコピペすると改行や記号が崩れることがあります。タイトル行は文頭に `# `（# + 半角スペース）を付けるなど、貼り付け後に体裁を整えてください。

---

## 6. セクション 4: エージェント作成（エージェント デザイナー）

### Q. エージェントはどう作りますか？
A. 左メニュー「エージェントを作成」からプロンプト欄に指示（テンプレート提供あり）を入力して実行するだけです。内容によっては自動で**複数のエージェント（マルチエージェント構成）**が生成され、「フロー」タブで構造（複数の箱）を確認できます。

### Q. ハンズオンで作る題材は？
A. 2 種類あります。

- **動画作成支援エージェント**: プロモーション動画制作を初期コンセプト〜シナリオ〜キービジュアル〜絵コンテまで支援。ユーザーと複数ステップで確認を取りながら進む対話型
- **論文調査レポートのマルチエージェント**（procedures ページ）: 編集長役の `Chief_Editor_Agent` の下に、arXiv API で直近 6 ヶ月の論文を検索する `Researcher_Agent` と、Wikipedia API で専門用語を解説する `Explainer_Agent` を配置する 3 階層構成

### Q. エージェントに画像や動画を直接生成させられますか？
A. いいえ。エージェント デザイナーのエージェントは直接的なメディア生成をサポートしていないため、ハンズオンでは Veo や Nano Banana（画像生成）向けの**プロンプトを生成する**役割に特化させています。

### Q. エージェントのデプロイが終わりません
A. デプロイには時間がかかる場合があります。数分待ってから画面を更新してください。

---

## 7. 発展編: カスタムエージェント登録（agent.md / ADK）

### Q. 何を作りますか？
A. FakeStore API を使って商品・カート・ユーザー情報を検索・照会する **EC サイト AI エージェント**を、Python + ADK で開発し、Vertex AI Agent Engine（Agent Runtime）にデプロイして Gemini Enterprise app に登録します。

### Q. 最初に設定する環境変数は？
A. `GOOGLE_CLOUD_PROJECT`、`GOOGLE_CLOUD_LOCATION`、`GOOGLE_CLOUD_GENAI_USE_VERTEXAI` の 3 つです。あわせて Vertex AI や IAM Credentials などの API を有効化し、`gcloud auth application-default login` で ADC 認証を行います。

### Q. ローカルでの動作確認方法は？
A. 以下のコマンドで ADK の Web UI を起動します。

```bash
cd ~/cloudshell_open/gemini-enterprise-handson
uv venv && source .venv/bin/activate && uv sync
adk web --allow_origins "*"
```

表示されるリンクをクリックして Web UI を開き、「どんな商品カテゴリーがありますか？」などで動作確認後、`Ctrl+C` で終了します。

### Q. 評価（テスト）はどう実行しますか？
A. ADK の評価フレームワークを使います。

```bash
adk eval store store/eval/data/eval_data1.evalset.json \
  --config_file_path store/eval/data/test_config.json
```

指標は 2 つで、`tool_trajectory_avg_score`（関数呼び出しの正確性）と `response_match_score`（応答一致度、閾値 0.7）です。失敗したら `--print_detailed_results` で詳細を確認します。

### Q. デプロイ時の注意点は？
A. 以下に注意してください。

- `GOOGLE_CLOUD_LOCATION="us-central1"` と `GOOGLE_CLOUD_STORAGE_BUCKET`（例: `ai-agents-[ユーザー名]-[タイムスタンプ]`）を追加設定する
- ストレージ バケットの作成と、サービスアカウントへの権限付与（2 コマンド）が必須
- デプロイには **5 分程度**かかる。成功すると ReasoningEngine リソースの作成通知が表示される

### Q. `AGENT_ENGINE_ID` が空になります
A. ID 取得の curl コマンドで Authorization トークンの取得に失敗している可能性が高いです。`gcloud auth print-access-token` が通るか、ADC 認証が切れていないかを確認してください。長時間作業しているとトークンの有効期限切れも起こります。

### Q. Gemini Enterprise への登録手順は？
A. 4 ステップです。

1. `agent_registration_tool` をクローン
2. 管理コンソールの URL から Gemini Enterprise の ID とロケーションを確認
3. `config.json` を作成（フィールドをすべて埋める）
4. `python3 as_registry_client.py register_agent` を実行

### Q. config.json はどこまで埋める必要がありますか？
A. `auth_id` と `icon_uri` は**空欄でも OK** ですが、それ以外のフィールドはすべて必須です。埋められないフィールドがある場合は、それ以前の手順がどこか抜けているサインです。

### Q. 登録したエージェントはどこに表示されますか？
A. Gemini Enterprise app の管理画面「エージェント」セクションに表示され、ウェブアプリから対話できるようになります。

---

## 8. トラブルシューティング（共通）

### Q. 認証エラー・権限エラーが頻発します
A. まず以下を確認してください。

- **シークレット ウィンドウ**で演習用アカウントのみでログインしているか（個人アカウントとの混在が最多原因）
- IAM でアプリケーションへのアクセス権・ライセンスが付与されているか
- ADC 認証（`gcloud auth application-default login`）をやり直す

### Q. ID 設定で「認証構成を更新できませんでした（Error while setting up AclConfig: Project's GCP Organization is not associated with a Cloud Identity customer id.）」と出ます
A. セクション 1.3 の Google Identity 設定で出る典型エラーです。Google Identity 認証には、プロジェクトが **Cloud Identity に紐づいた組織（Organization）配下**にあることが必須で、以下のどちらかが原因です。

1. プロジェクトが「組織なし」で作成されている（個人 gmail.com アカウントのプロジェクトが該当）
2. 組織はあるが Cloud Identity のカスタマー ID と関連付けられていない

対処は環境によって異なります。

- **Qwiklabs 環境**: 本来発生しません。発生した場合は演習用アカウント以外（個人アカウント）でログインしていないかを確認し、シークレット ウィンドウでやり直す
- **個人プロジェクトで検証中**: そのままでは設定不可。Cloud Identity Free にサインアップしてドメイン確認 → 組織作成 → プロジェクトを組織配下へ移動が必要（手間が大きいため、組織を持つ別環境での検証を推奨）

### Q. ロケーション関連のエラーが出ます
A. API エンドポイント URL の location パラメータの一貫性を確認してください。デプロイは `us-central1` を使うため、他のロケーションが混ざっているとエラーになります。

### Q. 環境変数が「設定したはずなのに無い」と言われます
A. Cloud Shell のセッションが切れたり別タブで作業したりすると環境変数は引き継がれません。各ステップの冒頭で `echo $GOOGLE_CLOUD_PROJECT` などで確認し、必要なら再設定してください。

### Q. 手順どおりの画面が表示されません
A. Gemini Enterprise の UI は更新が早いため、メニュー名やボタン配置が手順書と異なる場合があります。検索窓や左メニューから同等の機能名（構成 / 機能管理 / データストア / エージェント等）を探してください。

---

## 9. 参考資料：プロンプト活用ガイド

ハンズオンやハッカソンで Gemini をより効果的に使うためのプロンプトガイドです。用途に応じて参照してください。

| ガイド | 内容 | リンク |
| --- | --- | --- |
| 企業向け Gemini 活用ガイド | ビジネス現場での Gemini 活用シーンとプロンプト例 | [business-prompt.pdf](https://raw.githubusercontent.com/kkitase/hoso/main/business-prompt.pdf) |
| エンジニアのためのプロンプト活用ガイド | コーディング・設計・デバッグなど開発者向けの使いこなし | [engineer-prompt.pdf](https://raw.githubusercontent.com/kkitase/hoso/main/engineer-prompt.pdf) |
| 画像生成プロンプトガイド | 画像生成で狙いどおりの結果を得るためのプロンプト設計 | [image-prompt.pdf](https://raw.githubusercontent.com/kkitase/hoso/main/image-prompt.pdf) |
| Imagen プロンプトガイド | Imagen での画像生成プロンプトの組み立て方 | [prompt_guide_imagen.pdf](https://raw.githubusercontent.com/kkitase/hoso/main/prompt_guide_imagen.pdf) |
| 動画生成プロンプトガイド | 動画生成で意図を伝えるプロンプトのコツ | [video-prompt.pdf](https://raw.githubusercontent.com/kkitase/hoso/main/video-prompt.pdf) |
