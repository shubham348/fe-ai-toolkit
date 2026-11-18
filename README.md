# FE AI Toolkit — One Agent to Set Up Your AI-Powered Frontend Workflow

A shareable VS Code Copilot setup for frontend engineers. Run **one master agent** and it configures the MCP servers, VS Code extensions, and specialized agents you need for daily frontend work: testing, PR/MR creation, read-only code review, accessibility (WCAG), SEO, performance and Core Web Vitals, and more.

> **Status:** Documentation draft. The master agent (`fe-toolkit-setup`) implements exactly what is listed here.

---

## 1. Why this exists

| Problem | What the toolkit does |
|---|---|
| Developers don't know which MCP servers, agents, and extensions to use | Gives one curated, opinionated list for frontend work |
| Setup is manual and differs per machine | One agent detects your stack and sets everything up |
| AI agents can change things you didn't want changed | Read-only agents are restricted by their tool list, not just by instructions |
| Secrets end up in config files | Tokens are never written to files; they use input prompts or environment variables |
| Generic agents don't know your stack | Agents adapt to the framework, test runner, and Git host they detect |

---

## 2. Prerequisites

| Requirement | Notes |
|---|---|
| VS Code (latest stable) | Agent mode, custom agents, and MCP support enabled |
| GitHub Copilot + Copilot Chat | Agent mode must be allowed by your org policy |
| Node.js 18+ | Needed for `npx`-based MCP servers |
| Git | Needed for the PR/MR agents |
| Chrome / Chromium | Used by Chrome DevTools MCP and Playwright |
| Accounts (optional) | GitHub or GitLab, Figma, Snyk, Jira/Linear, Sentry — only for the servers you enable |

---

## 3. What the master agent does

**Agent:** `fe-toolkit-setup`

| Step | Action | Writes anything? |
|---|---|---|
| 1. Detect | Reads `package.json` and the repo to detect the framework, test runner, linter, styling, Git host, and package manager | No |
| 2. Ask | Asks which **packs** (Core / Extras / Integrations) and which **scope** (Workspace or User) you want | No |
| 3. Plan | Shows the exact list of MCP servers, extensions, and agents it will add, and skips anything already present | No |
| 4. Confirm | Waits for your explicit "yes" | No |
| 5. Apply | Installs extensions, merges the MCP config, and copies the agent files | **Yes** |
| 6. Report | Prints a summary table: what was added or skipped, what each agent does, and how to use it | No |

### Scope options

| Scope | MCP config location | Agents location | Best for |
|---|---|---|---|
| **Workspace** | `.vscode/mcp.json` | `.github/agents/` | Sharing with your team through the repo |
| **User** (default) | User profile `mcp.json` (Command Palette → *MCP: Open User Configuration*) | User prompts folder (Command Palette → *Chat: Configure Custom Agents*) | Using the same agents in every project |

> **WSL / remote windows:** user agents are read from the **local** VS Code profile (on WSL, the Windows `%APPDATA%\Code\User\prompts`), while MCP servers and the `npx` downloads run on the remote machine.

> **`~/.copilot/agents/`:** Copilot CLI / SDK chats read user agents from `~/.copilot/agents/` on the machine where they run (on WSL, inside Linux). Newer VS Code versions can also move user agents from the prompts folder into `%USERPROFILE%\.copilot\agents` on their own. If your agents vanish from the prompts folder but still show in Chat, look there first. The setup agent installs to every folder that exists and never removes agents from any of them.

> **Snyk:** if the Snyk extension is set to auto-configure its MCP server, it adds a server named `Snyk` itself. The setup agent skips adding another one and points `fe-security` and `dep-upgrader` at `Snyk/*`.

### Safety rules the master agent follows

| Rule | Detail |
|---|---|
| Plan before write | Nothing is written until you confirm |
| Safe to re-run | Existing servers, agents, and extensions are detected and skipped |
| No overwrites | If a file with the same name exists, the agent asks before replacing it |
| Copy, never move or delete | Existing agent, prompt, and `mcp.json` files are never moved, renamed, or deleted, even if they look like duplicates |
| No secrets in files | Uses `${input:...}` prompts or environment variables for tokens |
| No automatic Git publishing | Agents leave changes for you to review; commit or push only after your explicit request and confirmation |
| Official sources only | Every MCP server comes from the vendor's official package or endpoint |
| Read-only by design | Review and audit agents get no write or comment tools |

---

## 4. MCP servers added

### Core pack (always proposed)

| MCP server | Publisher | Type | Used for | Used by agents |
|---|---|---|---|---|
| **Playwright** (`@playwright/mcp`) | Microsoft | stdio (`npx`) | Browser automation, accessibility snapshots, E2E flows, device emulation | e2e-tester, a11y-auditor, responsive-qa, visual-diff, bug-hunter |
| **Chrome DevTools** (`chrome-devtools-mcp`) | Google Chrome team | stdio (`npx`) | Lighthouse audits, performance traces (LCP, INP, CLS), network and console inspection | perf-auditor, seo-auditor, a11y-auditor, bug-hunter, ssr-checker |
| **Context7** (`@upstash/context7-mcp`) | Upstash | stdio or remote | Up-to-date library docs (React, Vue, Next.js, Playwright, and others), which reduces made-up APIs | All coding agents |
| **GitHub** *(if GitHub is detected)* | GitHub | remote HTTP | Read and create PRs, diffs, commits | pr-creator, pr-reviewer |
| **GitLab** *(if GitLab is detected)* | GitLab | remote HTTP | Read and create MRs, diffs, commits | pr-creator, pr-reviewer |

### Extras pack (optional)

| MCP server | Publisher | Type | Used for | Used by agents |
|---|---|---|---|---|
| **Snyk** (`snyk mcp`) | Snyk | stdio (Snyk CLI) | Code and dependency security scans | fe-security, dep-upgrader |
| **Figma** | Figma | remote HTTP | Read design context, variables, and screenshots | figma-to-component, design-system-guard |

### Integrations pack (optional, team-specific)

| MCP server | Publisher | Used for | Used by agents |
|---|---|---|---|
| **Atlassian (Jira/Confluence)** | Atlassian | Ticket acceptance criteria and specs | pr-creator, pr-reviewer, bug-hunter |
| **Linear** | Linear | Issue context | pr-creator, bug-hunter |
| **Sentry** | Sentry | Production errors and stack traces | bug-hunter |
| **New Relic** | New Relic | Real-user performance data | perf-auditor |

> Before publishing, check each server's install command and URL against the vendor's official docs. Endpoints change.

---

## 5. VS Code extensions added

| Extension | ID | Pack | Why |
|---|---|---|---|
| GitHub Copilot Chat | `GitHub.copilot-chat` | Core (required) | Runs agents and MCP |
| ESLint | `dbaeumer.vscode-eslint` | Core | Linting feedback while you edit |
| Stylelint | `stylelint.vscode-stylelint` | Core | CSS/SCSS linting |
| Prettier | `esbenp.prettier-vscode` | Core | Formatting |
| Playwright Test | `ms-playwright.playwright` | Core | Run and debug E2E tests |
| Jest *(if Jest is detected)* | `Orta.vscode-jest` | Core | Run and debug unit tests |
| Vitest *(if Vitest is detected)* | `vitest.explorer` | Core | Run and debug unit tests |
| axe Accessibility Linter | `deque-systems.vscode-axe-linter` | Core | WCAG issues flagged in JSX/HTML as you type |
| Error Lens | `usernamehw.errorlens` | Core | Inline errors |
| GitHub Pull Requests *(if GitHub)* | `GitHub.vscode-pull-request-github` | Core | Browse PRs in the editor |
| GitLab Workflow *(if GitLab)* | `GitLab.gitlab-workflow` | Core | Browse MRs in the editor |
| Snyk Security | `snyk-security.snyk-vulnerability-scanner` | Extras | Security scanning |
| Figma for VS Code | `figma.figma-vscode-extension` | Extras | Inspect designs in the editor |

---

## 6. Agents added

### Legend

| Mode | Meaning |
|---|---|
| **Read-only** | Can read code and call read-only tools. Cannot edit files or post anything. |
| **Edit** | Can edit files and run commands in your local workspace |
| **Remote write** | Can create things on GitHub/GitLab (for example, a PR). Always asks first. |

**How to run an agent:** pick it from the agent dropdown in Chat, then type the example prompt. Custom agents are not invoked with `@`.

**Tool scope:** each agent can only use the servers in its own tool list. For example, `a11y-auditor` can't read Jira. This is intentional. Use the default Agent for anything else.

### Core pack

| Agent | Mode | What it does | MCP / tools | Example prompt |
|---|---|---|---|---|
| **test-writer** | Edit | Writes unit and component tests (Jest/Vitest + Testing Library) for a file, following existing test patterns, then runs them until they pass | Files, terminal, Context7 | `add tests for src/components/Header.jsx` |
| **e2e-tester** | Edit | Explores a page in a real browser, writes Playwright E2E tests with stable selectors, and fixes flaky ones | Playwright, files, terminal | `write a checkout happy-path test for /cart` |
| **pr-creator** | Remote write | Checks the branch and warns about risky scope (multi-ticket branches, config deletions), asks for the PR/MR template values and never invents them, keeps your title exactly, shows a preview, and creates the PR/MR **only after you confirm**. Runs lint and tests only if you ask | GitHub/GitLab, terminal, Jira (optional) | `create an MR for this branch` |
| **pr-reviewer** | **Read-only** | Reviews someone else's PR/MR and gives you a list of suggested comments (file, line, severity, suggested text). **It never posts comments, approves, or changes the PR/MR.** | GitHub/GitLab (read tools only), files | `review MR !1234` |
| **a11y-auditor** | Read-only → Edit (on request) | Audits a page or component against **WCAG 2.2 AA**: axe scan, keyboard navigation, focus order, contrast, ARIA, accessibility tree. Reports issues with the WCAG criterion, then fixes them if you ask | Playwright, Chrome DevTools (Lighthouse), files | `audit http://localhost:3000/search` |
| **seo-auditor** | Read-only → Edit (on request) | Checks title and meta tags, canonical, robots, Open Graph, JSON-LD structured data, heading structure, server-rendered content, sitemap, and internal links | Chrome DevTools (Lighthouse), files | `check the event detail page` |
| **perf-auditor** | Read-only → Edit (on request) | Measures **Core Web Vitals** (LCP, INP, CLS) and Lighthouse performance, finds the root cause (images, fonts, render-blocking resources, JS size, hydration, third parties), and suggests or applies fixes. Lighthouse measures page load only, so INP comes from a trace recorded while interacting with the page | Chrome DevTools (trace + Lighthouse), files, New Relic (optional) | `why is LCP slow on the home page (mobile)?` |

### Extras pack

| Agent | Mode | What it does | MCP / tools | Example prompt |
|---|---|---|---|---|
| **fe-security** | Read-only → Edit | Finds XSS sinks (`dangerouslySetInnerHTML`, `innerHTML`, `eval`), unsafe URLs, missing CSP, secrets in the client bundle, and vulnerable dependencies | Snyk, files, terminal | `scan src/ for XSS risks` |
| **bundle-analyzer** | Read-only | Reports large modules, duplicate packages, and missing code splitting, and suggests lazy-loading candidates | Terminal, files | `what makes the main bundle large?` |
| **ssr-checker** | Read-only → Edit | Finds hydration mismatches and code that touches `window`/`document` during server rendering, and compares server HTML with client HTML | Chrome DevTools, files | `check the product page for hydration errors` |
| **responsive-qa** | Read-only | Checks layouts at mobile, tablet, and desktop sizes and on Chromium/WebKit/Firefox; reports overflow, overlap, and small tap targets | Playwright | `check /search on all breakpoints` |
| **visual-diff** | Read-only | Takes screenshots before and after a change and reports visual differences | Playwright | `compare staging and local for /home` |
| **design-system-guard** | Read-only | Flags hard-coded colors, spacing, and fonts, and custom components that duplicate design system ones | Files, Figma (optional) | `review my changed files` |
| **figma-to-component** | Edit | Builds a component from a Figma frame, reusing your design system components and tokens, and adds a test | Figma, files, Context7 | `build <figma-url>` |
| **dep-upgrader** | Edit | Reads changelogs, upgrades a dependency, applies breaking-change fixes, and runs tests | Terminal, files, Context7, Snyk | `upgrade react-router to latest` |
| **bug-hunter** | Read-only → Edit | Reproduces a bug in the browser, reads console and network, finds the root cause, and proposes a fix with a regression test | Playwright, Chrome DevTools, Sentry (optional) | `the filter dropdown closes on mobile tap` |
| **refactor-planner** | Read-only → hands off | Produces a step-by-step refactor plan with risks, then hands off to an edit agent | Files | `plan splitting SearchPage.jsx` |
| **story-writer** | Edit | Writes Storybook stories and component docs covering states and variants | Files | `add stories for Button` |

---

## 7. Recommended dev packages (optional)

The master agent **suggests** these and only installs them if you approve.

| Package | Why |
|---|---|
| `@axe-core/playwright` | Automated WCAG checks inside your E2E tests |
| `@lhci/cli` | Lighthouse CI budgets for performance, SEO, and accessibility in your pipeline |
| `web-vitals` | Collect real-user LCP, INP, and CLS in production |
| A bundle analyzer for your bundler (`webpack-bundle-analyzer`, `rollup-plugin-visualizer`, and so on) | Used by bundle-analyzer |

---

## 8. Summary output (what you see after setup)

When setup finishes, the master agent prints a report like this:

| Category | Item | Status |
|---|---|---|
| MCP | Playwright | Added |
| MCP | Chrome DevTools | Added |
| MCP | Context7 | Added |
| MCP | GitLab | Already present, skipped |
| Extension | ESLint | Already installed |
| Extension | axe Accessibility Linter | Installed |
| Agent | pr-reviewer | Added (User scope) |
| Agent | perf-auditor | Added (User scope) |

It then prints the **Agents added** table (name, what it does, example prompt) and any **next steps**, such as "Sign in to GitLab when prompted" or "Run *MCP: List Servers* to start the servers".

---

## 9. How to use

| Step | Action |
|---|---|
| 1 | Copy `fe-toolkit-setup.agent.md` into your user prompts folder (or the repo's `.github/agents/`) |
| 2 | Open Copilot Chat, switch to **Agent** mode, and pick **fe-toolkit-setup** from the agent dropdown |
| 3 | Type `set up my frontend toolkit` |
| 4 | Answer the pack and scope questions, review the plan, and confirm |
| 5 | Pick an agent from the agent dropdown, then type your request. For example, pick **pr-reviewer** and type `review MR !1234` |

---

## 10. Troubleshooting

| Symptom | Fix |
|---|---|
| Agent not in the dropdown | Check the file location and that the frontmatter YAML is valid; reload the window. On WSL or remote, agents must be in the **local** profile's prompts folder. Also check `~/.copilot/agents/` (Copilot CLI / SDK, and `%USERPROFILE%\.copilot\agents` on Windows, where VS Code may move agents) |
| MCP server not starting | Command Palette → *MCP: List Servers* → *Show Output*; check Node version and network/proxy |
| MCP tools missing | Your org policy may block MCP. Ask your admin |
| Auth prompts keep appearing | Re-run the server and complete sign-in; never paste tokens into `mcp.json` |
| Reviewer agent tried to comment | It can't, because write tools aren't in its tool list. Report it as a bug if you see this |
| `npm error ETIMEDOUT` when a server starts | You're behind a company network. Add `"--registry", "<company npm mirror>"` before the package name in the server's `npx` args |
| Server stuck on "Waiting for server to respond to `initialize`" | Usually the same network problem. Open *Show Output* for the real error |
| Agent says a tool "is disabled" | The agent's `tools:` names don't match your server name. Open the agent file, click **Configure Tools**, and tick the server |
| `code --install-extension` fails on WSL | Install the extensions from the Extensions view instead |

---

## 11. Principles
1. **Least privilege:** each agent gets only the tools it needs.
2. **Human in the loop:** plan first, confirm, then apply.
3. **Enforce in config, guide in prompts:** tool lists enforce limits; instructions only guide behavior.
4. **Detect, don't assume:** adapt to the user's stack.
5. **Official sources and no secrets in files.**
6. **Safe to re-run:** running setup twice changes nothing.
