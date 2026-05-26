---
paths:
  - "app/controllers/**/*.rb"
---

Do not define empty controller action methods. Rails implicit rendering
will handle actions that only need to render their default view template.
Before actions still run without an explicit method definition.
