# Logic Table for `permitted` Field

This document describes the logic for setting the `permitted` field in the TherapistDirectAccessEntry backfill process.

## Overview

The `permitted` field is set by negating the result of the `restricted(value)` method:
```ruby
permitted = !restricted(properties["direct_access_restricted"])
```

## Logic Table

| `direct_access_restricted` value | `restricted(value)` returns | `permitted = !restricted(...)` result |
|----------------------------------|----------------------------|---------------------------------------|
| `nil` or empty                   | `false`                    | `true`                               |
| `"yes"` (any case)              | `true`                     | `false`                              |
| `"yes something"` (starts with) | `true`                     | `false`                              |
| `"true"`                        | `true`                     | `false`                              |
| `"1"`                           | `true`                     | `false`                              |
| `"restricted"`                  | `true`                     | `false`                              |
| `"TRUE"` (any case)             | `true`                     | `false`                              |
| `"RESTRICTED"` (any case)       | `true`                     | `false`                              |
| `"no"`                          | `false`                    | `true`                               |
| `"false"`                       | `false`                    | `true`                               |
| `"0"`                           | `false`                    | `true`                               |
| `"unrestricted"`                | `false`                    | `true`                               |
| Any other value                 | `false`                    | `true`                               |

## Summary Logic

**`permitted = true`** (therapist CAN practice) when:
- No restriction data exists
- Value is explicitly "no", "false", "0", or anything else not indicating restriction

**`permitted = false`** (therapist CANNOT practice) when:
- Value starts with "yes" (case-insensitive)
- Value is exactly "true", "1", or "restricted" (case-insensitive)

## Default Behavior

The method defaults to **permissive** (not restricted) unless explicitly told otherwise.