# Scout Manager Dashboard

Internal founder dashboard for Scout.

This app is intended to sit on top of Jira as a lightweight orchestration and context layer. It is not a Jira replacement.

## MVP Principles

- Optimize for founder speed over scalability
- Prefer simple readable code over abstraction
- Ship working flows before generalized systems
- Avoid rebuilding tools Jira already solves
- Mobile-first always

## Product Boundary

Jira remains the source of truth for:

- Tickets
- Epics
- Statuses
- Sprint tracking
- Development workflow

This dashboard should provide:

- A founder-friendly mobile layer
- Simplified operational visibility
- Lightweight context and notes
- AI-assisted summaries and planning
- Faster navigation across active work

This dashboard should not provide:

- A custom ticketing system
- Rebuilt Jira boards or workflows
- Enterprise abstractions
- Extra backend services without clear need
- Premature optimization

## Initial MVP Scope

Start with four pages:

- `/login`
- `/dashboard`
- `/initiatives`
- `/initiatives/:id`

The main founder flow is:

`Login -> Dashboard -> Initiative -> Linked Jira work -> Notes/context -> AI guidance -> Open Jira when needed`

## Tech Stack

- React
- Vite
- Tailwind
- React Router
- React Query
- Supabase Auth
- Supabase Postgres
- Supabase Edge Functions

## Architecture Doc

See [docs/mvp-architecture.md](/Users/anna/ScoutManager/docs/mvp-architecture.md) for the refined MVP architecture and implementation plan.
