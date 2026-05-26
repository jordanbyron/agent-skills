---
paths:
  - "app/controllers/**/*.rb"
---

# Name lookup callbacks `find_*`, not `set_*`

Before-action callbacks that look up a record should be named `find_child`,
`find_user`, etc. — not `set_child` or `set_user`. The method's job is to
find a record from the database; the name should say what it does.
