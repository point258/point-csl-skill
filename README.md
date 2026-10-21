# point老三样焚决

两个 agent skill：在 Linux 服务器上部署 AstrBot + NapCat 的 QQ 群聊机器人，并给它写一份"像活人"的人格。

不绑定具体客户端，任何能执行 shell / HTTP / SQL 的 harness 都能用。

---

## 内容物

```
point老三样焚决/
├── 00-先看这里.md
├── astrbot-qq-deploy/            部署
│   ├── SKILL.md
│   ├── references/
│   │   ├── 00-intake.md          信息收集与提问清单
│   │   ├── 10-server-docker.md   服务器、compose、端口
│   │   ├── 20-napcat-qq.md       NapCat 登录与 OneBot 连接
│   │   ├── 30-astrbot-core.md    核心配置逐项说明
│   │   ├── 40-plugins.md         四个插件的作用、槽位、坑
│   │   ├── 50-verify.md          验收清单
│   │   ├── 60-troubleshooting.md 15 条症状 → 根因 → 修法
│   │   └── 70-ops.md             备份、清理、改人格、升级
│   ├── assets/docker-compose.yml.tpl
│   └── scripts/
│       ├── patch_plugins.py      一键打补丁（幂等，可 --check）
│       ├── verify_deploy.py      一键验收
│       └── ssh_helper.py         SSH 兜底（仅依赖 paramiko）
└── laosan-persona-forge/         人格
    ├── SKILL.md
    ├── references/
    │   ├── mechanisms.md         活人感原理与 8 个机制
    │   ├── anti-slop.md          AI 味词表与结构禁忌
    │   ├── persona-skeleton.md   15 段骨架逐段写法
    │   ├── plugin-bridge.md      插件注入格式与对接
    │   └── sources.md            调研来源
    ├── assets/persona-template.md
    └── scripts/persona_lint.py   人格体检（无第三方依赖）
```

---

## 功能

**部署**

- 从空服务器到群里能说话：Docker 编排、NapCat 扫码登录、OneBot 反向连接
- 模型分工：对话 / 快速（读空气、识图、分段）/ 高推理（记忆整理），三档分开配
- 四个插件全部配好：group_chat_plus、livingmemory、self_learning、smart_segmentation
- 五个必打补丁 + 一个可选的每日好感度上限，一条命令应用
- 24 项验收：配置、人格、插件槽位、三个模型连通性、识图

**人格**

- 15 段骨架：元层前提、人形与称谓、人称与身份、亲近程度、此刻的心情、核心驱动、矛盾缺陷、语言指纹、表达落差、群内社交位、状态漂移、意外性协议、记忆与工具、表情与梗、红线
- 三段插件强依赖段：把注入的 `好感度: X/100` 和情绪类型翻译成具体行为
- 自动体检：查 AI 味词、破折号、否定式排比、三段式、句长单调、段末总结、形容词密度、台词示例数量、结构缺失

**排障与运维**

- 15 条症状 → 根因 → 修法，覆盖"不说话 / @ 不回 / 人格不生效 / 心情不更新"等
- 三档清理：只清会话 / 清会话加记忆 / 连好感度一起清，都保留回滚手段
- 备份、改人格流程、升级注意事项

---

## 特色

**不绑定 harness。** 全程只用标准 shell、HTTP、SQL。有 SSH 工具就用，没有就用 `ssh` 命令，再不行用附带的 `ssh_helper.py`。

**信息不足会停下来问。** 服务器地址、模型密钥属硬门槛，缺了不猜；角色设定属软信息，可用默认值但会标注假设；首轮已给的不重复问；用户说"你看着办"就列清楚替你决定了什么。

**补丁幂等可重放。** `patch_plugins.py --check` 先看缺什么，`--all` 应用，重复执行不会重复改。每个被改的源文件都留 `.bak-<时间戳>`。

**验收要证据，不要"应该没问题"。** `verify_deploy.py` 逐项打 PASS/FAIL，模型用各自的 base_url 和 key 实发请求，识图会造一张纯色图问颜色。

**不写套路。** 不给标准台词，因为示例越"好"模型越会反复复用；不写"温柔""傲娇"这类形容词，改成"什么情况下做什么"。

**隐私已清洗。** 无 IP、密码、API key、QQ 号、群号，可直接公开。

---

## 使用方法

### 1. 安装

把两个目录放进 harness 的技能目录：

```bash
# WorkBuddy
cp -r astrbot-qq-deploy laosan-persona-forge ~/.workbuddy/skills/

# 其它 harness：放进它约定的 skills 目录即可
```

### 2. 触发

不需要记命令，说人话即可：

| 你说 | 走哪个 |
|---|---|
| 「帮我部署一个 QQ 机器人」 | astrbot-qq-deploy |
| 「她说话太像客服了」/「给她写个人格」 | laosan-persona-forge |
| 「@ 她不理我」 | astrbot-qq-deploy 的排障 |

### 3. 典型流程

```
说需求 → 补齐信息 → 起容器 → 扫码登录 → 配模型 → 装插件 → 打补丁
      → 写人格 → 人库 → 验收 24 项 → 交付说明
```

### 4. 手工用脚本

```bash
# 在服务器上执行
python3 patch_plugins.py --check           # 看补丁状态
python3 patch_plugins.py --all             # 打基础补丁
python3 patch_plugins.py --daily-cap 10    # 可选：每日净增上限

python3 verify_deploy.py                   # 配置验收
python3 verify_deploy.py --models --vision # 加测模型与识图

python3 persona_lint.py 人格.md             # 人格体检，85 分以上可用
```

### 5. 依赖

| 环境 | 需要 |
|---|---|
| 服务器 | Linux、Docker 24+、内存 ≥1G（1G 要开 swap） |
| 本机/服务器脚本 | Python 3.9+，`ssh_helper.py` 需 `pip install paramiko` |
| 人格体检 | 无第三方依赖 |
