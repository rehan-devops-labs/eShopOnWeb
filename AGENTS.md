# Repository agent guidance

## Repository role

This is Rehan's AZ-400 learning copy of Microsoft's eShopOnWeb ASP.NET Core 8 reference application. It demonstrates a layered monolithic architecture and contains additional Azure DevOps/GitHub Actions labs, Bicep infrastructure, Docker configurations, and course notes.

Treat upstream application architecture and course evidence as reference material. Keep learning-specific changes focused and avoid broad modernization unrelated to the active lab.

## Architecture

- `src/ApplicationCore`: entities, specifications, interfaces, and domain services with no infrastructure dependency.
- `src/Infrastructure`: EF Core persistence, identity, logging, and external implementations.
- `src/Web`: MVC/Razor storefront and composition root.
- `src/PublicApi`: API endpoints used by the admin experience.
- `src/BlazorAdmin` and `src/BlazorShared`: browser admin application and shared contracts.
- `tests/`: unit, integration, and functional test projects.
- `infra/`, `.ado/`, and `.github/workflows/`: Bicep and CI/CD labs.

Preserve inward dependency direction: UI and infrastructure may depend on Application Core, not the reverse.

## Build and test

Use the SDK pinned by `global.json`.

```bash
dotnet restore eShopOnWeb.sln
dotnet build eShopOnWeb.sln --configuration Release --no-restore
dotnet test eShopOnWeb.sln --configuration Release --no-build
```

Use focused project tests while iterating, then run the solution gate. Database/integration tests must use local or disposable resources, never a production Azure database.

## Change rules

- Keep business rules in Application Core and data/provider behavior in Infrastructure.
- Add EF Core migrations deliberately; do not rewrite applied migrations.
- Never commit connection strings, Azure service connections, subscription/tenant IDs, Key Vault values, registry credentials, or generated deployment outputs.
- Keep CI examples internally consistent across trigger, build artifact, environment, and deploy stages.
- Do not change generated or vendored assets merely to reduce diff noise.

## Cloud and deployment safety

`azd up`, Bicep deployment, Azure Pipelines, GitHub deployment workflows, Docker publication, database migration, and lab cleanup can create cost or destroy resources. Do not run or trigger them as ordinary validation. Require explicit operator intent, review the target subscription/resource group, and prefer what-if/dry-run validation first.
