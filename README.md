# dotnet
Shared functionality across my .NET projects.

## Workflows

Reusable GitHub workflows - each repository keeps a small caller file with its own triggers (`on:`, workflow inputs) and passes in its values.

| Workflow | Inputs | Notes |
|---|---|---|
| `test.yml` | `solution`*, `name`*, `configuration` (`Debug`), `dotnet-versions` (8, 9, 10) | restore, style (`--verify-no-changes`), build, test, then `notify.yml` |
| `prepare.yml` | `solution`*, `dotnet-versions` (8, 9, 10) | restore, style, build `Release`, pack `Pack.csproj`, upload the `nupkgs` artifact |
| `notify.yml` | `name`*, `results`* (pass `${{ toJSON(needs.*.result) }}`), `success-message` (`<name> publish succeeded`), `failure-message` (`<name> publish failed`) | sends the failure message if any job failed or was cancelled, otherwise the success message |
| `workflow-cleanup.yml` | `days` (`60`) | deletes old workflow runs |

\* required. `name` is used for the notification username, e.g. `Wrap (GitHub)`.

```yaml
jobs:
  test:
    uses: bfren/dotnet/.github/workflows/test.yml@main
    with:
      solution: "Wrap.slnx"
      name: Wrap
    secrets: inherit
```

### Publishing to NuGet

NuGet trusted publishing only accepts a login from a workflow file in the package's own repository, so publishing is split:
`prepare` (shared, builds and uploads the `nupkgs` artifact) → `nuget` (in the repository: downloads `nupkgs`, `NuGet/login`, push) → `notify` (shared).
The `nuget` job needs `id-token: write` and `packages: write`, and the repository needs `NUGET_USER` and `SLACK_WEBHOOK` secrets.
