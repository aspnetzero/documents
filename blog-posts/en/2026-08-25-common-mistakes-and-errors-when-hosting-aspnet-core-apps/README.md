# Article Assets

Article: **Common Mistakes and Errors When Hosting ASP.NET Core Apps**

## Cover Image Needed

Suggested visual concepts:
- A browser window showing a generic `HTTP Error 500.30` page next to a terminal showing the real exception
- IIS / server iconography with warning triangles and a .NET logo
- A "20 problems, 20 fixes" style layout

Recommended dimensions: 1200x630px (standard blog/social sharing size)

Save as: `cover.png`

## In-Article Images — Done

Rendered at 2400x* (2x, 1200 CSS px wide), light theme, consistent visual style. Source HTML is disposable; regenerate by re-rendering if edits are needed.

| File | Content |
| --- | --- |
| `images/02-dotnet-list-runtimes.png` | Terminal card: `dotnet --list-runtimes` with the `Microsoft.AspNetCore.App` lines highlighted, plus a caption explaining what their absence means. |
| `images/06-405-method-not-allowed.png` | Request/response pair: `DELETE /api/users/42` answered with `405`, `Allow: GET, HEAD, OPTIONS, TRACE`, and no CORS header. |
| `images/07-webdav-pipeline-diagram.png` | Two-lane pipeline diagram: with WebDAV registered the request dies at the module (405); with it removed it reaches the controller (204). |
| `images/08-reverse-proxy-forwarded-headers-diagram.png` | Browser → proxy (HTTPS) → Kestrel (HTTP) with the `X-Forwarded-*` headers, and a with/without `UseForwardedHeaders` comparison. |
| `images/09-data-protection-key-ring-diagram.png` | Side-by-side: per-instance key rings failing across a load balancer vs. one shared persisted ring. |
| `images/10-cors-console-error.png` | Browser console panel with the full CORS policy error, plus the two origins and what each must match. |
| `images/11-request-size-limits-diagram.png` | A 50 MB upload stopped at the first of three limits (IIS request filtering → Kestrel → FormOptions). |

## In-Article Images — Still Needed

These are real UI screenshots and have to be captured on a Windows machine with IIS. The article references them at these paths.

| File | What to capture |
| --- | --- |
| `images/01-event-viewer-ancm-error.png` | Event Viewer > Windows Logs > Application, an error entry with source `IIS AspNetCore Module V2` selected, exception text visible in the detail pane. |
| `images/03-iis-modules-aspnetcoremodulev2.png` | IIS Manager, server node selected, **Modules** feature open, `AspNetCoreModuleV2` row highlighted. |
| `images/04-app-pool-32bit-setting.png` | IIS Manager > Application Pools > Advanced Settings, **Enable 32-Bit Applications** row highlighted showing `False`. |
| `images/05-sql-server-apppool-login.png` | SSMS Object Explorer > Security > Logins showing a login named `IIS APPPOOL\MyAppPool`. |

Capture at 1200px wide where possible, light theme, PNG. If any of these cannot be captured, the surrounding text stands on its own — remove the image line rather than substituting a mock-up.

## Regenerating the Rendered Images

The HTML sources live in `images/src/`. They share `style.css`. To re-render after an edit (headless Edge, 2x scale):

```bash
EDGE="/c/Program Files (x86)/Microsoft/Edge/Application/msedge.exe"
WD=$(cd images/src && pwd -W)

"$EDGE" --headless=new --disable-gpu --hide-scrollbars --force-device-scale-factor=2 \
  --screenshot="$WD/img08.png" --window-size=1200,570 "file:///$WD/img08.html"
```

Window heights per file: img02 600, img06 430, img07 530, img08 570, img09 660, img10 590, img11 530.
Then copy the result over the matching file in `images/`.
