AWS SSO（AWS Single Sign-On）现已更名为 IAM Identity Center（IAM 身份中心），它是 AWS 提供的一个集中身份管理服务，主要用于实现以下目的：

⸻

✅ 核心功能

功能	说明
集中管理用户访问权限	管理用户对多个 AWS 账户和应用程序的访问权限
支持外部身份提供商	可与 Azure AD、Okta、Google Workspace 等 IdP 集成
一次登录（SSO）体验	用户只需登录一次，即可访问所有授权的 AWS 账户、服务和第三方应用
集成 IAM 权限集	可为每个账户定义精细的访问权限，通过“权限集”分配


⸻

🧩 架构概览

[User] → [Identity Center (SSO Login Page)] → [IAM Permission Set] → [Target AWS Account or Application]

你可以通过以下方式管理用户：
	•	使用 AWS 内部身份目录（默认）
	•	连接现有的 IdP（SAML 2.0）
	•	手动创建用户并分配组和权限集

⸻

🛠️ 使用步骤概览（内部用户管理场景）
	1.	启用 IAM Identity Center
	•	在 AWS 控制台中启用 “IAM 身份中心”
	2.	添加用户和组
	•	创建用户、设置邮箱、初始密码（可选）
	•	创建组并分配用户
	3.	设置权限集（Permission Sets）
	•	类似于 IAM 角色，定义用户登录某个账户时的权限范围
	4.	将用户或组映射到 AWS 账户
	•	将权限集与特定账户绑定给特定用户/组
	5.	用户登录 SSO Portal
	•	用户使用链接登录 SSO Portal，即可访问已授权的 AWS 账户和应用

⸻

🌐 登录地址
	•	登录链接通常形如：

https://<your-aws-sso-instance>.awsapps.com/start



可在控制台中找到，或从邮件中获得（用户首次登录会收到激活邮件）

⸻

🔐 支持的外部身份源
	•	SAML 2.0
	•	Azure AD
	•	Okta
	•	Google Workspace
	•	Ping Identity
	•	JumpCloud 等

⸻

🎯 常见用途
	•	企业统一身份管理
	•	安全审计和访问控制
	•	多账户集中权限管理（适合组织结构复杂的企业）
	•	与 AWS Organizations 集成，实现统一多账户登录

⸻

如果你提供一下你的使用场景（比如：公司已有 Azure AD？目标是统一管理 AWS 子账户？开发者访问控制？），我可以给你更具体的实现建议和操作步骤。需要吗？
