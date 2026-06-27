# Agno OS Console Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the first independently deployable Agno OS-style console in `apps/web`, verified page by page against the captured Agno Demo OS behavior.

**Architecture:** Keep stage one frontend-heavy. `apps/web` renders the app shell, pages, components, and local data; `packages/shared` only receives cross-boundary domain types if needed. No new runtime dependency is required for this stage.

**Tech Stack:** Next.js App Router, React 19, Tailwind CSS 4, existing `@project-template/ui`, Vitest, TypeScript.

## Global Constraints

- User-visible copy defaults to Chinese; technical product labels such as AgentOS, Demo OS, Sessions, Metrics, Fastify, Prisma, Redis, BullMQ remain English.
- Dependencies must lock the major version if added; stage one should add no new dependency.
- Shared code goes in `packages/` only when it is actually reused across app boundaries.
- Do not hard-code business assumptions into the template beyond demo data needed for this Agno OS console.
- After changes, run the impacted package's `lint`, `typecheck`, and `test`; for this stage also run web `build`.
- Do not implement real authentication, real approval mutation, real schedule mutation, or real LLM streaming in stage one.

---

## File Structure

- Modify: `apps/web/app/page.tsx` - replace the template landing page with the Agent OS console entry.
- Modify: `apps/web/app/globals.css` - set app-level typography, background, and tiny utility refinements.
- Create: `apps/web/src/lib/agent-os/types.ts` - page data types used by the console.
- Create: `apps/web/src/lib/agent-os/data.ts` - local demo manifest, sessions, approvals, schedules, and metrics.
- Create: `apps/web/src/lib/agent-os/data.test.ts` - focused data-shape tests.
- Create: `apps/web/src/components/agent-os/shell.tsx` - persistent sidebar, banner, and OS header.
- Create: `apps/web/src/components/agent-os/cards.tsx` - Home entity cards and section layout.
- Create: `apps/web/src/components/agent-os/table-page.tsx` - shared table page primitives.
- Create: `apps/web/src/components/agent-os/pages.tsx` - page-specific content.
- Create: `apps/web/src/components/agent-os/chart.tsx` - tiny CSS/SVG line chart placeholders.

## Task 1: Local Data Model

**Files:**
- Create: `apps/web/src/lib/agent-os/types.ts`
- Create: `apps/web/src/lib/agent-os/data.ts`
- Create: `apps/web/src/lib/agent-os/data.test.ts`

**Interfaces:**
- Produces: `agentOsData`, `navItems`, `getPageBySlug(slug: string): AgentOsPage`
- Produces types: `AgentOsEntity`, `AgentOsSection`, `AgentOsSession`, `AgentOsApproval`, `AgentOsSchedule`, `AgentOsPage`

- [ ] **Step 1: Create `types.ts`**

```ts
export type AgentOsEntityKind = "agent" | "team" | "workflow" | "interface" | "os";

export type AgentOsEntity = {
  id: string;
  kind: AgentOsEntityKind;
  name: string;
  description: string;
  labels: string[];
  hidden?: boolean;
  path?: string;
  status?: "active" | "inactive";
};

export type AgentOsSection = {
  id: string;
  title: string;
  entities: AgentOsEntity[];
  defaultVisible: number;
};

export type AgentOsSession = {
  id: string;
  name: string;
  type: "agent" | "team" | "workflow";
  agentId: string;
  updatedAt: string;
  totalTokens: number;
};

export type AgentOsApproval = {
  id: string;
  action: string;
  detail: string;
  owner: string;
  date: string;
};

export type AgentOsSchedule = {
  id: string;
  name: string;
  cron: string;
  endpoint: string;
  nextRun: string;
  enabled: boolean;
};

export type AgentOsMetricSeries = {
  label: string;
  value: number;
  points: number[];
};

export type AgentOsPage = {
  slug: string;
  label: string;
  title: string;
  href: string;
};
```

- [ ] **Step 2: Create `data.ts`**

Use the observed Demo OS values:

```ts
import type {
  AgentOsApproval,
  AgentOsMetricSeries,
  AgentOsPage,
  AgentOsSchedule,
  AgentOsSection,
  AgentOsSession,
} from "./types";

export const navItems: AgentOsPage[] = [
  { slug: "home", label: "Home", title: "Home", href: "/" },
  { slug: "chat", label: "Chat", title: "Agents Chat", href: "/?page=chat" },
  { slug: "sessions", label: "Sessions", title: "Sessions", href: "/?page=sessions" },
  { slug: "traces", label: "Traces", title: "Traces", href: "/?page=traces" },
  { slug: "memory", label: "Memory", title: "Memory", href: "/?page=memory" },
  { slug: "knowledge", label: "Knowledge", title: "Knowledge", href: "/?page=knowledge" },
  { slug: "metrics", label: "Metrics", title: "Metrics", href: "/?page=metrics" },
  { slug: "evaluation", label: "Evaluation", title: "Evaluations", href: "/?page=evaluation" },
  { slug: "approvals", label: "Approvals", title: "Approvals", href: "/?page=approvals" },
  { slug: "scheduler", label: "Scheduler", title: "Scheduler", href: "/?page=scheduler" },
];

export const sections: AgentOsSection[] = [
  {
    id: "agents",
    title: "Agents",
    defaultVisible: 3,
    entities: [
      {
        id: "builder",
        kind: "agent",
        name: "Builder",
        description: "Describe your job and Builder composes a personalized assistant — interviewed, built behind approval, trial-run, and published live in the OS.",
        labels: ["studio-tools", "hitl", "agent-builder"],
      },
      {
        id: "researcher",
        kind: "agent",
        name: "Researcher",
        description: "Searches the web with Exa tools and generates polished, self-contained HTML reports.",
        labels: ["web-search", "file-generation"],
      },
      {
        id: "agno-expert",
        kind: "agent",
        name: "Agno Expert",
        description: "Answers Agno questions from the live documentation using MCP tools.",
        labels: ["mcp", "documentation"],
      },
      {
        id: "operator",
        kind: "agent",
        name: "Operator",
        description: "Turns an infra request into a risk-scored change plan with approval before execution.",
        labels: ["hitl", "approvals", "skills"],
        hidden: true,
      },
    ],
  },
  {
    id: "teams",
    title: "Teams",
    defaultVisible: 3,
    entities: [
      { id: "dash", kind: "team", name: "Dash", description: "Self-learning data analyst team that performs SQL queries and schema operations over your warehouse.", labels: ["coordinate", "sql", "data-analysis"] },
      { id: "mentor", kind: "team", name: "Mentor", description: "A coach that learns your profile, memories, and playbooks over time via the Learning Machine.", labels: ["learning-machine", "memory", "followups"] },
      { id: "clinic", kind: "team", name: "Clinic", description: "Reads the live clinic schedule via a context provider and patient-scoped records via filters.", labels: ["context-provider", "knowledge-filtering", "+1"] },
      { id: "newsroom", kind: "team", name: "Newsroom", description: "Research team that coordinates analysis and writing into a report.", labels: ["coordinate", "research"], hidden: true },
    ],
  },
  {
    id: "workflows",
    title: "Workflows",
    defaultVisible: 3,
    entities: [
      { id: "classifier", kind: "workflow", name: "Classifier", description: "Classifies any source and routes it to a specialist that reads it with real tools.", labels: ["router", "condition", "multi-source"] },
      { id: "scribe", kind: "workflow", name: "Scribe", description: "Content production pipeline with parallel research, a revision loop, and HITL.", labels: ["parallel", "loop", "hitl"] },
      { id: "code-scout", kind: "workflow", name: "Code Scout", description: "Analyzes a codebase, writes a script, and narrates it with TTS in a cross-modal chain.", labels: ["cross-modal", "coding-tools", "text-to-speech"] },
    ],
  },
];

export const sessions: AgentOsSession[] = [
  { id: "d47c4226", name: "What is Agno?", type: "agent", agentId: "sage", updatedAt: "19 Jun 2026, 07:34", totalTokens: 5987 },
  { id: "09c416a3", name: "What is Agno?", type: "agent", agentId: "sage", updatedAt: "19 Jun 2026, 07:27", totalTokens: 5019 },
  { id: "f86477d8", name: "Summarize Agno in one concise sentence.", type: "agent", agentId: "sage", updatedAt: "19 Jun 2026, 07:20", totalTokens: 1526 },
];

export const approvals: AgentOsApproval[] = [
  { id: "deploy-service", action: "deploy_service", detail: "service: api-gateway, environment: production", owner: "Deployment Workflow", date: "24 Feb 2025" },
  { id: "process-payment", action: "process_payment", detail: "amount: 15000, currency: USD, vendor: Cloud Services", owner: "Finance Agent", date: "24 Feb 2025" },
  { id: "publish-article", action: "publish_article", detail: "title: Q1 Product Update, channel: blog", owner: "Content Team", date: "23 Feb 2025" },
];

export const schedules: AgentOsSchedule[] = [
  { id: "hourly", name: "Hourly Sync", cron: "0 * * * *", endpoint: "/v1/agents/sync/runs", nextRun: "16 Jun 2025, 21:00", enabled: false },
  { id: "eval", name: "Daily Model Evaluation", cron: "30 22 * * *", endpoint: "/v1/agents/eval/runs", nextRun: "16 Jun 2025, 05:30", enabled: false },
  { id: "backup", name: "Weekly Backup", cron: "0 4 * * 0", endpoint: "/v1/agents/backup/runs", nextRun: "23 Jun 2025, 04:00", enabled: false },
];

export const metricSeries: AgentOsMetricSeries[] = [
  { label: "Agent Runs", value: 170, points: [0, 0, 0, 6, 6, 16, 5, 10, 5, 10, 5, 14, 9, 12, 15, 13, 0] },
  { label: "Agent Sessions", value: 170, points: [0, 0, 0, 6, 6, 17, 5, 10, 5, 10, 5, 15, 9, 12, 15, 13, 0] },
];

export const agentOsData = { sections, sessions, approvals, schedules, metricSeries };

export function getPageBySlug(slug: string): AgentOsPage {
  return navItems.find((item) => item.slug === slug) ?? navItems[0];
}
```

- [ ] **Step 3: Create `data.test.ts`**

```ts
import { describe, expect, it } from "vitest";
import { agentOsData, getPageBySlug, navItems } from "./data";

describe("agent OS demo data", () => {
  it("contains the observed primary navigation", () => {
    expect(navItems.map((item) => item.slug)).toEqual([
      "home",
      "chat",
      "sessions",
      "traces",
      "memory",
      "knowledge",
      "metrics",
      "evaluation",
      "approvals",
      "scheduler",
    ]);
  });

  it("keeps Home manifest sections populated", () => {
    expect(agentOsData.sections.map((section) => section.id)).toEqual(["agents", "teams", "workflows"]);
    expect(agentOsData.sections[0]?.entities.map((entity) => entity.name)).toContain("Builder");
  });

  it("falls back to Home for unknown pages", () => {
    expect(getPageBySlug("missing").slug).toBe("home");
  });
});
```

- [ ] **Step 4: Run the data test**

Run: `pnpm --filter @project-template/web test -- src/lib/agent-os/data.test.ts`

Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add apps/web/src/lib/agent-os
git commit -m "feat(AgentOS): 添加演示控制台数据模型"
```

## Task 2: App Shell

**Files:**
- Create: `apps/web/src/components/agent-os/shell.tsx`
- Modify: `apps/web/app/globals.css`
- Modify: `apps/web/app/page.tsx`

**Interfaces:**
- Consumes: `navItems`, `AgentOsPage`
- Produces: `AgentOsShell({ activePage, children }: { activePage: AgentOsPage; children: React.ReactNode })`

- [ ] **Step 1: Create shell component**

Implement `AgentOsShell` with sidebar nav, demo banner, OS header, support and refresh buttons. Use plain text/icon glyphs and CSS classes; do not add icon dependency.

- [ ] **Step 2: Update `page.tsx` to render the shell**

Read `searchParams.page`, resolve it with `getPageBySlug`, and render placeholder content inside `AgentOsShell`.

- [ ] **Step 3: Update global CSS**

Set body background to `#f6f6f5`, use `Inter, ui-sans-serif, system-ui`, and keep `box-sizing`.

- [ ] **Step 4: Run checks**

Run:

```bash
pnpm --filter @project-template/web lint
pnpm --filter @project-template/web typecheck
```

Expected: both PASS.

- [ ] **Step 5: Commit**

```bash
git add apps/web/app apps/web/src/components/agent-os apps/web/app/globals.css
git commit -m "feat(AgentOS): 搭建控制台应用壳"
```

## Task 3: Home Page

**Files:**
- Create: `apps/web/src/components/agent-os/cards.tsx`
- Create: `apps/web/src/components/agent-os/pages.tsx`
- Modify: `apps/web/app/page.tsx`

**Interfaces:**
- Consumes: `agentOsData.sections`
- Produces: `HomePage()`

- [ ] **Step 1: Implement entity cards**

Build compact cards with title, description, label chips, Chat and Config buttons, matching Agno's three-column desktop layout.

- [ ] **Step 2: Implement HomePage**

Render Agents, Teams, Workflows, Interfaces, and All OSes. Show hidden items collapsed and include non-functional `Show more` buttons for stage one.

- [ ] **Step 3: Wire HomePage in `page.tsx`**

Render HomePage when active slug is `home`.

- [ ] **Step 4: Run checks**

Run:

```bash
pnpm --filter @project-template/web lint
pnpm --filter @project-template/web typecheck
pnpm --filter @project-template/web test
```

Expected: all PASS.

- [ ] **Step 5: Commit**

```bash
git add apps/web/app/page.tsx apps/web/src/components/agent-os/cards.tsx apps/web/src/components/agent-os/pages.tsx
git commit -m "feat(AgentOS): 实现首页能力清单"
```

## Task 4: Data Pages

**Files:**
- Create: `apps/web/src/components/agent-os/table-page.tsx`
- Modify: `apps/web/src/components/agent-os/pages.tsx`
- Modify: `apps/web/app/page.tsx`

**Interfaces:**
- Consumes: `sessions`, `approvals`, `schedules`, `metricSeries`
- Produces: `ChatPage`, `SessionsPage`, `MemoryPage`, `KnowledgePage`, `MetricsPage`, `EvaluationPage`, `ApprovalsPage`, `SchedulerPage`, `TracesPage`

- [ ] **Step 1: Create table primitives**

Create small primitives for page headers, filter buttons, export buttons, table rows, and gated overlays.

- [ ] **Step 2: Implement ChatPage**

Render object selector, toolbar, centered loading/empty state, and bottom composer.

- [ ] **Step 3: Implement SessionsPage**

Render database/table header and three captured session rows.

- [ ] **Step 4: Implement Memory, Knowledge, Evaluation, Traces**

Reuse the table layout. Memory and Traces can render clear empty states; Knowledge and Evaluation render captured-style filters.

- [ ] **Step 5: Implement Metrics, Approvals, Scheduler**

Metrics and Scheduler render `Not available for Demo OS` overlays. Approvals renders `Admin access required` over approval rows.

- [ ] **Step 6: Wire all page branches**

Use `activePage.slug` in `page.tsx` to select each page component.

- [ ] **Step 7: Run checks**

Run:

```bash
pnpm --filter @project-template/web lint
pnpm --filter @project-template/web typecheck
pnpm --filter @project-template/web test
pnpm --filter @project-template/web build
```

Expected: all PASS.

- [ ] **Step 8: Commit**

```bash
git add apps/web/app/page.tsx apps/web/src/components/agent-os
git commit -m "feat(AgentOS): 实现核心数据页面"
```

## Task 5: Runtime Screenshot Verification

**Files:**
- Modify if needed: `apps/web/app/page.tsx`
- Modify if needed: `apps/web/src/components/agent-os/*.tsx`
- Modify if needed: `apps/web/app/globals.css`

**Interfaces:**
- Consumes: local `http://localhost:3000`
- Produces: verified visual parity notes in the final response

- [ ] **Step 1: Start local web app**

Run: `pnpm --filter @project-template/web dev`

Expected: local app serves on `http://localhost:3000`.

- [ ] **Step 2: Capture local pages in Chrome**

Capture:

- `http://localhost:3000/`
- `http://localhost:3000/?page=chat`
- `http://localhost:3000/?page=sessions`
- `http://localhost:3000/?page=metrics`
- `http://localhost:3000/?page=approvals`
- `http://localhost:3000/?page=scheduler`

- [ ] **Step 3: Compare against Agno screenshots**

Check shell geometry, active nav, top banner, card density, table headers, row spacing, overlays, and composer positioning.

- [ ] **Step 4: Patch obvious visual mismatches**

Only patch mismatches that block recognizability or usability. Do not chase pixel perfection.

- [ ] **Step 5: Run final verification**

Run:

```bash
pnpm --filter @project-template/web lint
pnpm --filter @project-template/web typecheck
pnpm --filter @project-template/web test
pnpm --filter @project-template/web build
```

Expected: all PASS.

- [ ] **Step 6: Commit**

```bash
git add apps/web
git commit -m "fix(AgentOS): 根据截图校准控制台界面"
```

## Self-Review

- Spec coverage: Tasks cover local data, shell, Home, core pages, and screenshot verification.
- Placeholder scan: No `TBD` or unspecified task remains.
- Type consistency: Type and function names in later tasks match Task 1 outputs.
