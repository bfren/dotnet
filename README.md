# dotnet
Shared functionality across my .NET projects.

## Workflows

Reusable GitHub workflows - each repository keeps a small caller file with its own triggers (`on:`, workflow inputs) and passes in its values.

| Workflow | Inputs | Notes |
|---|---|---|
| `test.yml` | `solution`*, `name`*, `configuration` (`Debug`), `dotnet-versions` (8, 9, 10) | restore, style (`--verify-no-changes`), build, test, Slack |
| `publish-nuget.yml` | `solution`*, `name`*, `dotnet-versions` (8, 9, 10), `dry_run` (`false`) | build `Release`, pack `Pack.csproj`, push to NuGet (trusted publishing) and GitHub Packages, Slack |
| `workflow-cleanup.yml` | `days` (`60`) | deletes old workflow runs |

\* required. `name` is used for the Slack username, e.g. `Wrap (GitHub)`.

```yaml
jobs:
  test:
    uses: bfren/dotnet/.github/workflows/test.yml@main
    with:
      solution: "Wrap.slnx"
      name: Wrap
    secrets: inherit
```

Callers of `publish-nuget.yml` must grant `contents: read`, `id-token: write` and `packages: write` on the calling job, and need `NUGET_USER` and `SLACK_WEBHOOK` secrets.
