---
name: copilot-path-instructions-maker
description: >
  Generate `.instructions.md` files with glob-based `applyTo` patterns for GitHub Copilot path-scoped
  instructions. Use this skill whenever the user wants to create file-type-specific, folder-specific,
  or language-specific Copilot instructions — for example, different rules for Vue components vs Go
  services, frontend vs backend, test files vs source files, or any scenario where different parts of
  the codebase need different AI behavior. Trigger when the user mentions "path instructions",
  "instructions.md files", "file-specific copilot rules", "per-folder instructions", "applyTo patterns",
  "language-specific AI rules", or asks things like "make Copilot treat my Vue files differently from
  my Go files", "add instructions for test files", or "split my copilot instructions by framework".
  Also trigger when the copilot-instructions-maker skill suggests splitting a large instructions file.
---

# Copilot Path Instructions Maker

Generate targeted `.instructions.md` files that apply different AI rules to different parts of the codebase using glob-based `applyTo` patterns. Where `copilot-instructions.md` sets project-wide rules, path instructions let you say "when working on Vue components, do X" or "when editing Go services, do Y" — giving Copilot the right context for each file type.

## Why path instructions exist

A full-stack project has fundamentally different conventions in different areas. Your Vue components use Composition API with `<script setup>`, your Go services follow stdlib patterns, your test files need specific assertion styles, and your migration files have strict naming rules. Cramming all of this into one `copilot-instructions.md` wastes context tokens on irrelevant rules and can even confuse the AI. Path instructions solve this by activating only when Copilot is working on matching files.

## Interview workflow

Use the `#vscode/askQuestions` tool to present questions as interactive carousels. The interview is shorter than the main instructions maker because it's focused on a specific slice of the codebase.

### Phase 1: Scope identification

Figure out what area of the codebase needs its own instructions.

- What part of the codebase needs specific rules? (frontend components, backend services, tests, docs, migrations, configs, etc.)
- What file extensions and paths are involved? (e.g., `**/*.vue`, `src/services/**/*.go`, `tests/**/*.test.ts`)
- Does this overlap with or extend existing instructions? (check for existing `.github/copilot-instructions.md` or other `.instructions.md` files)

Use `#search/fileSearch` and `#search/listDirectory` to discover the actual file structure rather than guessing. Look at the project tree to suggest appropriate glob patterns.

### Phase 2: Convention discovery

Understand the specific rules for this file scope.

- What framework/library patterns apply here? (Composition API vs Options API, Express middleware style, pytest fixtures, etc.)
- Are there naming conventions specific to this area? (component naming, test naming `*.spec.ts` vs `*.test.ts`, etc.)
- What imports or modules are standard? (auto-imports, barrel files, specific utility modules)
- Are there structural templates these files should follow? (component structure, service class shape, test arrange/act/assert)
- What should Copilot avoid generating in these files? (deprecated patterns, wrong library usage, prohibited approaches)

### Phase 3: Examples

Collect or discover concrete examples that show the conventions in action.

- Use `#search/codebase` to find representative files that exemplify the team's patterns
- Ask the user to point out a "gold standard" file if the codebase search doesn't surface clear patterns
- Extract the patterns that make these files good — structure, naming, import style, error handling

## File format

Each `.instructions.md` file uses YAML frontmatter to control when it activates:

```markdown
---
name: 'Display Name'
description: 'Brief description shown on hover in Chat view'
applyTo: 'glob/pattern/**/*.ext'
---

# Instructions content in Markdown
```

### Frontmatter fields

| Field | Required | Purpose |
|-------|----------|---------|
| `name` | No | Display name in the VS Code UI. Defaults to filename. |
| `description` | No | Shows on hover. Helps the user (and the AI) understand what this file governs. |
| `applyTo` | Yes* | Glob pattern relative to workspace root. Without this, the file won't auto-activate. |

*Technically optional per the spec, but always include it — an instructions file without `applyTo` only works when manually attached, which defeats the purpose.

### Common glob patterns

| Pattern | Matches |
|---------|---------|
| `**/*.vue` | All Vue single-file components |
| `**/*.ts,**/*.tsx` | All TypeScript files (comma-separated for multiple extensions) |
| `src/services/**/*.go` | Go files in the services directory |
| `**/*.test.ts,**/*.spec.ts` | All TypeScript test files |
| `docs/**/*.md` | Markdown files in the docs folder |
| `database/migrations/**` | All migration files |
| `**/*.php` | All PHP files |
| `src/components/**/*.vue` | Vue components in a specific directory |
| `**/*.py` | All Python files |
| `*.config.*,*.rc.*` | Config files at any level |
| `docker-compose*.yml,Dockerfile*` | Docker-related files |

## File organization

Place instruction files in `.github/instructions/` and organize by domain:

```
.github/instructions/
├── frontend/
│   ├── vue-components.instructions.md
│   ├── composables.instructions.md
│   └── styles.instructions.md
├── backend/
│   ├── go-services.instructions.md
│   ├── api-handlers.instructions.md
│   └── database.instructions.md
├── testing/
│   ├── unit-tests.instructions.md
│   └── e2e-tests.instructions.md
└── docs/
    └── documentation.instructions.md
```

VS Code searches `.github/instructions/` recursively, so subdirectories work naturally.

## Writing principles

- **Reference the main instructions.** If there's a `copilot-instructions.md`, link to it: `Apply the [general coding guidelines](../../copilot-instructions.md) to all code.` This avoids duplication.
- **Be surgical.** Path instructions should contain only what's unique to this file scope. If a rule applies everywhere, it belongs in the main instructions file, not repeated in every path file.
- **Lead with structure.** Show the expected shape of a file first (the template/skeleton), then explain individual rules. Developers (and AI) think in terms of "what does a good one look like?" before "what are all the rules?"
- **Include a concrete example.** One well-chosen example of a correctly written file in this scope is more effective than ten abstract rules.
- **Stay under 100 lines per file.** Path instructions are loaded alongside the main instructions and any other matching path files. Token budget matters. If you're going over 100 lines, the scope is probably too broad — split it.

## Generation process

1. **Scan the project** using `#search/listDirectory` and `#search/fileSearch` to understand the file tree
2. **Identify natural instruction boundaries** — areas where conventions differ significantly
3. **Ask the user** which areas need path instructions (suggest based on what you found)
4. **For each area**, run through the Phase 2-3 interview
5. **Generate the files** with proper frontmatter and Markdown content
6. **Cross-check** against the main `copilot-instructions.md` for contradictions or duplication
7. **Present to the user** for review

When generating multiple files in one session, show a summary table first:

```
| File | applyTo | Covers |
|------|---------|--------|
| vue-components.instructions.md | **/*.vue | SFC structure, Composition API, naming |
| go-services.instructions.md | src/services/**/*.go | Service patterns, error handling, stdlib |
| unit-tests.instructions.md | **/*.test.ts,**/*.spec.ts | Test structure, assertions, mocking |
```

Then generate each file and save to `.github/instructions/`.

## Example output

Here's what a well-crafted path instructions file looks like:

```markdown
---
name: 'Vue Component Standards'
description: 'Conventions for Vue 3 single-file components using Composition API'
applyTo: '**/*.vue'
---

# Vue Component Standards

Apply the [general coding guidelines](../../copilot-instructions.md).

## Component structure

Every `.vue` file follows this order:

1. `<script setup lang="ts">` (always TypeScript, always setup)
2. `<template>` (single root element not required in Vue 3)
3. `<style scoped>` (always scoped, use CSS modules for shared styles)

## Script section patterns

- Use `defineProps<{}>()` with TypeScript generics, not runtime declaration
- Use `defineEmits<{}>()` the same way
- Extract reusable reactive logic into `composables/use*.ts` files
- Import order: vue core → vue-router/pinia → project composables → project utils → types

## Naming

- Component files: `PascalCase.vue` (e.g., `UserProfileCard.vue`)
- Composables: `useCamelCase.ts` (e.g., `useUserAuth.ts`)
- Props: `camelCase` in script, `kebab-case` in template attributes

## Avoid

- Options API — all new components use `<script setup>`
- `this` keyword — doesn't exist in `<script setup>`
- `defineComponent()` wrapper — unnecessary with `<script setup>`
- Inline styles — use Tailwind classes or `<style scoped>`
```

## Output

Save files to `.github/instructions/` organized in subdirectories by domain. Create the directory structure if it doesn't exist.

After generating, remind the user:
- Path instructions auto-activate when Copilot works on matching files
- They can check which instructions are active via the Chat Customizations editor
- Instructions files should be committed to version control so the whole team benefits

## Reference

- [VS Code .instructions.md Docs](https://code.visualstudio.com/docs/copilot/customization/custom-instructions#_use-instructionsmd-files)
- Default search location: `.github/instructions/` (configurable via `chat.instructionsFilesLocations`)
- Also compatible with `.claude/rules/` folder format (uses `paths` instead of `applyTo`)
