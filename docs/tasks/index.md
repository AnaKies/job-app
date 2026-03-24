# Delivery Index

This directory contains the delivery package for implementing the approved CeeVee architecture.

## Documents

- [overview.md](./overview.md)
  Primary delivery sequence, feature goals, role-specific task list, dependencies, and parallelization guidance.

- [user-tests.md](./user-tests.md)
  Short manual user tests for visible feature outcomes.

- [task-briefs/f1-f4.md](./task-briefs/f1-f4.md)
  Detailed briefs for Features F1 to F4.

- [task-briefs/f5-f8.md](./task-briefs/f5-f8.md)
  Detailed briefs for Features F5 to F8.

- [task-briefs/f9-f12.md](./task-briefs/f9-f12.md)
  Detailed briefs for Features F9 to F12.

## Delivery Rules

- `overview.md` is the primary execution map.
- Task briefs are authoritative for task-level scope and done criteria.
- `user-tests.md` is the manual validation reference for visible product states.
- Features are ordered by dependency, but tasks inside a feature should run in parallel whenever the documented dependencies allow it.
