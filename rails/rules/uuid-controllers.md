---
paths:
  - "app/controllers/**/*.rb"
---

# UUID-Based Record Lookups

All record lookups in controllers must use the `uuid` column, never the
database `id`. Integer primary keys must never be exposed to clients.

## Prerequisite: override `to_param`

If not already present, add the following to `ApplicationRecord`:

```ruby
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true

  def to_param
    uuid
  end
end
```

This ensures all route helpers, `redirect_to`, `link_to`, and URL
generation use the UUID automatically. Do **not** add a fallback like
`uuid || super` — if a model is missing a `uuid` column, it should fail
loudly rather than silently expose an integer ID.

## How to look up records

- Use `find_by!(uuid: params[:id])` (or the equivalent scoped query) in
  place of `find(params[:id])`.
- For nested resources, apply the same pattern:
  `@parent.children.find_by!(uuid: params[:id])`.

## What to avoid

- **Never call `Model.find(params[:id])`** — this queries by integer
  primary key, not by `uuid`.
- **Never expose integer IDs** in routes, redirects, or response bodies.
  With `to_param` overridden this happens automatically for URL helpers,
  but take care in JSON serializers and manual string interpolation —
  always use `record.uuid`.
- **Do not override `to_param` on individual models** — the base class
  handles it. If a model needs a different slug strategy, discuss with
  the team first.