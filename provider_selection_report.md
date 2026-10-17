# Quota-Management for Providers in Jcode

## Summary
The current selection mechanism for providers in Jcode uses a fixed priority order based solely on configuration. This can lead to scenarios where exhausted providers take precedence over available ones, wasting requests and resulting in poor user experience.

## Problem Location
The affected code resides in `crates/jcode-provider-core/src/selection.rs`, specifically within the function `auto_default_provider` which operates between lines 44 and 60. The function is only informed about provider availability through the `ProviderAvailability` struct which contains basic boolean indicators alongside `copilot_premium_zero`, indicating if a provider is configured but not their capacity or remaining budget.

## How It Impacts Users
A concrete illustration of the problem: If a user has a fully utilized ChatGPT subscription (100% usage) and another with headroom (such as Copilot), the automatic selection favors the exhausted provider every time, leading to unnecessary failed requests.

Cross-provider failover is excellent for dynamically handling provider outages, but does not handle the proactive routing necessary when a provider is already known to be out of quota.

## Solution Proposal
1. Modify the mechanism for provider selection with an explicit order in the Jcode configuration:  
```plaintext
[provider]
# Configure the order of provider selection. The first listed provider is prioritized.
order = ["copilot", "claude", "openai"]
```

2. Update the `auto_default_provider` function such that it returns the first available provider from the configuration list, reverting to default behavior only when the list is unset.

3. **Enhanced Functionality**: Enabling the application to skip providers that are already known to limit for the current session, which the existing failover logic can facilitate with `RetryAndMarkUnavailable`.

## Workaround for Current Users
For immediate relief, users can manually check current usages with `jcode provider list` and `jcode usage` commands before launching. They can choose their provider using `-p <provider>`, as this flag is honored upon session start.

## Conclusion
Keeping the enrichment layer of an order key will guide provider selection more finely and help to avoid the pitfalls of currently available but exceeded quota. This PR can provide a much more streamlined experience for users.

## Additional Context
- The `default_provider` directive has already expressed a preference and can be utilized to manage individual cases; however, it does not represent the desired priority linked to provider headroom. A clearer, explicit order within the configuration can cater to broader use cases effectively.
- As consistency in user experience is vital, this change represents both backwards compatibility and a user-driven improvement. If the change seems reasonable, a pull request can be drafted for review.