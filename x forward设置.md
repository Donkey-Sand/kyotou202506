はい、おっしゃる通りです。
CloudFront を経由すると、WAF が直接見る IP アドレスは、実際のクライアントのものではなくなります。
これは非常に重要な仕様なので、以下に詳しく説明します。

⸻

📌 CloudFront を挟むと IP アドレスが変わる理由

🔁 通常の構成：

[クライアント] → CloudFront（エッジロケーション）→ オリジンサーバ（例：S3、ALB、Lambdaなど）

🔒 WAF はどの IP を見ているのか？
	•	WAF（Web ACL）は CloudFront に紐づいている場合、
→ CloudFront にリクエストを送ってきた “直近の接続元 IP” を見ます。
	•	つまり、**CloudFront 自身が WAF に対しての「クライアント」**になります。
	•	そのため、WAF のルールで使われる source IP は、実際のエンドユーザーではなく、CloudFront の内部通信の IP（≠クライアントIP）になることがあります。

⸻

✅ 正しいクライアントIPを WAF に認識させるには？

🔧 解決策：

WAF のルール設定で「X-Forwarded-For ヘッダーの最初の IP を使用する」オプションを有効化する

⸻

✅ 設定方法（手順概要）：
	1.	AWS 管理コンソールで「WAF」サービスを開く
	2.	対象の Web ACL を選択
	3.	IP アドレスマッチルールを新規作成または編集
	4.	「Use the first IP address in the X-Forwarded-For header」
　　（「X-Forwarded-For ヘッダー内の最初の IP を使用」）にチェックを入れる
	5.	保存して Web ACL を再適用

⸻

📎 補足：なぜ「最初の IP」なのか？
	•	X-Forwarded-For ヘッダーには複数の IP がカンマ区切りで入ります。
	•	最初の IP が、元のクライアントのグローバル IP。
	•	その後ろに CloudFront やプロキシサーバなどの IP が続きます。

例：

X-Forwarded-For: 203.0.113.10, 15.158.25.7

→ 最初の 203.0.113.10 がクライアントの IP。

⸻

✅ まとめ

状況	説明
CloudFront 経由	WAF は通常、CloudFront ノードからの IP を見てしまう
対策	WAF のルールで X-Forwarded-For の先頭 IP を使用するよう設定
結果	実際のクライアント IP に対して正しくアクセス制御が可能になる


⸻

ご希望であれば、この設定方法を画像付きの日本語マニュアルとしてお渡しすることも可能です。必要でしたらお申し付けください。



当然可以，以下是上面内容的中文翻译版：

⸻

📌 为什么通过 CloudFront 后 IP 地址会变化？

🔁 一般的架构：

[客户端] → CloudFront（边缘节点）→ 源站（如 S3、ALB、Lambda 等）

🔒 WAF 实际看到的是谁的 IP？
	•	当你在 CloudFront 上关联了 WAF（Web ACL）时，
→ WAF 看到的“客户端 IP”其实是连接到 CloudFront 的那一跳 IP。
	•	换句话说，CloudFront 自身成了 WAF 所看到的“客户端”。
	•	因此，WAF 判断使用的 IP，并不一定是真正访问你网站的用户 IP，而可能是 CloudFront 节点的中转 IP。

⸻

✅ 如何让 WAF 正确识别真实客户端 IP？

🔧 解决办法：

在 WAF 的规则中启用
**“使用 X-Forwarded-For 头部中的第一个 IP 地址”**选项。

⸻

✅ 设置方法（操作概述）：
	1.	登录 AWS 管理控制台 → 打开「WAF」服务
	2.	找到并打开目标 Web ACL
	3.	新建或编辑「IP 地址匹配规则」
	4.	勾选「使用 X-Forwarded-For 中的第一个 IP 地址」（Use the first IP address in the X-Forwarded-For header）
	5.	保存规则并重新应用 Web ACL

⸻

📎 补充说明：为什么选择“第一个 IP”？
	•	X-Forwarded-For 是一个 HTTP 头，里面通常有多个 IP 地址，用逗号分隔。
	•	第一个 IP 是最初发起请求的客户端的公网 IP
	•	后面的 IP 是 CloudFront 或中间代理等设备的 IP

例如：

X-Forwarded-For: 203.0.113.10, 15.158.25.7

→ 第一个 203.0.113.10 是真正的用户 IP。

⸻

✅ 总结

情况	说明
通过 CloudFront 请求	WAF 默认看到的是 CloudFront 节点的 IP，而不是用户 IP
解决方式	在 WAF 中启用 X-Forwarded-For 的首个 IP 用作判断依据
效果	能够对真实客户端 IP 实现准确的访问控制


⸻

如果你需要的话，我也可以为你制作一份图文版中文操作手册，帮助你更直观地设置该选项。是否需要？


根据你上传的截图，你在配置 AWS WAF 的 IP match 条件，系统让你选择以下两种方式来判断“请求来源的 IP 地址”：

⸻

✅ 两个选项的含义对比：

选项	含义	适用情况
Source IP address（默认）	使用 CloudFront 接收到的直接来源 IP（通常是 CloudFront 节点看到的 IP）	如果你不使用 CDN / Proxy 或仅限某些 IP 直连时使用
IP address in header	使用 HTTP 头部（如 X-Forwarded-For）中的 IP 地址	如果使用了 CloudFront、ALB、代理服务器 —— 建议选这个！


⸻

🔎 你的场景适用哪一个？

你当前情况为：
	•	使用 CloudFront（CDN） → WAF → 后端服务
	•	设置了公司 Global IP 白名单，但仍偶尔被误拦
	•	想让 WAF 判断真实客户端 IP（而不是 CloudFront 出口 IP）

✅ 应选择： IP address in header

并且保持以下设置：
	•	Header field name: X-Forwarded-For（保持默认即可）
	•	Position inside header: 选择 First IP address（首个 IP 地址），因为它才是真正的客户端 IP！

但从你截图来看，当前选的是：

Last IP address

这个表示用列表中最后一个 IP，它通常是 CloudFront 节点的 IP，不是客户端 IP！

⸻

✅ 正确设置方式如下：
	1.	勾选 IP address in header
	2.	Header field name：保持默认的 X-Forwarded-For
	3.	Position inside header：
	•	❌ 不要选 “Last IP address”
	•	✅ 应选择：“First IP address”

👉 但你当前画面里只显示了 “Last IP address”，没有 “First IP address” 的选项，可能是你还没展开那部分，或者 AWS 控制台语言/版本不同。

你可以试试看点击那个 “Last IP address” 的上面部分（应该是一个 radio button 列表），是否能展开选择“First IP address”。

⸻

📌 小结（中文总结）：
	•	如果你想让 WAF 识别 公司员工的真实 IP 地址，必须选择：
	•	IP address in header
	•	使用 X-Forwarded-For
	•	使用 第一个 IP（first IP address）

这样，WAF 就不会再误拦你们公司合法的请求了。

⸻

如你需要我为这部分出一份日语/中文图文手册，也可以告诉我。是否继续？


以下是刚才说明内容的日语翻译版本：

⸻

✅ どちらを選ぶべきか？（あなたの状況に最適な設定）

あなたのスクリーンショットでは、AWS WAF の「IP アドレスマッチ条件」を設定しており、
リクエスト元の IP アドレスの判定方法として、以下の2つの選択肢があります：

オプション	意味	適したケース
Source IP address（送信元IP）	CloudFront が受け取った直接の IP（CloudFrontノード側から見たIP）を使用する	CDNやプロキシを使っていない場合
IP address in header（ヘッダー内のIP）	X-Forwarded-For などの HTTP ヘッダーに含まれる IP を使用する	CloudFrontやALB、プロキシ経由のアクセスに対して推奨 ✅


⸻

🎯 あなたのケース（CloudFront使用）では：
	•	CloudFront を経由してリクエストを受けている
	•	会社の Global IP をホワイトリスト登録しているが、WAF でブロックされることがある
	•	本当のクライアント IP で判定してほしい

✅ この場合は IP address in header を選ぶのが正解です！

さらに、以下の設定を行う必要があります：
	1.	Header field name：そのまま X-Forwarded-For を使う（デフォルトでOK）
	2.	Position inside header（ヘッダー内の位置）：
	•	✅ 「First IP address（最初のIP）」を選ぶ
	•	→ 一番左のIPが本来のクライアントIP（会社の端末のIP）
	•	❌ 「Last IP address（最後のIP）」は CloudFront ノードなどのIPになってしまうため不適切

⸻

🔎 注意点

あなたのスクリーンショットでは、現在「Last IP address（最後のIP）」が選択されていますが、
これは CloudFront 側のIPであり、社内クライアントのIPではありません。

→ このままだと WAF は会社のIPを正しく判定できません。

⸻

✅ 正しい手順（まとめ）
	1.	IP address in header を選択
	2.	Header field name：X-Forwarded-For（変更不要）
	3.	Position inside header：First IP address（最初のIP）を選択すること！
　※ 表示されていない場合は、オプションが折りたたまれているか、スクロールで表示される可能性があります。

⸻

もしご希望であれば、**日本語の手順マニュアル（スクリーンショット付き）**としてまとめてお渡しすることも可能です。必要でしたらお知らせください。


谢谢上传图片！以下是该页面内容的中文说明（含日语对照），详细解释了 Position inside header 和 Fallback for missing IP address 的含义和设置建议。

⸻

🧾 一、Position inside header（在 Header 中选哪个 IP 地址？）

CloudFront 在请求中加入 X-Forwarded-For 头，内容可能如下：

X-Forwarded-For: 203.0.113.100, 70.41.3.18, 150.172.238.178

这表示：最左边是客户端 IP，右边是中间代理服务器或 CDN 节点。

你有三个选项来告诉 WAF 检查哪一个 IP：

选项	中文含义	建议	日语翻译
Last IP address（默认）	检查最后一个 IP（通常是 CloudFront 节点）	❌ 不建议（不是用户的真实 IP）	最後の IP アドレスを検査（CloudFrontノード）
First IP address	检查第一个 IP（真实客户端 IP）	✅ 推荐（一般是你想控制的公司出口 IP）	最初の IP アドレスを検査（クライアントIP）
Any IP address	检查 Header 中的任意 IP（最多检查前10个）	可用于宽松匹配（不精确）	全ての IP を検査（最大10個）


⸻

✅ 建议你选择：

👉 First IP address（推荐）
因为这是最靠近用户的一跳，最有可能是真正的公司 IP。

⸻

🧾 二、Fallback for missing IP address（没有 IP 时怎么办）

这个选项是说：如果请求里压根就没有你指定的 Header（比如没有 X-Forwarded-For），WAF 应该怎么处理？

选项	中文含义	建议	日语翻译
No match（默认）	如果 Header 缺失，则视为不匹配	✅ 推荐（更安全）	該当なしとする（安全）
Match	如果 Header 缺失，也视为匹配成功	❌ 有风险，容易被绕过	該当ありとみなす（非推奨）


⸻

✅ 建议你保持默认设置：
	•	First IP address
	•	No match

这样可以确保：
	•	使用正确的客户端 IP 来判断
	•	没有 Header 时默认阻止请求，防止伪造绕过

⸻

如果你需要，我可以为你生成完整的 WAF 设置说明手册（中文 / 日语 / 图文）。是否需要？



以下は、先ほどの内容の日本語翻訳版です。CloudFront + WAF 環境での Position inside header および Fallback for missing IP address の設定に関する解説です。

⸻

🧾 ①「Position inside header」（ヘッダー内のどのIPを使うか？）

X-Forwarded-For ヘッダーには、複数のIPアドレスがカンマ区切りで含まれます。例：

X-Forwarded-For: 203.0.113.100, 70.41.3.18, 150.172.238.178

この場合、一番左が実際のクライアントのIPアドレスです。右側はプロキシやCloudFrontノードのIPです。

WAFでは以下3つの選択肢があります：

オプション	説明	推奨度	日本語説明
Last IP address（デフォルト）	最後のIPアドレスを検査（CloudFrontノード）	❌ 推奨しない	一番最後のIP（CDNノード）を見るため、正確なクライアントIPにならない可能性が高い
First IP address	最初のIPアドレスを検査（クライアントIP）	✅ 推奨	クライアントが実際に持つIPアドレスを見るため、アクセス制御が正確にできる
Any IP address	ヘッダー内の最大10個のIPアドレスすべてをチェック	△ 条件が緩くなる	柔軟だが意図しないマッチングが起こる可能性あり


⸻

✅ 結論：「First IP address」がおすすめです！

これを選ぶことで、WAFは最も左側（＝元のクライアント）のIPアドレスを使って検査します。

⸻

🧾 ②「Fallback for missing IP address」（ヘッダーが無い場合の処理）

この設定は、「指定したヘッダーがリクエストに存在しなかった場合どうするか」を決めるものです。

オプション	説明	推奨度	日本語説明
No match（デフォルト）	ヘッダーが無い場合はマッチしない（＝ブロックされる可能性あり）	✅ 推奨	安全性が高い。攻撃者がヘッダーを省略した場合も通さない
Match	ヘッダーが無くてもマッチしたと見なす	❌ 非推奨	ヘッダーを偽装・省略してWAFをバイパスされる恐れあり


⸻

✅ 結論：「No match」を選ぶのが安全です！

⸻

✅ 最終的なおすすめ設定（まとめ）

項目	設定内容
IP判定元	IP address in header
ヘッダーフィールド名	X-Forwarded-For
検査対象IP位置	First IP address
ヘッダーが無いときの処理	No match


⸻

ご希望があれば、これをベースにした日本語の「WAF設定手順書（PDF or Markdown）」としてお渡し可能です。必要であればお知らせください。












