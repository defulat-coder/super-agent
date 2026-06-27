# Agno OS Console Design

## Goal

Build an independently deployable Agent OS console in this repository that moves toward feature parity with Agno OS, starting with a production-quality demo console that can be verified page by page against `os.agno.com`.

## Evidence From Agno OS

Chrome CDP analysis of `https://os.agno.com` and `https://os.agno.com/try-demo` found three product layers:

- Cloud control API: `https://os-api.agno.com/api/v1`.
- AgentOS runtime API: the demo points to `https://demo-os-production-823a.up.railway.app`; local installs are probed at `http://localhost:7777/health`.
- Browser console: a React/Vite single-page app with a persistent left navigation and route-level data pages.

The cloud API handles authentication, users, organizations, OS list, billing, and demo token creation. The runtime API exposes product data such as `/health`, `/config`, `/sessions`, and table-backed pages.

Observed top-level routes:

- Home
- Chat
- Sessions
- Traces
- Memory
- Knowledge
- Metrics
- Evaluation
- Approvals
- Scheduler

Observed demo runtime behavior:

- `POST /api/v1/auth/demo-sdk-token` creates the demo runtime token.
- `GET /health` confirms runtime availability.
- `GET /config` returns OS manifest, databases, agents, teams, workflows, knowledge instances, and quick prompts.
- `GET /sessions?page=1&limit=25&sort_by=updated_at&sort_order=desc&db_id=demo-os-db&table=agno_sessions` returns paginated session rows.
- Metrics, Approvals, and Scheduler show real layouts but are partly gated in Demo OS with blur overlays such as `Not available for Demo OS` or `Admin access required`.

## Product Scope

This spec covers the first implementation stage: a self-contained Demo OS console that runs from the current monorepo without depending on Agno's cloud or demo runtime.

It must include:

- A persistent Agent OS application shell.
- A demo OS banner and active OS header.
- Home with Agents, Teams, Workflows, Interfaces, and All OSes sections.
- Chat, Sessions, Memory, Knowledge, Metrics, Evaluation, Approvals, and Scheduler pages.
- Local data that mirrors the shape of Agno's `/config`, `/sessions`, and related table pages.
- Screenshot-driven comparison against Agno Demo OS and local `localhost:3000`.

It does not include in stage one:

- Real authentication.
- Mutating approvals or schedules.
- Real LLM streaming.
- New database tables.
- Full Agno API proxying.
- Third-party UI dependencies beyond what already exists in this repo.

## Architecture

Keep the first stage frontend-heavy. The existing `apps/web` Next.js app owns route rendering and local mock data. Shared domain types live in `packages/shared` only when they are useful across frontend and API boundaries. The API stays unchanged unless a page needs a local HTTP contract for deployment parity.

The implementation should avoid a broad framework rewrite:

- Use App Router routes under `apps/web/app`.
- Use colocated feature components under `apps/web/src/components/agent-os`.
- Use local data helpers under `apps/web/src/lib/agent-os`.
- Use current Tailwind CSS and `@project-template/ui`.
- Do not add charting, icon, state-management, or table dependencies for stage one.

## Visual Direction

Visual thesis: quiet operational console with a pale surface, narrow gray dividers, compact mono labels, orange-red AgentOS accents, and dense but readable data areas.

Content plan:

- Shell: orient the user with current OS status and navigation.
- Home: show capability inventory from the OS manifest.
- Operational pages: expose data tables, filters, sort controls, overlays, and empty/loading states.
- Chat: provide a credible agent workspace without pretending to stream real model output.

Interaction thesis:

- Navigation should preserve a stable shell and highlight the active page.
- Sections on Home should expand and collapse with minimal layout shift.
- Filter, sort, and tab controls should update local UI state and URL-shaped state where simple.

## Pages

### App Shell

The shell mirrors Agno OS:

- Thin left sidebar.
- Agno-style brand block.
- Navigation items for Home, Chat, Sessions, Traces, Studio, Learning, Memory, Knowledge, Metrics, Evaluation, Approvals, Scheduler, and Settings.
- Top demo banner warning that all demo data is public.
- OS header showing `Demo OS`, an active status dot, support and refresh controls.
- Footer utility links rendered as icon-like compact controls.

### Home

Home renders from an OS manifest:

- Agents: Builder, Researcher, Agno Expert, plus hidden items behind Show more.
- Teams: Dash, Mentor, Clinic, plus hidden items.
- Workflows: Classifier, Scribe, Code Scout, Troubleshooter, AI Digest.
- Interfaces: Slack, WhatsApp, Chat.
- All OSes: Production, Staging, Local Dev.

Each card shows:

- Name.
- Description.
- Labels.
- Chat and Config actions.

### Chat

Chat shows:

- Object selector for Agents / Teams / Workflows.
- Toolbar buttons: keyboard shortcut, See Config, Sessions, Fork Session, New Session.
- Disabled or mock composer with attachment/settings/send controls.
- Empty state or sample transcript depending on selected object.

Stage one should not send real messages.

### Sessions

Sessions shows:

- Database and table header.
- View filter.
- Export button.
- Select-all checkbox.
- Rows with session name and updated timestamp.
- Sort indicator for Updated at.

Use captured demo examples such as `What is Agno?` and `Summarize Agno in one concise sentence.`

### Memory, Knowledge, Evaluation

These pages share a table-page pattern:

- Database/table or knowledge instance selector.
- View or instance filter.
- Export button where appropriate.
- Select row controls.
- Updated at sorting.
- Empty, unavailable, and populated states as local data requires.

Knowledge should include `Clinic Records` and other instances from the demo manifest.

### Metrics

Metrics shows:

- Database and table header.
- Export action.
- Month picker controls.
- Blurred chart area with `Not available for Demo OS` overlay.
- Two to four simple CSS/SVG line chart placeholders, no chart dependency.

### Approvals

Approvals shows:

- Page title.
- View filter.
- Request rows with action name, arguments, agent/team, and date.
- Deny and Approve buttons rendered but disabled or non-mutating.
- `Admin access required` overlay matching Demo OS behavior.

### Scheduler

Scheduler shows:

- Page title.
- Schedule rows.
- Enable switches.
- Cron expression, endpoint, next run, and last status.
- `Not available for Demo OS` overlay matching Demo OS behavior.

## Data Model

The local data model should be small and explicit:

- `AgentOsManifest`
- `AgentOsEntity`
- `AgentOsDatabase`
- `AgentOsKnowledgeInstance`
- `AgentOsSession`
- `AgentOsApproval`
- `AgentOsSchedule`
- `AgentOsMetricSeries`

Use string literal unions for entity category and page status only where they prevent mistakes. Do not introduce generic repositories or factories.

## Error Handling

Stage one is static/local, so error handling is limited:

- Health display can keep existing API health behavior.
- If local page data is empty, render a clear empty state.
- Buttons that would mutate remote state should be visually present but non-mutating.
- External documentation links open to Agno docs where observed.

## Testing And Verification

Required checks after implementation:

- `pnpm --filter @project-template/web lint`
- `pnpm --filter @project-template/web typecheck`
- `pnpm --filter @project-template/web test`
- `pnpm --filter @project-template/web build`

Runtime verification:

- Start the app on `http://localhost:3000`.
- Use Chrome screenshots for Agno Demo OS and local pages.
- Compare Home, Chat, Sessions, Metrics, Approvals, and Scheduler first.
- Iterate until shell geometry, page hierarchy, controls, and gated overlays match the reference closely enough for functional parity.

## Open Decisions

None for stage one. Later stages can add API-backed data, Prisma persistence, worker integration, and real agent runtime calls after the console is visually and structurally aligned.
