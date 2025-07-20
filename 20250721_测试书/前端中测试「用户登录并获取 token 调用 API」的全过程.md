# 前端中测试「用户登录并获取 token 调用 API」的全过程

✅ 完整登录组件：Login.js or Login.tsx
tsx
复制
编辑
// Login.tsx（或 Login.js）
import React, { useState } from 'react';
import { Amplify, Auth } from 'aws-amplify';
import axios from 'axios';

// 你需要将这里替换为你自己的 Cognito 配置
Amplify.configure({
  Auth: {
    region: 'ap-northeast-1',
    userPoolId: 'ap-northeast-1_xxxxxxxx',
    userPoolWebClientId: 'xxxxxxxxxxxxxxxxxxxxxxxxxx',
    mandatorySignIn: true
  }
});

const Login = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [history, setHistory] = useState<string[]>([]);
  const [status, setStatus] = useState('');

  const handleLogin = async () => {
    try {
      // 用户登录
      await Auth.signIn(email, password);
      const session = await Auth.currentSession();
      const token = session.getIdToken().getJwtToken();

      setStatus("✅ 登录成功，正在获取搜索历史…");

      // 使用 token 调用后端 API（/history）
      const res = await axios.get("https://your-api-id.execute-api.ap-northeast-1.amazonaws.com/history", {
        headers: {
          Authorization: `Bearer ${token}`
        }
      });

      setHistory(res.data.history.map((item: any) => item.keyword));
      setStatus("✅ 搜索历史已加载");
    } catch (err: any) {
      console.error(err);
      setStatus("❌ 登录失败：" + err.message);
    }
  };

  return (
    <div style={{ padding: 20, maxWidth: 400 }}>
      <h2>🔐 登录</h2>
      <input
        type="email"
        placeholder="邮箱"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        style={{ width: '100%', padding: 8, marginBottom: 8 }}
      />
      <input
        type="password"
        placeholder="密码"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        style={{ width: '100%', padding: 8, marginBottom: 8 }}
      />
      <button onClick={handleLogin} style={{ padding: 10, width: '100%' }}>
        登录并获取搜索历史
      </button>

      <p style={{ marginTop: 16 }}>{status}</p>

      {history.length > 0 && (
        <>
          <h4>📜 搜索历史</h4>
          <ul>
            {history.map((kw, i) => (
              <li key={i}>{kw}</li>
            ))}
          </ul>
        </>
      )}
    </div>
  );
};

export default Login;
🧪 使用说明
把这个组件放进你的 React 项目里，例如 src/components/Login.tsx

修改里面的：

userPoolId

userPoolWebClientId

后端 API Gateway 地址（https://your-api-id...）

在主页面引入：

tsx
复制
编辑
import Login from './components/Login';

function App() {
  return (
    <div>
      <Login />
    </div>
  );
}
🚀 效果
用户填写邮箱和密码后，点击登录按钮会自动：

登录 Cognito 获取 JWT

调用 Lambda 后端 /history 接口

将搜索历史展示在页面上 ✅
