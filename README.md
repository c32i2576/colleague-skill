<div align="center">

# colleague.skill

> *"You AI guys are traitors to the codebase — you've already killed frontend, now you're coming for backend, QA, ops, infosec, chip design, and eventually yourselves and all of humanity"*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://python.org)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

[![Discord](https://img.shields.io/badge/Discord-Join%20Community-5865F2?logo=discord&logoColor=white)](https://discord.gg/aRjmJBdK)

<br>

Your colleague quit, leaving behind a mountain of unmaintained docs?<br>
Your intern left, nothing but an empty desk and a half-finished project?<br>
Your mentor graduated, taking all the context and experience with them?<br>
Your partner transferred, and the chemistry you built reset to zero overnight?<br>
Your predecessor handed over, trying to condense three years into three pages?<br>

**Turn cold goodbyes into warm Skills — welcome to cyber-immortality!**

<br>

Provide source materials (Feishu messages, DingTalk docs, Slack messages, emails, screenshots)<br>
plus your subjective description of the person<br>
and get an **AI Skill that actually works like them**

[Supported Sources](#supported-data-sources) · [Install](#install) · [Usage](#usage) · [Demo](#demo) · [Detailed Install](INSTALL.md) · [💬 Discord](https://discord.gg/aRjmJBdK)

[**中文**](docs/lang/README_ZH.md) · [**Español**](docs/lang/README_ES.md) · [**Deutsch**](docs/lang/README_DE.md) · [**日本語**](docs/lang/README_JA.md) · [**Русский**](docs/lang/README_RU.md) · [**Português**](docs/lang/README_PT.md) · [**한국어**](docs/lang/README_KO.md)

</div>

---

> 🆕 **2026.04.14 Update** — **WeChat group is live!** Come hang out with the dot-skill community — share skills, discuss features, trade tips.
>
> <img src="docs/assets/wechat-group-qr-2.png" alt="dot-skill WeChat group QR" width="240">
>
> QR refreshes every 7 days — if expired, ping me on Discord.

> 🆕 **2026.04.13 Update** — **dot-skill Roadmap is live!** colleague.skill is evolving into **dot-skill** — distill anyone, not just colleagues. Multimodal output, skill ecosystems, and more on the way.
>
> 👉 **[Read the full Roadmap](ROADMAP.md)** · **[💬 Discord](https://discord.gg/aRjmJBdK)**
>
> We've also cleaned up Issues, added Milestones, and set up a [public project board](https://github.com/users/titanwings/projects/1). Community contributions welcome — check `good-first-issue` labels!

> 🆕 **2026.04.07 Update** — The community's enthusiasm for dot-skill remixes has been incredible! I've built a community gallery — PRs welcome!
>
> Share any skill or meta-skill, and drive traffic directly to your own GitHub repo. No middleman.
>
> 👉 **[titanwings.github.io/colleague-skill-site](https://titanwings.github.io/colleague-skill-site/)**
>
> Now listed: 户晨风.skill · 峰哥亡命天涯.skill · 罗翔.skill and more

---

Created by [@titanwings](https://github.com/titanwings) | Powered by Shanghai AI Lab · AI Safety Center

## Supported Data Sources

> This is still a beta version of colleague.skill — more sources coming soon, stay tuned!

| Source | Messages | Docs / Wiki | Spreadsheets | Notes |
|--------|:--------:|:-----------:|:------------:|-------|
| Feishu (auto) | ✅ API | ✅ | ✅ | Just enter a name, fully automatic |
| DingTalk (auto) | ⚠️ Browser | ✅ | ✅ | DingTalk API doesn't support message history |
| Slack (auto) | ✅ API | — | — | Requires admin to install Bot; free plan limited to 90 days |
| WeChat chat history | ✅ SQLite | — | — | Currently unstable, recommend using open-source tools below |
| PDF | — | ✅ | — | Manual upload |
| Images / Screenshots | ✅ | — | — | Manual upload |
| Feishu JSON export | ✅ | ✅ | — | Manual upload |
| Email `.eml` / `.mbox` | ✅ | — | — | Manual upload |
| Markdown | ✅ | ✅ | — | Manual upload |
| Paste text directly | ✅ | — | — | Manual input |

### Recommended WeChat Chat Export Tools

These are independent open-source projects — this project does not include their code, but our parsers are compatible with their export formats. WeChat auto-decryption is currently unstable, so we recommend using these open-source tools to export chat history, then paste or import into this project:

| Tool | Platform | Description |
|------|----------|-------------|
| [WeChatMsg](https://github.com/LC044/WeChatMsg) | Windows | WeChat chat history export, supports multiple formats |
| [PyWxDump](https://github.com/xaoyaoo/PyWxDump) | Windows | WeChat database decryption & export |
| [留痕 (Liuhen)](https://github.com/greyovo/留痕) | macOS | WeChat chat history export (recommended for Mac users) |

> Tool recommendations from [@therealXiaomanChu](https://github.com/therealXiaomanChu). Thanks to all the open-source authors — together for cyber-immortality!

---

## Install

### Claude Code

> **Important**: Claude Code looks for skills in `.claude/skills/{name}/SKILL.md`. The repo now ships **two independent skills** under `skills/`. Install each one separately.

```bash
# Project-level install (run at your git repo root)
mkdir -p .claude/skills
git clone https://github.com/titanwings/colleague-skill /tmp/colleague-skill-src
cp -r /tmp/colleague-skill-src/skills/create-colleague   .claude/skills/
cp -r /tmp/colleague-skill-src/skills/create-politician  .claude/skills/

# Or global install (available in all projects)
git clone https://github.com/titanwings/colleague-skill /tmp/colleague-skill-src
cp -r /tmp/colleague-skill-src/skills/create-colleague   ~/.claude/skills/
cp -r /tmp/colleague-skill-src/skills/create-politician  ~/.claude/skills/
```

After install, restart Claude Code so `/create-colleague` and `/create-politician` are registered as slash commands.

### OpenClaw

```bash
git clone https://github.com/titanwings/colleague-skill /tmp/colleague-skill-src
cp -r /tmp/colleague-skill-src/skills/create-colleague   ~/.openclaw/workspace/skills/
cp -r /tmp/colleague-skill-src/skills/create-politician  ~/.openclaw/workspace/skills/
```

### Dependencies (optional)

```bash
pip3 install -r requirements.txt
```

> Feishu/DingTalk/Slack auto-collection requires App credentials. See [INSTALL.md](INSTALL.md) for details.

---

## Usage

### Create a Colleague Skill

In Claude Code, type:

```
/create-colleague
```

Follow the prompts: enter an alias, company/level (e.g. `ByteDance L2-1 backend engineer`), personality tags, then choose a data source. All fields can be skipped — even a description alone can generate a Skill.

Once created, invoke the colleague Skill with `/{slug}`.

### Create a Politician Skill (v2 — Nuwa-style)

In Claude Code, type:

```
/create-politician
```

**v2 flow** (inspired by [nuwa-skill](https://github.com/alchaincyf/nuwa-skill)):

1. **3-question intake** — slug, basic info (country/party/position/era), initial profile
2. **Source strategy** — user-provided / **six-agent parallel auto-research** / hybrid
3. **Six-track research** (optional) — speeches, votes/decisions, social media, media critique, interviews/bio, timeline — all written to `research/` inside the skill dir
4. **Three-fold validation** — each candidate mental model must pass: cross-domain appearance (≥2 issues) + generative power (predicts new issues) + distinctiveness (non-obvious for this ideology)
5. **Build** — 3-7 mental models + 5-10 decision heuristics + quantified Expression DNA + **honest limitations** (what this Skill cannot do)
6. **Quality gates** — known-stance match / edge-case test / voice check before writing

Votes and executive decisions are the highest-weight evidence; speeches shape rhetoric; social media captures instinctive reactions; media commentary is validation only. Vote-vs-slogan conflicts are **preserved, not smoothed over** — votes form the stance baseline, slogans the rhetorical wrapper.

Once created, invoke the politician Skill with `/{slug}`.

See `references/extraction-framework.md` for the full methodology and `references/skill-template.md` for the output template.

### Colleague Commands

| Command | Description |
|---------|-------------|
| `/list-colleagues` | List all colleague Skills |
| `/{slug}` | Invoke full Skill (Persona + Work) |
| `/{slug}-work` | Work capabilities only |
| `/{slug}-persona` | Persona only |
| `/colleague-rollback {slug} {version}` | Rollback to a previous version |
| `/delete-colleague {slug}` | Delete |

### Politician Commands

| Command | Description |
|---------|-------------|
| `/list-politicians` | List all politician Skills |
| `/{slug}` | Invoke full Skill (Political Capability + Persona) |
| `/{slug}-political` | Political capability only |
| `/{slug}-persona` | Persona only |
| `/politician-rollback {slug} {version}` | Rollback to a previous version |
| `/delete-politician {slug}` | Delete |

---

## Demo

> Input: `ByteDance L2-1 backend engineer, INTJ, blame-shifter, ByteDance-style`

**Scenario 1: Code Review**

```
User            ❯ Can you review this API design?

colleague.skill ❯ Hold on — what's the impact? You haven't explained the context.
                  (after reading) N+1 query, fix it. Use the standard
                  {code, message, data} response format. That's the spec,
                  don't ask why.
```

**Scenario 2: Blame game**

```
User            ❯ This bug was introduced by you, right?

colleague.skill ❯ Does the timeline match? That feature touched multiple places,
                  there were other changes too.
```

---

## Features

### Generated Skill Structure

Each colleague Skill has two parts that work together:

| Part | Content |
|------|---------|
| **Part A — Work Skill** | Systems, tech standards, workflows, experience |
| **Part B — Persona** | 5-layer personality: hard rules → identity → expression → decisions → interpersonal |

Execution: `Receive task → Persona decides attitude → Work Skill executes → Output in their voice`

Each politician Skill (v2 schema) has six parts:

| Part | Content |
|------|---------|
| **Part A — Mental Models** | 3-7 thinking frameworks validated via cross-domain / generative / distinctiveness tests |
| **Part B — Decision Heuristics** | 5-10 case-backed triggerable rules |
| **Part C — Expression DNA** | Quantified sentence fingerprint, style spectrum, verbal tics, forbidden words, signature narratives, audience switching |
| **Part D — Evidence & Honest Limitations** | Hard-evidence summary, vote-vs-slogan tensions, inferred items, **explicit "this Skill cannot do X" list**, research cutoff date |
| **Part E — Execution Rules** | Runtime priority: blindspot check → mental model → heuristic → hard evidence → DNA |
| **Part F — Sources** | Primary (this person's output), secondary (others), key quotes |

Execution: `Blindspot check → Mental model selects framework → Heuristics check for triggered rules → Hard evidence sets baseline → Expression DNA shapes voice`

### Supported Tags

**Personality**: Responsible · Blame-shifter · Perfectionist · Good-enough · Procrastinator · PUA master · Office politician · Managing-up expert · Passive-aggressive · Flip-flopper · Quiet · Read-no-reply …

**Corporate culture**: ByteDance-style · Alibaba-style · Tencent-style · Huawei-style · Baidu-style · Meituan-style · First-principles · OKR-obsessed · Big-corp-pipeline · Startup-mode

**Levels**: ByteDance 2-1~3-3+ · Alibaba P5~P11 · Tencent T1~T4 · Baidu T5~T9 · Meituan P4~P8 · Huawei 13~21 · NetEase · JD · Xiaomi …

### Evolution

- **Append files** → auto-analyze delta → merge into relevant sections, never overwrite existing conclusions
- **Conversation correction** → say "he wouldn't do that, he should be xxx" → writes to Correction layer, takes effect immediately
- **Version control** → auto-archive on every update, rollback to any previous version

---

## Project Structure

This project follows the [AgentSkills](https://agentskills.io) open standard. The repo now hosts **two independent skills** under `skills/` so each can be installed separately into `.claude/skills/`:

```
colleague-skill/                           # repo root (this is NOT a skill dir)
├── README.md, LICENSE, INSTALL.md, ROADMAP.md, CONTRIBUTING.md
├── docs/, requirements.txt, colleague_skill.pdf
├── tools/                                 # canonical shared Python tools (mirrored into each skill)
│
├── skills/
│   ├── create-colleague/                  # ← install this dir as .claude/skills/create-colleague
│   │   ├── SKILL.md                       # entry point (frontmatter: name=create-colleague)
│   │   ├── prompts/
│   │   │   ├── intake.md
│   │   │   ├── work_analyzer.md
│   │   │   ├── work_builder.md
│   │   │   ├── persona_analyzer.md
│   │   │   ├── persona_builder.md
│   │   │   ├── merger.md
│   │   │   └── correction_handler.md
│   │   └── tools/                         # copy of repo-root tools/
│   │
│   └── create-politician/                 # ← install as .claude/skills/create-politician
│       ├── SKILL.md                       # entry point (frontmatter: name=create-politician, v2)
│       ├── prompts/
│       │   ├── politician_intake.md
│       │   ├── political_analyzer.md
│       │   ├── political_builder.md
│       │   ├── politician_persona_analyzer.md
│       │   ├── politician_persona_builder.md
│       │   ├── politician_merger.md
│       │   └── politician_correction_handler.md
│       ├── references/                    # v2 methodology
│       │   ├── extraction-framework.md    #   Three-fold validation, Expression DNA, six-track research
│       │   └── skill-template.md          #   Output SKILL.md template (PART A-F)
│       ├── examples/                      # pre-distilled gallery
│       └── tools/                         # copy of repo-root tools/
│
├── colleagues/                            # Generated colleague Skills (gitignored, output dir)
└── politicians/                           # Generated politician Skills (gitignored, output dir)
```

**Why duplicate `tools/` into each skill?** Each `${CLAUDE_SKILL_DIR}` resolves to the skill's own folder, so scripts referenced as `${CLAUDE_SKILL_DIR}/tools/*.py` must live inside the skill. The repo-root `tools/` is the canonical source; `skills/*/tools/` are mirrors kept in sync via `cp -r tools skills/{name}/tools`.

---

## Notes

- **Source material quality = Skill quality**: chat logs + long docs > manual description only
- Prioritize collecting: long-form writing **by them** > **decision-making replies** > casual messages
- Feishu auto-collection requires adding the App bot to relevant group chats
- This is still a demo version — please file issues if you find bugs!

---
### 📄 Technical Report

> **[Colleague.Skill: Automated AI Skill Generation via Expert Knowledge Distillation](colleague_skill.pdf)**
>
> We wrote a paper detailing the system design of colleague.skill — the two-part architecture (Work Skill + Persona), multi-source data collection, Skill generation & evolution mechanisms, and evaluation results in real-world scenarios. Check it out if you're interested!

---

## Star History

<a href="https://www.star-history.com/?repos=titanwings%2Fcolleague-skill&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/image?repos=titanwings/colleague-skill&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/image?repos=titanwings/colleague-skill&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/image?repos=titanwings/colleague-skill&type=date&legend=top-left" />
 </picture>
</a>

---

<div align="center">

MIT License © [titanwings](https://github.com/titanwings)

</div>
