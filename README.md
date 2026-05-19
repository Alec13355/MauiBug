# MAUI iOS Build Regression — 10.0.50 → 10.0.60

Minimal reproduction for a build regression introduced in .NET MAUI **10.0.60**
(workload `26.5.10280`) that breaks iOS builds on GitHub Actions.

## Symptom

The same error appears on **every** project — whether or not it uses pinned
workloads — ruling out a local configuration issue.

```
SDK version:              10.0.203 (pinned — same failure occurs on unpinned builds)
Last MAUI workload that worked: 10.0.50
Broken since:             10.0.60 (workload 26.5.10280)
Affected platform:        iOS (net10.0-ios)
Unaffected:               Android (net10.0-android)
```

## How to reproduce

1. Push this repo to GitHub.
2. Go to **Actions → Regression Bisect (10.0.50 vs 10.0.60)** and run it manually.
3. The `10.0.50 (working)` matrix leg passes; the `10.0.60 (broken)` leg fails.

Alternatively, run the individual workflows:
- `build-working.yml` — pins workload via rollback file, passes.
- `build-broken.yml` — installs latest workload (10.0.60), fails.

## Artifacts

Both runs upload a `.binlog` artifact. Open it with
[MSBuild Structured Log Viewer](https://msbuildlog.com) to see the exact
target and task that fails.

## Workaround

Pin the workload to the last known-good version using `--from-rollback-file`:

```yaml
- name: Install MAUI workload (pinned)
  run: |
    dotnet workload install maui-ios \
      --from-rollback-file https://aka.ms/dotnet/10.0.1xx/daily/MauiKnownGoodRollbackFile.json \
      --source https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet10/nuget/v3/index.json \
      --source https://api.nuget.org/v3/index.json
```

## Project structure

```
MauiBugRepro/
  MauiBugRepro.csproj     # net10.0-ios;net10.0-android, plain MAUI template
  MauiProgram.cs
  App.xaml / .cs
  AppShell.xaml / .cs
  MainPage.xaml / .cs
  Platforms/
    iOS/
      AppDelegate.cs
      Program.cs
      Info.plist
    Android/
      MainActivity.cs
      MainApplication.cs
.github/workflows/
  build-working.yml       # pins to 10.0.50 rollback → passes
  build-broken.yml        # latest workload (10.0.60) → fails
  regression-bisect.yml   # matrix: both versions in one run
```
