了解了！你的目标是：

✅ 点击按钮后触发一个 GET 请求，并验证这个请求的 URL（或参数）中是否包含某个特定的内容。

下面是一个完整、精准的 Playwright 脚本例子，可以帮你完成这个测试：

⸻

✅ 示例：验证 GET 请求是否包含特定参数

import { test, expect } from '@playwright/test';

test('点击按钮后检查 GET 请求的参数', async ({ page }) => {
  await page.goto('https://your-website.com');

  // 等待并拦截 GET 请求
  const [request] = await Promise.all([
    page.waitForRequest(req =>
      req.url().includes('/api/target-endpoint') &&
      req.method() === 'GET'
    ),
    page.click('#your-button-id'), // 替换为你的按钮选择器
  ]);

  // 输出请求信息
  console.log('请求URL:', request.url());

  // ✅ 检查请求中是否包含特定内容，比如 query 参数 "keyword=special"
  expect(request.url()).toContain('keyword=special');
});


⸻

🔍 举例说明：

如果按钮触发了这个请求：

GET https://example.com/api/search?keyword=special&type=1

那上面的代码会捕获这个请求，并验证 URL 中是否包含 keyword=special。

⸻

✅ 如果你想检查多个参数：

你可以用 URL API 拆解：

const url = new URL(request.url());
expect(url.searchParams.get('keyword')).toBe('special');
expect(url.searchParams.get('type')).toBe('1');


⸻

如果你不确定接口路径：

你可以用更宽松的方式监听所有 GET 请求：

page.on('request', request => {
  if (request.method() === 'GET' && request.url().includes('keyword=')) {
    console.log('匹配请求:', request.url());
  }
});


⸻

需要我帮你生成一个适配你项目的测试脚本？你只需要告诉我以下两点：
	•	按钮的 selector 是什么？比如：#search-button 或 button:has-text("検索")
	•	API 的路径中包含什么关键词？比如 /api/search 或 keyword=xxx

我可以快速帮你定制。
