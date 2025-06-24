好的，以下是通过 AWS 管理控制台 设置 WAF，使 只有特定 IP 可以访问 CloudFront，其它一律拒绝 的详细步骤：

⸻

🌐 使用 AWS 控制台设置 WAF IP 白名单（仅允许特定 IP）

⸻

✅ 步骤 1：创建 IP Set
	1.	登录 AWS 控制台 → 搜索并打开「WAF & Shield」
	2.	左侧选择「IP sets（IP セット）」→ 点击「Create IP set」
	3.	填写以下信息：
	•	Name：例 AllowedIPs
	•	Region：选择 Global（CloudFront 用）
	•	IP version：选择 IPv4
	•	Addresses：输入允许访问的 IP，如：

203.0.113.0/24


	•	点击「Create IP set」

⸻

✅ 步骤 2：创建 Web ACL
	1.	左侧选择「Web ACLs（ウェブ ACL）」→ 点击「Create web ACL」
	2.	设置如下：
	•	Name：例如 CF-Only-Allow-Certain-IP
	•	Region：选择 CloudFront
	•	Associated AWS resource：可暂时跳过，稍后关联 CloudFront
	•	默认行为（Default action）：Block
	3.	点击「Next」进入添加规则

⸻

✅ 步骤 3：添加允许 IP 的规则
	1.	选择「Add my own rules and rule groups」
	2.	选择「IP set」类型 → 点「Add rule」
	3.	设置如下：
	•	Rule name：AllowOnlySpecificIPs
	•	IP set：选择刚刚创建的 AllowedIPs
	•	Action：Allow
	•	Rule priority：0
	4.	点击「Add rule」

⸻

✅ 步骤 4：完成设置
	1.	保持默认的 CloudWatch 可见性配置
	2.	点击「Next」直到「Review and create」
	3.	点击「Create web ACL」

⸻

✅ 步骤 5：关联 CloudFront Distribution
	1.	打开刚创建的 Web ACL 页面
	2.	下方「Associated AWS resources」→ 点击「Add AWS resource」
	3.	选择你的 CloudFront distribution
	4.	点击「Add」

⸻

🎉 设置完成！

此时，只有 IP Set 中列出的 IP 可以访问 CloudFront，其他请求都会被拦截。

⸻

📌 提示事项
	•	IP 格式必须是 CIDR（如 203.0.113.7/32 表示一个 IP）。
	•	你可以随时修改 IP set 中的地址，规则会自动更新。
	•	若前端报错，可打开 WAF 的请求日志查找被拦截的原因。

⸻

是否需要我为你补充 截图说明版（PDF） 或 CloudFront + WAF 整体示意图？


以下是上述 CloudFront に対する特定 IP 許可・その他全ブロックの WAF 設定手順の日本語版です：

⸻

🌐 AWS コンソールで WAF を設定し、特定 IP のみアクセス許可（それ以外は拒否）

⸻

✅ ステップ 1：IP セットを作成する
	1.	AWS マネジメントコンソールにログインし、「WAF & Shield」を検索して開く
	2.	左メニューから「IP sets（IP セット）」を選択 → 「IP set の作成（Create IP set）」をクリック
	3.	以下の情報を入力：
	•	Name（名前）：例）AllowedIPs
	•	Region（リージョン）：Global（CloudFront 用）
	•	IP version：IPv4 を選択
	•	Addresses（アドレス）：許可する IP を入力（例）

203.0.113.0/24


	•	最後に「Create IP set（IP セットを作成）」をクリック

⸻

✅ ステップ 2：Web ACL を作成する
	1.	左メニューの「Web ACLs（ウェブ ACL）」を選択 → 「Create web ACL（Web ACL の作成）」
	2.	以下のように設定：
	•	Name（名前）：例）CF-Only-Allow-Certain-IP
	•	Region（リージョン）：CloudFront
	•	Associated AWS resource（関連付けるリソース）：この時点ではスキップ可能（後で関連付ける）
	•	Default action（デフォルト動作）：Block（すべて拒否）
	3.	「Next（次へ）」をクリックしてルール追加画面へ

⸻

✅ ステップ 3：特定 IP のみ許可するルールを追加
	1.	「Add my own rules and rule groups（独自のルールを追加）」を選択
	2.	ルールタイプに「IP set」を選択し、「Add rule（ルールを追加）」をクリック
	3.	以下のように設定：
	•	Rule name（ルール名）：AllowOnlySpecificIPs
	•	IP set：先ほど作成した AllowedIPs を選択
	•	Action（動作）：Allow（許可）
	•	Rule priority（優先度）：0
	4.	「Add rule（ルールを追加）」をクリック

⸻

✅ ステップ 4：作成を完了
	1.	CloudWatch ログなどの設定はデフォルトのままで OK
	2.	「Next（次へ）」→「Review and create（確認して作成）」まで進み、「Create web ACL（Web ACL を作成）」をクリック

⸻

✅ ステップ 5：CloudFront と Web ACL を関連付ける
	1.	作成した Web ACL の画面を開く
	2.	下部の「Associated AWS resources（関連リソース）」→「Add AWS resource（リソースを追加）」
	3.	対象の CloudFront Distribution を選択
	4.	「Add（追加）」をクリック

⸻

🎉 設定完了！

これで、IP セットに登録された IP からのアクセスのみ許可され、それ以外はすべて拒否 されます。

⸻

📌 補足
	•	IP アドレスは CIDR 形式（例：203.0.113.7/32）で指定してください。
	•	IP set の内容を変更すれば、即座にルールに反映されます。
	•	WAF の「ログ」機能を有効にすれば、ブロックされたリクエストの詳細も確認できます。

⸻

ご希望であれば、この内容を図付きの PDF 手順書 にまとめたり、構成図 を作成することも可能です。必要でしたらお知らせください


。
