你可以使用 Playwright 来测试这个弹出页面（div）是否显示与隐藏。以下是一个典型的测试步骤和代码示例：

✅ 场景假设
	•	页面上有一个按钮，点击后显示一个 id="history" 的 div。
	•	弹出窗口里有一个“关闭”按钮，点击后会隐藏该 div。

⸻

🧪 Playwright 测试代码（TypeScript 示例）

import { test, expect } from '@playwright/test';

test('点击按钮显示和关闭history弹窗', async ({ page }) => {
  // 1. 打开页面
  await page.goto('https://example.com'); // 替换为你实际的网址

  // 2. 点击触发弹窗的按钮
  await page.click('#show-history-button'); // 替换为实际按钮的选择器

  // 3. 断言：弹窗（history div）应该可见
  const historyDiv = page.locator('#history');
  await expect(historyDiv).toBeVisible();

  // 4. 点击“关闭”按钮
  await page.click('#close-history-button'); // 替换为关闭按钮的选择器

  // 5. 断言：弹窗应该消失
  await expect(historyDiv).toBeHidden();
});


⸻

📝 说明：
	•	page.click(selector)：模拟点击按钮。
	•	page.locator('#history')：选中弹出的div。
	•	expect(...).toBeVisible()：断言元素可见。
	•	expect(...).toBeHidden()：断言元素已隐藏。

⸻

✅ 补充建议
	•	如果你用的是类名而不是 ID，比如 <div class="history">，可以用 .history 来选。
	•	如果隐藏是通过 CSS（比如 display: none）实现的，Playwright 的 toBeHidden() 是可以正确判断的。

⸻

如果你提供 HTML 结构或更具体的按钮选择器，我可以帮你更精确地写选择器。需要的话可以贴出来 👍
