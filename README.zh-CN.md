# stakeholder-writeup

**语言：** [English](README.md) | 中文

[@zhujian0409](https://github.com/zhujian0409) 写的一个适用于 Claude Code 和 Codex 的 skill——把一段多轮的技术调查对话整理成给非工程受众看的 Markdown 报告：只写有证据的事实、按受众调结构、显式调用才触发。

## 干什么用

跟 AI agent 调查一个线上问题或者一个新功能，聊了 20-50 轮之后，直接说"写一份报告给我老板"，拿回来的东西通常问题不少——把猜测和核实过的数据混在一起、往给高层看的稿里塞 SQL、影响范围用一些模糊的"已缓解"糊过去。

`stakeholder-writeup` 强制走一条不一样的路：

- **Step 0 — 受众菜单。** 调用时先让你选 1-4（技术 leader / 不懂技术的高层 / PM·业务 / 外部客户）。骨架、术语密度、结尾模板都根据你的选择切换。
- **事实核验。** 草稿里每一个数字结论都必须能追溯到你实际跑过的命令或你亲口说过的话。不准出现"大概改善了 25%"这种没人量过的数。
- **影响诚实。** 不知道就写不知道（"影响范围：未测量——需要额外 N+1 小时日志分析确认"），不拿"已缓解"之类的词糊过去。
- **12 项发送前自检。** 交付前跑一遍，任何一条没证据来源直接 fail，不给你偷懒。

输出语言自动识别：默认英文，当对话主要是中文时切中文。

## 什么时候调用

**只在显式调用时触发**——skill 会 hard-ignore 像"写报告""写个 md"这种随口一说。要触发请用：

- 斜杠命令：`/stakeholder-writeup`
- Codex skill 调用：`$stakeholder-writeup`
- 显式点名："用 stakeholder-writeup 写一份我们刚才的调试总结"
- 英文：`use the stakeholder-writeup skill`

Codex 侧通过 `stakeholder-writeup/agents/openai.yaml` 设置了 `allow_implicit_invocation: false`，所以默认也是显式调用才触发。

## 安装

### Claude Code

clone 本仓库，把 skill 目录拷到你的 Claude Code 用户级 skill 目录：

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/zhujian0409/stakeholder-writeup-skills.git
cp -r stakeholder-writeup-skills/stakeholder-writeup ~/.claude/skills/
```

或者保留本仓库，用软链接（这样 `git pull` 就能直接更新 skill）：

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/zhujian0409/stakeholder-writeup-skills.git
ln -s "$(pwd)/stakeholder-writeup-skills/stakeholder-writeup" ~/.claude/skills/stakeholder-writeup
```

放到 `~/.claude/skills/` 下的 skill 会被 Claude Code 自动注册。下一次会话里 `/stakeholder-writeup` 就会作为斜杠命令出现。

### Codex

clone 本仓库，把 skill 目录拷到 Codex 用户级 skills 目录：

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/zhujian0409/stakeholder-writeup-skills.git
cp -r stakeholder-writeup-skills/stakeholder-writeup ~/.codex/skills/
```

或者保留本仓库，用软链接：

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/zhujian0409/stakeholder-writeup-skills.git
ln -s "$(pwd)/stakeholder-writeup-skills/stakeholder-writeup" ~/.codex/skills/stakeholder-writeup
```

安装后重启 Codex。使用 `$stakeholder-writeup` 调用。

## 可选私有归档库

如果希望每份报告自动进入一个可搜索的私有归档库，可以先建一个 private git 仓库，然后设置：

```bash
export STAKEHOLDER_WRITEUPS_DIR=/path/to/private/stakeholder-writeups
```

skill 会使用：

```text
reports/YYYY/MM/YYYY-MM-DD_查询人_修复人_主题.md
index.md
index.json
```

如果报告里包含内部系统名、人员、日志、SQL、客户上下文或运维细节，归档库应保持 private。不要把密码、token、私钥、API key 或原始敏感凭据写进报告。

## 设计理念

这个 skill 贯彻两条：

1. **要证据，不要感觉。** 任何数字声明必须能追溯到具体命令输出或你亲口说过的话，不允许"估算"冒充"事实"。发送前的自检清单有任何条目没证据来源就直接 fail。
2. **按受众调输出。** CEO 不想看 SQL 附录，技术 leader 不想看类比。skill 会识别这份东西是给谁看的，相应调整篇幅、术语密度、结尾模板的形式。

完整 7 步流程、骨架映射表、受众变体对照表、以及 skill 拒绝犯的那些真实世界坑点，见 [SKILL.md](stakeholder-writeup/SKILL.md)。

## License

MIT——见 [LICENSE](LICENSE)。
