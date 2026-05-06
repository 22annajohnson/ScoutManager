# Scout Manager Dashboard MVP Architecture

## Design Philosophy

- Optimize for founder speed over scalability
- Prefer simple readable code over abstraction
- Ship working flows before generalized systems
- Avoid rebuilding tools Jira already solves
- Mobile-first always

The dashboard is primarily an orchestration and context layer, not a replacement for Jira.

It should optimize for:

- Fast solo-founder iteration
- Mobile-first usability
- Extremely low operational complexity
- Minimal infrastructure
- Easy future expansion without building it now

It should avoid:

- Custom ticketing systems
- Jira replacement architecture
- Enterprise abstractions
- Unnecessary backend services
- Premature optimization

## MVP Product Boundary

Jira remains the source of truth for:

- Tickets
- Epics
- Statuses
- Sprint tracking
- Development workflow

The dashboard should add:

- Lightweight initiative organization
- Mobile-friendly visibility into current work
- Founder notes and context
- AI-assisted summaries and planning
- Fast navigation into Jira when deeper actions are needed

The dashboard should not:

- Rebuild Jira boards
- Reimplement Jira workflows
- Create a second project management system
- Add automation/orchestration infrastructure in V1

## Initial Page Set

Start with only these routes:

- `/login`
- `/dashboard`
- `/initiatives`
- `/initiatives/:id`

V1 should avoid route sprawl. Epic and issue detail should live in sheets, cards, or sections inside the initiative view unless a real need emerges.

## Navigation and Mobile Layout

Use a mobile-first app shell:

- Bottom nav: `Home`, `Initiatives`, `Chat`
- Top bar: page title, refresh, logout

Layout rules:

- Single-column by default
- Slide-up sheets for issue detail, filters, and chat on phone
- Keep taps shallow and obvious
- Avoid dense tables as primary UI
- Let desktop become a simple wider layout, not a separate product

## Core Screen Responsibilities

### `/login`

- Magic link or basic email auth
- Minimal styling
- No complex auth flows

### `/dashboard`

Purpose: show what needs attention now.

Sections:

- Active initiatives
- Blocked or stale Jira work
- Recently updated relevant issues
- Short AI summary card

### `/initiatives`

Purpose: show the founder's internal planning layer above Jira.

Each initiative should include:

- Title
- Status
- Priority
- Short summary
- Linked work count

### `/initiatives/:id`

Purpose: the primary founder command center.

Sections:

- Initiative summary
- Linked epics and issues
- Lightweight Jira status visualization
- Notes/context
- Contextual AI chat

This should be the most valuable page in the MVP.

## Recommended Architecture

Keep the system intentionally thin:

- React frontend
- Supabase Auth and Postgres
- Supabase Edge Functions for Jira access
- Supabase Edge Functions for AI access
- No custom backend server

High-level flow:

1. React reads overlay data from Supabase
2. React calls Edge Functions for Jira and AI
3. Edge Functions hide secrets and normalize responses
4. Jira remains the source of truth for delivery work
5. Supabase stores only local overlay/context data

This keeps infrastructure, maintenance, and ops complexity extremely low.

## Jira Integration Guidance

Use Jira as the work engine, not as something to duplicate.

Recommended Jira approach:

- Fetch only the Jira data needed for the current screen
- Normalize Jira payloads in Edge Functions
- Keep frontend types small and stable
- Link back to Jira for deeper workflow interaction

Important:

Avoid Jira sync/caching entirely until clear pain emerges.

Do not build:

- A mirrored Jira database
- Webhook ingestion
- Scheduled sync jobs
- Board replication
- Workflow modeling in the app

Suggested Edge Functions:

- `jira-search`
- `jira-issue`
- `jira-initiative-context`
- `jira-status-summary`

## AI Architecture

Use a single AI service layer using Supabase Edge Functions.

V1 AI responsibilities:

- Summarize linked Jira work
- Summarize notes/context
- Suggest next actions
- Surface blockers or missing clarity

V1 AI should not:

- Edit Jira automatically
- Orchestrate agents
- Run background automation
- Mutate workflow state

Chat should be contextual rather than generic.

Suggested chat contexts:

- Dashboard context
- Initiative context

Prompt payloads should include:

- Page context
- Initiative metadata
- Linked Jira issue summaries
- Local notes

Suggested Edge Function:

- `ai-chat`

## Database Schema

Supabase should store only overlay and context data.

### `initiatives`

- `id`
- `title`
- `status`
- `priority`
- `summary`
- `created_at`
- `updated_at`

### `initiative_links`

- `id`
- `initiative_id`
- `jira_key`
- `link_type`

### `notes`

- `id`
- `entity_type`
- `entity_id`
- `body`
- `created_at`
- `updated_at`

### `chat_threads`

- `id`
- `entity_type`
- `entity_id`
- `created_at`

### `chat_messages`

- `id`
- `thread_id`
- `role`
- `content`
- `created_at`

Not needed initially:

- `saved_views`
- `user_preferences`
- `jira_cache`

## Frontend Structure

Keep the codebase boring and readable.

```text
src/
  app/
    router.tsx
    providers.tsx
    shell/
      AppShell.tsx
      TopBar.tsx
      MobileBottomNav.tsx

  pages/
    LoginPage.tsx
    DashboardPage.tsx
    InitiativesPage.tsx
    InitiativeDetailPage.tsx

  features/
    auth/
    dashboard/
    initiatives/
    jira/
    notes/
    chat/

  components/
    ui/

  lib/
    supabase.ts
    queryClient.ts
    constants.ts
```

Rules:

- Page files should own layout composition
- Feature folders should own fetching and domain-specific UI
- Shared UI should remain minimal
- Avoid premature platform/core/domain folder structures

## Component Plan

First-pass components:

- `AppShell`
- `TopBar`
- `MobileBottomNav`
- `DashboardOverview`
- `InitiativeList`
- `InitiativeCard`
- `InitiativeHeader`
- `LinkedJiraIssues`
- `JiraIssueCard`
- `JiraStatusVisualization`
- `NotesPanel`
- `ChatPanel`
- `EmptyState`
- `LoadingState`

Do not build a large design system first. Add reusable UI only when it is actually reused.

## State Management

Use:

- React Query for server/data state
- `useState` for local UI state
- A tiny store later only if one specific global UI concern appears

Avoid:

- Redux
- Event buses
- Heavy client-side data architecture
- Generalized repository/service patterns without real repetition

## Lightweight Jira Status Visualization

Show a simplified mobile-friendly grouping of Jira work.

Suggested status buckets:

- `Backlog`
- `Ready`
- `In Progress`
- `Done`

Important framing:

- This is a lightweight Jira status visualization
- It is not a recreated Jira board
- It is not drag-and-drop
- It does not own workflow logic

Each item can show:

- Issue key
- Title
- Assignee
- Priority
- Tap for issue details
- Link to open in Jira

## What To Mock First

Start with mocked data before integrating real services.

Mock:

- Auth state
- Initiative list
- Initiative detail data
- Linked Jira issues
- Lightweight Jira status visualization
- AI responses
- Notes content

This is the fastest way to validate:

- Mobile usability
- Founder workflow quality
- Information density
- Screen hierarchy
- Whether the initiative page is genuinely helpful

## Implementation Order

1. Scaffold app with Vite, Tailwind, router, and shell
2. Build mobile nav and top bar
3. Create mocked `DashboardPage`
4. Create mocked `InitiativesPage`
5. Create mocked `InitiativeDetailPage`
6. Add notes UI and persistence in Supabase
7. Add Supabase auth
8. Add initiative persistence in Supabase
9. Add Jira read integration through Edge Functions
10. Replace mocks with real Jira-linked content
11. Add AI chat panel through the AI service layer
12. Polish loading, empty, and mobile interaction states

## What Should Stay Hardcoded For Now

Hardcode initially:

- Status bucket mapping
- Dashboard section ordering
- Priority labels
- Basic initiative statuses
- Simple AI prompt structure
- Default page layouts

Do not try to make these configurable yet.

## What Waits Until V2

- Dedicated epic pages if truly needed
- Dedicated issue pages if sheets become insufficient
- Jira write-back actions
- Drag/drop interactions
- Saved filters/views
- Sync/caching systems
- Multi-user permissions
- Notifications
- Analytics
- Agent systems
- Automation systems
- Workflow customization

## Biggest Risks To Avoid

1. Rebuilding Jira accidentally through helpful-seeming features
2. Adding abstraction before real repetition exists
3. Modeling too much complexity for a single founder user
4. Adding a custom backend service when Edge Functions are enough
5. Turning status visualization into a board product
6. Spending time on sync/caching before pain is proven
7. Designing desktop-first instead of phone-first
8. Letting AI expand into orchestration before the core dashboard is useful

## Guiding Founder Flow

The MVP should optimize around this path:

`Login -> Dashboard -> Initiative -> Linked Jira work -> Notes/context -> AI guidance -> Open Jira when needed`

If a proposed feature does not improve that flow, it should probably wait.
