# 35 AI agents that draft everything and send nothing

I set up a personal office of 35 AI agents for my job search, LinkedIn, MBA thesis and life admin. They can read my Gmail, Calendar, Drive, Slack and Notion and prepare the work. Nothing goes out, gets booked or gets changed until I approve it.

The office software is [Agents Office](https://github.com/ajsahni/agents-office) by Sahni.ai. **This repository contains only my configuration and the approval layer I built around it.** It does not include or modify Agents Office, and it is not affiliated with Sahni.ai. See [NOTICE.md](NOTICE.md).

**Stack:** Agents Office, Claude Code, MCP connectors (Gmail, Google Calendar, Google Drive, Slack, Notion), Claude Code permission rules, a Windows launcher.

![Agents Office running my configuration](docs/office.png)

*Agents Office (by Sahni.ai) running my configuration: six departments, 35 renamed desks, and my notes as "the Brain" in the middle. The numbers on the department cards are the office's built-in demo animation, not my results.*

## Why I built it

Much of my week as an MBA student applying for roles in Germany goes into work that follows a pattern: tailoring applications, answering recruiters, preparing interviews, drafting LinkedIn posts, tracking deadlines and reading for my thesis. I wanted to answer two questions:

1. Which of these tasks can an AI agent draft well enough that I only review and send?
2. How do you give agents access to real accounts without risking a message going out that nobody approved?

The second question decides whether agents get used at all. A company faces the same question before it lets agents near customer data.

## What I set up

I mapped my workstreams onto the office's six fixed departments and rewrote every desk:

| My workstream | Office department | Example agents |
|---|---|---|
| Inbox | Emails | Inbox Lead, Recruiter Replies, German Writer |
| Job search | Sales | Career Lead, Job Scout, Applications, Interview Prep |
| LinkedIn | Marketing | LinkedIn Lead, Research, Post Writer, Comments |
| Planning and research | Operations | Planning Lead, Reg Watch, Contract Check, Skills Coach |
| Money | Accounting | Money Lead, Bills & Subs, Tax & Documents |
| Thesis and coursework | Delivery | Thesis Lead, Literature, Interview Desk, Academic QA |

All 35 agents are listed in [docs/agents.md](docs/agents.md). Each agent gets its instructions from three places:

- **Role** (`brain/Agents Office/agents.json`): a job description per desk, plus standing instructions for 11 of them.
- **Skills** (`brain/Agents Office/skills/`): how a type of work is done. There are seven, covering approvals, house style, email replies, applications, LinkedIn posts, the thesis and the research proposal. Each is bound to a department or a single agent.
- **Notes** (`brain/`): Markdown notes the agents read for context, organised as one map per department. My real notes stay private; the ones here are generic versions so the structure is visible.

## The approval layer

Out of the box, an agent given a task can use every tool of every connected app: post to Slack, create calendar invitations, edit Notion, run automations. The office's own "needs my OK" step only covers scheduled routines. So I added three locks:

1. **App allow-list.** The agents never receive n8n, Composio, my browser or two connectors I have not set up.
2. **45 tool-level deny rules** ([agent-permissions.json](setup/agent-permissions.json)). Claude Code itself blocks every tool that sends, schedules, posts, publishes, edits or deletes. Reading and drafting stay open. The launcher installs the rules before every start and refuses to start the office if that fails.
3. **A standing instruction to every agent** ([approvals skill](brain/Agents%20Office/skills/approvals/SKILL.md)). Draft instead of sending, never commit me to anything, and end with "DECISION NEEDED" when a choice is mine.

Approval happens where I already work. Emails wait in Gmail Drafts and Slack messages in Drafts & Sent, and nothing leaves until I press Send. Full details, the per-app table and the known limits are in [docs/approval-layer.md](docs/approval-layer.md).

## Decisions and trade-offs

- **Skills over notes for anything that must not be wrong.** The office's source shows that for each task an agent sees only about four of my notes, picked by keyword and cut at roughly 1,800 characters. So the facts an agent must never get wrong, like my CV facts and the approval rule, live in skills, which load in full every time.
- **Drafts instead of a new approval queue.** I could have built an approval inbox. Routing every outbound action into the drafts folder of the app it belongs to covers every task and asks me to learn nothing new.
- **Enforced rules over instructions.** An instruction is a request to the model. The deny rules are enforced by the runtime, so they hold even if an agent misreads a task.
- **Fail closed.** If the rules cannot be installed, the office does not start.
- **Generic in public, specific in private.** The skills here are templates. My real CV facts, thesis brief and LinkedIn strategy stay on my machine.

## How it was checked

- **Agents Office's own test suite**, run with my private configuration and with this public version: 34 of 35 checks pass in both. The one failure is a browser version mismatch in the test environment.
- **My real connector list**, replayed through the office's allow-list code: agents receive Gmail, Calendar, Drive, Slack, Notion, Claude Docs and web search, nothing else.
- **All 45 rules**, matched against each connector's tool list: read and draft tools pass, while send, edit and invite tools are blocked. For example Gmail `create_draft` passes and Calendar `respond_to_event` is blocked.
- **Still to do:** a live test where an agent is asked to "send" an email and a Slack message, and both must end as drafts.

## What I will measure next

I have not measured results yet. Over the next four weeks I will track:

- time per application, before and after
- share of drafts I send without edits
- how many of my corrections (`revise: ...`) turn into standing rules for an agent

## What the build taught me about rolling out agents

- **Permissions are the real work.** The apps connected themselves the moment I logged in to Claude Code. Deciding what each agent may touch, and enforcing it, took most of the build.
- **Take an inventory first.** My Claude account had ten connected apps, two of them not even signed in. You cannot set permissions for tools you have not listed.
- **Approval belongs where people already work.** A drafts folder gets checked. A separate approval dashboard would be one more tab to forget.

## How I built it

I set the goal and the rules: personal use, and nothing goes out without my approval. I chose the workstreams and ran the setup on my own Windows PC. I worked with Claude (Anthropic) as a build partner for the implementation. It read the Agents Office source, wrote the launcher and permission rules, and debugged the Windows install with me.

## What is in this repository

```
README.md                      this page
docs/agents.md                 all 35 agents, their jobs and standing instructions
docs/approval-layer.md         the three locks, per-app table, checks and known limits
setup/start-agents-office.cmd  Windows launcher: installs the rules, then starts the office
setup/install-agents-office.cmd  one-time install of Agents Office next to this folder
setup/agent-permissions.json   the 45 deny rules
setup/office.config.local.json office settings, including the app allow-list
brain/Agents Office/agents.json the 35 roles
brain/Agents Office/skills/    the 7 skills (generic versions)
brain/                         generic notes: one map per department, voice, index
```

## Run it yourself

1. Install Claude Code and log in. Then copy `setup/` so it sits next to where Agents Office should live, and run `install-agents-office.cmd`. It downloads [Agents Office](https://github.com/ajsahni/agents-office) from its official repository.
2. Use `brain/` as your notes folder (or copy `brain/Agents Office/` into your own), point `brain` in `office.config.local.json` at it, and fill in the example files (`cv-facts.md`, `brief.md`, `strategy.md`, `profile.md`).
3. Start the office with `start-agents-office.cmd`.

Agents Office is free for personal and internal use under its own licence. Read it before you use it for anything else.

## Credits

Agents Office is a Sahni.ai product by AJ Sahni, licensed under PolyForm Noncommercial 1.0.0 with Sahni.ai's additional terms. Claude and Claude Code are made by Anthropic. Details in [NOTICE.md](NOTICE.md).
