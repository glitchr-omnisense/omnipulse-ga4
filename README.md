# omnipulse/ga4

Formerly `omnireach/ga4` (renamed on 2026-11-10 with its family: `Omnireach\` is now `Omnipulse\`;
this package replaces the former name).

[omnisense/omnipulse](https://github.com/glitchr-omnisense/omnipulse)'s Google Analytics 4, under
any PHP host: visitors, sessions, pageviews, bounce rate and session length of a property, by
landing page, page, source, country, device or day, combined as needed.

## Install and a first call

```sh
composer require omnipulse/ga4
```

No account for this first call: the Analytics Data API's answer is recorded
(`Omnipulse\Testing\RecordedHttpClient` answers in place of the API), so nothing leaves the process.
A real call needs a Google Cloud service account given the Viewer role on the GA4 property (its JSON
key as `credentials`), or an access token obtained elsewhere.

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Omnipulse\Ga4\Ga4Factory;
use Omnipulse\Model\{Dimension, Metric, ReachQuery};
use Omnipulse\Testing\RecordedHttpClient;

// the Analytics Data API's answer, recorded: nothing sent. HttpClient::create() (symfony/http-client) for a real call.
$http = RecordedHttpClient::routes([
    'POST /properties/123456789:runReport' => '{"dimensionHeaders": [{"name": "landingPage"}], "metricHeaders": [{"name": "totalUsers", "type": "TYPE_INTEGER"}, {"name": "sessions", "type": "TYPE_INTEGER"}], "rows": [{"dimensionValues": [{"value": "/"}], "metricValues": [{"value": "1840"}, {"value": "2210"}]}]}',
]);
$ga4 = (new Ga4Factory($http))->create(['property' => '123456789', 'access_token' => 'recorded']);

$report = $ga4->audience(new ReachQuery(new DateTimeImmutable('2026-10-01'), new DateTimeImmutable('2026-10-31'), [Dimension::EntryPage], [Metric::Visitors, Metric::Visits]));
echo json_encode($report->rows[0]->dimensions + $report->rows[0]->metrics, JSON_UNESCAPED_SLASHES), "\n";   // {"entry_page":"/","visitors":1840,"visits":2210}
```

With your credentials, through the family's registry:

```php
use Omnipulse\Ga4\Ga4Factory;
use Omnipulse\Registry;
use Symfony\Component\HttpClient\HttpClient;

$registry = new Registry([new Ga4Factory(HttpClient::create())], [
    'ga' => ['factory' => 'ga4', 'options' => ['property' => '123456789', 'credentials' => '/secrets/service-account.json']],
]);
```

Options: `property` (the numeric property id, not the `G-` measurement id), `credentials` or
`access_token` (scope `analytics.readonly`; the service account given the Viewer role on the
property), `url`.

## The API

`POST properties/{id}:runReport` of the Data API v1beta
(<https://developers.google.com/analytics/devguides/reporting/data/v1>): `dateRanges`,
`dimensions` (`landingPage`, `pagePath`, `sessionSource`, `countryId`, `deviceCategory`,
`date`), `metrics` (`totalUsers`, `sessions`, `screenPageViews`, `bounceRate`,
`averageSessionDuration`), `dimensionFilter` (EXACT string filters, an `andGroup` for
several), `limit` (250,000 at most) and `offset`. Values come as strings and are typed here;
dates `YYYYMMDD` become Y-m-d.

**Terms and limits** (<https://developers.google.com/analytics/devguides/reporting/data/v1/quotas>):
token quotas per property (per day, per hour, concurrent requests); rows may be thresholded
(Google signals) or grouped under "(other)" on high-cardinality reports.

**Google credentials**: a service account's JSON key (`credentials`: the JSON or the path of
its file) - the token is asked with a JWT signed by its key (RS256, the OAuth 2.0 JWT bearer
grant) and kept until a minute before it expires - or an `access_token` obtained elsewhere.
`GoogleToken` is the same class in omnipulse/search-console and omnipulse/ga4.

**Reuse**: Google's PHP libraries were set aside: `google/apiclient` with
`google/apiclient-services` (every Google API generated, a very large package, its own Guzzle
stack), `google/analytics-data` (gRPC and protobuf through google/gax), `google/auth` (Guzzle
again). The family injects the application's HTTP client and needs one JSON call per report and
one signed JWT, done here with ext-openssl.

## Usage policy

Google Analytics 4's usage policy is declared as data in [`policy.json`](policy.json) (omnisense/contracts'
schema, named in `composer.json`'s `extra.omnisense.policy`), its terms read on
2026-11-10 (https://developers.google.com/analytics/devguides/reporting/data/v1/quotas): 200,000 core tokens a day and 40,000 an hour per property, 14,000 an hour per project per property (written as `units`, counted one per report: a lower bound), 120 potentially thresholded requests an hour (written over every report, the stricter form), 10 concurrent requests; rights unknown; OAuth credentials. omnipulse's registry enforces it - a call over a
limit is never made, a 429 and its `Retry-After` hold every next call
(`UsageLimitReachedException`) - and a site may tighten it, never loosen it
(omnisense/omnipulse's docs/policies.md).

## Verified, and not

- Verified (the core's harness, PHP 8.4): landing pages with a filter, a daily history with two
  filters (`andGroup`) in date order, what GA4 cannot answer here, a refused property, the
  factory with a service account (a key made by the test) - against answers composed in the shape
  the runReport reference documents, not recorded; every class loading with the package's
  requirements alone (`BareTest`).
- Not verified live: no Google account was used.

## Licence

MIT. An unofficial client, affiliated with Google in no way.
