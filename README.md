# CC Switch Skills

42 个本地已启用技能的公开副本，涵盖 Dify 支持、软件开发、测试、数据库、文档和设计。

导出日期：2026-10-09。保留原目录名称、`SKILL.md`、参考资料、脚本和资源；`agent-brower` 等已有目录名称也保留原样。

## 使用

1. 克隆仓库或通过 GitHub 的 **Code → Download ZIP** 下载。
2. 每个 `skills/<目录>/SKILL.md` 是技能入口。按所用客户端的技能安装方式导入需要的目录。
3. 在目标机器安装技能要求的 CLI、运行时和 MCP，并完成自己的登录与授权。复制技能文件不会复制这些依赖和权限。
4. Windows 用户需按实际环境调整 shell 命令和路径；这些技能来自 macOS 使用环境，尚未在 Windows 逐项运行验证。

这是技能文件集合，不包含 Claude 或其他服务的账号、模型额度、API 密钥。需要专用工具的 Dify 支持技能，必须连接自己有权使用的服务后才能执行调查。

## 公开版本的调整

个人姓名、内部服务地址、知识库与 Notion 标识、文档示例令牌已替换为示例或占位符；另为 `verification-loop` 补齐了技能元数据。详见 [PUBLIC_EXPORT.md](PUBLIC_EXPORT.md)。未启用的 `wechat-miniprogram` 不在本次导出中。

## 技能目录

| 技能 | 说明 |
| --- | --- |
| [agent-browser](skills/agent-brower/SKILL.md) | Automates browser interactions for web testing, form filling, screenshots, and data extraction. Use when the user needs to navigate websites, interact with web pages, fill forms, take screenshots, test web applications, or extract information from web pages. |
| [backend-patterns](skills/backend-patterns/SKILL.md) | Backend architecture patterns, API design, database optimization, and server-side best practices for Node.js, Express, NestJS, FastAPI, and Next.js API routes. |
| [browser-testing-with-devtools](skills/browser-testing-with-devtools/SKILL.md) | Tests in real browsers via Chrome DevTools MCP. Use when building or debugging anything that runs in a browser. Use when you need to inspect the DOM, capture console errors, analyze network requests, profile performance, or verify visual output with real runtime data. Requires the chrome-devtools MCP server to be configured. |
| [code-review-and-quality](skills/code-review-and-quality/SKILL.md) | Conducts multi-axis code review. Use before merging any change. Use when reviewing code written by yourself, another agent, or a human. Use when you need to assess code quality across multiple dimensions before it enters the main branch. |
| [code-simplification](skills/code-simplification/SKILL.md) | Simplifies code for clarity. Use when refactoring code for clarity without changing behavior. Use when code works but is harder to read, maintain, or extend than it should be. Use when reviewing code that has accumulated unnecessary complexity. |
| [codebase-design](skills/codebase-design/SKILL.md) | Shared vocabulary for designing deep modules. Use when the user wants to design or improve a module's interface, find deepening opportunities, decide where a seam goes, make code more testable or AI-navigable, or when another skill needs the deep-module vocabulary. |
| [continuous-learning-v2](skills/continuous-learning-v2/SKILL.md) | Instinct-based learning system that observes sessions via hooks, creates atomic instincts with confidence scoring, and evolves them into skills/commands/agents. |
| [d3-viz](skills/d3js/SKILL.md) | Creating interactive data visualisations using d3.js. This skill should be used when creating custom charts, graphs, network diagrams, geographic visualisations, or any complex SVG-based data visualisation that requires fine-grained control over visual elements, transitions, or interactions. Use this for bespoke visualisations beyond standard charting libraries, whether in React, Vue, Svelte, vanilla JavaScript, or any other environment. |
| [deepagents](skills/deepagents/SKILL.md) | LangChain Deep Agents framework for building autonomous coding agents. Use for agent harness, backends, subagents, human-in-the-loop, long-term memory, middleware, and CLI-based agent development. |
| [dify-cloud-browser-repro](skills/dify-cloud-browser-repro/SKILL.md) | Reproduce Dify Cloud issues quickly in the user's already signed-in Chrome session while preserving an evidence-grounded support workflow. Use for Dify SaaS support tickets or requests that require reproducing, verifying, or capturing browser-visible behavior in cloud.dify.ai Studio, Workflow, Chatflow, Agent, WebApp, plugin, dataset, RAG, file-upload, or multimodal paths. Reuse the current browser and tab, minimize snapshots and tool round-trips, prepare prerequisites with safe test apps or DSL when the UI setup itself is not under test, and defer targeted Console and Network analysis until the symptom has been triggered. |
| [dify-education-whitelist](skills/dify-education-whitelist/SKILL.md) | Use when judging whether a school, education email, email domain, university domain, education discount request, or 教育邮箱/教育域名/教育优惠白名单 request qualifies for Dify SaaS education whitelist handling. |
| [dify-enterprise-support](skills/dify-enterprise-support/SKILL.md) | 调查 Dify Enterprise/私有部署工单，按客户实际版本核验证据并生成未发送回复。 |
| [dify-support](skills/dify-support/SKILL.md) | Handle Dify SaaS or Enterprise support work with right-sized, evidence-grounded investigation and customer-safe drafts. Use for Dify-specific Zendesk tickets, reply reviews, read-only production DB, SigNoz, or Cloudflare edge checks, billing/refund, plugin, workflow, dataset, quota, WebApp/API, and developer or SRE handoff. Route explicitly requested or already-confirmed reusable SaaS knowledge to dify-saas-knowledge. Do not use for unrelated products or generic support tickets. |
| [domain-modeling](skills/domain-modeling/SKILL.md) | Build and sharpen a project's domain model. Use when discussing codebase terminology, writing or editing a GLOSSARY.md, or recording or editing an ADR. |
| [find-skills](skills/find-skills/SKILL.md) | Helps users discover and install agent skills when they ask questions like "how do I do X", "find a skill for X", "is there a skill that can...", or express interest in extending capabilities. This skill should be used when the user is looking for functionality that might exist as an installable skill. |
| [frontend-ui-engineering](skills/frontend-ui-engineering/SKILL.md) | Builds production-quality UIs. Use when building or modifying user-facing interfaces. Use when creating components, implementing layouts, managing state, or when the output needs to look and feel production-quality rather than AI-generated. |
| [grill-me](skills/grill-me/SKILL.md) | A relentless interview to sharpen a plan or design. |
| [grilling](skills/grilling/SKILL.md) | Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases. |
| [hatch-pet](skills/hatch-pet/SKILL.md) | Create, repair, validate, visually QA, and package Codex-compatible v2 animated pets from character art, generated images, company or prospect brand cues, or visual references. Use for any new Codex pet, custom mascot, non-pixel pet style, brand-inspired pet, existing-pet repair, or 8x11 spritesheet workflow requiring all 9 standard animation rows, 16 look directions, deterministic assembly, QA artifacts, and spriteVersionNumber 2 packaging. |
| [improve-codebase-architecture](skills/improve-codebase-architecture/SKILL.md) | Scan a codebase for deepening opportunities, present them as a visual HTML report, then grill through whichever one you pick. |
| [incremental-implementation](skills/incremental-implementation/SKILL.md) | Delivers changes incrementally. Use when implementing any feature or change that touches more than one file. Use when you're about to write a large amount of code at once, or when a task feels too big to land in one step. |
| [iterative-retrieval](skills/iterative-retrieval/SKILL.md) | Pattern for progressively refining context retrieval to solve the subagent context problem |
| [migrate-to-codex](skills/migrate-to-codex/SKILL.md) | Migrate supported instruction files, skills, agents, and MCP config into Codex project and global files. |
| [mysql-patterns](skills/mysql-patterns/SKILL.md) | MySQL database patterns for query optimization, schema design, indexing, and security. Quick reference for common patterns. |
| [pdf](skills/pdf/SKILL.md) | Use when tasks involve reading, creating, or reviewing PDF files where rendering and layout matter; prefer visual checks by rendering pages (Poppler) and use Python tools such as `reportlab`, `pdfplumber`, and `pypdf` for generation and extraction. |
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.md) | Breaks work into ordered tasks. Use when you have a spec or clear requirements and need to break work into implementable tasks. Use when a task feels too large to start, when you need to estimate scope, or when parallel work is possible. |
| [postgres-patterns](skills/postgres-patterns/SKILL.md) | PostgreSQL database patterns for query optimization, schema design, indexing, and security. Based on Supabase best practices. |
| [process-faq](skills/process-faq/SKILL.md) | Process and transform FAQ documents (xlsx, word, pdf, txt) into RAG-optimized format. Use when working with FAQ files, knowledge base documents, or when the user needs to analyze and restructure FAQ content for RAG systems. |
| [security-and-hardening](skills/security-and-hardening/SKILL.md) | Hardens code against vulnerabilities. Use when handling user input, authentication, data storage, or external integrations. Use when building any feature that accepts untrusted data, manages user sessions, or interacts with third-party services. |
| [security-review](skills/security-review/SKILL.md) | Use this skill when adding authentication, handling user input, working with secrets, creating API endpoints, or implementing payment/sensitive features. Provides comprehensive security checklist and patterns. |
| [sequelize-patterns](skills/sequelize-patterns/SKILL.md) | Sequelize Node.js ORM for SQL databases. Use for database models, migrations, associations, queries, transactions, validations, hooks, and working with PostgreSQL, MySQL, MariaDB, SQLite, SQL Server. |
| [source-command-e2e](skills/source-command-e2e/SKILL.md) | Generate and run end-to-end tests with Playwright. Creates test journeys, runs tests, captures screenshots/videos/traces, and uploads artifacts. |
| [source-command-plan](skills/source-command-plan/SKILL.md) | Restate requirements, assess risks, and create step-by-step implementation plan. WAIT for user CONFIRM before touching any code. |
| [source-command-setup-pm](skills/source-command-setup-pm/SKILL.md) | Configure your preferred package manager (npm/pnpm/yarn/bun) |
| [source-command-tdd](skills/source-command-tdd/SKILL.md) | Enforce test-driven development workflow. Scaffold interfaces, generate tests FIRST, then implement minimal code to pass. Ensure 80%+ coverage. |
| [spec-driven-development](skills/spec-driven-development/SKILL.md) | Creates specs before coding. Use when starting a new project, feature, or significant change and no specification exists yet. Use when requirements are unclear, ambiguous, or only exist as a vague idea. |
| [tdd-workflow](skills/tdd-workflow/SKILL.md) | Use this skill when writing new features, fixing bugs, or refactoring code. Enforces test-driven development with 80%+ coverage including unit, integration, and E2E tests. |
| [tdesign-miniprogram](skills/tdesign-miniprogram/SKILL.md) | TDesign Mini Program UI component library by Tencent. Use for building WeChat mini apps with TDesign components, design system, and best practices. |
| [test-driven-development](skills/test-driven-development/SKILL.md) | Drives development with tests. Use when implementing any logic, fixing any bug, or changing any behavior. Use when you need to prove that code works, when a bug report arrives, or when you're about to modify existing functionality. |
| [thesis-writer](skills/thesis-writer/SKILL.md) | Use when writing, editing, or formatting a Chinese undergraduate graduation thesis (.docx) by copying a reference template and replacing only text content. Covers the complete workflow from template analysis to content replacement, diagram generation, screenshot capture, and quality verification. |
| [ui-ux-pro-max](skills/ui-ux-pro-max/SKILL.md) | UI/UX design intelligence. 50 styles, 21 palettes, 50 font pairings, 20 charts, 9 stacks (React, Next.js, Vue, Svelte, SwiftUI, React Native, Flutter, Tailwind, shadcn/ui). Actions: plan, build, create, design, implement, review, fix, improve, optimize, enhance, refactor, check UI/UX code. Projects: website, landing page, dashboard, admin panel, e-commerce, SaaS, portfolio, blog, mobile app, .html, .tsx, .vue, .svelte. Elements: button, modal, navbar, sidebar, card, table, form, chart. Styles: glassmorphism, claymorphism, minimalism, brutalism, neumorphism, bento grid, dark mode, responsive, skeuomorphism, flat design. Topics: color palette, accessibility, animation, layout, typography, font pairing, spacing, hover, shadow, gradient. Integrations: shadcn/ui MCP for component search and examples. |
| [verification-loop](skills/verification-loop/SKILL.md) |  |

## 来源与许可

本仓库汇集本地自定义与第三方技能，不声明对全部内容拥有原创权，也不对整个集合重新授予统一许可证。已有许可证随对应技能保留；可核实的来源及许可证见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。没有记录许可证的条目不能据此推定可任意再分发或商用。

机器可读目录见 [manifest.json](manifest.json)。
