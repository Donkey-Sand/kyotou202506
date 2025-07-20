在 Amazon DynamoDB 中启用日志主要有两种方式，分别用于不同的场景：

⸻

✅ 方法一：启用 CloudWatch Contributor Insights（行为模式分析日志）

用于分析访问模式和热点分布。

开启步骤：
	1.	打开 AWS 管理控制台 → 进入 DynamoDB
	2.	选择目标表 → 左侧导航栏点击 “Contributor Insights”
	3.	点击 “Create rule”
	4.	选择要监控的项（通常选择 partition key 或 partition + sort key）
	5.	保存并启用规则

📌 日志将发送到 CloudWatch Logs，你可以在 CloudWatch 中查询和设置告警。

⸻

✅ 方法二：启用 CloudTrail 日志记录（API调用追踪）

用于记录对 DynamoDB 的所有 API 调用（如 PutItem、GetItem、DeleteItem 等）。

开启步骤：
	1.	打开 AWS 控制台 → 进入 CloudTrail
	2.	创建或使用已有的 trail（跟踪器）
	3.	确保选中 “管理事件” → 包括读写操作
	4.	保存设置后，CloudTrail 会自动将 DynamoDB 的调用记录存入指定的 S3 桶或 CloudWatch Logs

📌 这对于安全审计和操作溯源很有用。

⸻

✅ 方法三：开启 DynamoDB Streams（变更数据流日志）

如果你希望捕捉 数据变更日志（如新增、更新、删除项），可以启用 Streams。

开启步骤：
	1.	打开 DynamoDB 控制台 → 选择目标表
	2.	点击 “Exports and streams”
	3.	在 “DynamoDB Stream details” 区域点击 “Enable”
	4.	选择流的内容（如 NEW_IMAGE、OLD_IMAGE 等）
	5.	保存设置

📌 然后你可以通过 Lambda 订阅 Stream，对数据变更实时处理或记录。

⸻

总结：三种日志用途

日志类型	功能	存储位置
CloudWatch Contributor Insights	热点分析、性能优化	CloudWatch Logs
CloudTrail	审计操作、API调用记录	S3 或 CloudWatch
DynamoDB Streams	数据变更追踪	DynamoDB Stream 接口


⸻

如果你能告诉我你希望记录哪类日志（数据变更 / 性能分析 / 操作审计），我可以提供更具体的设置与代码示例。


你想记录 Lambda 对 DynamoDB 的所有操作，可以使用 CloudTrail + CloudWatch Logs 方式进行完整审计追踪。

下面是完整设置指南：

⸻

✅ 目标：

记录 Lambda 对 DynamoDB 的所有访问操作（如 PutItem / GetItem / UpdateItem 等），包括：
	•	谁访问了 DynamoDB
	•	访问了哪个表
	•	使用了什么操作（API）
	•	请求时间、来源 IP、调用者（Lambda ARN）

⸻

🔧 设置步骤（CloudTrail + CloudWatch Logs）

1️⃣ 开启 CloudTrail

CloudTrail 会记录所有对 AWS 服务的 API 调用（包括 Lambda 调用 DynamoDB 的请求）。

步骤如下：
	1.	打开 CloudTrail 控制台
	2.	选择左侧 Trails（跟踪器）
	3.	点击 [Create trail] 创建一个新的 trail，或编辑已有 trail
	4.	勾选：
	•	✅ “管理事件”（记录控制台和 SDK 的调用）
	•	✅ “读写事件” → 选择 All
	5.	选择 存储目标：
	•	日志发送到 S3 或 CloudWatch Logs（推荐）
	6.	确认并创建

⸻

2️⃣ 验证是否成功记录 Lambda → DynamoDB 操作

你可以：
	•	打开 CloudWatch Logs
	•	找到 CloudTrail 相关的日志组
	•	使用过滤条件查找，如：

eventSource = "dynamodb.amazonaws.com"
userIdentity.invokedBy = "lambda.amazonaws.com"

或在 CloudTrail 控制台中按以下方式查找：
	•	事件名称：PutItem, GetItem, UpdateItem, BatchWriteItem 等
	•	用户类型：AWSService（Lambda 会作为一个服务角色调用）
	•	调用者 ARN：Lambda 函数的 ARN

⸻

📌 可选：按表级别限制日志范围

你可以通过 CloudTrail 的 高级事件选择器 精细过滤：

{
  "Field": "resource.name",
  "Equals": ["your-dynamodb-table-name"]
}


⸻

📥 示例日志内容

{
  "eventTime": "2025-07-01T03:04:05Z",
  "eventSource": "dynamodb.amazonaws.com",
  "eventName": "PutItem",
  "userIdentity": {
    "type": "AWSService",
    "invokedBy": "lambda.amazonaws.com"
  },
  "requestParameters": {
    "tableName": "user-table"
  },
  "sourceIPAddress": "lambda.amazonaws.com",
  "userAgent": "aws-sdk-nodejs/2.1352.0"
}


⸻

✅ 总结

项目	内容
目的	记录 Lambda 对 DynamoDB 的所有读写操作
方法	使用 CloudTrail 跟踪所有 API 调用
日志存储	CloudWatch Logs / S3
可筛选字段	表名、操作类型、调用者、时间


⸻

需要我帮你写一段 CloudWatch Insights 查询语句，来筛选出 Lambda 的所有 DynamoDB 操作吗？




