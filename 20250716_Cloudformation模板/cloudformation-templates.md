#####
来源：
https://docs.aws.amazon.com/zh_cn/prescriptive-guidance/latest/patterns/deploy-a-react-based-single-page-application-to-amazon-s3-and-cloudfront.html
#####

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: >
  Creates the infrastructure to host and expose a Single Page Application:
      - An Amazon S3 bucket for hosting the application
      - An Amazon CloudFront distribution to expose the application and the API
      - An Amazon S3 bucket for hosting bucket and cloudfront access logs
      - A public API to be used by the application

Parameters: {}

Resources:
  # Our simple REST API
  SimpleAPI:
    Type: 'AWS::ApiGateway::RestApi'
    Properties:
      Description: A simple REST API
      Name: SimpleAPI
      EndpointConfiguration:
        Types:
          - REGIONAL

  # The Resource (/hello) of our API
  SimpleAPIResource:
    Type: 'AWS::ApiGateway::Resource'
    Properties:
      ParentId: !GetAtt 
        - SimpleAPI
        - RootResourceId
      PathPart: hello
      RestApiId: !Ref SimpleAPI

  # The method to call (GET) for our API
  HelloAPIGETMethod:
    Type: 'AWS::ApiGateway::Method'
    #checkov:skip=CKV_AWS_59: "This API does not expose backend service"
    DependsOn:
      - CFDistributionSPA
    Properties:
      ApiKeyRequired: false
      AuthorizationType: NONE
      HttpMethod: GET
      Integration:
        Type: MOCK
        PassthroughBehavior: WHEN_NO_MATCH
        RequestTemplates:
          application/json: "{\n \"statusCode\": 200\n}"
        IntegrationResponses:
          - StatusCode: 200
            SelectionPattern: 200
            ResponseParameters:
              method.response.header.Access-Control-Allow-Origin: !Sub  '''https://${CFDistributionSPA.DomainName}'''
            ResponseTemplates:
              application/json: "{\"message\": \"Hello World!\"}"
      MethodResponses:
        - StatusCode: 200
          ResponseParameters:
            method.response.header.Access-Control-Allow-Origin: true
          ResponseModels:
            application/json: Empty
      RestApiId: !Ref SimpleAPI
      ResourceId: !Ref SimpleAPIResource

  # A deployment resource for deploying our API
  Deployment:
    Type: 'AWS::ApiGateway::Deployment'
    DependsOn:
      - HelloAPIGETMethod
    Properties:
      RestApiId: !Ref SimpleAPI
      StageName: v1

  # The Amazon S3 bucket into which our Single Page Application build files must be deployed
  S3Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: !Sub 'react-cors-spa-${SimpleAPI}'
      PublicAccessBlockConfiguration:
        BlockPublicAcls : true
        BlockPublicPolicy : true
        IgnorePublicAcls : true
        RestrictPublicBuckets : true
      LoggingConfiguration:
        DestinationBucketName: !Ref LoggingBucket
        LogFilePrefix: s3-access-logs
      VersioningConfiguration:
        Status: Enabled
      BucketEncryption:
        ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault:
            SSEAlgorithm: 'AES256'

  # The Amazon S3 bucket policy for securing the bucket hosting the application
  BucketPolicy:
    Type: 'AWS::S3::BucketPolicy'
    Properties:
      PolicyDocument:
        Id: MyPolicy
        Version: 2012-10-17
        Statement:
          - Sid: PolicyForCloudFrontPrivateContent
            Effect: Allow
            Resource: !Sub ${S3Bucket.Arn}/*
            Principal:
              Service: cloudfront.amazonaws.com
            Condition:
              StringEquals:
                AWS:SourceArn: !Sub arn:aws:cloudfront::${AWS::AccountId}:distribution/${CFDistributionSPA}
            Action: 's3:GetObject*'
      Bucket: !Ref S3Bucket

  BucketPolicyForLogging:
    Type: 'AWS::S3::BucketPolicy'
    Properties:
      PolicyDocument:
        Id: S3LoggingPolicy
        Version: 2012-10-17
        Statement:
          - Sid: PolicyForS3ServerLog
            Effect: Allow
            Resource: !Sub ${LoggingBucket.Arn}/*
            Principal:
              Service: logging.s3.amazonaws.com
            Condition:
              StringEquals:
                AWS:SourceAccount: !Sub ${AWS::AccountId}
            Action: 's3:PutObject'
      Bucket: !Ref LoggingBucket

  # The Amazon S3 bucket into which access logs from S3 (for the application) and CloudFront will be put
  LoggingBucket:
    #checkov:skip=CKV_AWS_18: "This bucket is private and only for storing logs"
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: !Sub 'react-cors-spa-${SimpleAPI}-logs'
      PublicAccessBlockConfiguration:
        BlockPublicAcls : true
        BlockPublicPolicy : true
        IgnorePublicAcls : true
        RestrictPublicBuckets : true
      AccessControl: LogDeliveryWrite
      VersioningConfiguration:
        Status: Enabled
      BucketEncryption:
        ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault:
            SSEAlgorithm: 'AES256'
      OwnershipControls:
        Rules:
          - ObjectOwnership: BucketOwnerPreferred
    DeletionPolicy: Delete

  # The Amazon CloudFront distribution exposing our Single Page Application
  # Policies can be found here: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-managed-response-headers-policies.html
  CFDistributionSPA:
    #checkov:skip=CKV_AWS_68: "For demo purposes and to reduce cost, no WAF is configured"
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        Origins:
        - DomainName: !GetAtt S3Bucket.RegionalDomainName
          Id: myS3Origin
          S3OriginConfig:
            OriginAccessIdentity: ""
          OriginAccessControlId: !GetAtt CloudFrontOriginAccessControl.Id
        Enabled: 'true'
        DefaultRootObject: index.html
        DefaultCacheBehavior:
          AllowedMethods:
          - GET
          - HEAD
          - OPTIONS
          TargetOriginId: myS3Origin
          CachePolicyId: 658327ea-f89d-4fab-a63d-7e88639e58f6 # CachingOptimized
          OriginRequestPolicyId: 88a5eaf4-2fd4-4709-b370-b4c650ea3fcf # CORS-S3Origin
          ResponseHeadersPolicyId: eaab4381-ed33-4a86-88ca-d9558dc6cd63 # CORS-with-preflight-and-SecurityHeadersPolicy
          ViewerProtocolPolicy: redirect-to-https
        CustomErrorResponses:
          - ErrorCode: 403
            ResponseCode: 200
            ResponsePagePath: /index.html
          - ErrorCode: 501
            ResponseCode: 501
            ErrorCachingMinTTL: 0
        PriceClass: PriceClass_All
        Logging:
          Bucket: !GetAtt LoggingBucket.RegionalDomainName
          Prefix: 'cf-spa-access-logs'
        ViewerCertificate:
          CloudFrontDefaultCertificate: true
          MinimumProtocolVersion: 'TLSv1.2_2021'

  # The Amazon CloudFront distribution exposing our Simple REST API
  # Policies can be found here: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-managed-response-headers-policies.html
  CFDistributionAPI:
    #checkov:skip=CKV_AWS_68: "For demo purposes and to reduce cost, no WAF is configured"
    Type: 'AWS::CloudFront::Distribution'
    Properties:
      DistributionConfig:
        Origins:
        - DomainName: !Sub "${SimpleAPI}.execute-api.${AWS::Region}.amazonaws.com"
          Id: myAPIGTWOrigin
          CustomOriginConfig:
            OriginProtocolPolicy: https-only
        Enabled: 'true'
        DefaultCacheBehavior:
            AllowedMethods:
            - GET
            - HEAD
            - OPTIONS
            TargetOriginId: myAPIGTWOrigin
            CachePolicyId: 4135ea2d-6df8-44a3-9df3-4b5a84be39ad # CachingDisabled
            OriginRequestPolicyId: b689b0a8-53d0-40ab-baf2-68738e2966ac # AllViewerExceptHostHeader
            ResponseHeadersPolicyId: eaab4381-ed33-4a86-88ca-d9558dc6cd63 # CORS-with-preflight-and-SecurityHeadersPolicy
            ViewerProtocolPolicy: redirect-to-https
        CustomErrorResponses:
          - ErrorCode: 501
            ResponseCode: 501
            ErrorCachingMinTTL: 0
        PriceClass: PriceClass_All
        Logging:
          Bucket: !GetAtt LoggingBucket.RegionalDomainName
          Prefix: 'cf-api-access-logs'
        ViewerCertificate:
          CloudFrontDefaultCertificate: true
          MinimumProtocolVersion: 'TLSv1.2_2021'

  # The Amazon CloudFront origin access control
  CloudFrontOriginAccessControl:
    Type: AWS::CloudFront::OriginAccessControl
    DependsOn:
      - S3Bucket
    Properties: 
      OriginAccessControlConfig:
        Description: Default Origin Access Control
        Name: !Ref AWS::StackName
        OriginAccessControlOriginType: s3
        SigningBehavior: always
        SigningProtocol: sigv4

Outputs:
  BucketName:
    Value: !Sub "react-cors-spa-${SimpleAPI}"
  SPADomain:
    Value: !GetAtt CFDistributionSPA.DomainName
  APIDomain:
    Value: !GetAtt CFDistributionAPI.DomainName
```

这里新建的S3，有设置访问控制策略吗？还是说是公开的，所有人都可以访问？

✅ 1. PublicAccessBlockConfiguration 明确阻止公开访问
yaml
复制
编辑
PublicAccessBlockConfiguration:
  BlockPublicAcls : true
  BlockPublicPolicy : true
  IgnorePublicAcls : true
  RestrictPublicBuckets : true
这表示：

阻止所有 public ACL（即便某个对象被设置为公开）

阻止 public policy（禁止将整桶设置为公开）

忽略 public ACL 设置

禁止任何公开访问

这四个配置合起来意味着：
🛡️ 该 S3 Bucket 是私有的，任何匿名或未经授权的访问请求都会被拒绝。

✅ 2. BucketPolicy 明确授权 只有 CloudFront 可以访问
yaml
复制
编辑
Statement:
  - Sid: PolicyForCloudFrontPrivateContent
    Effect: Allow
    Resource: !Sub ${S3Bucket.Arn}/*
    Principal:
      Service: cloudfront.amazonaws.com
    Condition:
      StringEquals:
        AWS:SourceArn: !Sub arn:aws:cloudfront::${AWS::AccountId}:distribution/${CFDistributionSPA}
    Action: 's3:GetObject*'
这条策略只允许：

来源是你这个 CloudFront 分发（CFDistributionSPA）的请求

由 cloudfront.amazonaws.com 服务发起

使用 s3:GetObject* 权限读取对象内容

🔐 普通用户无法直接访问 S3 URL；只能通过 CloudFront 的 URL 访问前端页面。

✅ 结论
💡 你的 S3 Bucket 是 私有的，只允许 通过 CloudFront 分发访问，这是一种推荐的安全做法。


✅ 1. PublicAccessBlockConfiguration により公開アクセスを明確にブロック
yaml
复制
编辑
PublicAccessBlockConfiguration:
  BlockPublicAcls : true
  BlockPublicPolicy : true
  IgnorePublicAcls : true
  RestrictPublicBuckets : true
この設定により：

すべてのパブリックACLをブロック（たとえオブジェクトが公開設定されていても無効）

パブリックポリシーの適用をブロック

パブリックACLを無視

バケットを公開状態にすることを制限

この4つを組み合わせることで、
🛡️ このS3バケットは完全にプライベートになり、匿名アクセスや未認証アクセスは拒否されます。

✅ 2. BucketPolicy により CloudFront のみアクセス可能に制限
yaml
复制
编辑
Statement:
  - Sid: PolicyForCloudFrontPrivateContent
    Effect: Allow
    Resource: !Sub ${S3Bucket.Arn}/*
    Principal:
      Service: cloudfront.amazonaws.com
    Condition:
      StringEquals:
        AWS:SourceArn: !Sub arn:aws:cloudfront::${AWS::AccountId}:distribution/${CFDistributionSPA}
    Action: 's3:GetObject*'
このポリシーは以下の条件に合致するアクセスのみ許可します：

CloudFrontディストリビューション（CFDistributionSPA）からのリクエスト

サービスが cloudfront.amazonaws.com

アクションは s3:GetObject*

🔐 ユーザーがS3のURLに直接アクセスすることはできず、CloudFront経由でのみアクセス可能です。

✅ 結論
💡 このS3バケットは非公開であり、CloudFront経由のみアクセス可能です。これは推奨されるセキュリティ構成です。

もし、IAMユーザーやCognitoロールによるアクセス制御を追加したい場合は、バケットポリシーにそれらのPrincipalを追加することも可能です。必要であれば、その例もご提供しますか？


---------


上面的yaml文件里，OriginAccessIdentity: ""  这个为啥是空值呢


ChatGPT 说：
很好的问题。

在你提供的 CloudFormation 模板中：

yaml
复制
编辑
S3OriginConfig:
  OriginAccessIdentity: ""
  OriginAccessControlId: !GetAtt CloudFrontOriginAccessControl.Id
这里的 OriginAccessIdentity: "" 是一个无效或无用的设置，原因如下：

🧠 背景知识：OAI vs OAC
CloudFront 要访问私有 S3 bucket，有两种方式：

方式一：Origin Access Identity (OAI)（旧方法）
S3OriginConfig.OriginAccessIdentity 字段必须设置为：

yaml
复制
编辑
OriginAccessIdentity: origin-access-identity/cloudfront/xxxxxxxx
配合 S3 Bucket Policy 授权这个 OAI 用户访问对象

方式二：Origin Access Control (OAC)（推荐的新方法）
使用 CloudFront 的 OriginAccessControl 资源

将 OriginAccessControlId 设置给 CloudFront Distribution 中的 Origin

不需要设置 OriginAccessIdentity 字段

🛠 为什么你的模板里 OriginAccessIdentity: "" 是空的？
因为你已经正确使用了新方式（OAC）：

yaml
复制
编辑
OriginAccessControlId: !GetAtt CloudFrontOriginAccessControl.Id
但同时保留了 OriginAccessIdentity: ""，虽然这个值为空字符串，但它仍然是一个 多余的字段，甚至可能引起一些工具的误报（如 linter、checkov、cfn-lint）。

✅ 正确做法
如果你使用的是 OAC（OriginAccessControl），就应该完全删除 OriginAccessIdentity 这一行：

yaml
复制
编辑
S3OriginConfig: {}  # 或者干脆写成 null/空字典形式
✅ 总结（翻译成日语）：
CloudFront が S3 にアクセスするための方法は2種類あります：

OriginAccessIdentity（OAI）→ 古いやり方

OriginAccessControl（OAC）→ 新しく推奨されているやり方

あなたのテンプレートはすでに OriginAccessControl を正しく設定していますので、OriginAccessIdentity: "" は 不要な設定 です。
この行は削除して問題ありません。


-------------


AuthorizationType: NONE  这个值为啥这样子设置


ChatGPT 说：
AuthorizationType: NONE 这个设置出现在以下部分：

yaml
复制
编辑
HelloAPIGETMethod:
  Properties:
    AuthorizationType: NONE
    HttpMethod: GET
    ...
✅ 它的作用是：
表示这个 API Gateway 方法（即 GET /hello）不需要认证（Authorization），任何人都可以访问。

也就是说：

无需 API Key

无需 IAM 签名

无需 Cognito 用户池

无需 Lambda 授权器

✅ 为什么这么设置？
在你的 CloudFormation 模板中，这个 API 是：

json
复制
编辑
{
  "message": "Hello World!"
}
这是一个很典型的「公开测试用 API」，没有任何敏感数据，因此默认不需要访问权限控制。常见的用途包括：

静态页面调用的公共 API

用于演示或初期开发测试

跨域访问测试（CORS）

✅ 若将来想增加认证怎么办？
你可以把 AuthorizationType 设置为以下其中之一：

类型	说明
NONE	不需要认证（默认）✅
AWS_IAM	要求使用 IAM 签名 v4（常用于内网或 CLI）
COGNITO_USER_POOLS	使用 Amazon Cognito 认证用户
CUSTOM	自定义 Lambda 授权器

🔒 重要提醒
如果你的 API 最终会处理用户数据、敏感操作，或者部署到生产环境，建议不要长期使用 NONE，可以改用：

Cognito 用户池（推荐）

或 API Key / IAM 签名（取决于客户端环境）

🇯🇵 日语翻译
text
复制
编辑
`AuthorizationType: NONE` は、このAPIメソッド（GET /hello）が「認証なしで誰でもアクセス可能」であることを意味します。

つまり：
- APIキー不要
- IAM署名不要
- Cognitoユーザー認証不要
- Lambdaオーソライザー不要

今回のテンプレートでは、"Hello World!" を返すだけのシンプルなテスト用APIなので、認証を求めない設定が適しています。

ただし、将来的にユーザーデータを扱う場合や本番環境に公開する場合は、
`COGNITO_USER_POOLS` などの認証方式を導入することを強くおすすめします。










