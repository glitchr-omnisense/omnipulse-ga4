# Changelog

## Unreleased (1.x)

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

