# claude-skills

Reusable skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code/skills). Drop any skill into `~/.claude/skills/` and it's available across all projects.

## Install

**All skills:**
```bash
git clone https://github.com/unattachedgray/claude-skills.git /tmp/cs
cp -r /tmp/cs/*/ ~/.claude/skills/
rm -rf /tmp/cs
```

**Single skill:**
```bash
SKILL=pretext-layout  # change to any skill name
mkdir -p ~/.claude/skills/$SKILL
curl -sL https://raw.githubusercontent.com/unattachedgray/claude-skills/main/$SKILL/SKILL.md \
  -o ~/.claude/skills/$SKILL/SKILL.md
```

## Skills (84)

### Development Lifecycle

| Skill | Description |
|-------|-------------|
| [dev](dev/) | 7-phase development process (think/plan/build/review/test/ship/reflect) |
| [build](build/) | PRD-driven autonomous build loop with quality gates |
| [eval](eval/) | Test skills against predefined eval cases |
| [hats](hats/) | Switch into a focused persona: architect, reviewer, debugger, etc. |
| [writing-plans](writing-plans/) | Plan multi-step tasks from specs before touching code |
| [brainstorming](brainstorming/) | Explore intent, requirements, and design before implementation |
| [simplify](simplify/) | Review changed code for reuse, quality, and efficiency |

### Frontend & UI

| Skill | Description |
|-------|-------------|
| [pretext-layout](pretext-layout/) | Text-aware frontend design via canvas measureText() — zero-reflow text measurement |
| [browser](browser/) | Chrome automation: tabs, screenshots, forms, extraction, accessibility |
| [react-development](react-development/) | React hooks, TypeScript, Server Components, state management, routing |
| [nextjs-development](nextjs-development/) | Next.js App Router, Server Components, data fetching, Vercel deployment |
| [senior-frontend](senior-frontend/) | Modern web apps with React, Next.js, TypeScript, Tailwind CSS |
| [frontend-guidelines](frontend-guidelines/) | Frontend patterns, architecture, and best practices |
| [tailwind-patterns](tailwind-patterns/) | Tailwind CSS v4: CSS-first config, container queries, design tokens |
| [3d-web-designer](3d-web-designer/) | Three.js, React Three Fiber, WebGPU for 3D web experiences |
| [web-design-guidelines](web-design-guidelines/) | UI review for Web Interface Guidelines compliance |
| [ui-design](ui-design/) | Design systems, component libraries, design tokens, responsive design |
| [mobile-design](mobile-design/) | Mobile-first design for iOS and Android |
| [theme-factory](theme-factory/) | Apply pre-set or custom themes to any artifact |

### Backend & Architecture

| Skill | Description |
|-------|-------------|
| [backend-development](backend-development/) | Node.js/Express/TypeScript microservices with layered architecture |
| [senior-architect](senior-architect/) | System design with React, Next.js, Node, Go, Python, Postgres, GraphQL |
| [senior-fullstack](senior-fullstack/) | Full-stack apps with React, Next.js, Node.js, GraphQL, PostgreSQL |
| [database-design](database-design/) | Schema architecture, SQL/NoSQL, normalization, indexing, migrations |
| [api-integration](api-integration/) | Third-party API integration, OAuth, webhooks, rate limiting |
| [typescript-expert](typescript-expert/) | Type-level programming, performance, monorepos, migration strategies |
| [javascript-mastery](javascript-mastery/) | 33+ essential JS concepts from fundamentals to advanced patterns |
| [bun-development](bun-development/) | Bun runtime: package management, bundling, testing, Node.js migration |

### DevOps & Infrastructure

| Skill | Description |
|-------|-------------|
| [senior-devops](senior-devops/) | CI/CD, infrastructure automation, containerization, cloud platforms |
| [docker-expert](docker-expert/) | Multi-stage builds, image optimization, container security, Compose |
| [vercel-deployment](vercel-deployment/) | Deploying to Vercel with Next.js |
| [github-workflow-automation](github-workflow-automation/) | PR reviews, issue triage, CI/CD, Git operations |
| [linux-production-shell-scripts](linux-production-shell-scripts/) | Production shell script templates for system automation |
| [bash-linux](bash-linux/) | Bash/Linux terminal patterns, commands, scripting |
| [powershell-windows](powershell-windows/) | PowerShell Windows patterns, operator syntax, error handling |

### AI & Machine Learning

| Skill | Description |
|-------|-------------|
| [senior-ml-engineer](senior-ml-engineer/) | ML productionization, MLOps, model deployment, feature stores |
| [senior-data-scientist](senior-data-scientist/) | Statistical modeling, experimentation, causal inference, analytics |
| [senior-data-engineer](senior-data-engineer/) | Data pipelines, ETL/ELT, Spark, Airflow, dbt, Kafka |
| [long-context](long-context/) | Extend transformer context windows with RoPE, YaRN, ALiBi |
| [senior-computer-vision](senior-computer-vision/) | Image/video processing, object detection, segmentation, vision AI |
| [segment-anything-model](segment-anything-model/) | SAM for zero-shot image segmentation |
| [blip-2-vision-language](blip-2-vision-language/) | Vision-language: captioning, VQA, image-text retrieval |
| [stable-diffusion-image-generation](stable-diffusion-image-generation/) | Text-to-image generation with Stable Diffusion |
| [whisper](whisper/) | Speech recognition: transcription, translation, 99 languages |
| [audiocraft-audio-generation](audiocraft-audio-generation/) | Text-to-music (MusicGen) and text-to-sound (AudioGen) |
| [langchain](langchain/) | LLM apps with agents, chains, RAG, 500+ integrations |

### AI Agents

| Skill | Description |
|-------|-------------|
| [agent-development](agent-development/) | Building autonomous AI agents with tool integration and memory |
| [advanced-agents](advanced-agents/) | Computer-use agents, parallel systems, vision-based control |
| [agent-manager-skill](agent-manager-skill/) | Manage local CLI agents via tmux, cron scheduling |
| [agent-memory](agent-memory/) | Persistent memory systems for AI agents |
| [agent-md-refactor](agent-md-refactor/) | Refactor bloated AGENTS.md/CLAUDE.md into organized docs |

### Prompt Engineering & Claude

| Skill | Description |
|-------|-------------|
| [prompt-engineer](prompt-engineer/) | Prompt structure, context management, output formatting |
| [senior-prompt-engineer](senior-prompt-engineer/) | Advanced prompt patterns, structured outputs, AI product dev |
| [claude-api](claude-api/) | Build apps with Claude API, Anthropic SDK, Agent SDK |
| [mcp-development](mcp-development/) | Build and integrate MCP servers for Claude Code |
| [gemini](gemini/) | Run Gemini CLI for code review and big context (>200k) processing |

### Security

| Skill | Description |
|-------|-------------|
| [security-audit](security-audit/) | Audit secrets, permissions, ports, auth, DB integrity |
| [security-review](security-review/) | Auth, input validation, secrets management, vulnerability detection |
| [code-quality](code-quality/) | Code review, refactoring, coding standards, best practices |

### Content & Media

| Skill | Description |
|-------|-------------|
| [pdf](pdf/) | PDF manipulation: extract, create, merge, split, fill forms |
| [meme-factory](meme-factory/) | Generate memes via memegen.link API |
| [notebook](notebook/) | NotebookLM-style document-grounded analysis with source citations |
| [notebooklm](notebooklm/) | Query Google NotebookLM notebooks from Claude Code |

### Business & Strategy

| Skill | Description |
|-------|-------------|
| [ceo-advisor](ceo-advisor/) | Executive leadership, strategy, financial modeling, board governance |
| [marketing-ideas](marketing-ideas/) | Marketing strategies and growth ideas for SaaS products |
| [app-store-optimization](app-store-optimization/) | ASO toolkit for Apple App Store and Google Play |
| [ux-research](ux-research/) | User interviews, usability testing, personas, journey mapping |
| [email-systems](email-systems/) | Transactional email, marketing automation, deliverability |

### Workflow Automation

| Skill | Description |
|-------|-------------|
| [n8n-workflows](n8n-workflows/) | Manage n8n automations via MCP tools |
| [zapier-workflows](zapier-workflows/) | Orchestrate Zapier automations via MCP tools and webhooks |
| [jira](jira/) | Jira issue management, sprint status, workflow automation |
| [monitor](monitor/) | Metric watchers with alerts: URLs, shell commands, response times |
| [loop](loop/) | Run commands on a recurring interval |
| [schedule](schedule/) | Scheduled remote agents on cron |

### Platform-Specific

| Skill | Description |
|-------|-------------|
| [wordpress-development](wordpress-development/) | Block themes, plugins, REST API, Site Editor, wp.org compliance |
| [game-development](game-development/) | Game development orchestrator, routes to platform-specific skills |

### Skill Management

| Skill | Description |
|-------|-------------|
| [skill-enhance](skill-enhance/) | Propagate a technology across your skill library |
| [skill-development](skill-development/) | Full skill lifecycle: search, creation, testing, improvement |
| [skill-factory](skill-factory/) | Create new skills by researching and composing existing ones |
| [find-skills](find-skills/) | Discover and install skills from the ecosystem |

### System (internal)

| Skill | Description |
|-------|-------------|
| [charlie-system](charlie-system/) | Core personality and session context |
| [context-flush](context-flush/) | Pre-compaction memory flush to SQLite |
| [morning-briefing](morning-briefing/) | Daily briefing: reviews tasks, conversations, plans the day |
| [daily-goals](daily-goals/) | Goal-driven task generation |
| [task-review](task-review/) | Pick highest-priority task and work it |

## How Skills Work

A skill is a markdown file at `~/.claude/skills/<name>/SKILL.md`. Claude Code loads skill metadata at startup and loads the full content when relevant.

**Triggering**: Type `/<skill-name>` in Claude Code, or Claude auto-loads the skill when your request matches its description.

**Writing skills**: See [Anthropic's skill authoring docs](https://docs.anthropic.com/en/docs/claude-code/skills). Use `/skill-enhance` to propagate new technologies across existing skills.

## Recent Changes

- Added Temporal (durable execution) guidance to agent-development, advanced-agents, senior-devops, senior-data-engineer, and backend-development skills
- Pretext-layout text verification integrated across frontend skills
- 84 skills total (up from 16)

## License

MIT
