# Bump Project Guide Index

## Overview
This directory contains implementation guides, patterns, and troubleshooting information for the Bump application. Use this index to navigate to the appropriate guide based on your current task or milestone.

## Directory Structure

```
.cursorguides/
├── 00-index.md                    # This file - navigation and overview
├── concepts/                      # High-level architectural patterns
│   ├── auth-flow.md               # Authentication flow concepts
│   ├── realtime-patterns.md       # Realtime data patterns
│   └── testing-strategy.md        # Overall testing approach
├── implementations/               # Milestone-specific implementation guides
│   ├── 01-initialize-project.md   # Project initialization
│   ├── 02-supabase-integration.md # Supabase backend setup
│   ├── 03-home-checkin.md         # Home page and check-in feature
│   └── ...                        # Additional milestone guides
├── troubleshooting/               # Common issues and solutions
│   ├── supabase-testing.md        # Supabase testing challenges
│   └── typescript-errors.md       # Common TypeScript issues
└── templates/                     # Reusable code patterns
    ├── component-template.md      # Component structure template
    ├── test-template.md           # Test file template
    └── hook-template.md           # Custom hook template
```

## Milestone Guides

| Milestone | Guide | Status | Description |
|-----------|-------|--------|-------------|
| 1 | [Initialize Project](./implementations/01-initialize-project.md) | ✅ Complete | Project setup, routing, theme system |
| 2 | [Supabase Integration](./implementations/02-supabase-integration.md) | ⏳ In Progress | Backend setup, authentication, database schema |
| 3 | Home & Check-In | 🔲 Planned | Home page with status list and check-in |
| 4 | Gathering Places | 🔲 Planned | Places page with list/map toggle |
| 5 | Friends & Intent | 🔲 Planned | Friends page with connections and intent |
| 6 | Meetups | 🔲 Planned | Meetup logging and history |
| 7 | Settings | 🔲 Planned | User preferences |
| 8 | Notifications | 🔲 Planned | Notification functionality |
| 9 | Polish & Deploy | 🔲 Planned | Final touches and deployment |

## Concept Guides

| Guide | Description |
|-------|-------------|
| [Auth Flow](./concepts/auth-flow.md) | Email OTP authentication flow and session management |
| [Realtime Patterns](./concepts/realtime-patterns.md) | Patterns for realtime data with Supabase |
| [Testing Strategy](./concepts/testing-strategy.md) | Overall testing approach and patterns |

## Troubleshooting Guides

| Guide | Description |
|-------|-------------|
| [Supabase Testing](./troubleshooting/supabase-testing.md) | Solutions for common Supabase testing challenges |
| [TypeScript Errors](./troubleshooting/typescript-errors.md) | Resolving common TypeScript errors |

## Template Guides

| Guide | Description |
|-------|-------------|
| [Component Template](./templates/component-template.md) | Template for creating new components |
| [Test Template](./templates/test-template.md) | Template for writing tests |
| [Hook Template](./templates/hook-template.md) | Template for creating custom hooks |

## How to Use These Guides

1. **Starting a new milestone**: Begin with the corresponding implementation guide in the `implementations/` directory.
2. **Understanding architecture**: Refer to concept guides in the `concepts/` directory.
3. **Troubleshooting issues**: Check the `troubleshooting/` directory for solutions to common problems.
4. **Creating new code**: Use templates from the `templates/` directory for consistent structure.

## Contributing to Guides

When updating these guides:

1. **Keep implementation details separate from concepts**: Implementation guides should focus on specific tasks, while concept guides should explain the underlying patterns.
2. **Document challenges and solutions**: Add troubleshooting entries when you encounter and solve issues.
3. **Extract reusable patterns**: When you identify a pattern that could be reused, add it to the templates directory.
4. **Update the index**: Keep this index file up to date with new guides and status changes.

## Next Steps

After reviewing this index, proceed to the guide that corresponds to your current task or milestone. 