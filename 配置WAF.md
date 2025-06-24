在 AWS 中将 WAF（Web Application Firewall） 应用于 CloudFront，可以帮助你防护如 SQL 注入、跨站脚本（XSS）、恶意 IP 等常见的 Web 攻击。以下是设置 CloudFront 绑定 WAF 的详细步骤：

⸻

✅ 前提条件
	•	已经有一个 CloudFront distribution。
	•	已经创建了一个 Web ACL（WAF 的规则集合）。

⸻

🛠 设置步骤

步骤 1：创建 Web ACL（如果还没有）
	1.	登录 AWS 管理控制台
	2.	搜索并打开 AWS WAF & Shield
	3.	点击 “Web ACLs” → “Create web ACL”
	4.	填写：
	•	Name：任意名称（如 cf-waf）
	•	Region：选择 CloudFront（此处必须选择 Global）
	5.	添加规则（你可以选择 AWS 管理规则组，或自定义规则）
	•	示例：添加 AWS Managed Rule AWSManagedRulesCommonRuleSet
	6.	默认行为选择 Allow
	7.	点击 Next，直到最后确认并 创建

⸻

步骤 2：将 Web ACL 关联到 CloudFront
	1.	在 Web ACL 列表中，点击你刚创建的 Web ACL
	2.	在“Associated AWS resources”部分点击 “Add AWS resource”
	3.	选择 CloudFront Distribution
	4.	选择你希望绑定的 CloudFront 资源
	5.	点击 Add

⸻

🔁 生效时间

更改 CloudFront 设置（包括 WAF 绑定）通常需要 5–10 分钟 来传播。

⸻

🔍 验证绑定是否成功

你可以在以下位置查看关联状态：
	•	打开你的 Web ACL → 查看“Associated AWS resources”里是否有目标 CloudFront
	•	打开 CloudFront Distribution → “Settings” → “Web Application Firewall (WAF)” 部分是否已显示绑定

⸻

📌 注意事项
	•	WAF 与 CloudFront 的绑定必须使用 Global（全球）Web ACL。
	•	若你使用的是 API Gateway、ALB 等服务，Web ACL 的 region 需要选择对应区域。
	•	一旦关联成功，所有经过 CloudFront 的请求都会通过 WAF 筛查。

⸻

如需，我可以帮你生成一个 WAF 规则模板 或 Terraform/CloudFormation 的配置文件来自动化这些设置，告诉我你使用的方式即可（控制台 / IaC）。




以下是将 CloudFront に WAF を設定する方法の日本語翻訳です：

⸻

✅ 前提条件
	•	すでに CloudFront ディストリビューション が存在していること
	•	すでに Web ACL（WAF のルールセット） を作成済みであるか、これから作成すること

⸻

🛠 設定手順

ステップ 1：Web ACL を作成（未作成の場合）
	1.	AWS マネジメントコンソールにログイン
	2.	「AWS WAF & Shield」を検索して開く
	3.	「Web ACLs」→「Create web ACL（ACL の作成）」をクリック
	4.	以下を入力：
	•	Name（名前）：例：cf-waf
	•	Region（リージョン）：CloudFront（Global） を選択
	5.	ルールを追加（AWS 管理ルールまたはカスタムルール）
	•	例：AWSManagedRulesCommonRuleSet などを選択
	6.	デフォルトの動作は「Allow（許可）」を選択
	7.	「次へ」をクリックして、最後に「Create（作成）」

⸻

ステップ 2：CloudFront に Web ACL を関連付ける
	1.	作成済みの Web ACL を開く
	2.	「Associated AWS resources（関連リソース）」セクションで「Add AWS resource（リソースを追加）」をクリック
	3.	CloudFront を選択
	4.	対象の CloudFront ディストリビューションを選ぶ
	5.	「Add（追加）」をクリック

⸻

🔁 反映時間

CloudFront の設定（WAF 連携を含む）は、通常 5〜10分 程度で世界中に反映されます。

⸻

🔍 設定確認方法
	•	Web ACL の詳細画面で「Associated AWS resources」に CloudFront が表示されているか確認
	•	CloudFront の設定画面で「Web Application Firewall (WAF)」の項目に関連付けが表示されているか確認

⸻

📌 注意点
	•	CloudFront に WAF を設定する場合は、グローバル（Global）Web ACL を選択する必要があります。
	•	API Gateway や ALB など別のサービスで WAF を使う場合は、それぞれのサービスのリージョンを指定してください。
	•	関連付けが完了すると、CloudFront 経由の全リクエストは WAF を通過して検査されます。

⸻

ご希望であれば、WAF のルールテンプレートや Terraform / CloudFormation 用の自動設定コード の作成も可能です。利用環境（コンソール or IaC）を教えていただければ対応します。












