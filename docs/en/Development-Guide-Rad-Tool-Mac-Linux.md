# ASP.NET Zero Power Tools Cross-Platform Usage

This legacy cross-platform RAD Tool document has been replaced by the new Power Tools global tool documentation.

Power Tools now uses the same .NET global tool and web UI flow on Windows, macOS, and Linux. Older platform-specific RAD Tool workflows are no longer needed for normal usage.

## Install and Run

Run the install command from the ASP.NET Zero `aspnet-core` folder so the project-provided `NuGet.Config` is used:

```bash
dotnet tool install -g AspNetZero.PowerTools
```

Then launch Power Tools:

```bash
aspnetzero-powertools
```

For headless generation from an entity JSON file, see [Entity JSON Reference](Power-Tools-Creating-Entity-Json-File-Manually.md).

## Current Documentation

* [Power Tools Getting Started](Power-Tools-Getting-Started.md)
* [Using Power Tools Web UI](Power-Tools-Using-Web-UI.md)
* [Entity JSON Reference](Power-Tools-Creating-Entity-Json-File-Manually.md)
