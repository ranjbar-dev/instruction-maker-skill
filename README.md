# Instruction Maker Skills for GitHub Copilot

Two Claude.ai skills that generate tailored [GitHub Copilot custom instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions) for any project — through an interactive interview, not guesswork.

## What's in here

```
instruction-maker-skill/
├── copilot-instructions-maker/     # Generates .github/copilot-instructions.md
│   └── SKILL.md
├── copilot-path-instructions-maker/ # Generates .github/instructions/*.instructions.md
│   └── SKILL.md
└── README.md
```

### Copilot Instructions Maker

Generates a project-wide `.github/copilot-instructions.md` file. This is the single file that GitHub Copilot reads on every chat request to understand your project's conventions.

**What it does:**

Runs a 5-phase interactive interview using VS Code's `#vscode/askQuestions` tool to gather your project's stack, conventions, patterns, security rules, and AI-specific preferences — then produces a comprehensive instructions file with concrete code examples and explanations.

**Phases:** Project identity → Stack & tooling → Conventions & patterns → Security guardrails → AI-specific rules

### Copilot Path Instructions Maker

Generates targeted `.instructions.md` files with glob-based `applyTo` patterns under `.github/instructions/`. These activate only when Copilot is working on matching files.

**What it does:**

Scans your project structure using `#search/fileSearch` and `#search/listDirectory`, identifies natural instruction boundaries (frontend vs backend, different languages, test files vs source), interviews you about scope-specific conventions, and generates organized instruction files by domain.

**Example output structure:**

```
.github/instructions/
├── frontend/
│   ├── vue-components.instructions.md    # applyTo: **/*.vue
│   └── composables.instructions.md       # applyTo: **/composables/**/*.ts
├── backend/
│   └── go-services.instructions.md       # applyTo: **/*.go
└── testing/
    └── unit-tests.instructions.md        # applyTo: **/*.test.ts,**/*.spec.ts
```

## Installation

### Option 1: Install `.skill` files (recommended)

Download the `.skill` files from [Releases](https://github.com/ranjbar-dev/instruction-maker-skill/releases) and install them in Claude.ai or Claude Code.

### Option 2: Copy into your skills directory

Clone the repo and copy the skill folders into your skills directory:

```bash
git clone https://github.com/ranjbar-dev/instruction-maker-skill.git

# For Claude Code — copy to your user skills
cp -r instruction-maker-skill/copilot-instructions-maker ~/.claude/skills/
cp -r instruction-maker-skill/copilot-path-instructions-maker ~/.claude/skills/

# For Claude.ai — upload the SKILL.md files as user skills
```

## Usage

### Generating project-wide instructions

Ask Claude something like:

- *"Set up copilot instructions for my project"*
- *"Create a copilot-instructions.md for this repo"*
- *"Make Copilot understand my coding conventions"*
- *"Generate AI instructions for my workspace"*

The skill will walk you through the interview and produce `.github/copilot-instructions.md`.

### Generating path-specific instructions

Ask Claude something like:

- *"Create path instructions for my Vue components and Go services"*
- *"Add file-specific Copilot rules for test files"*
- *"Split my copilot instructions by framework"*
- *"Make Copilot treat my frontend and backend code differently"*

The skill will scan your project, suggest instruction boundaries, and generate files under `.github/instructions/`.

## How it works

Both skills leverage VS Code Copilot's built-in tools during the interview:

| Tool | Purpose |
|------|---------|
| `#vscode/askQuestions` | Present questions as interactive carousels for fast answers |
| `#search/fileSearch` | Discover actual file structure for accurate glob patterns |
| `#search/listDirectory` | Understand project organization |
| `#search/codebase` | Find representative code examples to extract patterns from |

The generated instructions follow VS Code's [custom instructions format](https://code.visualstudio.com/docs/copilot/customization/custom-instructions) and are compatible with GitHub Copilot in VS Code, GitHub Copilot CLI, and AGENTS.md-compatible tools.

## Requirements

- Claude.ai or Claude Code with skills support
- VS Code with GitHub Copilot extension (for the generated instructions to take effect)

## License

MIT
