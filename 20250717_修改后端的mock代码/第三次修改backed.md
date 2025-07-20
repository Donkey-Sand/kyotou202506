非常好的问题！你提到的核心需求是：

⸻

🎯 目标：实现高度通用的错误模拟机制
	•	✅ 不论未来增加多少个 API 接口，都无需重复写 if 分支
	•	✅ 不论未来增加多少种错误模式，都能自动支持
	•	✅ 错误信息（状态码、消息等）统一集中管理
	•	✅ 最好只在每个 endpoint 中添加一行即可启用 Mock 功能

⸻

✅ 最佳方案：使用配置驱动 + FastAPI Depends + 错误类型注册表

我们来设计一个具备高度通用性的解决方案。

⸻

🧱 配置文件：mock_config.yaml

mode:
  method: error         # 可选：normal / error / network / timeout 等
  search: network


⸻

🧱 错误模式注册表（错误类型、状态码、消息等）

# mock_errors.py

from fastapi import HTTPException

MOCK_ERROR_REGISTRY = {
    "network": lambda ep: HTTPException(status_code=503, detail=f"{ep}: 模拟网络错误"),
    "error": lambda ep: HTTPException(status_code=500, detail=f"{ep}: 模拟服务器错误"),
    "timeout": lambda ep: HTTPException(status_code=504, detail=f"{ep}: 模拟超时错误"),
    "unauthorized": lambda ep: HTTPException(status_code=401, detail=f"{ep}: 模拟未授权访问"),
    # 你可以随时在这里新增类型
}


⸻

🧱 通用 Depends 工具函数

# mock_checker.py

from fastapi import Depends
import yaml
from mock_errors import MOCK_ERROR_REGISTRY

# 全局加载一次配置（你也可以改成热加载）
with open("mock_config.yaml", "r") as f:
    CONFIG = yaml.safe_load(f)

def mock_mode_checker(endpoint_name: str):
    def checker():
        mode = CONFIG.get("mode", {}).get(endpoint_name, "normal")
        if mode != "normal":
            error_func = MOCK_ERROR_REGISTRY.get(mode)
            if error_func:
                raise error_func(endpoint_name)
            else:
                raise HTTPException(status_code=500, detail=f"{endpoint_name}: 未知的模拟模式 [{mode}]")
    return Depends(checker)


⸻

🧱 FastAPI 中的使用方式（高度通用）

from mock_checker import mock_mode_checker

@app.post("/api/method")
async def method_endpoint(
    request: Request,
    _: None = mock_mode_checker("method")
):
    # 这里只处理正常流程
    ...

@app.post("/api/search")
async def search_endpoint(
    request: Request,
    _: None = mock_mode_checker("search")
):
    ...


⸻

✅ 整体优势总结

特性	实现方式	好处
模拟模式切换	YAML 配置	一行切换所有接口的行为，无需改代码
错误类型定义集中	MOCK_ERROR_REGISTRY	所有错误逻辑统一管理、易于扩展
与接口解耦	Depends 注入	每个接口只需写一行即可引入错误模拟
易于扩展	只需增加配置项和注册表项	支持任意数量的接口和错误类型
热更新支持	可选改成每次读取配置	无需重启即可改变模拟行为（用于开发）


⸻

✨ 示例：以后要新增一个接口怎么办？

你只需做两件事：
	1.	在 YAML 中加上新接口的模式，比如：

mode:
  new_endpoint: timeout

	2.	在新接口中添加这一行：

_: None = mock_mode_checker("new_endpoint")

无需再写分支、抛异常，其他交给通用逻辑。

⸻

是否需要我把这套结构打包成完整 Python 项目 demo 给你？或者需要添加「日志记录」、「热加载配置」、「多环境支持」等高级功能吗？




以下是上述内容的日语翻译：

⸻

🎯 目標：汎用的なエラーモック機構の実現
	•	✅ 将来 API エンドポイントが増えても、毎回 if 分岐を書かなくて済む
	•	✅ エラータイプが増えても、既存のコードを変更せずに対応可能
	•	✅ エラーメッセージやステータスコードなどを一元管理したい
	•	✅ 各エンドポイントには 1行だけ でモックを有効化できるのが理想

⸻

✅ 最適解：設定駆動 + FastAPI Depends + エラー種別レジストリ

この目的を満たすために、汎用的で拡張性の高い実装方法をご提案します。

⸻

🧱 設定ファイル：mock_config.yaml

mode:
  method: error         # 選択肢：normal / error / network / timeout など
  search: network


⸻

🧱 エラー種別の登録辞書（ステータスコードとメッセージを定義）

# mock_errors.py

from fastapi import HTTPException

MOCK_ERROR_REGISTRY = {
    "network": lambda ep: HTTPException(status_code=503, detail=f"{ep}: ネットワークエラーのモック"),
    "error": lambda ep: HTTPException(status_code=500, detail=f"{ep}: サーバーエラーのモック"),
    "timeout": lambda ep: HTTPException(status_code=504, detail=f"{ep}: タイムアウトのモック"),
    "unauthorized": lambda ep: HTTPException(status_code=401, detail=f"{ep}: 認証エラーのモック"),
    # 新しい種類をここに追加するだけでOK
}


⸻

🧱 共通 Depends チェック関数

# mock_checker.py

from fastapi import Depends
import yaml
from mock_errors import MOCK_ERROR_REGISTRY

# 設定を一度だけ読み込む（必要ならホットリロードにも変更可能）
with open("mock_config.yaml", "r") as f:
    CONFIG = yaml.safe_load(f)

def mock_mode_checker(endpoint_name: str):
    def checker():
        mode = CONFIG.get("mode", {}).get(endpoint_name, "normal")
        if mode != "normal":
            error_func = MOCK_ERROR_REGISTRY.get(mode)
            if error_func:
                raise error_func(endpoint_name)
            else:
                raise HTTPException(status_code=500, detail=f"{endpoint_name}: 未知のモックモード [{mode}]")
    return Depends(checker)


⸻

🧱 FastAPI での使い方（すべて共通化）

from mock_checker import mock_mode_checker

@app.post("/api/method")
async def method_endpoint(
    request: Request,
    _: None = mock_mode_checker("method")
):
    # ここには正常系の処理だけを書けばOK
    ...

@app.post("/api/search")
async def search_endpoint(
    request: Request,
    _: None = mock_mode_checker("search")
):
    ...


⸻

✅ この仕組みのメリットまとめ

特徴	実現方法	メリット
モード切替	YAML 設定	コードを変えずにモード変更可能
エラータイプ集中管理	MOCK_ERROR_REGISTRY	保守しやすく拡張もしやすい
各APIと分離	Depends で注入	各APIに1行追加するだけでOK
拡張性	モードもAPIも設定だけで追加可能	コード変更不要
ホットリロードも可能	CONFIG 読み直せば対応可	開発中に即時反映できる


⸻

✨ 将来新しいAPIを追加する場合

やることはたった2つ：
	1.	YAML にエンドポイント名を追加：

mode:
  new_endpoint: timeout

	2.	エンドポイントに以下を追加：

_: None = mock_mode_checker("new_endpoint")

これだけで完了。条件分岐もエラー処理も一切書く必要なし。

⸻

必要であれば、この構成をまるごと Python プロジェクトとして構築するサンプルや、ホットリロード／ログ出力／環境別切り替えなどの高度機能もご提供可能です。ご希望があれば教えてください。

