在 AWS CloudFront 中启用 CSP（Content-Security-Policy），主要是通过设置自定义响应头实现的。

⸻

✅ 开启 CSP 的方法（CloudFront + S3 场景）

📌 方法一：使用 CloudFront 响应头策略（推荐）

CloudFront 提供了“响应头策略”功能，可以统一为 CloudFront 分发设置 HTTP 响应头。

🧾 操作步骤：

1. 登录 AWS 控制台

打开 CloudFront 控制台

2. 创建响应头策略（Response Headers Policy）
	•	左侧点击 “Policies（ポリシー）” → “Response headers policy（レスポンスヘッダーポリシー）”
	•	点击 “Create response headers policy（レスポンスヘッダーポリシーの作成）”

3. 配置内容如下：
	•	Name: CSP-Policy
	•	Security headers > Content-Security-Policy:
	•	✔️ Enable
	•	Policy value 示例（最常用配置）：

default-src 'self'; script-src 'self'; style-src 'self'; object-src 'none'; frame-ancestors 'none';



4. 绑定该策略到 CloudFront Distribution
	•	回到 CloudFront 控制台主页面
	•	选择你要修改的 Distribution → 点击“Behaviors（ビヘイビア）” → 编辑对应行为（Edit）
	•	在 “Response headers policy” 处，选择刚刚创建的 CSP-Policy
	•	保存并等待部署完成

⸻

📌 方法二：S3 设置对象级别的响应头（不推荐）

在 S3 对象的元数据中设置 Content-Security-Policy 响应头，但这种方式仅对单个对象生效，不适合大规模统一管理。

⸻

🎯 示例 CSP 字符串

用途场景	CSP 示例
最基本安全	default-src 'self';
允许加载前端 API	default-src 'self'; connect-src 'self' https://your-api.example.com;
加载字体、CSS、图片等	default-src 'self'; style-src 'self' 'unsafe-inline'; font-src 'self'; img-src 'self';


⸻

✅ 验证是否生效

部署完成后，使用浏览器 DevTools → Network → Response Headers 检查是否返回了 Content-Security-Policy 头。

或用 curl 检查：

curl -I https://your-domain.cloudfront.net


⸻

如果你有具体需求（如要允许加载 API、CDN、字体等），我也可以帮你定制 CSP 字符串。是否需要？





在 CloudFront 中启用 Strict-Transport-Security（HSTS） 同样是通过 响应头策略（Response Headers Policy） 来配置的。下面是详细步骤：

⸻

✅ HSTS（Strict-Transport-Security）开启方法（CloudFront 场景）

📌 背景说明

Strict-Transport-Security 响应头告诉浏览器 强制使用 HTTPS 访问，可防止中间人攻击或协议降级。

⸻

🧾 操作步骤

1. 打开 CloudFront 控制台

👉 https://console.aws.amazon.com/cloudfront/

2. 创建响应头策略（或编辑已有策略）
	•	点击左侧导航栏的 「Policies（ポリシー）」→「Response headers policy」
	•	点击「Create response headers policy（レスポンスヘッダーポリシーの作成）」或选择已有的策略并编辑

3. 在“Security headers”部分配置 HSTS：
	•	✔️ Strict-Transport-Security → 启用
	•	输入设置项：

设置项	示例值	说明
Max age	63072000（两年，单位：秒）	浏览器缓存 HSTS 策略的时间
Include subdomains	✔️	是否对子域名也生效
Preload	✔️	是否希望被加入到浏览器的 HSTS preload 列表（可选）

示例配置最终将产生如下响应头：

Strict-Transport-Security: max-age=63072000; includeSubDomains; preload

4. 保存策略后，回到你的 CloudFront 分发
	•	进入目标 Distribution → 选择「Behaviors」→ 编辑相关行为（Edit）
	•	在 Response headers policy 处，绑定刚才创建/修改的策略
	•	保存并部署（通常几分钟内生效）

⸻

🧪 验证是否启用成功

执行以下命令检查响应头：

curl -I https://your-domain.cloudfront.net

结果中应包含：

Strict-Transport-Security: max-age=63072000; includeSubDomains; preload


⸻

⚠️ 注意事项
	•	HSTS 只能对 HTTPS 访问生效，所以你的 CloudFront Distribution 必须启用 HTTPS。
	•	请勿对尚未全面支持 HTTPS 的网站启用 preload，否则浏览器将无法回退 HTTP。

⸻

需要我为你的前端项目（如 S3 + CloudFront）提供一份「推荐的完整 Response Headers 策略」模板吗？包括 CSP、HSTS、X-Content-Type-Options 等。


