# FLMX 已发布使用手册独立审计报告

审计日期：2026-11-26。审计人：Codex。结论：**NO-GO**。

本次共记录 **20 项：B 3 项、P1 10 项、P2 7 项**。B 表示阻止本版通过内容审定，P1 表示应修，P2 表示建议改善。NO-GO 的主要原因是新装登录口令错误、把已删除的订阅链式写成现存功能，以及订阅撤权与用户配额的说明会造成错误的访问控制预期。这是对已发布内容的审计结论，不代表执行了下线、修改或重新发布。

## 1. 对象、基准与证据边界

- 在线对象：[FLMX 使用手册](https://iiiiiiiii7125-art.github.io/flmx-manual/)，实际 HTTP 200，18 页（首页加 17 篇）。
- 仓库快照：`main` = `99f540cd396a0501cb9503dec7b29a14460e6296`；手册内容首次提交为 `bf3fea2b663e734d3caec81c6d1d9aa14f45cff1`（v6）。
- 线上 HTML 与该 `main/index.html` 逐字节相同，SHA-256 均为 `9fbca4fd3eaa76a05a235d835b2160d8aea9fe3faac0badf41843d37d460094b`。以下 M 链接固定到该快照，避免后续 main 变化。
- 原工作区 `/home/adminuser/flmx-max` 的 HEAD 是 `eae3cd8aa9dfc561d7a8d2f0c399e52013c1d9bf`，`git describe` 为 `2.1.30-20-geae3cd8a`，属于另一个代码审计任务。**未用这个 HEAD 作发布行为的证据。**
- 实际源码基准：通过 `git archive 2.1.30` 导出到独立临时目录，tag 解析为 `0057995ff023fa1865197443127e797757b83c66`。以下 S 链接均固定到此发布 commit。`vite-frontend/package.json` 的 version 是 `0.0.0`，不能据此认定面板版本；前端运行版本来自 `VITE_APP_VERSION`（[vite-frontend/src/config/site.ts:15](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/config/site.ts#L15)）。
- 主要逐项对照 frontend，涉及口令、权限、计数、拨号方向等实现事实时补查同一发布 commit 的 backend / agent。**未读取或采用 `doc/guide` 的旧说明；未混用 PR #38 的修改。**
- **094 live：未核实。**本次没有该面板的管理员密码及可用登录会话，也没有取得本次任务提供的 live 地址，因此没有登录、修改数据、猜测口令或声称复现面板业务操作。
- 对 `index.html` 全部 31 处内联图片解码并逐张目视检查；其中第 1、2 图完全相同，共 30 张不同图片。编号按 HTML 中 `<figure>` 的出现顺序，附录逐张登记。图片可证明图面内容，不能单独证明拍摄来源、清理结果或历史测试过程。
- 本报告只新增报告文件；不修改手册 HTML、README、图片、Pages 设置或发布状态，不改动 flmx-max 工作区。

## 2. 发现

### F-01 · B · 新装管理员密码写成旧的公开固定口令

- **位置／原文：**安装部署 → 访问面板，[index.html:L376](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L376)：“默认管理员账号 `admin@user`、密码 `admin@user`。”并称“默认口令是公开的”。
- **正确事实：**发布版新播种管理员用户名仍为 `admin@user`，初始密码优先使用 `FLMX_ADMIN_PASSWORD`，否则随机生成；正常安装完成输出会给出密码或取回位置，容器内一次性文件为 `/app/data/initial-admin-password.txt`。首次登录必须先改密。已有数据库不因升级重播口令。
- **证据：**[go-backend/internal/store/repo/initial_admin.go:240](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/store/repo/initial_admin.go#L240) 的实际播种分支、[go-backend/internal/store/repo/initial_admin.go:274](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/store/repo/initial_admin.go#L274) 的用户名，以及 [panel_install.sh:3084](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/panel_install.sh#L3084)–3099 的安装输出逻辑。该错误会使新用户按手册无法登录；不把旧部署可能仍保留旧密码推成新装默认。

### F-02 · B · 把已删除的订阅链式配置写成当前可用功能

- **位置／原文：**首页 [index.html:L303](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L303)；对接中转 → 两种链的表格 [index.html:L670](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L670)；订阅设置 [index.html:L803](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L803)–804：“在订阅里给节点配前置……换链路只改订阅、立刻对所有导入生效”；FAQ [index.html:L1003](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L1003) 重复引导“订阅里配前置”。
- **正确事实：**2.1.30 没有该订阅链式编辑入口；订阅链式选项已删除。现存“订阅前置地址”只是订阅 HTTP 拉取地址的选择，不会在客户端创建代理链。服务器侧隧道转发链与入站出站仍需分别解释。
- **证据：**[go-backend/internal/http/handler/subscription.go:29](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/subscription.go#L29) 明确记录删除，并可核对紧随其后的 `subPipeline` 实际字段及 [go-backend/internal/http/handler/subscription.go:76](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/subscription.go#L76) 的输出选项；[vite-frontend/src/pages/subscription.tsx:518](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/subscription.tsx#L518) 起的 Editor 没有订阅链式配置；[vite-frontend/src/pages/config/sub-front-card.tsx:49](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/config/sub-front-card.tsx#L49) 说明前置只反代订阅路径、token 不变。不是将两种“前置”混同后的新功能。

### F-03 · B · 把取消分配、用户停用及用户额度写成订阅访问的通用控制

- **位置／原文：**订阅分配 [index.html:L821](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L821)：“用户侧能不能用，还受该用户的状态、过期时间与流量限制约束”；[index.html:L822](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L822)：“要收回使用权时，取消分配或停用用户即可”。
- **正确事实：**本页分配给用户的是订阅自身的独立 token 链接，流量及到期按该订阅计算。撤销分配只收回面板内可见性，已复制的订阅链接不会随之失效；不能依赖用户账号停用或用户转发配额来撤销这类裸 token 链接。需要使已发链接失效时，应操作对应订阅，并区分旧客户端凭据的限制（F-04）。
- **证据：**图 24 自身写着“流量与到期按订阅单独计算”；[vite-frontend/src/pages/subscription/assign.tsx:165](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/subscription/assign.tsx#L165)–167；[go-backend/internal/http/handler/user_subscription.go:104](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/user_subscription.go#L104)–130 明确返回撤销不使链接失效的告警；[go-backend/internal/http/handler/user_subscription.go:194](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/user_subscription.go#L194) 说明卡片发的是不带用户参数的 token 链接；[go-backend/internal/http/handler/kernel_relay.go:1769](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/kernel_relay.go#L1769)、[go-backend/internal/http/handler/kernel_relay.go:1815](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/kernel_relay.go#L1815)、[go-backend/internal/http/handler/kernel_relay.go:1905](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/kernel_relay.go#L1905) 可区分裸 token、个性化用户过滤和订阅自身计量。原文虽补充了“链接已发出时重置”，仍与“取消分配或停用即可收回使用权”的总述冲突。

### F-04 · P1 · “重置后旧配置都连不上”遗漏共用凭据及下发失败边界

- **位置／原文：**导入到客户端底部 [index.html:L855](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L855)：“重置后已导入的旧配置也连不上”；订阅字段说明 [index.html:L776](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L776) 也把独立凭据和失活断连写成通用效果。
- **正确事实：**旧订阅 URL 在 token 轮换成功后失效；已导入的连接配置是否断开取决于对应凭据和节点下发。共用凭据的外部节点、未完成原地多用户的 SS 等不会因为订阅重置而自动改掉源节点密码。独立凭据的下发失败也会留下待同步状态，不能笼统承诺瞬时全部断开。
- **证据：**[vite-frontend/src/pages/subscription.tsx:665](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/subscription.tsx#L665)–668 的编辑警告、[vite-frontend/src/pages/subscription.tsx:1780](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/subscription.tsx#L1780)–1787 的重置警告；[go-backend/internal/http/handler/subscription.go:1237](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/subscription.go#L1237)–1262 的 `SharedIDs` 判定及 [go-backend/internal/http/handler/subscription.go:1854](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/subscription.go#L1854)–1861 的轮换／下发状态说明。

### F-05 · P1 · 节点详情的“监控”按钮被误写为返回节点列表

- **位置／原文：**节点详情与监控 → 按钮表 [index.html:L591](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L591)：“管理员点它会回到节点列表（当前版本无独立监控页）”。
- **正确事实：**管理员直接访问 `/monitor` 会跳回 `/node`，这一点正确。但节点详情里的“监控”调用节点页提供的回调，在节点页中展开该节点的监控详情，列表被隐藏，并可返回列表；这是不同入口。
- **证据：**[vite-frontend/src/pages/monitor.tsx:31](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/monitor.tsx#L31)–32 是路由入口重定向；[vite-frontend/src/pages/node.tsx:3330](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/node.tsx#L3330)–3335 是按钮回调；[vite-frontend/src/pages/node-hub.tsx:94](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/node-hub.tsx#L94)–106 实际挂载 `NodeMonitorDetail`。无需新增独立监控章节，也不能因此抹去节点内已有监控。

### F-06 · P1 · 把节点可用端口范围错误地套到内核入站

- **位置／原文：**搭建节点 → 可用端口 [index.html:L460](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L460)：“入站随机分配端口、隧道入口端口都从这个范围里取”；新建入站 [index.html:L483](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L483)：“自己指定（要在节点可用端口范围内……）”。
- **正确事实：**节点可用端口控制转发／中转／隧道入口的分配与校验，**内核入站监听端口不受该范围限制**。入站留空的随机选口实现为 `[20000,60000)`，并检查冲突；不是读取节点配置范围。
- **证据：**[vite-frontend/src/pages/node/node-form-modal.tsx:272](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/node/node-form-modal.tsx#L272)–276 的字段原文已经明确区分；[go-backend/internal/http/handler/kernel.go:1851](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/kernel.go#L1851)–1864 的实际入站选口。默认范围 `10000-65535` 本身正确。

### F-07 · P1 · 指示将域名填到 IPv4 字段会被拒绝

- **位置／原文：**快速上手 [index.html:L339](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L339)：“把节点 IPv4 改成公网 IP 或域名”；订阅设置 [index.html:L793](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L793) 再称“把节点 IPv4 改成公网 IP / 域名”。
- **正确事实：**IPv4 字段只能填 IPv4 字面地址；域名要填独立的“域名/主机名”字段。恢复订阅还需分享地址选择正确，不能让读者直接把域名粘进 IPv4 输入框。
- **证据：**[vite-frontend/src/pages/node/node-form-modal.tsx:231](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/node/node-form-modal.tsx#L231)–255 的两个字段及说明；[vite-frontend/src/pages/node.tsx:984](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/node.tsx#L984)–1002 对 IPv4／IPv6／hostname 分别校验，域名填 IPv4 会得到“请输入有效的IPv4地址”。不否定手册描述的默认分享地址为回环地址时会被过滤的场景。

### F-08 · P1 · 质量检测目标留空不等于无可测对象

- **位置／原文：**隧道质量 → 质量检测目标从哪来，[index.html:L705](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L705)：“没填目标的隧道没有可测对象——先回隧道编辑里补上目标”。
- **正确事实：**Host 与 Port 都留空使用默认 `www.bing.com:443`；只填一部分或填入无效值才需要按校验处理。无需为了有探测结果而强制显式填写默认目标。
- **证据：**[vite-frontend/src/pages/tunnel.tsx:3801](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/tunnel.tsx#L3801)–3807 的字段说明；[go-backend/internal/http/handler/tunnel_probe_target.go:18](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/tunnel_probe_target.go#L18)–35 的默认值及空值归一化分支。

### F-09 · P1 · “连接 IP”被解释成来源 IP，实际是下一节点的拨号目标

- **位置／原文：**对接中转 → 连接 IP 不是第四档策略，[index.html:L664](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L664)：“用它指定这一跳从哪个 IP 连出去”。
- **正确事实：**它选择到当前配置的下一节点时使用的目标地址，并非绑定拨号源 IP／本机出口地址。多节点跳与多出口还会禁用自定义连接 IP，使用各节点默认 IP。
- **证据：**[go-backend/internal/http/handler/mutations.go:9961](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/mutations.go#L9961)–9987 明确返回到 `toNode` 的 host，并优先返回 `connectIp`；[vite-frontend/src/pages/tunnel.tsx:3274](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/tunnel.tsx#L3274)–3292、[vite-frontend/src/pages/tunnel.tsx:3620](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/tunnel.tsx#L3620) 起的禁用条件。它确实是独立字段，手册关于“不是第四种策略”的判断保留。

### F-10 · P1 · 订阅前置地址步骤漏掉“链接地址”选择

- **位置／原文：**订阅前置地址 [index.html:L875](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L875)–876：“在网站配置里填前置地址……回订阅详情重新复制链接，确认链接里的地址已换成前置地址”。
- **正确事实：**配置保存的是备用地址列表，不会自动替换当前链接 base。订阅管理中须在“链接地址”选择器选择相应前置，再打开／复制该订阅的链接；默认仍是面板地址。地址应包含协议，可带端口，不带路径。
- **证据：**[vite-frontend/src/pages/config/sub-front-card.tsx:49](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/config/sub-front-card.tsx#L49)–53；[vite-frontend/src/pages/subscription.tsx:982](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/subscription.tsx#L982)–985 的 `frontBase` 初值为空；[vite-frontend/src/pages/subscription.tsx:1062](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/subscription.tsx#L1062)–1063 用 `frontBase || panelOrigin()`；[vite-frontend/src/pages/subscription.tsx:1422](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/subscription.tsx#L1422)–1427 的选择器；[vite-frontend/src/pages/subscription/front-picker.tsx:27](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/subscription/front-picker.tsx#L27)–39 的实际字段。缺失部分虽然没有配图，不因此算问题；文字操作步骤漏关键动作才是本项问题。

### F-11 · P1 · 流量说明将 SS 的形态限制泛化为所有入站的排除项

- **位置／原文：**订阅设置 [index.html:L799](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L799)：“另有几类入站同样不计入：ShadowTLS 前置、仅中转、带链式出口、已配置分流、开启 PROXY 协议的入站”；FAQ [index.html:L1005](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L1005) 也没有限定为 SS。
- **正确事实：**独立用户凭据协议和 SS 原地计量的资格判定不同。上述 SS 形态限制不能写成对所有协议一律不计量；SS 原地多用户开启后还需要配置实际下发成功。外部导入节点不计量、按本周期累计的说明正确。
- **证据：**[go-backend/internal/http/handler/subscription.go:1023](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/subscription.go#L1023)–1061：导入节点先排除；`SupportsUsers()` 且非 SS 的协议在 1036–1037 返回可计量；SS 再经过开关、方法、形态和下发确认的各层检查。[go-backend/internal/http/handler/subscription_metering.go:96](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/subscription_metering.go#L96) 起的 `ssMeteringShapeBlock` 是 SS 路径。建议保持协议限定，避免读者将 VLESS 等的统计误判为无效。

### F-12 · P2 · 用户流量重置日期可选范围写少了三天

- **位置／原文：**用户管理 → 字段表 [index.html:L913](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L913)：“每月 1–28 号中的一天”。
- **正确事实：**本版 UI 提供不重置及每月 1–31 号，后端允许 0–31。
- **证据：**[vite-frontend/src/pages/user.tsx:1846](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/user.tsx#L1846)–1868 的 `Array.from({ length: 31 })`；[go-backend/internal/http/handler/mutations.go:123](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/mutations.go#L123)–125 的范围校验。这里仅报告选项范围，不推断短月实际执行日期。

### F-13 · P2 · 用户页第二个 tab 误写成“散户”

- **位置／原文：**用户管理开头 [index.html:L889](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L889)：“用户 / 散户两个标签”。
- **正确事实：**“用户 / 限速”。
- **证据：**图 26（[index.html:L891](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L891)）顶部实际显示“限速”；[vite-frontend/src/pages/user-hub.tsx:10](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/user-hub.tsx#L10)–13 的两个 tab 名一致。不是被删除或隐藏的第三个功能。

### F-14 · P2 · “订阅源”的数据方向解释反了

- **位置／原文：**新建入站 → 订阅源，[index.html:L534](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L534)：“管理这个面板对外提供的订阅来源”。
- **正确事实：**这里管理从外部订阅链接／文件导入并同步到面板的源；面板对外分发链接由订阅管理处理。
- **证据：**图 11（[index.html:L536](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L536)）直接写明“填机场或别人的订阅链接，或上传订阅文件”；[vite-frontend/src/pages/inbound/sources.tsx:413](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/inbound/sources.tsx#L413) 的相同说明。图注对空态的标注正确，不将已声明的初始状态重复报为问题。

### F-15 · P2 · 入站名称的两张图注与实际填写／列表不一致

- **位置／原文：**新建入站，图 7 图注 [index.html:L490](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L490)：“名称填「演示-VLESS」”；图 8 图注 [index.html:L494](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L494)：“演示-VLESS · VLESS + TCP + Reality + Vision · 52668……”。两图 alt 同样将名称简化。
- **正确事实：**两图名称栏实际均为 **`VLESS + Reality + Vision演示-VLESS`**。图 7 在模板自动带出的名称之后追加了文字；图 8 保存后的列表也保留该前缀。图 15 后续显示“演示-VLESS”是另一时刻，不能回填到前两图。
- **证据：**解码图 7、8 的名称栏可直接逐字读取，分别为 [index.html:L489](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L489)、[index.html:L493](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L493)；对应 SHA 前缀见附录。模板可自动填名的代码为 [vite-frontend/src/pages/inbound/presets.ts:373](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/inbound/presets.ts#L373)。这是本次对图面的直接核对，不假定中间发生过何种改名操作。

### F-16 · P1 · 截图的前后端版本声明缺少可闭合的证据

- **位置／原文：**侧栏 [index.html:L281](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L281) 及多处图注统一称“前端 2.1.30 / 后端 2.1.23”；例如图 1 图注 [index.html:L313](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L313)。
- **正确事实／边界：**图 1、2、4、8 等左下角可见的是 **`v2.1.23` 加红色 `2.1.30`**。红标表示可更新的最新版本，不能凭它证明当前前端已经是 2.1.30。同版源码的页脚版本来自前端构建变量。对“前端 2.1.30 / 后端 2.1.23”的实际部署组合，本次结论是 **未核实，且现有图面与本版页脚实现之间存在待解释差异**。
- **证据：**[vite-frontend/src/layouts/admin.tsx:549](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/layouts/admin.tsx#L549)–552 将 `siteConfig.version` 传给页脚；[vite-frontend/src/config/site.ts:15](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/config/site.ts#L15) 取 `VITE_APP_VERSION`；[vite-frontend/src/components/version-footer.tsx:94](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/components/version-footer.tsx#L94)–100 区分当前 version 和 latestUpdateVersion。图 3 的“已检查到最新版本 2.1.30”也仅证明更新检查结果。应补充实际构建标识／可复核版本读数，或收窄声明；**不据此断言截图伪造，也不擅自判定 live 前端实际为 2.1.23。**

### F-17 · P2 · 质量截图把被截断的目标名当作完整图面证据

- **位置／原文：**隧道质量图 19 图注 [index.html:L696](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L696)：“出口→测试目标 www.bing.com:443 为 2 ms”；alt 同样写完整目标。
- **正确事实：**图中可见目标标签是截断的 `www.bing.c…`，`2ms` 清楚可见，端口 `443` 没有显示。源码默认值确实是 `www.bing.com:443`，但这不能证明该次截图没有自定义目标。
- **证据：**原图 [index.html:L695](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L695)；[vite-frontend/src/pages/node/tunnel-monitor-view.tsx:79](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/node/tunnel-monitor-view.tsx#L79)–90 说明目标标签可取运行数据或默认值。建议图注分别注明“图面可见的截断标签”和“默认目标／文字补充”。不把源码默认值冒充该 live 截图的完整读数。

### F-18 · P2 · 搜索仅索引正文前 400 字，正文关键字会漏检

- **位置／原文／行为：**顶部“搜索文档”，正文 [index.html:L505](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L505)、[index.html:L507](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L507)、FAQ [index.html:L994](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L994) 均存在 `26.3.27`，线上搜索该词却返回“没有找到相关内容”。
- **正确事实：**搜索弹窗与结果跳转可用，但并非全文搜索；每篇只收开头 400 字与 h2 标题，不收后段正文和 h3 正文。
- **证据：**线上 Chromium 实际输入 `26.3.27`，结果链接数 0；输入“负载策略”返回 1 条并正确跳到 `#/guide/relay`；[index.html:L1186](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L1186)–1195 的索引构建可直接解释漏检。建议扩大索引正文范围。这不是重复报告上轮已经修复的搜索打开／关闭交互。

### F-19 · P1 · Cloudflare 缓存验证将 BYPASS 设成唯一通过条件

- **位置／原文：**订阅前置地址 [index.html:L876](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L876)：“cf-cache-status 必须是 BYPASS”；[index.html:L879](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L879)–880 的自检命令只接受这个期望输出。
- **正确事实：**命中 Cache Rules 的 Bypass cache 设置时，Cloudflare 也会返回 `DYNAMIC`；它表示请求时即判为不具备缓存资格。不能仅凭不是 `BYPASS` 就断言配置失败。应核实请求确实到达订阅端点、绕过规则匹配且未被缓存，而非限定一个头值。
- **证据：**[Cloudflare 官方 cache responses 文档的 DYNAMIC 段](https://developers.cloudflare.com/cache/concepts/cache-responses/#dynamic) 将命中 Bypass cache 的 Cache Rule 列为原因。面板源码 [vite-frontend/src/pages/config/sub-front-card.tsx:81](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/config/sub-front-card.tsx#L81)–82 也写着“应显示 BYPASS”，故这是手册沿用了源码内的说明错误；不将它写成手册与源码不一致。该项只针对外部 CDN 的验证判据；未访问或猜测用户的 Cloudflare 配置。

### F-20 · P2 · “8 个客户端一键导入”把通用复制入口算成调起应用

- **位置／原文：**首页 [index.html:L305](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L305)：“8 个客户端入口一键导入”；导入到客户端 [index.html:L839](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L839) 笼统说“点客户端名直接导入，面板会调起对应客户端”。
- **正确事实：**确有 8 个入口，但其中 7 个配置应用 URL scheme；“通用”入口只复制 base64 格式订阅链接，不调起第 8 个应用。该章下方“通用”的表格已写复制，与前面的总述不一致。
- **证据：**[vite-frontend/src/pages/subscription/clients.ts:12](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/subscription/clients.ts#L12)–61，最后一项 `importURL: null`；[vite-frontend/src/pages/subscription.tsx:1632](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/subscription.tsx#L1632)–1639 明确走复制分支。图 25 能证明入口布局，不能证明实际调起客户端成功。

## 3. 按任务 11 项的覆盖结果

| 审计项 | 本次结果与依据 |
|---|---|
| 1. 事实准确性 | 阅读全部页面正文、字段表、提示及 FAQ，按发布 tag 查前端并补查实现。主要差异见 F-01 至 F-14、F-20；没有登录 094 的操作均不标成 live 实测。|
| 2. 图注与图面 | 31 处图逐张看，名称错误 F-15、版本证据 F-16、目标标签截断 F-17；其余逐图说明见附录。|
| 3. 空态 | 节点、入站、中转线路／隧道／规则／质量、订阅、用户列表均有对象；图 11 订阅源、图 24 分配的初始状态已诚实声明，通过。概览也诚实显示落地／中转 0 和流量空态。|
| 4. 已删功能 | Gist、PostgreSQL 设置均只在 FAQ 的已不存在说明出现；登录无通行证入口的说明与前端相符。13 类备份中保留通行证密钥不等于登录功能。管理员 `/monitor` 重定向正确，但按钮行为有 F-05；另发现已删订阅链式仍被写成存在（F-02）。|
| 5. 安装指令 | 安装脚本下载只有 `https://example.com/flmx/panel_install.sh`；订阅自检使用 `sub.example.com`，`-a`、`-s` 使用中文面板地址／节点密钥占位。未发现私人仓库下载地址、真实个人域名或真实 token／密钥／口令。`admin@user` 是旧公开口令错误 F-01，不声称泄露现用口令。截图里的真实 IP、协议伪装／探测域名不是安装下载链接。|
| 6. 概率 | 三策略 UI 与解释总体正确：在候选健康、等权且集合固定的前提下 fifo 用第一候选、其余平时 0；round 循环按连接分配，各约 1/N；rand 等权各约 1/N、短期不均。[go-gost/x/selector/strategy.go:28](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-gost/x/selector/strategy.go#L28)–34、50–70、73–87。**这是连接选择比例，不保证各出口字节流量相等**；随机实现还支持权重，不能将源码的全部使用场景一律推为等权。未把这些限定另报成已证实的概率错误。|
| 7. 订阅口径 | 回环分享地址会被过滤，与图 13 的 `127.0.0.1` 和图 23 的 0 节点相容，但本次未独立复现地址修改前后因果。公网地址不是“保证有节点”的充分条件，还受分享地址选择、中转范围、启用状态及格式过滤影响。节点数是对应格式可渲染的订阅端点数，不能等同物理服务器数或连通性测试结果；[go-backend/internal/http/handler/subscription.go:962](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/subscription.go#L962)、[go-backend/internal/http/handler/subscription.go:1271](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/subscription.go#L1271)–1295；回环拒绝见 [go-backend/internal/kernel/import.go:87](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/kernel/import.go#L87)，分享生成见 [go-backend/internal/http/handler/kernel_relay.go:1633](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/kernel_relay.go#L1633)。统计的周期、导入不计入等基本口径一致，具体限制见 F-11，IPv4 填域名见 F-07。|
| 8. 链式代理 | 广州→深圳 IX→美国→SOCKS5 已明确标成用户个例，通过措辞核查；其历史运行及 UDP 端到端结果未核实。出口区未拍到已如实声明，源码同一弹窗下部确有出口区（[vite-frontend/src/pages/tunnel.tsx:3335](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/tunnel.tsx#L3335)）。链式新建的在线、去重、必选出口等要求可由 [go-backend/internal/http/handler/mutations.go:8530](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/mutations.go#L8530)、8651–8671 支持；nftables 直连／单入口限制见 [go-backend/internal/http/handler/nftables_runtime.go:118](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/go-backend/internal/http/handler/nftables_runtime.go#L118)。客户端订阅链式可用性的说明不通过（F-02）。|
| 9. 导航与链接 | 18 页逐页导航、33 次前后翻页通过；正文路由链接静态目标检查无错链；图片 31/31 加载。没有需要探测的外部 a 链接，example.com 代码示例不按死链误报。搜索基本交互通过、全文命中不足见 F-18。|
| 10. 移动端与 console | Chromium 140 / Playwright Core 1.55.0 实测线上站点；390×844 下 18 页 `scrollWidth=390`，无整页横向溢出。抽屉 top=64、bottom=844、clientHeight=scrollHeight=780；Esc、遮罩点击、选链接后关闭均成功。console error=0、pageerror=0、HTTP >=400 响应=0。只覆盖该浏览器与这组视口，未推断其他设备表现。|
| 11. 诚实性 | 可以确认图片内含真实 UI 形态及文本，但不能独立确认“094 拍摄”“演示数据已清理”“节点 IP 已恢复”及历史实测过程。版本说明有 F-16 的证据冲突，保留未核实。登录页英文标签可从 [vite-frontend/src/pages/index.tsx:274](https://github.com/iiiiiiiii7125-art/flmx-max/blob/0057995ff023fa1865197443127e797757b83c66/vite-frontend/src/pages/index.tsx#L274)、291、313 证实；切中文浏览器的历史试验未核实，缺登录截图本身不报问题。|

## 4. 本次线上交互记录

直接访问已发布 HTTPS URL，桌面 1440×1000、移动视口 390×844，使用无既有登录／缓存的浏览器上下文。只浏览手册，不向 094 发送业务请求。

| 操作 | 实际结果 |
|---|---|
| 首页 + 全部文章导航 | 18/18 选中正确 `.page.on`，桌面无整页横向溢出 |
| 上一页／下一页 | 16 次 next、17 次 prev，共 33 次，所有目的页与 ORDER 一致；末页无下一页有效链接 |
| 内部路由检查 | 没有指向未知 `#/guide/...` 的链接；本页目录由实际 h2/h3 生成 |
| 搜索“订阅” | 返回 14 条（实现有 14 条上限） |
| 搜索“负载策略”并点结果 | 1 条，跳转 `#/guide/relay`，关闭搜索层 |
| 搜索“nftables” | 1 条；不能据此证明全文覆盖 |
| 搜索“26.3.27” | 0 条，正文实际存在，见 F-18 |
| 搜索不存在的词 | 显示无结果提示 |
| Ctrl+K / Esc | 搜索打开／关闭成功 |
| 手动深色切换 | 切到 dark 成功；浅色系统下刷新重新跟随浅色，与 FAQ 一致 |
| 系统深色 | 设置系统偏好 dark 后刷新，使用 dark |
| 移动抽屉 | 最底部在视口内；打开时 body 锁滚动；Esc、遮罩、点击“订阅设置”均关闭 |
| 移动搜索／主题 | 负载策略 1 条；主题切换成功 |
| 错误与图片 | 无 console error、JS 异常及失败 HTTP 响应；31 处图片均 complete 且 naturalWidth > 0 |

没有把已修复的抽屉超高、无交互记录、弹窗误当页面、空表单误当已填写、节点下半部裁切等旧问题重新列为发现；本次独立记录上述已通过行为。

## 5. 未核实项目

1. 094 的当前前后端部署版本、截图实际拍摄来源、演示对象删除结果、节点公网 IP 是否恢复。截图和作者声明不足以独立验证历史清理操作；F-16 只报告现有版本证据矛盾。
2. 修改 127.0.0.1 前后的 live 订阅变化、最低客户端版本改成 26.3.27 后特定 Clash／Mihomo 版本的连接结果、广州→深圳 IX→美国→SOCKS5 个例及其 UDP 测试。本次没有管理员会话或对应客户端实测，不能宣称复现成功。
3. 链式出口区的实际下拉交互。源码确认区块存在且位于同弹窗后部，但本次未在 live 打开；手册已经声明未拍到，不因这个声明本身扣分。
4. 升级执行、节点安装、通知实际投递、数据导入覆盖与恢复、真实订阅链接和各客户端 URL scheme 的成功调起。本次仅对照入口／字段／实现，没有执行这些有副作用的操作。

## 附录 A：31 处截图逐张检查

M 行号是内联图片所在行，图注在紧接的下一行。SHA 为解码后原始图片字节 SHA-256 的前 16 位，供定位，不重新压缩。全局的版本／来源限制统一见 F-16 与未核实项目，不重复计数。

| 图 | 页面／HTML 位置 | SHA-256 前缀 | 实际图面与图注核查 |
|---|---|---|---|
| 01 | `home`／[index.html:L312](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L312) | `9c8cbe56bb7f0998` | 概览有在线节点及有效订阅；落地 0/0、中转 0、流量空态如实展示。图面只有六个可见导航项，设置入口由源码支持，不把其余裁切部分当作已显示。 |
| 02 | `getting-started`／[index.html:L328](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L328) | `9c8cbe56bb7f0998` | 与图 1 字节完全相同，不是第二次独立操作证据。图注谈五步后核对是操作建议；该图尚无落地和中转。 |
| 03 | `upgrade`／[index.html:L409](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L409) | `1068d110e6f06ccb` | 升级确认弹窗、toast、三条升级前确认及底部按钮均可见。已明确不是升级区全页，未重复报旧问题。 |
| 04 | `nodes`／[index.html:L433](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L433) | `893da69499078bb4` | 节点 B/C 离线及地址／空统计可见；第三张卡只露出顶部，图注已明确在线节点未完整拍到，通过裁切声明核查。 |
| 05 | `nodes`／[index.html:L446](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L446) | `32b113269b019732` | 新增节点上半部、空名称、备注示例占位、未分组及续费周期可见；图注把下面字段说成需要滚动，没有假称拍全。 |
| 06 | `nodes`／[index.html:L467](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L467) | `4e0b0bbef5ed77b1` | Xray 26.11.9 与 sing-box 1.14.4-flmx1 均显示运行中，入站数均为 0；不是空列表。 |
| 07 | `inbound`／[index.html:L489](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L489) | `3282773f0e944d61` | 模板选中、名称含模板前缀、端口留空；名称图注错误 F-15。 |
| 08 | `inbound`／[index.html:L493](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L493) | `22991985aa4d627a` | 一条已启用入站，52668、节点 r5-node-0241、上下行 0 B；名称仍带模板前缀，见 F-15。 |
| 09 | `inbound`／[index.html:L517](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L517) | `13205db3888b9a50` | 高级设置中段；上端监听／分享输入仅残留底边及说明，出站策略、空闲超时和不限开关可见；不作为完整上半部字段截图证据。 |
| 10 | `inbound`／[index.html:L530](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L530) | `8e141dcead121843` | 三个中转相关开关及说明、备注、JSON 示例和创建按钮可见，图注相符。 |
| 11 | `inbound`／[index.html:L536](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L536) | `f91c74701d1a34cc` | 订阅源空态明确声明初始状态，通过；正文数据方向有 F-14。 |
| 12 | `import`／[index.html:L549](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L549) | `9e30af8d68f41eb8` | 导入节点弹窗是未粘贴内容的表单，展示支持格式／文件入口；图注没有冒称已成功导入。 |
| 13 | `node-detail`／[index.html:L577](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L577) | `c98a6b2bc7e175ff` | 节点详情弹窗：127.0.0.1、在线、2.1.15-p2pacing、1天11小时、6.22/3.28 GB、nftables installed 及按钮可见。按钮点击效果不能由此静态图推断，见 F-05。 |
| 14 | `node-detail`／[index.html:L582](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L582) | `6b83ec5f29fcdd74` | CPU 5%、内存 37%、磁盘 63%，28.0K/s、30.5K/s、9.7G 和 1天13小时，与图注相符。 |
| 15 | `relay`／[index.html:L615](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L615) | `3a089131ab7b9b08` | 线路一条，演示-VLESS、入口 3.113.251.92:16123、1ms 均可见。这里只证明显示值，不证明端到端业务成功。 |
| 16 | `relay`／[index.html:L619](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L619) | `5260c3c2c1187ca5` | 隧道卡有端口转发／启用／1入口0跳1出口／双向计算／1x及操作按钮，图注相符。 |
| 17 | `relay`／[index.html:L640](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L640) | `8d718ce88a09d910` | 端口转发、面板转发、入口节点及高级设置标题可见；高级字段内容在可见区外，图注作为继续填写步骤，没有声称内容拍全。 |
| 18 | `relay`／[index.html:L652](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L652) | `6f5da9a5c01524b4` | 第1跳、添加一跳、空节点选择、TLS、轮询及连接IP标签可见；连接IP输入区仅顶边可见。出口未拍到声明诚实，不报已知缺口。 |
| 19 | `tunnel-quality`／[index.html:L695](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L695) | `7d1b9914dc4e3e42` | 有数据的质量卡，隧道1/1、每秒探测、08:48:43、入口→出口为横线、出口段2ms可见；目标名截断见 F-17。 |
| 20 | `rules`／[index.html:L721](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L721) | `b921ee75f64ade6d` | 有两条规则、正常／主备、入口目标地址截断、0 B及操作按钮可见；第二条名称也截断，完整含义需文字补充。没有伪装空列表。 |
| 21 | `subscription`／[index.html:L758](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L758) | `91d7cfb91e3f5611` | 两条订阅、各2个节点、不限与100GB、两个tab及模板按钮均可见，图注相符。 |
| 22 | `subscription`／[index.html:L771](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L771) | `2ffa82a34ebe72ec` | 新订阅表单上半部，名称为空且显示示例占位；到期快捷按钮、流量档、每月重置均可见。未假称已填写。 |
| 23 | `subscription`／[index.html:L789](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L789) | `db54b952e43b1cd5` | 0节点的订阅详情、100GB、提示、小时日月与近24小时0 B均可见；该图诚实标成0节点状态，非空列表冒充有数据。 |
| 24 | `sub-assign`／[index.html:L814](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L814) | `7f1308ea5f93f66f` | 尚未选择用户的订阅分配初始状态，图注明确，且图面说明流量与到期按订阅单独计算（F-03）。 |
| 25 | `sub-clients`／[index.html:L833](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L833) | `db220eeb6f60196f` | 复制／二维码、两行8个入口、底部按钮和计量文字可见；Shadowrocket、Quantumult X 的视觉标签被截断，完整名有源码支持，不能声称逐字全显。通用动作见 F-20。 |
| 26 | `users`／[index.html:L891](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L891) | `da97fb7fff11e941` | 一个演示用户及本人配额区有数据；1000GB、规则10、每月1号、到期日及底部按钮可见。顶部实际是用户／限速，见 F-13。 |
| 27 | `users`／[index.html:L903](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L903) | `72768d1b27be0d8e` | 用户名／密码未填、流量1000及规则10为默认，最大连接／日期留空；已修图注诚实，不重复报旧问题。状态与不重置是选择器默认态。 |
| 28 | `config`／[index.html:L927](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L927) | `c5d19f69115ddcad` | 品牌设置及96x96说明／实时预览可见，图注相符。 |
| 29 | `config`／[index.html:L940](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L940) | `90a3e18f07842847` | 通知下半部、资源告警可见文字、阈值20%/0、9点、发送测试、测试点标题可见；通道及10事件不在图中，正文明确用上半部补充。 |
| 30 | `config`／[index.html:L966](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L966) | `55b2549b64830f22` | 备份与恢复两块、0/13及选择按钮完整可见，图注相符。 |
| 31 | `config`／[index.html:L970](https://github.com/iiiiiiiii7125-art/flmx-manual/blob/99f540cd396a0501cb9503dec7b29a14460e6296/index.html#L970) | `f03a2a6f3ced746b` | 导出内容弹窗的13个类型、全选／清空／确认导出可见，图注相符；含通行证密钥数据类型不等于通行证登录入口存在。 |

## 附录 B：只读复核方法

可用下面的标准库脚本从本次固定仓库快照还原图片。输出目录建在临时目录即可，无须改动手册：

```python
import base64, hashlib, re
from pathlib import Path

html = Path('index.html').read_text()
out = Path('/tmp/flmx-manual-images')
out.mkdir(exist_ok=True)
for number, match in enumerate(re.finditer(
    r'data:(image/[^;]+);base64,([^\s"\'<>]+)', html
), 1):
    mime, payload = match.groups()
    raw = base64.b64decode(payload)
    suffix = {'image/jpeg': 'jpg', 'image/webp': 'webp'}.get(mime, 'bin')
    target = out / f'{number:02}.{suffix}'
    target.write_bytes(raw)
    print(number, hashlib.sha256(raw).hexdigest(), target)
```

版本复核使用 `git rev-parse 2.1.30^{commit}`；导出使用 `git archive 2.1.30 vite-frontend go-backend go-gost install.sh panel_install.sh` 到独立目录。本报告不使用未发布工作区修改。本次没有修改产品代码，未运行 flmx-max 构建或回归套件；所需验证为手册逐文／逐图审读、发布版源码追踪和已发布网站浏览器交互检查。
