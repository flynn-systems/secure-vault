# Vault

A secure document vault, built in public, one architectural decision at a time.

This repo is the companion code for the *[series on Flynn Systems](https://flynnsystems.substack.com/t/secure-vault)* exploring not just how to build production software, but why it's built the way it is: the trade-offs, the shortcuts we don't take, and the ones we do.

Each article corresponds to a tagged [Release](../../releases) in this repo. Check out the tag that matches whatever you're reading rather than `main`, since `main` reflects the current state of the project, which will be ahead of older articles.

| Article | Tag | What it covers |
|---|---|---|
| [Seven Projects You Don't Need Yet](https://flynnsystems.substack.com/p/seven-projects-you-dont-need-yet) | `article-03` | Solution structure: two projects, vertical slice folders, no logic yet |

## Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download) or later
- Any editor with C# support (Visual Studio 2022 17.14+, VS Code with the C# Dev Kit extension, or JetBrains Rider)

Check your SDK version:

```bash
dotnet --version
```

## How this project was scaffolded

For anyone following along who wants to build the same structure from scratch rather than cloning:

```bash
# Create a folder for the solution and projects and change to it
mkdir Vault && cd Vault

# Solution file (uses the new .slnx format, default from .NET 10 SDK onward)
dotnet new sln -n Vault

# API project, minimal APIs explicitly (not controllers), OpenAPI scaffolded in
dotnet new webapi -n Vault.Api --use-minimal-apis

# Test project
dotnet new xunit -n Vault.Tests

# Wire both into the solution
dotnet sln add Vault.Api Vault.Tests

# Let the test project reference the API project
dotnet add Vault.Tests reference Vault.Api

# Standard .NET gitignore, generated rather than hand-written
dotnet new gitignore

# Init git repo
git init

# Scalar UI for browsing the OpenAPI document. Not included by the webapi
# template by default, the template only scaffolds the OpenAPI document
# generator (Microsoft.AspNetCore.OpenApi), the UI is a separate package.
cd Vault.Api
dotnet add package Scalar.AspNetCore
cd ..
```

Then add a `using` directive and the Scalar UI mapping to `Vault.Api/Program.cs`, right next to the existing `MapOpenApi()` call:

```csharp
using Scalar.AspNetCore;

if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference();
}
```

That's the whole scaffold. No Clean Architecture layers, no seven projects, see [Seven Projects You Don't Need Yet](https://flynnsystems.substack.com/p/seven-projects-you-dont-need-yet) for why.

## Running it

```bash
cd Vault.Api
dotnet run
```

The OpenAPI document is served at `/openapi/v1.json` in development. .NET 10's `webapi` template pairs this with [Scalar](https://github.com/scalar/scalar) rather than classic Swagger UI for a browsable API reference, available at `/scalar/v1` when running in development.

Running the tests:

```bash
dotnet test
```

## Project structure

```
Vault.Api/
  Features/
    Documents/
      UploadDocument.cs
      GetDocument.cs
      DeleteDocument.cs
      Document.cs
    Auth/
      Login.cs
      Register.cs
    Sharing/
      CreateShareLink.cs
      RevokeShareLink.cs
  AuditLog/
    AuditEntry.cs
    AuditLogger.cs
  Infrastructure/
    VaultDbContext.cs
    Migrations/
  Program.cs
Vault.Tests/
```

Each feature folder owns its endpoint, request/response types, and handler logic together. No Controllers/Services/Repositories split. See the article for the full reasoning.

## License

MIT. See [LICENSE](LICENSE).