---
paths:
  - "db/migrate/**/*.rb"
  - "app/models/**/*.rb"
---

# UUID Required on All Models

Every model must have a `uuid` column used as the public identifier.

## Migration requirements

- Every table creation migration must include a `uuid` column:
  `t.uuid :uuid, null: false, default: "gen_random_uuid()"`
- Add a unique index on `uuid` for every table.
- If adding a new model to an existing schema that lacks `uuid`, generate
  an `add_column` migration to backfill it before any controller work.

## Model requirements

- Never expose the integer `id` column in URLs, API responses, or
  serializers. Use `uuid` everywhere external-facing.
- Do not override `to_param` to return `uuid` in each model, instead override it in `ApplicationRecord` (see controller rule).

## What NOT to change

- Internal ActiveRecord associations (`belongs_to`, `has_many`) still use
  integer foreign keys for performance. The `uuid` column is for external
  lookup, not for join columns.
