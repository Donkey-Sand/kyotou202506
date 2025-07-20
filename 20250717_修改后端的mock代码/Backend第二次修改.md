ご指摘の通り、エラーモード分岐のロジックを各エンドポイントに都度書くのは保守性が悪くなりがちです。
これを FastAPI の Depends やデコレーター を使って共通化することは十分可能です。

⸻

✅ 方針：Depends で共通チェック関数を使う

以下のように check_mock_mode という関数を Depends に渡すことで、リクエストの事前チェックを共通化できます。

⸻

🔧 実装ステップ

① config_loader.py に共通チェック関数を追加

from fastapi import HTTPException, Request, Depends
import yaml

def load_mock_mode():
    with open("config.yaml", "r") as f:
        config = yaml.safe_load(f)
    return config.get("mode", {})

mock_mode = load_mock_mode()

def check_mock_mode(endpoint_name: str):
    def checker():
        mode = mock_mode.get(endpoint_name, "normal")
        if mode == "network":
            raise HTTPException(status_code=503, detail=f"{endpoint_name}: Simulated network error")
        elif mode == "error":
            raise HTTPException(status_code=500, detail=f"{endpoint_name}: Simulated internal error")
    return Depends(checker)


⸻

② main.py 側のエンドポイントで共通化

from config_loader import check_mock_mode

@app.post("/api/method")
async def method_endpoint(
    request: Request,
    _: None = check_mock_mode("method")  # ← ここでモード判定
):
    # ここからは正常系の処理のみ書けばOK
    ...

@app.post("/api/search")
async def search_endpoint(
    request: Request,
    _: None = check_mock_mode("search")
):
    ...

こうすることで 分岐ロジックは 1 箇所に集中し、各エンドポイントには正常処理だけ書けます。

⸻

✅ メリット
	•	✅ 分岐ロジックの重複排除（DRY原則）
	•	✅ エラーメッセージも一括管理
	•	✅ 新しいメソッド追加時の実装が簡単
	•	✅ テストもしやすい

⸻

🧠 おまけ：デコレーターで書くことも可能

ただし FastAPI では Depends の方が公式に推奨されており、CORS や middleware と相性も良いため、Depends の形がベストプラクティスです。

⸻

必要であれば、この方式に対応したテストコードや他エンドポイントの追加例も提供できます。続けますか？



以下是上述内容的中文翻译：

⸻

✅ 方案概述：使用 Depends 实现共通化检查

你提到的担忧非常合理 —— 每增加一个新接口都手动实现相同的错误模式分支逻辑，会造成代码重复、维护困难、错误信息分散。

这个问题可以通过 FastAPI 的 Depends 或函数装饰器 来优雅地解决，实现「共通化处理逻辑」。

⸻

🔧 实现步骤

① 在 config_loader.py 中添加共通检查函数

from fastapi import HTTPException, Depends
import yaml

def load_mock_mode():
    with open("config.yaml", "r") as f:
        config = yaml.safe_load(f)
    return config.get("mode", {})

mock_mode = load_mock_mode()

def check_mock_mode(endpoint_name: str):
    def checker():
        mode = mock_mode.get(endpoint_name, "normal")
        if mode == "network":
            raise HTTPException(status_code=503, detail=f"{endpoint_name}: 模拟网络错误")
        elif mode == "error":
            raise HTTPException(status_code=500, detail=f"{endpoint_name}: 模拟内部错误")
    return Depends(checker)

这个 check_mock_mode("method") 会根据配置文件判断当前接口是否处于错误模式，并在必要时抛出异常。

⸻

② 在主程序中使用 Depends 实现共通处理

from config_loader import check_mock_mode

@app.post("/api/method")
async def method_endpoint(
    request: Request,
    _: None = check_mock_mode("method")  # ← 在这里自动判断错误模式
):
    # 这里就只写正常处理逻辑即可
    ...

@app.post("/api/search")
async def search_endpoint(
    request: Request,
    _: None = check_mock_mode("search")
):
    ...


⸻

✅ 优点总结

优点	说明
✅ 去除重复	所有接口共用同一个判断函数
✅ 错误信息集中管理	不再需要在每个接口里写 error 分支
✅ 可扩展性强	新接口只需加一行 check_mock_mode
✅ 更适合测试	各种模式都能统一管理、易于切换


⸻

🧠 额外说明：也可以用装饰器实现

虽然你也可以用 Python 装饰器（@xxx）来实现同样的功能，但 FastAPI 官方更推荐使用 Depends —— 它与路由、CORS、middleware 等集成得更自然，代码可读性也更高。

⸻

如有需要，我也可以为你提供配套的测试脚本或者更复杂的共通处理逻辑。是否继续？
