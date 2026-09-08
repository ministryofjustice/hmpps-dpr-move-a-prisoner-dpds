# hmpps-dpr-move-a-prisoner-dpds

Data Product Definitions for move-a-prisoner.

DPD JSON files live under `definitions/` and are published to S3 by
[`.github/workflows/publish-dpds.yml`](.github/workflows/publish-dpds.yml).

## Publish

Actions tab → **Publish DPDs** → Run workflow → pick environment
(`development`, `test`, `preproduction`, `production`).

The reports/dashboards will be available in the prisons and probation reporting platforms around 10 minutes after the publication.
Platform links for the development environment:
https://digital-prison-reporting-mi-ui-dev.hmpps.service.justice.gov.uk/
https://hmpps-probation-mi-ui-dev.hmpps.service.justice.gov.uk

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
