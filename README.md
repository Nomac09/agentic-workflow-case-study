# Agentic workflow case study: La Donna Di Ferro

How I rebuilt and now upgrade a production e-commerce site with Claude Code agents, without letting the agents near production.

**Live site:** [ladonnadiferro.com](https://ladonnadiferro.com), a Dubai womenswear label.
**Stack:** Next.js 15 (App Router), React 19, TypeScript strict, Tailwind CSS v4, Supabase (Postgres, RLS), Stripe Checkout, Resend, Vercel, Vitest, Playwright, GitHub Actions.

The application code stays private because it is a client's live business. This repository shows the process, with the real agent definitions, settings and excerpts from the spec and decision logs.

*Version française en bas de page.*

---

## The project in two stages

1. **Migration.** The label ran on Wix. I scraped the live site (pages, catalogue, images, hero video), rebuilt it in Next.js with the same look and content, and added what Wix could not do: a real Stripe checkout, transactional email, an admin area and a Supabase catalogue with a committed JSON fallback, so the site runs with no environment variables at all. Every old Wix URL is a permanent redirect, so links and search results survived the move.
2. **Upgrade in phases.** Before changing anything else, I ran an audit (Phase 0), shipped a security hotfix, then put the engineering guardrails in place (Phase 1) so that every later phase is built on them.

---

## The workflow

```
SPECS.md        spec with ID, requirement, acceptance criteria, test
                approved by me before any code is written
   |
test-writer     writes tests from the acceptance criteria and the public
                interface only, without reading the implementation
   |
implementer     builds exactly one spec ID; stops and reports if it needs
                a file outside that spec's scope
   |
reviewer        read-only; checks the diff against the spec, returns
                "approve" or "changes needed" with numbered findings
   |
CI              lint, typecheck, unit tests, E2E, dependency audit gate,
                database tests on a local Supabase stack
   |
pull request    I review the diff and merge only on a green run
   |
DECISIONS.md    one entry per significant choice or incident:
                context, options, decision, consequence
```

Every commit names its spec IDs (Conventional Commits, for example `fix(scripts): refuse to seed unless --target matches the project (SEC-06)`), and a traceability table maps each spec to its pull request and its test files.

The three agent definitions are in [`examples/agents/`](examples/agents/).

---

## Guardrails are code, not instructions

Telling an agent "never touch production" is not enough. The rules are enforced:

- **Permission deny list** ([`examples/claude-settings.json`](examples/claude-settings.json)): agents cannot read or edit env files, run the seed script, touch Vercel env or production deploys, push to `main`, or edit their own `.claude/` rules, git config or git hooks.
- **Hooks:** a guard hook checks every tool call before it runs; a post-edit hook runs lint and typecheck after each edit; a pre-commit hook runs the tests. The hooks have their own tests.
- **End-to-end tests run without credentials.** `next build` loads the local env file whatever the shell says, so Playwright builds and serves from a clean copy of the repository that contains only what git would commit, with every Supabase, Stripe and Resend variable removed from the server environment. I verified it: a fake key exported in the shell is absent from the build output with the copy, and present in 17 files without it. The config also refuses to run against the live domains.
- **Seeding is guarded.** The seed script refuses to run unless its `--target` matches the project it is pointed at.

The guard is described in the decision log as "a seatbelt against mistakes, not a sandbox": the last line of defence is still a human reviewing the diff.

---

## Rules that came from mistakes

- **Red evidence is required.** In Phase 1, the test-writer sometimes ran after the implementer, so two new tests could never fail and nobody noticed until a mutation check. Now every new test must be seen failing for the right reason, either before the implementation or through a named one-line mutation that is applied and reverted. The reviewer treats missing red evidence as blocking.
- **Stop patching, cap the rounds.** Each patch to the guard's command parser surfaced new edge cases. After the third review, I simplified the rules and accepted the remaining risk explicitly, covered by human review, instead of chasing an endless parser.
- **Security first.** The Phase 0 audit found an access-control flaw in the database rules. It was fixed in a hotfix pull request with a regression test, and audit queries confirmed it had never been exploited. Four smaller hardening fixes shipped in the same pull request (safe redirects, HTML escaping in emails, safe JSON-LD).

---

## Numbers

| | |
|---|---|
| Commits | 85 |
| Specs, hotfix + Phase 1 | 21, all done or dropped with a reason |
| Specs, Phase 2 | 13 (12 approved, 1 proposed) |
| Test files | 18 (Vitest unit and component, Playwright E2E, SQL database tests) |
| CI jobs | checks, E2E, audit gate, DB tests |

---

## What is in this repository

```
examples/
  agents/implementer.md    one spec, nothing else
  agents/test-writer.md    tests from acceptance criteria, red evidence
  agents/reviewer.md       read-only verdict with a fixed template
  claude-settings.json     permission deny list and hook wiring
  SPECS-excerpt.md         traceability table and one full spec
  DECISIONS-excerpt.md     three real entries from the decision log
```

---

## En français

Étude de cas sur la refonte de ladonnadiferro.com : migration complète de Wix vers Next.js, puis montée en qualité par phases avec des agents Claude Code. Chaque évolution part d'une spécification avec critères d'acceptation validée avant le code ; un agent écrit les tests à partir des critères, un autre implémente une seule spec, un troisième relit sans pouvoir modifier ; la CI tourne sur chaque pull request et chaque décision est consignée. Les garde-fous sont du code (liste d'interdictions, hooks, tests E2E construits sans identifiants de production), pas de simples consignes. Le code de l'application reste privé ; ce dépôt montre la méthode avec les vrais fichiers d'agents et des extraits des journaux.

**Nasim Sawalma** · [GitHub](https://github.com/Nomac09) · [LinkedIn](https://www.linkedin.com/in/nasim-sawalma) · nasim.sawalma@gmail.com
