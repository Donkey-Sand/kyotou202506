✅ 一、前提条件
你已经部署好 CloudFormation，获得：

Cognito User Pool ID

Cognito User Pool Client ID

API Gateway 网址

你有一个 React 项目（用 CRA 或 Vite 初始化的都可）

✅ 二、安装依赖（项目根目录执行）
bash
复制
编辑
npm install aws-amplify @aws-amplify/ui-react
✅ 三、配置 aws-exports.js 文件
你可以手动在项目根目录下添加一个配置文件 src/aws-exports.js：

js
复制
编辑
const awsConfig = {
  Auth: {
    region: 'ap-northeast-1',
    userPoolId: 'ap-northeast-1_XXXXXXXXX',
    userPoolWebClientId: 'xxxxxxxxxxxxxxxxxxxxxxxxxx',
    authenticationFlowType: 'USER_SRP_AUTH',
  }
};

export default awsConfig;
⚠️ 请将 userPoolId 和 clientId 替换为你 CloudFormation 输出中的实际值。

✅ 四、修改前端入口代码（App.tsx）
tsx
复制
编辑
// src/App.tsx
import React, { useEffect, useState } from 'react';
import { Amplify, Auth } from 'aws-amplify';
import awsConfig from './aws-exports';
import { withAuthenticator } from '@aws-amplify/ui-react';
import '@aws-amplify/ui-react/styles.css';

Amplify.configure(awsConfig);

const App = () => {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const callApi = async () => {
      const session = await Auth.currentSession();
      const idToken = session.getIdToken().getJwtToken();

      const res = await fetch('https://your-api-id.execute-api.ap-northeast-1.amazonaws.com/v1/hello', {
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
请将上面 URL 中的 https://your-api-id... 替换为 CloudFormation 输出的 APIURL。

✅ 五、构建部署前端至 S3 + CloudFront
bash
复制
编辑
npm run build
然后你可以将 build/ 文件夹中的内容上传至你 CloudFormation 创建的 S3 Bucket：

bash
复制
编辑
aws s3 sync build/ s3://your-bucket-name --delete
（将 your-bucket-name 替换为你的 S3 Bucket 名，CloudFormation 输出中也有）

✅ 六、通过 CloudFront 访问前端页面
使用 CloudFormation 输出的 CloudFront 域名访问，即可看到登录页面，登录后访问 API。
