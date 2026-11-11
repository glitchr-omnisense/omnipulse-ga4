# Changelog

## Unreleased (1.x)

### Changed
- Token requests counted: the service account's token request (oauth2.googleapis.com) goes through the provider's guard, checked before it is sent (a
  site's limit naming `token`, the provider's waits) and counted after, in the registry's limiter.
  `Ga4Factory` gives `GoogleToken` the provider's guard (`Config::guard()`, its `guard:` argument): each token request is counted under `token`. GA4's limits count every operation, so they count it too (one request an hour, the stricter side: the OAuth endpoint spends no core tokens). Needs omnisense/contracts 16c8da1 or later (`Config::guard()`; 1568f2c, what the sites'
  vendors hold, has it). Tests/Ga4ProviderTest: a site's limit of 0 tokens a day, the token request never sent.

### Added
- Google Analytics 4's usage policy declared as data: `policy.json` (omnisense/contracts' schema), its terms
  read on 2026-11-10, named in `composer.json` (`extra.omnisense.policy`, and
  `omnisense/contracts` required). The provider implements `DeclaresUsagePolicyInterface` and gives
  the policy to its Capabilities; the factory hands the provider's Backoff to its HTTP calls, so
  that a 429 and its `Retry-After` hold every next call. `Tests/PolicyTest.php`.

### Changed
- `omnireach/ga4` is now `omnipulse/ga4` (2026-11-10: the omnireach family is renamed omnipulse,
  this repository's history kept): the namespace `Omnipulse\Ga4`, requiring `omnisense/omnipulse`;
  `composer.json` replaces `omnireach/ga4`, so that a project never installs both.

