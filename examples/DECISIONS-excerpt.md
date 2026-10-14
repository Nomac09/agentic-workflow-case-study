# DECISIONS.md (excerpt)

One entry per significant technical choice or incident: context, options considered, decision, consequence. Three real entries, unedited.

---

## 2026-10-11 - E2E server runs from a clean copy of the repo with service keys removed (TEST-03)

**Context.** Playwright's `webServer` builds and starts the app for the smoke test. The repo's local env file holds production Supabase, Stripe and Resend credentials, and `@next/env` loads it for `next build` and `next start` whatever the process environment says: blanking a variable does not stop the file's value being used. Playwright also merges its own `process.env` into the server's environment and offers no way to remove a key. Without those keys the app uses the committed seed catalogue and disables checkout and auth, which is what E2E should test against until Phase 7 adds a test backend.

**Options.** (a) Blank the keys in `webServer.env`: does not work, for the reason above. (b) A committed `.env.test` with dummy values: Next only reads it when `NODE_ENV=test`, which `next build` does not use, and the local file would still load and win for any key the dummy file leaves out. (c) Move or rename the local env file during the run: touches a file agents must not touch, and a killed run leaves it moved. (d) Build and serve from a directory that holds only what git would commit (`git ls-files -co --exclude-standard`, which leaves out ignored files such as the local env file), with `node_modules` symlinked, and spawn Next with the service keys deleted from its environment.

**Decision.** (d), in `e2e/web-server.ts`, run by `webServer.command`. It refreshes `.e2e-app/` (kept between runs only for `.next` and the symlink, so the build cache is reused), throws if any env file other than `.env.example` ends up there, then runs `next build` and `next start` with `sanitizeServerEnv` from `e2e/safety.ts`, which deletes every variable whose name contains SUPABASE, STRIPE or RESEND, plus `ORDER_NOTIFICATION_EMAIL`. The copy keeps production config out; the deletion keeps out keys exported in the shell. The server listens on 3100 rather than 3000, and `reuseExistingServer` is off: a server already on the port could be one started from the repo with real credentials, so Playwright always starts its own (after review; about 15 seconds per run). `NEXT_PUBLIC_SITE_URL` is set to the local origin. Separately, the config throws if `baseURL` (overridable with `E2E_BASE_URL` for Vercel previews) is on `ladonnadiferro.com`, `ladonnadiferro.fr` or a subdomain.

**Consequence.** Verified by building with a fake `NEXT_PUBLIC_SUPABASE_URL` exported: the value is absent from `.e2e-app/.next`, and present in 17 files when the deletion is disabled. A cold run (copy, build, start, two projects) takes about 15 seconds locally. The copy is ignored by git, ESLint, Prettier and tsc. If port 3100 is already in use, the run fails rather than reusing it. `nextRuns()` in `e2e/web-server.ts` is the pure description of what gets spawned, so a test proves both Next runs use the clean copy and the cleaned environment. `e2e/` has its own `tsconfig.json` (like `.claude/hooks/`) because Node's type stripping needs `.ts` import extensions, and a `package.json` with `"type": "module"` so Node does not reparse it.

---

## 2026-10-11 - Phase 1 retrospective: red evidence is now required (WF)

**Context.** In Phase 1 the test-writer often ran alongside or after the implementer, so new tests never ran red. Twice that hid a test that could not fail (TOOL-02's lint rule tests; a first config test). The coordinator caught both with mutation checks done ad hoc.

**Options.** (a) Always run the test-writer strictly first. Not always possible: some tests need the interface the implementer defines. (b) Require evidence that each new test has been seen to fail, from either order.

**Decision.** (b). The test-writer must report red evidence for every new test (a failing run before the implementation, or a named one-line mutation the coordinator applies and reverts). The reviewer treats missing red evidence as a blocking finding and records it per acceptance criterion. Also from the retrospective: agent instructions no longer list `npm run build` as safe, since in this repo it loads production credentials.

**Consequence.** One extra step per spec. A guard rule that blocks `npm run build` and `next dev` in the repository itself is proposed for Phase 2 (alongside WF-05) rather than relying on instructions.

---

## 2026-10-11 - Prettier: settings, and Markdown left out (TOOL-01)

**Context.** TOOL-01 adds Prettier with the Tailwind class-sorting plugin and reformats the repository once.

**Options.** Settings were measured, not guessed: formatting the 121 JS/TS files in memory under each candidate, 80 columns with trailing commas everywhere changed the fewest lines (about 180), so the code already followed Prettier's defaults. For Markdown: format it, or leave it out.

**Decision.** Prettier defaults (80 columns, double quotes, semicolons, trailing commas) plus `prettier-plugin-tailwindcss`. Ignored: scraped material, `public/`, the generated `products.json`, the Supabase email templates (pasted verbatim into the dashboard), the frozen Lenses section, and all Markdown. Prettier pads every row of a table to the widest cell, so a one-word status change in the SPECS.md traceability table would rewrite the whole table and hide the real change in review and in `git blame`.

**Consequence.** Markdown style is kept by hand. The one-off reformat commit is listed in `.git-blame-ignore-revs`; run `git config blame.ignoreRevsFile .git-blame-ignore-revs` once locally (GitHub applies it automatically).
