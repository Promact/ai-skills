# Feature Flag Management — `Promact.FeatureFlagService`

Provider-agnostic feature-flag checks. One `IFeatureFlagService` interface,
three independent provider implementations. Source:
`nuget-packages/feature-flag-management/` in
[Promact/reusable-components](https://github.com/Promact/reusable-components).

## Packages
| Provider | PackageId | Notes |
|---|---|---|
| Core (always needed) | `Promact.FeatureFlagService` | Abstractions: `IFeatureFlagService` |
| PostHog | `Promact.FeatureFlagService.PostHog` | Calls the PostHog API live on every check — no local caching |
| LaunchDarkly | `Promact.FeatureFlagService.LaunchDarkly` | Evaluates against a locally cached ruleset |
| Unleash | `Promact.FeatureFlagService.Unleash` | Evaluates against a locally cached ruleset; adds a subscription-level check |

## Public API
```csharp
namespace Promact.FeatureFlag;

public interface IFeatureFlagService
{
    Task<bool> IsFeatureEnabledAsync(string featureName);
}
```
Unleash adds an extra method on its concrete class (inject
`UnleashFeatureFlagService` directly to use it, not the interface):
```csharp
public class UnleashFeatureFlagService : IFeatureFlagService
{
    public Task<bool> IsFeatureEnabledAsync(string featureName);
    public Task<bool> IsFeatureForSubscriptionEnabledAsync(string featureName);
}
```

## DI wiring — pick one provider

**PostHog**
```csharp
services.AddPostHogFeatureFlagService(options =>
{
    options.ApiKey = configuration["PostHog:ApiKey"];
    options.Host = configuration["PostHog:Host"];
    options.ProjectId = int.Parse(configuration["PostHog:ProjectId"]);
});
```
Config: `PostHog:ApiKey`, `PostHog:Host`, `PostHog:ProjectId` (int).

**LaunchDarkly**
```csharp
services.AddLaunchDarklyFeatureFlagService(options =>
{
    options.SdkKey = configuration["LaunchDarkly:SdkKey"];
    options.Environment = configuration["LaunchDarkly:Environment"];
    // Optional — override the default per-request user context:
    // options.UserContextFactory = provider => new LaunchDarklyUserContextDetails { UserKey = ..., Email = ..., Subscription = ... };
});
```
Config: `LaunchDarkly:SdkKey`, `LaunchDarkly:Environment`. Auto-registers
`AddHttpContextAccessor()`. Set `options.Offline = true` for local
dev/tests (no network calls; flags evaluate to default `false`).

**Unleash**
```csharp
services.AddUnleashFeatureFlagService(options =>
{
    options.AppName = configuration["Unleash:AppName"];
    options.Environment = configuration["Unleash:Environment"];
    options.UnleashApi = configuration["Unleash:APIUrl"];
    options.ApiToken = configuration["Unleash:ApiToken"];
    // Optional — override the default subscription context:
    // options.SubscriptionContextFactory = provider => ...;
});
```
Config: `Unleash:AppName`, `Unleash:Environment`, `Unleash:APIUrl`,
`Unleash:ApiToken`. Auto-registers `AddHttpContextAccessor()`.

## Usage
```csharp
using Promact.FeatureFlag;

public class MyClass(IFeatureFlagService featureFlagService)
{
    public async Task DoWork()
    {
        if (await featureFlagService.IsFeatureEnabledAsync("my-feature-flag"))
        {
            // ...
        }
    }
}

// Unleash-only subscription-level check:
public class MyOtherClass(UnleashFeatureFlagService unleashFeatureFlagService)
{
    public Task<bool> IsForSubscriber() =>
        unleashFeatureFlagService.IsFeatureForSubscriptionEnabledAsync("my-feature-flag");
}
```

## Gotchas
- LaunchDarkly and Unleash both build their **default** per-request
  user/subscription context from `HttpContext.User` claims. Outside an HTTP
  request pipeline (background worker, console app), supply
  `options.UserContextFactory` / `options.SubscriptionContextFactory`
  explicitly — otherwise the default falls back to hardcoded placeholder
  values (e.g. `"default@yopmail.com"` / `"Basic"`).
- PostHog hits the live API on **every single check** (no caching) — avoid
  calling it in a hot path without your own caching layer.
- LaunchDarkly/Unleash evaluate against a **local cached ruleset**, so
  despite the `async` signature, `IsFeatureEnabledAsync` is effectively
  synchronous under the hood for those two.
