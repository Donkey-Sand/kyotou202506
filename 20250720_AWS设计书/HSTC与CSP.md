谢谢截图！你现在处于 **CloudFront 的「响应头策略（Response headers policy）」创建界面**，一切正常 ✅。在这个界面下：

你看到的：

---

### 🔐 **Security headers - optional（安全标头，可选）**

你可以直接勾选以下几个标准安全响应头（AWS 提供快捷设置）：

| 选项                            | 作用说明                    |
| ----------------------------- | ----------------------- |
| ✅ `Strict-Transport-Security` | 启用 HSTS（强制 HTTPS）       |
| ✅ `Content-Security-Policy`   | 启用 CSP，防止 XSS 和资源注入     |
| ✅ `X-Content-Type-Options`    | 防止 MIME 类型猜测攻击（推荐）      |
| ✅ `X-Frame-Options`           | 防止点击劫持（frame embedding） |
| ✅ `X-XSS-Protection`          | 老浏览器的 XSS 过滤器（现代浏览器不推荐） |
| ✅ `Referrer-Policy`           | 控制 Referer 的发送行为        |

---

### ✅ 如何启用 HSTS 和 CSP（建议设置）

#### 步骤 1：打开 `Strict-Transport-Security` 的开关

然后会弹出详细设置项，请填入如下：

* `max-age`: `63072000`（2 年）
* ✅ Include subdomains: 勾选
* ✅ Preload: 勾选（可选，如果你打算提交 preload list）

#### 步骤 2：打开 `Content-Security-Policy` 的开关

你需要填入 CSP 的策略内容，例如：

```text
default-src 'self'; script-src 'self' https://apis.example.com
```

可根据你站点使用的 JS/CDN 实际情况做调整。

---

### ✍️ 如果你要设置自定义 Header（不在上面列表中）

可以使用下方的 `Custom headers` → 点击 `Add header` 来手动添加，例如：

```text
Header name: Permissions-Policy
Header value: geolocation=(), microphone=()
```

---

### 🔚 最后步骤

1. 设置完毕后，点击页面底部的「**Create**」
2. 回到 CloudFront 的 Distribution 设置中，把这个响应头策略关联到行为（Behaviors）中即可

---

如果你希望我帮你：

* 给出更严格或更宽松的 CSP 策略模板
* 提供完整操作过程截图说明或 CloudFormation 代码
* 翻译成日语

都可以告诉我！是否需要继续？


もちろん、以下は上記の内容を日本語に翻訳したものです：

---

## ✅ 現在の画面について（CloudFront のレスポンスヘッダーポリシー作成画面）

あなたのスクリーンショットは、**CloudFront の「Response headers policy（レスポンスヘッダーポリシー）」作成画面**です。問題なく正しい場所にいます ✅。

---

### 🔐 **Security headers - optional（セキュリティヘッダー - 任意）**

このセクションでは、AWS が用意した代表的なセキュリティヘッダーを簡単に設定できます：

| 項目                            | 説明                      |
| ----------------------------- | ----------------------- |
| ✅ `Strict-Transport-Security` | HSTS を有効化（HTTPS を強制）    |
| ✅ `Content-Security-Policy`   | CSP を設定（XSS やリソース注入を防止） |
| ✅ `X-Content-Type-Options`    | MIME タイプ推測を防止（推奨）       |
| ✅ `X-Frame-Options`           | クリックジャッキング防止            |
| ✅ `X-XSS-Protection`          | 古いブラウザ向けの簡易 XSS フィルター   |
| ✅ `Referrer-Policy`           | `Referer` の送信方法を制御      |

---

## ✅ HSTS と CSP の設定手順（おすすめ構成）

### 🔹 ステップ 1：`Strict-Transport-Security` をオンにする

スイッチをオンにすると、以下の入力欄が表示されます：

* `max-age`: `63072000`（2年間）
* ✅ サブドメインを含める：チェックを入れる
* ✅ Preload：チェック（希望する場合）

### 🔹 ステップ 2：`Content-Security-Policy` をオンにする

CSP のポリシーを入力してください。例：

```text
default-src 'self'; script-src 'self' https://apis.example.com
```

※自分のサイトで使用している JS や CDN に合わせて調整してください。

---

### ✍️ カスタムヘッダーを追加する場合

画面下の `Custom headers` セクションで「**Add header**」をクリックして、任意のヘッダーを追加できます。

例：

```text
Header name: Permissions-Policy  
Header value: geolocation=(), microphone=()
```

---

### 🔚 最後のステップ

1. すべての設定が完了したら、画面下部の「**Create**（作成）」をクリックします
2. CloudFront の「Distribution（配信）」に戻り、「Behaviors（動作）」からこのポリシーを関連付けてください

---

さらに以下のような内容が必要でしたら、お知らせください：

* より厳格または柔軟な CSP ポリシー例
* スクリーンショット付きの詳細な操作ガイド
* CloudFormation による自動化設定コード

必要があれば、どうぞご連絡ください！


