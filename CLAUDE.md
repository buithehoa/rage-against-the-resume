# Rage Against the Resume

## Project Concept

A static portfolio site that replaces the traditional prose-based resume/LinkedIn profile with evidence pulled directly from public GitHub activity. The core thesis: LinkedIn-style employment history is self-composed narrative; this site instead surfaces what was actually built — personal projects, side projects, open-source contributions — as the primary "proof."

Core interaction: a recruiter (or the user) can filter the project list by language/topic/skill (e.g. "Ruby, Rails, Cryptocurrency"), and the filtered view's state is reflected in the URL, so it can be copied and shared as a link that shows exactly the curated subset intended.

## Scope Decisions

1. **Curation over raw dump.** Not just an unfiltered pull of every public repo. A mechanism will be designed (later, in the design phase) to highlight "good" projects — ones with solid descriptions and, where possible, live demos — rather than treating all repos as equally representative.
2. **Articles / blog posts / writing** are an intended future feature but explicitly out of scope for v1. No data source or design decision has been made for this yet.
3. **Data pipeline is build-time, not runtime.** A scheduled process (e.g. GitHub Actions) fetches GitHub data and writes it to a static JSON file at build time. The site itself only ever reads static data — no client-side calls to the GitHub API (avoids the 60 req/hour unauthenticated rate limit, which is shared per IP and could be exhausted by ordinary traffic).
4. **Build cadence:** scheduled roughly once a day to keep data fresh. Cron-based scheduling in GitHub Actions is not exact (GitHub deprioritizes scheduled runs under load), so a manual `workflow_dispatch` trigger will also be included to force an on-demand rebuild after pushing something the user wants reflected immediately. The workflow should run: `schedule` (daily cron at 00:00 UTC+7, i.e. `0 17 * * *` UTC) + `workflow_dispatch` only — deliberately no `push` trigger, since the user pushes to `main` often as a solo developer; the fetch-GitHub-data step runs before `nuxt build`.

## Tech Stack Decisions

- **Framework: Nuxt (Vue 3)**, using static generation to produce a fully static site deployable to GitHub Pages. Build with `npx nuxt build --preset github_pages` (prerenders to static HTML in `.output/public`) or `nuxt generate`.
  - Rejected React (user wants to try something else on this project) and Astro (user prefers a single coherent framework over Astro's static-builder + islands split).
  - Switched from SvelteKit to Nuxt because the user already uses Svelte on another project and wants variety here. Vue is also more commonly listed in job postings.
  - Rationale for Nuxt: file-based routing (`pages/`), prerendering of every route (including future per-project detail pages) without SPA-fallback workarounds, build-time data loading via `useAsyncData`, and reactive URL query state via `useRoute().query` + `router.replace`.
  - Known trade-off: Nuxt is bigger than this site needs (server/Nitro features go unused). The server layer is build-time only and does not ship to the browser; the client gets Vue + vue-router + Nuxt's small runtime.
- **Hosting:** GitHub Pages.
  - If not using a custom domain, set `NUXT_APP_BASE_URL=/<repository>/` for the build step (site is served from `https://<user>.github.io/<repository>/`).
  - Deploy via GitHub Actions: `actions/upload-pages-artifact` (path `./.output/public`) + `actions/deploy-pages`. Job needs `pages: write` and `id-token: write` permissions. Node 20 recommended.
- **Data source:** GitHub public repos (via GitHub API, fetched at build time only).
- **Scaffold:** `npx nuxi@latest init rage-against-the-resume` (Node.js 18+, even-numbered release recommended). Verify against the current Nuxt docs if the command fails.
- **Naming:** repo and package name `rage-against-the-resume` (kebab-case).

## Core Feature (v1)

- Pull the user's public GitHub repos at build time into a static JSON file.
- Render a filterable list/grid of projects.
- Filters: language, topic (and eventually curated "featured" status once the design step defines it).
- Filter state syncs to URL query params, so a specific filtered view is shareable via a copyable link — recruiter sees exactly the curated subset the user intends.

## Deferred / Future Scope

- Articles, blog posts, and other writing — data source and integration not yet decided (may be a second data source: RSS, Dev.to/Medium API, or manual entry, since these won't come from GitHub repos unless the writing itself lives in a repo).
- Design mechanism for surfacing "good" projects (descriptions, demo links, featured flag) — to be defined in the design phase.

## Project Name

**Rage Against the Resume**
