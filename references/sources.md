# 数据源（2026-09-29 实测）

**实测时间**：2026-09-29 15:00–15:30（北京时间），本机 curl 逐个调通/证伪。反爬注意写在每条"请求要求"里。

## 一、主力：天天基金移动端 API（FundMNewApi）✅

域名 `fundmobapi.eastmoney.com`（本沙箱实测可达；天天基金 web 全站 `fund.eastmoney.com` / `fundf10.eastmoney.com` 在本沙箱 egress 层 TCP 超时，UA/Referer 无效，网络层先挂——见"证伪"节）。

通用请求要求：桌面或移动端 UA；公共参数 `deviceid=Wap&plat=Wap&product=EFund&version=2.0.0`。返回 JSON（UTF-8），`Datas` 节点为业务数据。

### 1. 基金基本信息（净值/收益/费率/规模/经理名）

`https://fundmobapi.eastmoney.com/FundMNewApi/FundMNBasicInformation?FCODE={code}&deviceid=Wap&plat=Wap&product=EFund&version=2.0.0`

- 实测：2026-09-29 一次调通（FCODE=161005）
- 关键字段：`SHORTNAME`（简称）、`FTYPE`（类型）、`FSRQ`（净值日期，如 2026-09-28）、`RZDF`（日涨跌幅，如 -2.88）、`DWJZ`（单位净值 2.8935）、`LJJZ`（累计净值 5.9415）、`SYL_Z/Y/3Y/6Y/1N/2N/3N/JN/LN`（近1周/1月/3月/6月/1年/2年/3年/今年来/成立来收益率）、`SOURCERATE`（原申购费率 1.50%）、`RATE`（现行申购费率 0.15%）、`MAXRETRA1`（近1年最大回撤 12.5725）、`FEGM`（规模，口径以详情接口为准）、`JJJL`（经理姓名）、`ESTABDATE`（成立日）、`_estabdate`
- 用途：诊断模块的净值/回撤/费率/规模体检；定投测算的费率参数

### 2. 基金详情（规模季报/费率明细/基准/策略）

`https://fundmobapi.eastmoney.com/FundMNewApi/FundMNDetailInformation?FCODE={code}&deviceid=Wap&plat=Wap&product=EFund&version=2.0.0`

- 实测：2026-09-29 一次调通（FCODE=161005）
- 关键字段：`FULLNAME`（全称）、`ENDNAV`（季报规模 18396711445.65）、`FEGMRQ`（规模日期 2026-06-30）、`MGREXP`（管理费 1.20%）、`TRUSTEXP`（托管费 0.20%）、`SALESEXP`（销售服务费 0.00%）、`RISKLEVEL`（风险等级 4=中高）、`JJGS`（富国基金）、`TGYH`（托管行 工商银行）、`PERFCMP`（业绩基准：沪深300指数收益率×70%+中债综合全价指数收益率×25%+同业存款利率×5%）、`INVTGT`/`INVSTRA`（投资目标/策略文本）
- 用途：费率体检、规模健康度（警戒线：股票型<2 亿提示清盘风险）、基准（风格剥离用）

## 二、证伪与反爬（不要用，写清原因）

1. **天天基金 web 端全站**（`fund.eastmoney.com/pingzhongdata/{code}.js` 净值走势、`f10/FundArchivesDatas.aspx?type=lsjz|jjcc|jjjl` 历史净值/十大重仓/经理履历、`fundgz.1234567.com.cn/js/{code}.js` 盘中估值）：2026-09-29 本沙箱实测 **TCP 超时（HTTP 000，15 秒无响应）**，带桌面 UA + Referer `https://fund.eastmoney.com/` 依然不通。同日早间（03:08）新浪接口曾可用，属本沙箱 egress 到中国财经站点波动，**非反爬层面问题**。不要因为"换了 UA 就能用"反复死磕，直接走降级链。
2. **FundMNewApi 其余端点**（`FundNetDiagramNew` 净值走势、`FundManagerInfoNew` 经理信息、`FundMNewApi/FundMNFInfo` 基金列表）：HTTP 200 但返回 `{"ErrCode":404,"ErrMsg":"网络繁忙，请稍后重试！"}`——**反爬稳定拒绝**（间隔 10 秒重试一次，依然拒绝；基本信息/详情接口同时正常，排除临时波动）。定投测算的历史净值序列、经理任职起止与十大重仓因此**没有稳定程序化接口**，走降级链。
3. **新浪基金行情** `https://hq.sinajs.cn/list=fu{code}`（如 fu161005）：同上，本沙箱 egress 超时，不可达。
4. **腾讯** `https://qt.gtimg.cn/q=ff{code}`：本沙箱 egress 超时，不可达。

## 三、降级链

**Fallback 总链**：FundMNewApi（基本信息/详情）→ 网页搜索 → 用户手动提供 → 标注"未核验"。

| 数据项 | 主力 | 备选（降级） | 实在拿不到 |
|---|---|---|---|
| 单位净值/日涨跌/分段收益 | FundMNBasicInformation | 搜索 `{代码} 基金净值`（天天基金/晨星页面，注明来源与日期） | 标注"净值未核验"，诊断降级为"框架版" |
| 费率（管理/托管/申购） | FundMNDetailInformation（MGREXP/TRUSTEXP）+ Basic（SOURCERATE/RATE） | 搜索 `{代码} 基金 费率` 或基金产品资料概要 PDF（pdf.dfcfw.com） | 标注"费率未核验" |
| 规模 | FundMNDetailInformation（ENDNAV + FEGMRQ） | 搜索 `{代码} 最新规模`（新浪基金∞文章常带"最新规模XXX亿"） | 标注"规模未核验" |
| 基金经理/任职日期 | Basic（JJJL 姓名）+ 产品资料概要 PDF（任职日期） | 搜索 `{经理名} 累计任职`（新浪基金∞文章带"累计任职X年X天/现任资产总规模/任职最佳最差回报"） | 姓名标注"未核验" |
| 十大重仓/持仓集中度 | ——（无稳定接口） | 搜索 `{代码} 十大重仓股` / `{代码} 二季报`（新浪基金∞"从基金十大重仓股角度"表格：持股数/占净值比/变动）；巨潮季报 PDF | 标注"持仓未披露（季报未出或搜不到），集中度项跳过" |
| 净值历史序列（定投测算输入） | ——（接口被反爬） | 用户按 SKILL.md 附录模板提供净值表（CSV：日期,单位净值） | 测算降级为"公式说明+示例"，**绝不编造净值** |
| 基准同期涨跌幅（风格剥离用） | ——（无稳定程序化接口） | 搜索 `{基准指数} 近一年涨跌幅`（如"沪深300 近一年涨跌幅"，取新浪财经/东方财富指数页） | 风格剥离只列基准口径、不硬判 Alpha/Beta，标注"基准同期表现未核验" |
| 买卖点验尸（需历史逐季持仓） | ——（重型审计，超出 MVP） | 网页搜索经理代表作季报观点（"季报观点"文本） | 审计降级为"轻量三问版"（会不会卖/风格白送/抄作业价值），深度验尸明确标注"本 skill 不做" |

## 四、搜索模板（备选路径的标准关键词）

- 净值与业绩：`{代码} 基金净值` / `{代码} 今年以来收益 同类排名`
- 经理行为：`{基金名} {经理名} 累计任职` / `{经理名} 季报观点` / `{代码} 十大重仓股`
- 定投参考：`{代码} 定投 收益`（只取公开测算口径，不直接引用结论）
- 搜索结果优先取：天天基金/晨星/新浪基金∞/基金公司官网 PDF；每条数据注明"来源+日期"，单源标注来源，双源交叉后写"双源一致"。
