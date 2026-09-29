# 基金诊断

`laogu-fund`

买基前先做体检：**基金诊断**（净值/回撤/费率/规模/持仓集中度体检定级）+ **基金经理行为审计**（经理会不会卖/收益多少是风格白送/季报抄作业值不值/造神检测）+ **定投测算**（每月投 X 元、投 N 年的历史测算），三件套一次看清这只基健不健康。

只做体检与逻辑陈述，**不做买卖推荐**。数据注明日期时点与来源，缺数据直说不编造。

## 文件结构

- `SKILL.md` — 主流程（平台中立，纯 Markdown，可移植）
- `references/sources.md` — 数据源实测记录（天天基金移动端 API + 降级链）
- `MARKET.md` — 市场调研（供给/需求/槽点/差异化）
- `mcp-config.json` — MCP 热更新配置（接口模板/字段映射/降级顺序）
- `manifest.yaml` / `icon-512.png` / `LICENSE` — 发布配套文件

## 输出结构

- 一句话定级：**值得跟踪 / 保持观望 / 提示风险**（三档之一，不构成推荐）
- 体检表：净值与收益 / 回撤 / 费率 / 规模 / 持仓集中度（通过-注意-预警）
- 经理行为审计：会不会卖 / 风格剥离（禁止把 Beta 当 Alpha）/ 抄作业价值 / 造神检测（只陈述可验证事实）
- 定投测算：累计投入 / 总份额 / 赎回金额 / 收益率（确定性计算，不编数；历史测算不代表未来）
- 反偏见自查清单 + 作者声明

## 一键安装

```bash
# 方式一：git clone（仓库地址）
git clone https://github.com/laogu-caibao/laogu-fund.git
```

- ZIP 下载：https://github.com/laogu-caibao/laogu-fund/archive/refs/heads/main.zip
- 一键安装命令：`npx skills add laogu-caibao/laogu-fund`
- 扣子（Coze）：扣子编程 → 技能面板 → 创建技能 → 本地上传（仓库根目录已有 SKILL.md，直接压缩仓库文件夹即可）；页面要求 `.skill` 后缀时由扣子导入后自动生成，**不要只改 zip 扩展名**
- Trae：设置 → 技能 → 上传技能（同上 zip）；或手动放到 `~/.trae/skills/laogu-fund/`（TRAE Work 国区版路径为 `~/.trae-cn/skills/laogu-fund/`）
- MCP 一次装全：把 `uvx laogu-mcp` 配进 Agent 的 MCP 设置即得全部基金/股票工具（skill 负责流程指导、MCP 负责工具调用）

---

## 出品

**老谷拆财报** —— 以数据为刃，剖市场真相

- 抖音 / 微信视频号 / 今日头条 / 快手：搜索「老谷拆财报」
- 固定栏目：「价值投资之财报解读」（全网连载中）

> 作者声明：个人观点，仅供参考，不构成投资建议。
