# 使用 chrome.identity 进行身份验证

## 配置

```json
{
  "permissions": ["identity"],
  "oauth2": {
    "client_id": "YOUR_CLIENT_ID.apps.googleusercontent.com",
    "scopes": [
      "https://www.googleapis.com/auth/userinfo.profile",
      "https://www.googleapis.com/auth/userinfo.email"
    ]
  }
}
```

## 获取 OAuth 令牌

```js
async function signIn() {
  return new Promise((resolve, reject) => {
    chrome.identity.getAuthToken({ interactive: true }, (token) => {
      if (chrome.runtime.lastError) {
        reject(new Error(chrome.runtime.lastError.message));
      } else {
        resolve(token);
      }
    });
  });
}
```

或使用基于 Promise 的 API（Chrome 116+）：
```js
const { token } = await chrome.identity.getAuthToken({ interactive: true });
```

## 获取用户资料

```js
async function getUserProfile(token) {
  const response = await fetch('https://www.googleapis.com/oauth2/v3/userinfo', {
    headers: { Authorization: `Bearer ${token}` }
  });
  if (!response.ok) throw new Error('Failed to fetch profile');
  return response.json();
  // Returns: { sub, name, given_name, family_name, picture, email, email_verified }
}
```

## 退出登录

```js
async function signOut(token) {
  // Remove cached token
  await chrome.identity.removeCachedAuthToken({ token });

  // Optionally revoke the token server-side
  await fetch(`https://accounts.google.com/o/oauth2/revoke?token=${token}`);
}
```

## 错误处理

```js
try {
  const { token } = await chrome.identity.getAuthToken({ interactive: true });
  const profile = await getUserProfile(token);
  displayProfile(profile);
} catch (err) {
  if (err.message.includes('canceled')) {
    showMessage('Sign-in was cancelled');
  } else if (err.message.includes('not granted')) {
    showMessage('Permission was denied');
  } else {
    showMessage('Sign-in failed: ' + err.message);
  }
}
```

## 配置 Google Cloud Console

1. 访问 console.cloud.google.com
2. 创建项目（或选择现有项目）
3. 启用 "Google People API" 或 "Google OAuth2 API"
4. 创建 OAuth 2.0 凭据 → Chrome Extension 类型
5. 将 Application ID 设为扩展的 ID
6. 将 client_id 复制到 manifest.json

### 扩展 ID：开发环境与生产环境

**这一点至关重要，且常被忽略。** OAuth `client_id` 绑定到特定的扩展 ID。
扩展 ID 会因加载方式不同而变化：

| 场景 | ID 的确定方式 |
|---------|---------------------|
| 未打包（开发） | 由扩展的目录路径派生——移动文件夹后会变化 |
| 已打包（.crx） | 由打包所用的私钥派生 |
| Chrome Web Store | 由商店分配，永久不变 |

**要在开发期间获得稳定的 ID**，请在 manifest.json 中添加 `"key"` 字段。
这样无论目录路径如何，扩展 ID 都保持一致：

1. 打包一次扩展（`chrome://extensions` → Pack Extension）
2. 将生成的 `.crx` 作为 ZIP 打开，从其 manifest 中提取 `key`
3. 将该 key 添加到开发用的 manifest：

```json
{
  "key": "MIIBIjANBgkqhk...your-public-key-here...",
  "manifest_version": 3,
  "name": "My Extension"
}
```

或者，从 `chrome://extensions` 记下未打包扩展的 ID，并为该特定 ID 配置
OAuth 客户端。但要注意，文件夹移动后 ID 会变化。

**务必告知用户：** "发布到 Chrome Web Store 后，请用商店分配的扩展 ID
更新 OAuth 客户端配置。"

## 非 Google OAuth（launchWebAuthFlow）

对于第三方 OAuth 提供商（GitHub、Twitter 等）：

```js
const redirectUrl = chrome.identity.getRedirectURL();
// Returns: https://<extension-id>.chromiumapp.org/

const authUrl = `https://github.com/login/oauth/authorize?client_id=XXX&redirect_uri=${redirectUrl}`;

const responseUrl = await chrome.identity.launchWebAuthFlow({
  url: authUrl,
  interactive: true
});

// Parse the token from responseUrl
const url = new URL(responseUrl);
const code = url.searchParams.get('code');
```