以下是 限制某个 IP 在一定时间内访问 CloudFront 频率 的设置方法（使用 AWS WAF 的 Rate-based rule），以 Markdown 格式整理：

⸻

📘 AWS WAF 设置 Rate-based Rule 限制 IP 访问频率（适用于 CloudFront）

✅ 目标

限制单个 IP 地址在 5 分钟内最多只能访问 CloudFront 100 次（WAF 最小时间粒度为 5 分钟）。

⸻

🛠 前提条件
	•	已创建 CloudFront 分发（Distribution）
	•	有权限访问 AWS WAF 控制台
	•	使用 WAF v2（非 WAF Classic）

⸻

📍 步骤一：创建 Web ACL 并关联 CloudFront
	1.	登录 AWS 管理控制台
	2.	打开服务：WAF & Shield
	3.	点击左侧菜单：Web ACLs
	4.	点击右上角：Create web ACL

填写以下内容：

项目	内容
Name	CloudFrontRateLimitACL（可自定义）
Region	Global (CloudFront)
Associated AWS resource	选择你的 CloudFront 分发（Distribution）


⸻

📍 步骤二：添加 Rate-based 规则
	1.	在 “Add rules and rule groups” 页面中点击：
Add my own rules and rule groups
	2.	点击：Add rule
	3.	选择规则类型：Rate-based rule

填写以下内容：

项目	内容
Rule name	LimitIPRate（可自定义）
Rate limit	100（表示 5 分钟内最多 100 次）
Aggregation key	IP address（每个 IP 单独统计）
Action	Block 或 Count（建议 Block）


⸻

📍 步骤三：设置规则优先级并保存
	1.	设置规则的优先级（数字越小，优先级越高）
	2.	点击 “Add rule”
	3.	回到 Web ACL 设置界面后点击：Next
	4.	确认设置无误后点击：Create web ACL

⸻

✅ 完成后效果

当某个 IP 地址在 5 分钟内请求数超过设定阈值（例如 100 次）时，WAF 会自动阻止其访问 CloudFront。

⸻

📝 注意事项
	•	Rate limit 的时间粒度为 5 分钟（AWS 固定，无法更改）
	•	只能按照 单个 IP 地址 聚合，无法组合多个字段
	•	需要等待几分钟后 WAF 才会开始生效
	•	可配合其他规则（如 IPSet、地理限制、User-Agent 过滤）进行组合防护

⸻

如果你还想加上例如 “只对特定路径（如 /api）生效” 或 “对公司 IP 白名单放行”，也可以继续添加 Scope-down statements，我可以继续为你补充规则组合说明。需要的话请说一声。
