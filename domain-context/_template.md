---
# DOMAIN CONTEXT TEMPLATE
# Copy this file, rename it (e.g., "api-layer.md"), and fill in the sections below.
# Files starting with "_" are ignored during trigger matching.

name: "Your Domain Name"          # Display name (shown in context pack header)
triggers:
  paths:                           # Path prefixes that activate this context
    - "src/example/"               # Include when plan/review touches these paths
    - "src/other-example/"
  keywords:                        # Keywords that activate this context
    - "example-keyword"            # Include when these words appear in the subject
    - "another-keyword"            #   (case-insensitive matching)
---

<!-- DELETE EVERYTHING ABOVE THE --- AND REPLACE WITH YOUR CONTENT -->

## Overview

<!-- 2-3 sentences: What does this part of the codebase do? -->

## Key Patterns

<!-- List the architectural patterns, conventions, or rules that a reviewer -->
<!-- should know about when looking at code in this domain. -->

- **Pattern:** Description
- **Pattern:** Description

## File Locations

<!-- Map the key files/directories in this domain. -->

| Path | Purpose |
|------|---------|
| `src/example/` | What lives here |
| `src/other/` | What lives here |

## Notes

<!-- Gotchas, known tech debt, or anything else a reviewer should watch for. -->
