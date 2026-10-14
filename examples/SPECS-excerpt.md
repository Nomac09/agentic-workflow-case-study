# SPECS.md (excerpt)

Every change starts as a spec with an ID, a requirement, acceptance criteria and a test. Specs are approved before implementation; commits and pull requests reference spec IDs.

## Traceability

| Spec | Phase | PR | Tests | Status |
|---|---|---|---|---|
| SEC-01 | Hotfix | #1 | `supabase/tests/profiles_role_escalation.test.sql` | done |
| SEC-02 | Hotfix | #1 | `src/app/auth/callback/route.test.ts`, `src/lib/safeRedirect.test.ts` | done |
| SEC-03 | Hotfix | #1 | `src/lib/safeRedirect.test.ts` (unit); TEST-02 (component) | done |
| SEC-04 | Hotfix | #1 | `src/lib/emailTemplates.test.ts`, `src/lib/html.test.ts` | done |
| SEC-05 | Hotfix | #1 | `src/lib/jsonLd.test.ts` | done |
| WF-01 | 1 | #2 | review check | done |
| WF-02 | 1 | #2 | review check + dry run | done |
| WF-03 | 1 | #2 | `.claude/hooks/*.test.ts` | done |
| WF-04 | 1 | #2 | `.claude/hooks/*.test.ts` | done |
| TOOL-01 | 1 | #2 | CI `format:check` | done |
| TOOL-02 | 1 | #2 | `src/test/lintRules.test.ts` | done |
| TEST-01 | 1 | #2 | `src/test/setup.test.tsx`, `src/test/environment.test.ts` | done |
| TEST-02 | 1 | #2 | `src/components/auth/AuthModal.test.tsx` | done |
| TEST-03 | 1 | #2 | `e2e/smoke.spec.ts`, `e2e/env.guard.test.ts` | done |
| TEST-04 | 1 | #2 | CI job `db-tests` (run 38154395721: 16/16 PASS) | done |
| CI-01 | 1 | #2 | the Phase 1 PR run itself (run 38154395721 green; earlier runs red) | done |
| ENV-01 | 1 | #2 | `src/test/repoHygiene.test.ts` | done |
| DEP-01 | 1 | #2 | CI build + `scripts/audit-check.test.ts` | done |
| CLEAN-01 | 1 | #2 | typecheck + build + `src/test/repoHygiene.test.ts` | done |
| CLEAN-02 | 1 | - | - | dropped |
| SEC-06 | 1 | #2 | `scripts/seedGuard.test.ts` | done |
| WF-05 | 2 | - | `.claude/hooks/guard.test.ts`, `.claude/hooks/settings.test.ts` | approved |
| DOC-01 | 2 | - | review check | approved |
| SEC-07 | 2 | - | `src/lib/envGuard.test.ts`, `src/instrumentation.test.ts`, `scripts/seedGuard.test.ts` | approved |
| FORM-01 | 2 | - | `src/lib/spam.test.ts`, `src/app/actions/contact.test.ts`, `src/app/actions/newsletter.test.ts`, `src/components/contact/ContactForm.test.tsx`, `src/components/home/Newsletter.test.tsx`, `supabase/tests/form_rate_limits.test.sql` | proposed |
| DATA-01 | 2 | - | CI `db-tests` drift step; typecheck | approved |
| DATA-02 | 2 | - | `src/lib/data/products.test.ts`, `src/lib/data/orders.test.ts`, `src/test/repoHygiene.test.ts` | approved |
| DATA-03 | 2 | - | `src/lib/data/products.test.ts`, `src/test/repoHygiene.test.ts` | approved |
| CACHE-01 | 2 | - | `src/lib/data/products.test.ts`, `src/app/actions/admin.test.ts` | approved |
| DATA-04 | 2 | - | `src/app/api/checkout/route.test.ts` | approved |
| ARCH-01 | 2 | - | `src/components/layout/useHeaderIdentity.test.tsx`, `e2e/static.spec.ts` | approved |
| ARCH-02 | 2 | - | `src/middleware.test.ts` | approved |
| RSC-01 | 2 | - | `e2e/product-card.spec.ts`, `src/test/repoHygiene.test.ts` | approved |
| FMT-01 | 2 | - | `src/lib/format.test.ts` (Nasim) | approved |

---

---

## Example spec

### WF-02 Agent roles

**Requirement.** Three Claude Code subagents in `.claude/agents/`:

| Agent | Job | Tools |
|---|---|---|
| `implementer` | Implements exactly one spec ID per run. Does not touch files outside that spec's scope; if it needs to, it stops and reports. | Read, Edit, Write, Grep, Glob, Bash |
| `test-writer` | Writes tests from a spec's acceptance criteria and the public interface (types, function signatures, props) only. Does not read the implementation body first. | Read, Edit, Write, Grep, Glob, Bash |
| `reviewer` | Reviews a diff against its spec IDs and the Quality requirements in `UPGRADE.md`. Returns `approve` or `changes needed` with numbered reasons. Never edits. | Read, Grep, Glob, Bash (read-only commands) |

**Acceptance criteria.**
1. Each agent file has a `description` that says when to use it and a system prompt that states its scope limits, the style rules from `UPGRADE.md` (no em dashes, comments explain why, Conventional Commits with spec IDs) and the production safety rules.
2. `reviewer` has no Edit or Write tool in its frontmatter.
3. `test-writer`'s prompt names the files it may read (the spec, type/interface files, existing tests and fixtures) and requires it to state in its output which files it read.
4. `reviewer`'s output follows a fixed template: verdict, spec IDs checked, numbered findings (each with file:line, why, suggested fix), and a "not checked" section.

**Test.** Review check that the three files meet 1 to 4. Dry run: the `reviewer` is run on one Phase 1 commit and its verdict is pasted into the PR description.
