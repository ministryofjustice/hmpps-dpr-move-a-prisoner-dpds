# hmpps-dpr-move-a-prisoner-dpds

Data Product Definitions (DPR reports) for the services owned by the Move a Prisoner team. This is the
one place for new reports for all of those services, not only one service.

DPD JSON files live under `definitions/`, one folder per service, and are published to S3 by
[`.github/workflows/publish-dpds.yml`](.github/workflows/publish-dpds.yml).

| Folder | Service |
|---|---|
| `definitions/locations/` | Locations inside prison |
| `definitions/property/` | Prisoner Property |
| `definitions/csra/` | Cell Sharing Risk Assessment |

Add a folder when another service gets its first report. Once a folder holds a report, delete its
`.gitkeep`. Only `.json` files are published.

Existing reports in `hmpps-dpr-data-product-definitions` stay there until someone chooses to move them.
A report moved here must first be removed from that repo by the DPR team, using their deletion workflow.

Everything under `definitions/` is published to one S3 folder, `move-a-prisoner/`, and DPR builds each
report id from that folder name and the file's `id`, not from the file's path. So:

- every published report id starts `move-a-prisoner_`, whichever service folder it is in
- a file's `id` must be unique across the whole repo, not just within its folder
- moving a file between folders does not change its report id, but the old copy stays in S3 until it
  is deleted (see [Delete](#delete)), leaving two definitions with the same id

## Publish

Actions tab → **Publish DPDs** → Run workflow → pick environment
(`development`, `test`, `preproduction`, `production`).

The workflow publishes everything in `definitions/`, not a single report, so check what else has
changed before publishing to production. Publishing to production also needs you to confirm that the
reports have been tested in development, test, preproduction and the Tools UI in production.

The reports/dashboards appear in the prisons and probation reporting platforms up to 30 minutes after
publication. The reporting service caches its list of reports for 30 minutes, so a new report returns
an error until that cache expires.

The published report id is the file's `id` prefixed with `move-a-prisoner_`, e.g.
`move-a-prisoner_ors-property-location-history`.

Platform links for the development environment:
https://digital-prison-reporting-mi-ui-dev.hmpps.service.justice.gov.uk/
https://hmpps-probation-mi-ui-dev.hmpps.service.justice.gov.uk

## `experimental/` folders

A service folder can have an `experimental/` subfolder, e.g. `definitions/property/experimental/`, for
throwaway DPDs used to diagnose problems, usually by stripping parts of a real definition until a
failure goes away. They are published to the lower environments like anything else, but the publish
workflow **excludes them from production**.

They are not products. Delete them once the investigation that created them is closed.

| File | Purpose |
|---|---|
| `definitions/locations/experimental/sample-residential-location-occupancy.json` | Working example, copied from the Residential location occupancy report |
| `definitions/csra/experimental/sample-csra-current-levels.json` | Example reading NOMIS, because CSRA data is not yet in the Data Hub. There is no NOMIS CSRA data in the dev datamart, so it returns nothing there |

These two samples are for developers to copy, so keep them to development: choose `development` when publishing.

## Writing a report

- Datamart tables are named `prisons.<service>_<table>`, e.g. `prisons.locationsinsideprison_location`.
  The columns for each service are in
  [digital-prison-reporting-data-contracts](https://github.com/ministryofjustice/digital-prison-reporting-data-contracts/tree/main/prisons/digital-prisons-services).
- The datasource named `datamart` runs on Redshift. Any other name runs on Athena, so the name decides
  which SQL dialect applies. Filter drop-downs are safest on `datamart`.
- Limit reports to the user's caseload with a `row-level` policy, which needs the filtered column in the
  dataset, e.g. `prison_id IN (${caseloads})`.
- Every query is wrapped in a chain of `WITH` clauses, so recursive queries do not work.
- The DPR Tools test rig rejects some keys the schema allows: `type` on report fields and `formula` on
  dashboard measures. Remove them from the copy you upload to the rig, not from the committed file.
- A filter cannot be both `mandatory` and `interactive`. The report needs the value before it runs, so leave
  `interactive` out, and the filter then appears on the request page.
- The test rig refuses any key the DPR library does not recognise, including a mistyped one such as
  `sortorder`, where the correct key is `sortdirection`. Published reports ignore such keys, so a report
  copied from production may need correcting before the rig accepts it.
- To run an uploaded report in the rig, go to
  `https://dpr-tools-ui-dev.hmpps.service.justice.gov.uk/dpr/request-report/report/<file id>/<report id>/filters`.
  The link on the rig's own report card gives "Not Found".
- A query Athena rejects leaves nothing in Athena query history. Look in App Insights (`exceptions` for
  `hmpps-dpr-tools-api`) for the reason.

## Validation 

The DPDs are validated against the following schema:
https://raw.githubusercontent.com/ministryofjustice/hmpps-digital-prison-reporting-data-product-definitions-schema/main/schema/1.0.0/data-product-definition-schema.json

This validation is part of the [`.github/workflows/publish-dpds.yml`](.github/workflows/publish-dpds.yml) Workflow.

If you would like to also validate locally you could run:
```sh
npm ci
curl -fsSL \
  https://raw.githubusercontent.com/ministryofjustice/hmpps-digital-prison-reporting-data-product-definitions-schema/main/schema/1.0.0/data-product-definition-schema.json \
  -o schema.json
SCHEMA_LOCATION=$PWD/schema.json npm run validate
```

## Delete
You can delete published DPDs by running the [`.github/workflows/delete-dpds.yml`](.github/workflows/delete-dpds.yml).
This will remove the DPDs from S3, but they will still remain present in the GitHub repo.

File names are relative to `definitions/`, so include the service folder, e.g.
`property/ors-property-location-history.json`.
