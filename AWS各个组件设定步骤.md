# 📘 IAM 設定手順書

---

## ✅ 前提条件（Prerequisites）

- 管理者ユーザーでログイン済み
- AWS リソース（Lambda、Amplifyなど）の操作権限が必要

---

## ✅ Step-by-Step 手順

### Step 1：IAM ロール作成
1. IAM コンソール → 「ロール」→「ロールを作成」
2. ユースケース：「Lambda」または「Amplify」などを選択

### Step 2：ポリシーの割り当て
- Lambda 用：`AWSLambdaBasicExecutionRole`
- DynamoDB 用：`AmazonDynamoDBFullAccess`（またはカスタム）
- Amplify 用：`AWSAmplifyAdminAccess` 等

### Step 3：ポリシーのカスタマイズ（必要に応じて）
- JSON で限定的なアクセス権限を記述（最小権限の原則）

---

## 🔎 確認すべき情報まとめ

| 項目         | 例（サンプル）                |
|--------------|-------------------------------|
| ロール名      | `lambda-dynamo-role`           |
| アタッチ済ポリシー | `AWSLambdaBasicExecutionRole` 等 |

---

## 📎 参考リンク

- https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html


# 📘 Amazon S3 設定手順書（静的ファイル）

---

## ✅ 前提条件（Prerequisites）

- バケット名が他のユーザーと重複していないこと
- 公開バケットではなく、CloudFront 経由でアクセスすることを前提

---

## ✅ Step-by-Step 手順

### Step 1：バケット作成
1. S3 コンソール → 「バケットを作成」
2. バケット名：`myapp-assets`
3. リージョン：東京（ap-northeast-1）

### Step 2：パブリックアクセスブロック
- すべてのパブリックアクセスをブロック（推奨）

### Step 3：CloudFront オリジンとして設定
- OAI または OAC を使用して、CloudFront のみがアクセスできるようにバケットポリシーを設定

---

## 🔎 確認すべき情報まとめ

| 項目             | 例（サンプル）            |
|------------------|---------------------------|
| バケット名        | `myapp-assets`            |
| ホスティング形式   | 静的ウェブサイト（SPA対応） |
| オリジン使用先     | CloudFront               |

---

## 📎 参考リンク

- https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html


# 📘 CloudFront 設定手順書

---

## ✅ 前提条件（Prerequisites）

- S3 バケットが存在し、静的コンテンツが配置済み
- ドメイン名をカスタム使用する場合は、証明書（ACM）も事前に準備

---

## ✅ Step-by-Step 手順

### Step 1：ディストリビューション作成
1. CloudFront コンソール →「ディストリビューションを作成」
2. オリジンドメインに S3 バケットを指定（例：`myapp-assets.s3.amazonaws.com`）

### Step 2：ビヘイビア設定
- Viewer protocol policy：Redirect HTTP to HTTPS
- キャッシュポリシー：CachingOptimized（必要に応じて変更）

### Step 3：SPA 対応設定（404 → index.html）
- エラーページ設定で 403, 404 のレスポンスを `index.html` にリダイレクト

---

## 🔎 確認すべき情報まとめ

| 項目              | 例（サンプル）                          |
|-------------------|-----------------------------------------|
| ディストリビューションID | `E1234ABC5678Z`                        |
| アクセスドメイン       | `d1234.cloudfront.net`                 |
| オリジン名            | `myapp-assets.s3.amazonaws.com`        |

---

## 📎 参考リンク

- https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html


---

### 📘 DynamoDB 設定手順書

```markdown
# 📘 DynamoDB 設定手順書

---

## ✅ 前提条件（Prerequisites）

- 管理者 IAM アカウントでログイン済み
- Lambda または API からアクセスする必要がある

---

## ✅ Step-by-Step 手順

### Step 1：テーブル作成
1. DynamoDB コンソール → 「テーブルを作成」
2. テーブル名：`UserSearchHistory`
3. パーティションキー：`user_id`（文字列）
4. ソートキー：`timestamp`（数値）

### Step 2：読み書き設定
- キャパシティ：オンデマンドを選択（スケーラブル）

### Step 3：TTL 設定（任意）
- 属性名：`ttl` を指定し、Unix 時間で有効期限を記録

### Step 4：アクセス権限
- Lambda 実行ロールに以下のポリシー追加：
  - `dynamodb:PutItem`
  - `dynamodb:Query`

---

## 🔎 確認すべき情報まとめ

| 項目            | 例（サンプル）             |
|------------------|----------------------------|
| テーブル名        | `UserSearchHistory`        |
| パーティションキー | `user_id`（文字列）        |
| ソートキー         | `timestamp`（数値）         |

---

## 📎 参考リンク

- https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html


# 📘 AWS Lambda（FastAPI）設定手順書

---

## ✅ 前提条件（Prerequisites）

- Python および FastAPI 環境がローカルに構築されていること
- AWS CLI またはマネジメントコンソールで Lambda を操作可能であること
- IAM に Lambda 実行用ロールが存在していること

---

## ✅ Step-by-Step 手順

### Step 1：FastAPI アプリの準備
1. `main.py` に FastAPI アプリケーションを作成  
2. Lambda 対応のために `mangum` を導入し、ASGI エンドポイントを変換
   ```python
   from fastapi import FastAPI
   from mangum import Mangum

   app = FastAPI()
   handler = Mangum(app)

Step 2：依存ライブラリとコードを zip に圧縮


pip install -r requirements.txt -t ./package
cd package
zip -r ../app.zip .
cd ..
zip -g app.zip main.py

Step 3：Lambda 関数の作成
	1.	Lambda コンソールで関数を新規作成（ランタイムは Python）
	2.	上記の zip ファイルをアップロード
	3.	ハンドラー名を main.handler に設定

Step 4：IAM ロールの設定
	•	実行ロールに以下の権限を追加
	•	AWSLambdaBasicExecutionRole
	•	AmazonDynamoDBFullAccess（またはカスタム）


# 📘 API Gateway 設定手順書

---

## ✅ 前提条件（Prerequisites）

- Lambda 関数が作成済みであること
- Cognito ユーザープールが存在していること（認証用）
- 管理者 IAM ユーザーでログイン済み

---

## ✅ Step-by-Step 手順

### Step 1：REST API 作成
1. API Gateway コンソールを開く →「API を作成」→ REST API 選択
2. API 名・説明を入力し、作成

### Step 2：リソースとメソッドの追加
1. `/search`, `/history` などのリソースを追加
2. メソッド（GET/POST）を追加 → 統合先として Lambda を選択

### Step 3：Cognito 認証の設定
1. 「Authorizer」→ Cognito を追加
2. ユーザープールを指定
3. `Authorization` ヘッダーを使用してトークンを渡す設定にする

### Step 4：CORS 有効化（必要に応じて）
1. OPTIONS メソッドを手動追加
2. 必要なヘッダーをレスポンスに設定

---

## 🔎 確認すべき情報まとめ

| 項目            | 例（サンプル）                                     |
|------------------|----------------------------------------------------|
| API 名           | `user-search-api`                                  |
| 認証タイプ       | Cognito User Pool Authorizer                       |
| 認証トークン     | `Authorization: Bearer <id_token>`                 |
| API エンドポイント | `https://xxxxx.execute-api.ap-northeast-1.amazonaws.com/prod/search` |

---

## 📎 参考リンク

- https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-integrate-with-cognito.html

# 📘 AWS Cognito 設定手順書

本ドキュメントは、ユーザーログイン・認証のために AWS Cognito（ユーザープール）を使用する際の設定手順をまとめたものです。

---

## ✅ 前提条件（Prerequisites）

- AWS アカウントにログイン可能であること
- 管理者権限の IAM ユーザーを使用していること
- 使用リージョンを確認（例：ap-northeast-1）

---

## ✅ ステップ1：ユーザープールの作成

1. [Cognito コンソール](https://console.aws.amazon.com/cognito/) にアクセス  
2. 「**ユーザープールを作成**」をクリック  
3. 「ユーザープール」を選択 → 「次へ」  
4. ユーザープール名を入力（例：`user-login-pool`）  
5. **サインインオプションの設定**  
   - 「メールアドレスまたは電話番号でサインイン」を選択  
   - 「メールアドレス」にチェック  
6. **パスワードポリシー**を設定  
   - 最低8文字、英大文字/小文字、数字などを有効に  
7. **MFA（多要素認証）**：オフ（必要に応じて SMS / TOTP を有効化）  
8. **ユーザー属性**を必要に応じて追加（氏名、住所など）

---

## ✅ ステップ2：アプリクライアントの作成

1. アプリクライアントを作成（例：`frontend-client`）  
2. 「**クライアントシークレットを生成**」は**オフ**（フロントエンドで使用する場合）  
3. 作成後、App client ID を控える  

---

## ✅ ステップ3：ホスト型 UI（Hosted UI）設定（任意）

1. OAuth 2.0 を有効化：  
   - フロー：`Authorization code grant`（推奨）  
   - コールバックURL：`https://example.com/callback`  
   - サインアウトURL：`https://example.com/logout`  
2. OAuth スコープ：`email`、`openid`、`profile` にチェック

---

## ✅ ステップ4：ドメイン名の設定

1. 「ドメイン名」セクションへ移動  
2. 一意のドメイン名を入力（例：`yourapp.auth.ap-northeast-1.amazoncognito.com`）  
3. 保存すると、Hosted UI ログインURLが使用可能になる

---

## ✅ ステップ5：API Gateway またはフロントエンドとの連携

### 🔐 API Gateway に統合する場合

1. API Gateway コンソールを開く  
2. 任意の API を選択 → 「認証（Authorizer）」を追加  
3. Authorizer タイプ：`Cognito` を選択  
4. 対象ユーザープール ID、リージョン、トークンソース（`Authorization`）を指定

### 🌐 フロントエンド（例：React）と連携する場合

- AWS Amplify Auth や Amazon Cognito Identity JS を利用してログイン機能を実装  
- サインイン成功後に `ID Token` / `Access Token` を取得  
- API 呼び出し時に `Authorization: Bearer <token>` を付与

---

## 🔎 確認すべき情報まとめ

| 項目               | 値の例                                                             |
|--------------------|---------------------------------------------------------------------|
| ユーザープールID   | `ap-northeast-1_xxxxxxxx`                                           |
| アプリクライアントID | `xxxxxxxxxxxxxxxxxxxxxxxxx`                                        |
| Hosted UIログインURL | `https://yourapp.auth.ap-northeast-1.amazoncognito.com/login?...` |
| API 認証方法       | Cognito Authorizer + `Authorization: Bearer <ID Token>`             |

---

## 📎 参考リンク

- [Cognito 公式ドキュメント](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)
- [Amplify Auth の使い方](https://docs.amplify.aws/lib/auth/getting-started/q/platform/js/)
- [API Gateway × Cognito 認証設定](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-integrate-with-cognito.html)# 📘 AWS Cognito 設定手順書

本ドキュメントは、ユーザーログイン・認証のために AWS Cognito（ユーザープール）を使用する際の設定手順をまとめたものです。

---

## ✅ 前提条件（Prerequisites）

- AWS アカウントにログイン可能であること
- 管理者権限の IAM ユーザーを使用していること
- 使用リージョンを確認（例：ap-northeast-1）

---

## ✅ ステップ1：ユーザープールの作成

1. [Cognito コンソール](https://console.aws.amazon.com/cognito/) にアクセス  
2. 「**ユーザープールを作成**」をクリック  
3. 「ユーザープール」を選択 → 「次へ」  
4. ユーザープール名を入力（例：`user-login-pool`）  
5. **サインインオプションの設定**  
   - 「メールアドレスまたは電話番号でサインイン」を選択  
   - 「メールアドレス」にチェック  
6. **パスワードポリシー**を設定  
   - 最低8文字、英大文字/小文字、数字などを有効に  
7. **MFA（多要素認証）**：オフ（必要に応じて SMS / TOTP を有効化）  
8. **ユーザー属性**を必要に応じて追加（氏名、住所など）

---

## ✅ ステップ2：アプリクライアントの作成

1. アプリクライアントを作成（例：`frontend-client`）  
2. 「**クライアントシークレットを生成**」は**オフ**（フロントエンドで使用する場合）  
3. 作成後、App client ID を控える  

---

## ✅ ステップ3：ホスト型 UI（Hosted UI）設定（任意）

1. OAuth 2.0 を有効化：  
   - フロー：`Authorization code grant`（推奨）  
   - コールバックURL：`https://example.com/callback`  
   - サインアウトURL：`https://example.com/logout`  
2. OAuth スコープ：`email`、`openid`、`profile` にチェック

---

## ✅ ステップ4：ドメイン名の設定

1. 「ドメイン名」セクションへ移動  
2. 一意のドメイン名を入力（例：`yourapp.auth.ap-northeast-1.amazoncognito.com`）  
3. 保存すると、Hosted UI ログインURLが使用可能になる

---

## ✅ ステップ5：API Gateway またはフロントエンドとの連携

### 🔐 API Gateway に統合する場合

1. API Gateway コンソールを開く  
2. 任意の API を選択 → 「認証（Authorizer）」を追加  
3. Authorizer タイプ：`Cognito` を選択  
4. 対象ユーザープール ID、リージョン、トークンソース（`Authorization`）を指定

### 🌐 フロントエンド（例：React）と連携する場合

- AWS Amplify Auth や Amazon Cognito Identity JS を利用してログイン機能を実装  
- サインイン成功後に `ID Token` / `Access Token` を取得  
- API 呼び出し時に `Authorization: Bearer <token>` を付与

---

## 🔎 確認すべき情報まとめ

| 項目               | 値の例                                                             |
|--------------------|---------------------------------------------------------------------|
| ユーザープールID   | `ap-northeast-1_xxxxxxxx`                                           |
| アプリクライアントID | `xxxxxxxxxxxxxxxxxxxxxxxxx`                                        |
| Hosted UIログインURL | `https://yourapp.auth.ap-northeast-1.amazoncognito.com/login?...` |
| API 認証方法       | Cognito Authorizer + `Authorization: Bearer <ID Token>`             |

---

## 📎 参考リンク

- [Cognito 公式ドキュメント](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)
- [Amplify Auth の使い方](https://docs.amplify.aws/lib/auth/getting-started/q/platform/js/)
- [API Gateway × Cognito 認証設定](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-integrate-with-cognito.html)




