バックエンドモックに 正常系／異常系の切替機能 を導入するために、以下のように実装を拡張できます。

⸻

✅ 対応概要
	•	config.yaml ファイルで モックの動作モード（正常系・エラー系）を指定できるようにします。
	•	method と search のエンドポイントで、設定に応じて正常応答またはエラーを返すようにします。

⸻

🛠 1. config.yaml を新規作成

mode:
  method: normal        # normal / error / network
  search: error         # normal / error / network


⸻

🛠 2. config_loader.py を新規作成

import yaml

def load_mock_mode():
    with open("config.yaml", "r") as f:
        config = yaml.safe_load(f)
    return config.get("mode", {})


⸻

🛠 3. main ファイルの修正（設定を読み込み）

まずファイル冒頭に追加：

from config_loader import load_mock_mode

mock_mode = load_mock_mode()


⸻

🛠 4. /api/method のモック動作切替追加

@app.post("/api/method")
async def method_endpoint(request: Request):
    mode = mock_mode.get("method", "normal")

    if mode == "network":
        raise HTTPException(status_code=503, detail="Simulated network error")
    elif mode == "error":
        raise HTTPException(status_code=500, detail="Simulated internal error")

    try:
        body = await request.json()
        search_query = body.get("search_query", "")
        search_options = body.get("search_options", "")
        mode_options = body.get("mode_options", "")
        inst_options = body.get("inst_options", "")
    except Exception as e:
        raise HTTPException(status_code=400, detail=f"Invalid request body: {e}")

    params = {
        "search_query": search_query,
        "search_options": search_options.split(",") if search_options else [],
        "mode_options": mode_options.split(",") if mode_options else [],
        "inst_options": inst_options.split(",") if inst_options else [],
    }
    log_queue = asyncio.Queue()

    async def process_and_log():
        async def log_output_async(message):
            await log_queue.put({"log": message})

        try:
            method_str = await method.generate_async(params, log_output_async=log_output_async)
            await log_queue.put({"method": method_str})
        except Exception as e:
            await log_queue.put({"log": f"ERROR: {str(e)} {traceback.format_exc()}"})
        finally:
            await log_queue.put(None)

    asyncio.create_task(process_and_log())
    return StreamingResponse(method.log_generator(log_queue), media_type="text/event-stream")


⸻

🛠 5. /api/search にも同様の切替を追加

@app.post("/api/search")
async def search_endpoint(request: Request):
    mode = mock_mode.get("search", "normal")

    if mode == "network":
        raise HTTPException(status_code=503, detail="Simulated network timeout")
    elif mode == "error":
        raise HTTPException(status_code=500, detail="Simulated internal server error")

    try:
        body = await request.json()
        search_query = body.get("search_query", "")
        doc_num = int(body.get("doc_num", 10))
        extract_method = bool(body.get("extract_method", True))
    except Exception as e:
        raise HTTPException(status_code=400, detail=f"Invalid request body: {e}")

    params = {"search_query": search_query, "doc_num": doc_num, "extract_method": extract_method}
    event_queue = asyncio.Queue()

    async def process_and_log():
        async def log_output_async(message_dict):
            await event_queue.put(message_dict)

        try:
            search_summary = await search.fetch_async(params, log_output_async=log_output_async)
            await event_queue.put({"event": "process_complete", "summary": search_summary})
        except Exception as e:
            await event_queue.put({"event": "error", "message": f"ERROR: {str(e)} {traceback.format_exc()}"})
        finally:
            await event_queue.put(None)

    asyncio.create_task(process_and_log())
    return StreamingResponse(search.event_generator(event_queue), media_type="text/event-stream")


⸻

✅ 完了条件
	•	config.yaml によって動作モードを切り替え可能
	•	正常系・エラー系・ネットワーク系を個別エンドポイントでテスト可能

⸻

📌 例：テスト切替パターン

設定値	動作内容
normal	通常の応答
error	HTTP 500 の内部エラーを返す
network	HTTP 503 のネットワークエラーを返す


⸻

必要であれば、ログ出力や curl テストコマンド例も追加可能です。続きを希望されますか？








以下是上述内容的中文翻译：

⸻

✅ 对应概要
	•	通过 config.yaml 配置文件，可以切换 Mock 后端的运行模式（正常 / 错误 / 网络错误）。
	•	在 /api/method 和 /api/search 接口中，根据配置文件的设定返回正常响应或模拟错误。

⸻

🛠 1. 新建 config.yaml

mode:
  method: normal        # 可选值：normal / error / network
  search: error         # 可选值：normal / error / network


⸻

🛠 2. 新建 config_loader.py 加载配置

import yaml

def load_mock_mode():
    with open("config.yaml", "r") as f:
        config = yaml.safe_load(f)
    return config.get("mode", {})


⸻

🛠 3. 修改主程序文件，加载配置

在主程序文件开头加入：

from config_loader import load_mock_mode

mock_mode = load_mock_mode()


⸻

🛠 4. 为 /api/method 添加模式切换逻辑

@app.post("/api/method")
async def method_endpoint(request: Request):
    mode = mock_mode.get("method", "normal")

    if mode == "network":
        raise HTTPException(status_code=503, detail="模拟网络错误")
    elif mode == "error":
        raise HTTPException(status_code=500, detail="模拟内部服务器错误")

后面逻辑与正常代码相同（处理请求体、调用 method.generate_async() 并返回日志流）。

⸻

🛠 5. 为 /api/search 添加模式切换逻辑

@app.post("/api/search")
async def search_endpoint(request: Request):
    mode = mock_mode.get("search", "normal")

    if mode == "network":
        raise HTTPException(status_code=503, detail="模拟网络超时")
    elif mode == "error":
        raise HTTPException(status_code=500, detail="模拟内部服务器错误")

同样，后续逻辑照常处理请求体并生成日志流。

⸻

✅ 完成条件
	•	可通过 config.yaml 切换各接口的模拟运行模式。
	•	/api/method 与 /api/search 可根据设置模拟正常响应、服务器内部错误或网络错误。

⸻

📌 测试切换示例

配置值	模拟行为
normal	返回正常响应
error	返回 HTTP 500 内部错误
network	返回 HTTP 503 网络异常（服务不可用）


⸻

如果你需要进一步添加日志输出、命令行测试例子（如 curl）等内容，也可以告诉我，我可以继续补充。是否需要？


