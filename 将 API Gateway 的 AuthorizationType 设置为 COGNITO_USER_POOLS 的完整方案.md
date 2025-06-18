## 将 API Gateway 的 AuthorizationType 设置为 COGNITO_USER_POOLS 的完整方案
✅ 实现目标
将你的 API Gateway 接口 /hello 设置为：

只能通过登录的 Cognito 用户访问（例如：前端 SPA 登录后获得 ID Token）

验证用户是否已登录，并限制未授权访问

📐 实现步骤概览
创建 Cognito User Pool

创建 User Pool Client

配置 API Gateway 的 Cognito Authorizer

将 AuthorizationType 设为 COGNITO_USER_POOLS

在前端使用 ID Token 调用 API

🧱 CloudFormation 模板片段（新增部分）
1. 创建 Cognito 用户池
yaml
复制
编辑
CognitoUserPool:
  Type: AWS::Cognito::UserPool
  Properties:
    UserPoolName: SampleUserPool
    AutoVerifiedAttributes:
      - email
    Policies:
      PasswordPolicy:
        MinimumLength: 8
        RequireUppercase: true
        RequireLowercase: true
        RequireNumbers: true
        RequireSymbols: false
2. 创建 User Pool Client（用于登录）
yaml
复制
编辑
CognitoUserPoolClient:
  Type: AWS::Cognito::UserPoolClient
  Properties:
    ClientName: SampleUserPoolClient
    UserPoolId: !Ref CognitoUserPool
    GenerateSecret: false
    AllowedOAuthFlows:
      - code
      - implicit
    AllowedOAuthScopes:
      - email
      - openid
      - profile
    CallbackURLs:
      - https://example.com/callback
    LogoutURLs:
      - https://example.com/logout
    SupportedIdentityProviders:
      - COGNITO
    AllowedOAuthFlowsUserPoolClient: true
3. API Gateway Authorizer（与 Cognito 连接）
yaml
复制
编辑
CognitoAuthorizer:
  Type: AWS::ApiGateway::Authorizer
  Properties:
    Name: CognitoAuthorizer
    IdentitySource: method.request.header.Authorization
    RestApiId: !Ref SimpleAPI
    Type: COGNITO_USER_POOLS
    ProviderARNs:
      - !GetAtt CognitoUserPool.Arn
🔄 修改 HelloAPIGETMethod
将原本的 AuthorizationType: NONE 改为以下配置：

yaml
复制
编辑
AuthorizationType: COGNITO_USER_POOLS
AuthorizerId: !Ref CognitoAuthorizer
完整结构如下：

yaml
复制
编辑
HelloAPIGETMethod:
  Type: 'AWS::ApiGateway::Method'
  DependsOn:
    - CFDistributionSPA
  Properties:
    AuthorizationType: COGNITO_USER_POOLS
    AuthorizerId: !Ref CognitoAuthorizer
    HttpMethod: GET
    ...
🧪 前端调用方式（概念）
在前端 SPA 中用户登录成功后，获取 Cognito 的 ID Token，然后调用 API：

javascript
复制
编辑
fetch('https://your-api-url/hello', {
  method: 'GET',
  headers: {
    'Authorization': '<ID_TOKEN_FROM_COGNITO>'
  }
});
✅ 总结（日本語まとめ）
text
复制
编辑
API Gateway に Cognito 認証を追加するには、以下の手順が必要です：

1. Cognitoユーザープールを作成
2. ユーザープールクライアント（ログイン用）を設定
3. API Gateway に「Authorizer」を追加して Cognito を連携
4. API の AuthorizationType を COGNITO_USER_POOLS に変更
5. フロントエンドで ID Token を取得し、Authorization ヘッダーに付加してAPIを呼び出す

これにより、未ログインユーザーは API にアクセスできなくなり、セキュリティが向上します。



------


整合之后的yaml文件：


```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: >
  SPA hosting with secured API using Cognito authentication

Parameters: {}

Resources:
  # Cognito User Pool
  CognitoUserPool:
    Type: AWS::Cognito::UserPool
    Properties:
      UserPoolName: SampleUserPool
      AutoVerifiedAttributes:
        - email
      Policies:
        PasswordPolicy:
          MinimumLength: 8
          RequireUppercase: true
          RequireLowercase: true
          RequireNumbers: true
          RequireSymbols: false

  # Cognito User Pool Client
  CognitoUserPoolClient:
    Type: AWS::Cognito::UserPoolClient
    Properties:
      ClientName: SampleUserPoolClient
      UserPoolId: !Ref CognitoUserPool
      GenerateSecret: false
      AllowedOAuthFlows:
        - code
        - implicit
      AllowedOAuthScopes:
        - email
        - openid
        - profile
      CallbackURLs:
        - https://example.com/callback
      LogoutURLs:
        - https://example.com/logout
      SupportedIdentityProviders:
        - COGNITO
      AllowedOAuthFlowsUserPoolClient: true

  # Simple REST API
  SimpleAPI:
    Type: 'AWS::ApiGateway::RestApi'
    Properties:
      Description: A simple REST API
      Name: SimpleAPI
      EndpointConfiguration:
        Types:
          - REGIONAL

  # API Gateway Cognito Authorizer
  CognitoAuthorizer:
    Type: AWS::ApiGateway::Authorizer
    Properties:
      Name: CognitoAuthorizer
      IdentitySource: method.request.header.Authorization
      RestApiId: !Ref SimpleAPI
      Type: COGNITO_USER_POOLS
      ProviderARNs:
        - !GetAtt CognitoUserPool.Arn

  # /hello Resource
  SimpleAPIResource:
    Type: 'AWS::ApiGateway::Resource'
    Properties:
      ParentId: !GetAtt 
        - SimpleAPI
        - RootResourceId
      PathPart: hello
      RestApiId: !Ref SimpleAPI

  # GET Method with Cognito Auth
  HelloAPIGETMethod:
    Type: 'AWS::ApiGateway::Method'
    DependsOn:
      - CFDistributionSPA
    Properties:
      AuthorizationType: COGNITO_USER_POOLS
      AuthorizerId: !Ref CognitoAuthorizer
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
              method.response.header.Access-Control-Allow-Origin: "'*'"
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

  # API Deployment
  Deployment:
    Type: 'AWS::ApiGateway::Deployment'
    DependsOn:
      - HelloAPIGETMethod
    Properties:
      RestApiId: !Ref SimpleAPI
      StageName: v1

  # Remaining S3/CloudFront/Logging bucket definitions are omitted here
  # but assumed to be integrated as in your original template.

Outputs:
  UserPoolId:
    Value: !Ref CognitoUserPool
  UserPoolClientId:
    Value: !Ref CognitoUserPoolClient
  APIURL:
    Value: !Sub "https://${SimpleAPI}.execute-api.${AWS::Region}.amazonaws.com/v1/hello"
    Description: API endpoint protected by Cognito


```

## React SPA 前端代码（使用 AWS Amplify 登录并调用 API）

```tsx
// src/App.tsx
import React, { useEffect, useState } from 'react';
import { Amplify, Auth } from 'aws-amplify';
import { withAuthenticator } from '@aws-amplify/ui-react';
import '@aws-amplify/ui-react/styles.css';

Amplify.configure({
  Auth: {
    region: 'ap-northeast-1', // 改成你的区域
    userPoolId: 'ap-northeast-1_XXXXXXXXX',
    userPoolWebClientId: 'xxxxxxxxxxxxxxxxxxxxxxxxxx',
    authenticationFlowType: 'USER_SRP_AUTH',
  },
});

const App = () => {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const callApi = async () => {
      const session = await Auth.currentSession();
      const idToken = session.getIdToken().getJwtToken();

      const res = await fetch('https://your-api-url.execute-api.ap-northeast-1.amazonaws.com/v1/hello', {
        headers: {
          Authorization: idToken,
        },
      });

      const data = await res.json();
      setMessage(data.message);
    };

    callApi();
  }, []);

  return (
    <div>
      <h1>Hello Cognito Auth API</h1>
      <p>{message}</p>
    </div>
  );
};

export default withAuthenticator(App);
```

---

- 使用 AWS Amplify 自动集成 Cognito 登录 UI
- 登录后自动获取 ID Token 并调用 `/hello` API
- 请根据你的实际 `userPoolId` 和 `clientId` 替换配置
- 构建后可上传至对应的 S3 Bucket + CloudFront

------


