<!-- ─── How to adapt ────────────────────────────────────────────────── -->
<!-- architecture.md is the *how it's built*. It captures the choices    -->
<!-- that every other layer of the stack assumes — DI shape, MVUX vs.   -->
<!-- MVVM, navigation pattern, where async lives, where platform code   -->
<!-- lives.                                                              -->
<!--                                                                     -->
<!-- It is *not* an exhaustive system diagram. It's the choices a new   -->
<!-- contributor needs to make their first PR without re-deriving them   -->
<!-- from scratch.                                                       -->
<!-- ─────────────────────────────────────────────────────────────────── -->

# FieldKit — Architecture Brief

## One-line summary

A single Uno Platform project, MVUX for app state, Uno.Extensions for hosting/DI/nav/auth, offline-first via SQLite + a sync queue. Targets Windows desktop, iOS, and Android from one codebase.

## Project shape

```
FieldKit/
  FieldKit.csproj             single project, multi-TFM (desktop / ios / android / wasm)
  App.xaml(.cs)               hosting setup, top-level DI registration, theme bootstrap
  GlobalUsings.cs             one place for project-wide usings
  Presentation/
    Pages/                    *Page.xaml + *Page.xaml.cs (binding only)
    Controls/                 reusable XAML controls
    Templates/                ItemTemplates, ControlTemplates
  Models/                     MVUX records and feeds (the "M" in MVUX)
  Services/                   platform-agnostic services + interfaces
  Data/                       SQLite context, repository wrappers
  Sync/                       sync queue + reconciliation
  Platforms/
    Android/                  #if ANDROID code, manifest pieces
    iOS/                      #if IOS code, Info.plist additions
    Windows/                  #if WINDOWS code
  Themes/                     Colors.xaml, Fonts.xaml, Motion.xaml, Spacing.xaml
  Assets/                     fonts, images
```

The single-project model is intentional. We don't split presentation, domain, and data into separate assemblies. The cost of project plumbing is real; the benefit at this scale is not.

## Pattern: MVUX, not MVVM

**Rule:** Default to **MVUX**. Use **MVVM** (CommunityToolkit.Mvvm) only for sample/throwaway pages, and say why in the PR.

Why MVUX is the default here:

- FieldKit is dominated by **async data flows** — paginated inventory, sync queue, push-driven updates. MVUX feeds (`IFeed<T>`, `IListFeed<T>`, `IState<T>`) handle these natively without manual `INotifyPropertyChanged` plumbing.
- `FeedView` renders loading / ready / error / undefined from one binding. The brief on interactions ([`interactions.md`](./interactions.md)) assumes this is how those states get rendered.
- State-as-data fits the offline-first model: the sync layer mutates state, the UI reacts. We never reach into views to "refresh."

Where MVVM still makes sense:

- Single-page samples or admin tools where state is mostly imperative toggles.
- Pages we expect to throw away in a phase or two.

Both libraries are referenced. Don't mix them in the same page.

## Hosting and DI

Uno.Extensions hosting. `App.xaml.cs` builds the host once.

```csharp
public partial class App : Application
{
    public IHost Host { get; private set; } = default!;

    protected override async void OnLaunched(LaunchActivatedEventArgs args)
    {
        var builder = this.CreateBuilder(args)
            .Configure(host => host
                .UseLogging(configure: (context, logBuilder) =>
                    logBuilder.SetMinimumLevel(LogLevel.Information))
                .UseConfiguration(c => c.EmbeddedSource<App>().Section<AppConfig>())
                .UseLocalization()
                .UseAuthentication(auth => auth.AddOidc())
                .UseHttp((context, services) => services
                    .AddTransient<IInventoryApi, InventoryApi>())
                .ConfigureServices(services =>
                {
                    services.AddSingleton<ISyncQueue, SyncQueue>();
                    services.AddSingleton<ILocationProvider, LocationProvider>();
                    services.AddTransient<IJobsService, JobsService>();
                })
                .UseNavigation(RegisterRoutes));

        MainWindow = builder.Window;
        Host = await builder.NavigateAsync<Shell>();
    }
}
```

**Rules:**

- All services register here. No service-locator pattern, no `App.Current.Services` lookups.
- Use `Singleton` for stateful long-lived services (sync queue, location, auth state). `Transient` for thin wrappers and per-call clients. `Scoped` we don't use.
- Configuration via `IOptions<T>` only. No `IConfiguration` reads in business code.

## Navigation

Uno.Extensions region-based navigation. One `ContentControl` host per shell, declared with `uen:Region.Attached="True"`.

Routes are registered in `App.xaml.cs`:

```csharp
private static void RegisterRoutes(IViewRegistry views, IRouteRegistry routes)
{
    views.Register(
        new ViewMap<SignInPage, SignInModel>(),
        new ViewMap<TodayPage, TodayModel>(),
        new DataViewMap<JobDetailPage, JobDetailModel, Job>());

    routes.Register(
        new RouteMap("", View: views.FindByViewModel<ShellModel>(),
            Nested: new[]
            {
                new RouteMap("SignIn", View: views.FindByViewModel<SignInModel>()),
                new RouteMap("Today", View: views.FindByViewModel<TodayModel>(),
                    Nested: new[]
                    {
                        new RouteMap("Job", View: views.FindByViewModel<JobDetailModel>())
                    })
            }));
}
```

**Rules:**

- Navigation always goes through `INavigator`. Don't reach for `Frame.Navigate` or `Window.Content = ...`.
- Pass data via `DataViewMap<TPage, TModel, TData>`. Don't shoehorn through query parameters.
- Deep links resolve to routes, not to views directly. We get this for free from Uno.Extensions.

## Data flow

```
   API (HTTP)              Local SQLite
       │                        │
       ▼                        ▼
   InventoryApi          InventoryRepository
       │                        │
       └────► InventoryService ─┘
                    │
                    ▼
              IListFeed<Part>   (MVUX feed)
                    │
                    ▼
              JobDetailModel
                    │
                    ▼
              FeedView in XAML
```

**Read path.** UI binds to a feed. Feed asks service. Service reads from local repo first; if stale, refreshes from API; emits.

**Write path.** UI calls a command on the model. Model writes via service. Service writes to local repo + enqueues to sync queue. Sync queue drains in the background.

**The UI never knows whether data came from network or local store.** That's the entire point of the offline-first model.

## Sync queue

Single-producer, single-consumer. SQLite-backed `OutboxItem` rows with a state field (`pending`, `inflight`, `done`, `failed`). The `SyncQueue` service:

- Subscribes to network connectivity changes.
- Drains pending items oldest-first when connected.
- Retries with exponential backoff on transient failures.
- Surfaces failure state via an `IState<SyncStatus>` that the chrome's `SyncBadge` binds to.

Conflict policy (Phase 1–2): **last-write-wins server-side.** The reconcile UI in [`ux-flows.md`](./ux-flows.md) is informational, not interactive. We revisit when Phase 3 starts.

## Platform-specific code

`Platforms/` is for things that genuinely cannot be expressed cross-platform.

**Allowed in `Platforms/`:**

- Permissions (camera, location) — different idioms per OS.
- Native auth callbacks (iOS URL handlers, Android intents).
- Background sync registration (foreground service vs. background task).

**Not allowed in `Platforms/`:**

- UI styling. If a control needs a different look per platform, do it with `<win:` / `<android:` XAML namespaces inside one file, not a separate platform implementation.
- "Just easier this way." Two implementations of the same logic is the most expensive code in the project.

When platform code is unavoidable, write the interface in `Services/` and the impls in `Platforms/<OS>/`. Register the right impl in the platform-conditional hosting setup.

## Testing strategy

- **Unit tests** for services and MVUX models. Run on `net10.0` host. Mock external dependencies (API, repository) at the interface boundary.
- **No mocked database** in repository tests. Use a temporary SQLite file. Mocks here have masked migration bugs before.
- **No UI tests in Phase 1.** When Phase 2 lands, we add Uno UI Test on the desktop target only — covers ~95% of regressions at 5% of the cost of three-platform UI tests.

## Build and CI

- One `dotnet build` should be enough for any target combination. If it isn't, the build is broken.
- CI runs `dotnet build` for desktop and Android (cheapest two platforms). iOS builds nightly on a Mac runner.
- We don't gate merges on UI tests. We gate on the `Verification checklist` in [`CLAUDE.md`](./CLAUDE.md).

## What's not in this file

- **Visual styling and tokens** — see [`design.md`](./design.md).
- **Motion timings and state machines** — see [`interactions.md`](./interactions.md).
- **What we're building when** — see [`plan.md`](./plan.md).
- **Decisions log** — lives in [`CLAUDE.md`](./CLAUDE.md) until it earns its own file.
