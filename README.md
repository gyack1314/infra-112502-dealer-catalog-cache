# 经销网络的可承诺目录缓存与跨节点读下限

目录每个键独立发布修订。门店客户端的目录凭据以键到修订号的映射表达已读下限。合成记录中的金额为整数分。

## 输入消息

每个 JSONL 文件是一段独立输入记录。每行包括整数秒点 at、消息名称 kind 与对象 record。at 代表中间件收到消息的时刻，同一秒内按行序提交。业务时间和对应回执引用写在 record 中。

- **publish**　区域目录发布 key、revision、values 与可选 requires。requires 是所引用的其他目录键及其最低 revision，按相同区域解释。发布意味着版本存在，门店仍须经实际回源或缓存取得内容。
- **invalidate**　区域撤除或替换 key 的单项 item，以 boundary 表达从哪个 revision 起生效。旧内容被撤下后不可作为承诺，其他项仍可独立使用。
- **lookup**　门店发来 mode 为 browse 或 promise 的请求。keys 是所需目录，item 是待确认的适配名称，floor 为报价方带来的最低版本向量，可省略。stale_for 为展示允许的最大缓存年龄。
- **origin**　仿真回源的实际返回。store、key、request 对应原查询，提供 revision、values、requires。issued_at 是该答复所属请求发起时点。
- **link**　指定 store 和 region 的回源连接状态，以 online 布尔值表示。 online 时附 cursor，代表该节点已追上的撤下通知号。区域发出 invalidation_head 以 number 表示最新通知；每条 invalidate 含 number。 目录读取链路和撤下通知链路独立，断开某门店的查询链路期间仍可接收失效消息。
- **inspect**　查看 store、key 或 id 的状态。restart 用 store 表达门店缓存服务重启。
- **invalidation_head**　区域最新撤下通知号，0 表示此前无通知。
- **restart**　模拟门店缓存重启，store 说明影响哪个节点。

## 示例

- same-key-different-floor.jsonl
- invalidate-during-disconnect.jsonl

## 读取演练资料

运行环境为 Python 3.11，只使用标准库。在根目录运行以下命令可核对文件格式及读取接口。

    python3 scenario_reader.py examples
    python3 -m unittest discover -v

资料读取器会列出各文件包含的消息和时间范围。演练记录只描述各参与方交给系统的输入。
