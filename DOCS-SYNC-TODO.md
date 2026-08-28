# Documentation Sync — Follow-up TODO

Working notes from the documentation review carried out against the
[aspnet-zero-core](https://github.com/aspnetzero/aspnet-zero-core) `dev` branch.

**Branch:** `feature/docs-sync-15.5` (not pushed)

## Source repository state at the time of the review

| | |
|---|---|
| Latest release branch | `origin/rel-15.4` (last commit 2026-07-23) |
| `dev` branch | `common.props` still says `15.4.0`, but the content is the upcoming **15.5** (ng-zorro, API keys, Metronic 8.3.3, ABP 11.3.1) |
| Target framework | `net10.0` |
| ABP | 11.3.1 · `Abp.AspNetZeroCore` 6.1.0 |
| Angular | 21 |
| React | 19 + Vite |
| Mobile | .NET MAUI (no Xamarin project left) |
| Auth server | OpenIddict (IdentityServer4 fully removed) |
| Mapping | Mapperly |

## What was already done on this branch

1. **Prerequisites** — "Visual Studio 2017 (v15.9.0+)" replaced with Visual Studio 2026 plus an explicit .NET 10 SDK requirement in the four getting started documents and the solution structure documents.
2. **IdentityServer4 removed** — two integration documents deleted, navigation entries dropped, and the remaining references (`"IdentityServer"` appsettings key, ConsoleApiClient, used libraries) pointed at OpenIddict.
3. **Email Templates documented** — a feature document per UI (`Features-Mvc-Core-Email-Templates.md`, `Features-Angular-Email-Templates.md`, `Features-React-Email-Templates.md`), wired into the three navigations and the Next-link chain.
4. **Sending Emails rewritten** — it still described a single `default.html` template file that no longer exists.
5. **React navigation** — the missing **QR Login** entry added.
6. **Version-Differences rewritten** — it compared ASP.NET Core against MVC 5.x/AngularJS and claimed Twitter login is unavailable in Angular, which is no longer true.
7. **Change-Logs** — 15.4.0 entry added.
8. **Used-libraries lists refreshed** for Angular and MVC against the actual package manifests.
9. **Broken links fixed** — MAUI file-name casing, `Adding-New-Localization-React.md` created, `Overview-React` reference repointed.
10. **Health Check page documented** in `HealthChecks.md`.
11. **Xamarin → MAUI** across all navigation-linked documents, and the non-existent `*.Maui.sln` corrected to `*.Mobile.sln`.
12. **MVC Tenant Sign Up documented** — `Features-Mvc-Core-Tenant-Sign-Up.md` created and wired in (Angular and React already had this page).

---

## TODO

### 1. Research consolidating the AI assistant configuration files

The template ships a separate configuration tree for every AI coding assistant:

```
AGENTS.md
mcp.json
.agent/       agents/, commands/
.claude/      AGENT-INDEX.md, COMMAND-INDEX.md, agents/, commands/, skills/
.cursor/      agents/, commands/, mcp.json
.gemini/      settings.json
.windsurf/    rules/, skills/
.github/      copilot-instructions.md, prompts/
.vscode/      mcp.json
```

That is roughly 7 agent definitions, ~15 commands and ~20 skill files, duplicated across five vendors.

- [ ] **Investigate whether these can be reduced to a single, vendor-neutral structure** — one general `AGENTS.md` (plus a shared `skills/` or `commands/` folder) that every editor can read, instead of one copy per assistant. Check the current level of support in Claude Code, Cursor, Windsurf, GitHub Copilot and Gemini for a shared `AGENTS.md` convention, and what would still need a vendor-specific shim.
- [ ] Once the structure is settled, decide whether it deserves its own documentation section (today it is only mentioned by a single line in the 15.2.0 change log).

### 2. Missing screenshots

Images referenced by documents but absent from `docs/en/images/`:

- [ ] `images/external-logins-set-password.png` — referenced by `Features-Mvc-Core-Social-Account-Linking.md` and `Features-React-Social-Account-Linking.md`
- [ ] `images/customizable-dashboard-filters.png` — referenced by `Developing-React-Customizable-Dashboard.md`
- [ ] `images/customizable-dashboard-save-page-request.png` — referenced by `Developing-React-Customizable-Dashboard.md`
- [ ] `images/customizable-dashboard-save-page-payload.png` — referenced by `Developing-React-Customizable-Dashboard.md`

Screenshots still needed for the newly written documents:

- [ ] **Email Templates page** — one screenshot per UI (list on the left, editor with the Preview / Template body tabs on the right). The three documents carry a `<!-- TODO -->` comment where the image should go. Suggested names: `features-email-templates-mvc.png`, `features-email-templates-angular.png`, `features-email-templates-react.png`.
- [ ] **MVC Tenant Sign Up** — `Features-Mvc-Core-Tenant-Sign-Up.md` currently reuses the shared `tenant-signup-v3.png` and `new-tenant-select-edition-2.png` images, the same way the Angular and React documents do. Replace them with MVC specific captures if exact fidelity is wanted.

### 3. Health Check page for the React UI

The in-application **Health Check** administration page exists in the MVC and Angular UIs (`HealthCheckAppService`, permission `Pages.Administration.Host.HealthCheck`), but the React UI only has the generated service proxy — there is no page, route or menu item.

- [ ] **Implement the Health Check page in the React UI** so that all three UIs are at parity.
- [ ] After it ships, remove the "not implemented in the React UI yet" notes from `HealthChecks.md` and `Version-Differences.md`.
- [ ] Until then, decide whether the **Health Checks** entry should stay in `nav-aspnet-core-react.json`. It currently points at `HealthChecks.md`, which is mostly about the `/health` and `/healthchecks-ui` endpoints — those do work in the React project type — so leaving it is defensible.

### 4. Audit the Angular tutorial series for PrimeNG remnants

The Angular UI moved from PrimeNG to ng-zorro-antd on `dev`. `Infrastructure-Angular-Used-Libraries-Frameworks.md` and the Power Tools custom code documents already reflect this, but the tutorial series was not checked.

- [ ] Go through `Developing-Step-By-Step-Angular-*.md` (and the Angular parts of the other guides) and replace any PrimeNG components, imports and screenshots with their ng-zorro equivalents.
- [ ] Same check for `Feature-Dynamic-Entity-Parameters-*-Angular.md`, `Developing-Angular-Customizable-Dashboard.md` and `Extending-Existing-Entities-Core-Angular.md`.

---

## Open questions

- [ ] **Change log release date.** The 15.4.0 entry was added without a date because it could not be determined from the source repository (`origin/rel-15.4` ends on 2026-07-23, but the 15.3.0 entry's date does not match its branch either). Fill in the real release date.
- [ ] **Version number on `dev`.** `aspnet-core/common.props` still says `15.4.0` on `dev` even though 15.4 has been released and the documentation already refers to the current development version as v15.5. Bump it, or confirm that the next release really is 15.4.x.

## Out of scope

Around 80 documents are not referenced by any navigation file (the DevExtreme tutorial series, the MVC 5.x / AngularJS documents, the old RAD Tool guides, `Index-*.md`, `Overview-*.md`, `Security-Report-*.md`, the Xamarin documents, Elsa, DevExpress and the file upload tutorials). They were deliberately left untouched during this review.
