---
name: copilot-instructions-maker
description: >
  Generate a comprehensive `.github/copilot-instructions.md` file for any project by interviewing
  the user about their stack, conventions, and workflow. Use this skill whenever the user asks to
  create, generate, set up, or configure GitHub Copilot instructions, copilot-instructions.md,
  repo-level AI instructions, or mentions wanting Copilot to understand their project conventions.
  Also trigger when the user says things like "set up copilot for my project", "make copilot follow
  my coding style", "configure AI instructions", "create AGENTS.md", or "I want Copilot to know
  how we code". Even if the user just says "copilot instructions" — use this skill.
---

# Copilot Instructions Maker

Generate a tailored `.github/copilot-instructions.md` file by understanding the user's project through an interactive interview. The goal is a single, comprehensive instructions file that makes GitHub Copilot (and other AI agents) produce code that fits the project like a glove — right conventions, right patterns, right libraries, first try.

## Why this matters

Without repo-level instructions, Copilot guesses. It might use `moment.js` when the team uses `date-fns`, generate class components when the project is all hooks, or use `var` in a strict TypeScript codebase. A good `copilot-instructions.md` eliminates this guesswork and saves every developer on the team from repeating themselves in every prompt.

## Interview workflow

Use the `#vscode/askQuestions` tool to present questions as interactive carousels so the user can tap answers instead of typing essays. This keeps the session fast and focused. Ask in batches of 2-3 questions, not all at once. Adapt follow-up questions based on answers.

### Phase 1: Project identity

Understand what this project *is* before asking how it should be built.

- What kind of project is this? (web app, API, CLI, library, monorepo, mobile app, etc.)
- What's the primary language/runtime? (TypeScript, Python, Go, Rust, PHP, Java, etc.)
- What frameworks are in use? (React, Vue, Nuxt, Next.js, Laravel, Express, FastAPI, Spring, etc.)
- Is this a monorepo or single-package project?

### Phase 2: Stack and tooling

Drill into the specifics based on Phase 1 answers.

- Package manager (npm, pnpm, yarn, bun, composer, pip, go modules, etc.)
- Build tools (Vite, Webpack, esbuild, Turbopack, etc.)
- Database and ORM (PostgreSQL + Prisma, MySQL + Eloquent, MongoDB + Mongoose, etc.)
- Testing frameworks (Vitest, Jest, pytest, Go testing, PHPUnit, etc.)
- Linting/formatting (ESLint, Prettier, Biome, gofmt, PHP-CS-Fixer, Ruff, etc.)
- CI/CD platform (GitHub Actions, GitLab CI, Jenkins, etc.)
- Deployment target (Vercel, AWS, Hetzner, Docker/Compose, Kubernetes, etc.)

### Phase 3: Conventions and patterns

These are the rules that make or break AI-generated code quality.

- Naming conventions (PascalCase components, camelCase functions, snake_case DB columns, etc.)
- File organization patterns (feature-based, layer-based, domain-driven, etc.)
- Import style (absolute vs relative, barrel exports, path aliases like `@/` or `~/`)
- Error handling approach (Result types, try/catch patterns, error boundaries, custom error classes)
- State management approach (composables, stores, hooks, context, Redux, Pinia, etc.)
- API patterns (REST conventions, GraphQL schema style, RPC patterns)
- Preferred libraries the team has standardized on (and libraries to avoid)

### Phase 4: Security and quality guardrails

- Any security requirements? (input validation, output encoding, auth patterns, secrets handling)
- Code review standards? (PR size limits, required tests, documentation requirements)
- Performance considerations? (bundle size limits, lazy loading requirements, caching strategies)

### Phase 5: AI-specific instructions

- Should Copilot explain its reasoning in comments?
- Should it prefer existing patterns from the codebase or suggest improvements?
- Any files or directories Copilot should never modify?
- Preferred commit message format?

## Generating the instructions file

After gathering answers, generate a `.github/copilot-instructions.md` that follows this structure:

```markdown
# Project Copilot Instructions

## Project overview
<!-- One paragraph: what this is, primary language, key frameworks -->

## Tech stack
<!-- Concise list of the exact versions/tools in use -->

## Code conventions
### Naming
### File organization
### Imports and modules
### Error handling

## Preferred patterns
<!-- Show concrete code examples of the RIGHT way to do things -->

## Avoid
<!-- Explicit list of anti-patterns, deprecated libraries, wrong approaches -->

## Security
<!-- Non-negotiable security rules -->

## Testing
<!-- How tests should be structured, what to test, naming conventions -->

## Git and workflow
<!-- Commit message format, branch naming, PR conventions -->
```

### Writing principles

- **Be specific, not generic.** "Use `date-fns` for date manipulation" beats "Use appropriate date libraries." Include actual function names, actual file paths, actual config values.
- **Show, don't just tell.** Include short code examples for non-obvious conventions. A 3-line example of the correct import style is worth more than a paragraph explaining it.
- **Explain the why.** "Use `composables/` for shared Vue logic (not `utils/`) because composables signal Vue-specific reactivity" helps the AI generalize to new situations.
- **Keep it under 300 lines.** Copilot instructions are always-on context — every line costs tokens on every request. Be ruthless about what earns a spot. Move detailed framework-specific rules to `.instructions.md` path files instead (suggest this to the user if the file is getting long).
- **Use the `applyTo: "**"` frontmatter** at the top so it explicitly applies to all files.

### Before finalizing

Read the generated file back with fresh eyes. Check for:
- Contradictions (e.g., "use functional components" in one section and a class component example elsewhere)
- Vagueness that could be made specific
- Missing obvious conventions (if they mentioned React but there's no JSX/TSX guidance)
- Anything that duplicates what a linter already enforces (remove it — it wastes context)

Ask the user to review the draft. Offer to adjust sections, add examples, or split overly long sections into separate `.instructions.md` path files.

## Output

Save the file to `.github/copilot-instructions.md` in the workspace root. If a `.github/` directory doesn't exist yet, create it.

If the instructions file exceeds ~250 lines, suggest splitting framework-specific or language-specific sections into separate `.instructions.md` files under `.github/instructions/` and mention the **copilot-path-instructions-maker** skill for that.

## Reference

- [VS Code Custom Instructions Docs](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- The `copilot-instructions.md` file is automatically picked up by VS Code Copilot for all chat requests in the workspace
- It also works with GitHub Copilot in other IDEs and with AGENTS.md-compatible tools
