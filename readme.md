# AI-DLC Rules

Shared AI-Driven Development Life Cycle (AI-DLC) workflow rules for all projects in this organization.

Based on [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows) — an intelligent software development workflow that adapts to your needs, maintains quality standards, and keeps you in control.

---

## What This Is

These rules guide Claude Code through a structured 3-phase development workflow:

- **Inception** — requirements gathering, user stories, application design, risk assessment
- **Construction** — component design, code generation, testing, QA
- **Operations** — deployment, monitoring *(coming soon)*

The rules live here as a single source of truth and are pulled into each project via Git submodule. Update once, propagate everywhere.

---

## Repository Structure

```
ai-dlc-rules/
├── README.md                        ← you are here
├── CLAUDE.md                        ← core AI-DLC workflow rules (do not edit)
└── .aidlc-rule-details/
    ├── common/                      ← rules shared across all phases
    ├── inception/                   ← requirements & design rules
    ├── construction/                ← implementation & testing rules
    ├── operations/                  ← deployment rules
    └── extensions/
        ├── security/
        │   └── baseline/            ← baseline security rules
        └── jira/
            └── jira-integration.md  ← Jira-driven workflow (requires Atlassian MCP)
```

---

## Adding to a New Project

### 1. Add the submodule

```bash
cd /path/to/your-project
git submodule add git@bitbucket.org:your-org/ai-dlc-rules.git .ai-dlc
```

### 2. Create a `CLAUDE.md` at your project root

```markdown
# [Project Name] — AI-DLC Workflow

@.ai-dlc/CLAUDE.md

## Project Context
- Stack: [e.g. Laravel, Vue, MySQL]
- Standards: [e.g. PSR-12]
- Tests: [e.g. PHPUnit]
- Deploy: [e.g. your deploy process]

## Project-Specific Conventions
- [Add anything Claude should always know about this project]
```

### 3. Commit

```bash
git add .gitmodules .ai-dlc CLAUDE.md
git commit -m "Add AI-DLC workflow via submodule"
git push
```

### 4. Verify

Open Claude Code in your project terminal and ask:
> *"What instructions are currently active in this project?"*

---

## Adding to an Existing Project That Already Has a `CLAUDE.md`

If your project already has a `CLAUDE.md` with project-specific context, **do not overwrite it**. Instead, prepend the submodule import at the top so the shared rules load first, followed by your existing content.

> ⚠️ **Production repos:** Never commit directly to `master`. Always create a setup branch and raise a PR so the change is reviewed like any other.

### 1. Create a setup branch

```bash
git checkout -b feature/add-ai-dlc-workflow
```

### 2. Add the submodule

```bash
git submodule add git@bitbucket.org:clearlinkit/ai-dlc-rules.git .ai-dlc
git submodule update --init --recursive
```

### 3. Add `.claude/` and `aidlc-docs/` to `.gitignore`

```bash
echo ".claude/" >> .gitignore
echo "aidlc-docs/" >> .gitignore
```

### 4. Prepend the import to your existing `CLAUDE.md`

Open your existing `CLAUDE.md` and add this as the very first line:

```markdown
@.ai-dlc/CLAUDE.md

---

# [Your existing CLAUDE.md content below — unchanged]
```

### 5. Commit and push the setup branch

```bash
git add .gitmodules .ai-dlc CLAUDE.md .gitignore
git commit -m "Add AI-DLC workflow via submodule"
git push origin feature/add-ai-dlc-workflow
```

### 6. Raise a PR

Create a PR from `feature/add-ai-dlc-workflow` → `master` and get it reviewed before merging.

---

## How Teammates Clone a Project Using This

After cloning any project that uses this submodule, run:

```bash
git clone git@bitbucket.org:your-org/your-project.git
cd your-project
git submodule update --init --recursive
```

> **Tip:** Add this to your project's `Makefile` or setup script so teammates don't have to remember it.

---

## How to Use AI-DLC in Claude Code

**Standard — describe what you want to build:**
> *"Using AI-DLC, [describe what you want to build]"*

**Jira-driven — paste a ticket URL and let the ticket be the requirements:**
> *"Using AI-DLC, implement https://clearlink.atlassian.net/browse/FUSE-123"*

Claude Code will activate the workflow and guide you through structured phases automatically. When a Jira URL is provided, it fetches the ticket via the Atlassian MCP, maps the acceptance criteria to the Inception phase, and skips the manual requirements questionnaire.

> **Prerequisite for Jira:** The Atlassian MCP must be configured in Claude Code. See [Jira Integration](#jira-integration) below.

---

## Updating the Rules

When rules in this repo are updated:

```bash
# Step 1 — In this repo, make your changes and push
git add .
git commit -m "Update: [describe what changed]"
git push

# Step 2 — In each project, pull the latest rules
cd /path/to/your-project
git submodule update --remote .ai-dlc
git add .ai-dlc
git commit -m "Update AI-DLC rules to latest"
git push
```

---

## Adding Custom Extensions

Extensions let you layer additional rules (security, compliance, org policies) on top of the core workflow.

1. Create a folder under `.aidlc-rule-details/extensions/your-extension-name/`
2. Add a markdown file following the same format as `security/baseline/security-baseline.md`
3. Include an **Applicability Question** so Claude asks whether to enforce it per project
4. Commit and push — all projects will pick it up on their next `git submodule update`

---

## Jira Integration

The Jira extension (`.aidlc-rule-details/extensions/jira/jira-integration.md`) enables a Jira-driven workflow where a ticket URL replaces the manual requirements questionnaire.

### Prerequisites

The Atlassian MCP must be added to Claude Code's config. Run this once per machine:

```bash
claude mcp add --transport sse atlassian https://mcp.atlassian.com/v1/sse
```

Then restart `claude` and verify with `/mcp` — you should see `atlassian` listed.

### How it works

1. You paste a Jira ticket URL into Claude Code using the trigger phrase
2. Claude fetches the ticket via the Atlassian MCP
3. It maps the ticket Summary and Acceptance Criteria to AI-DLC's Inception phase
4. It confirms its understanding before writing any code
5. All generated artifacts in `aidlc-docs/` are prefixed with the ticket ID (e.g. `FUSE-123-design.md`)
6. A branch name is suggested automatically (e.g. `feature/FUSE-123-description`)

### Trigger phrase

```
Using AI-DLC, implement https://clearlink.atlassian.net/browse/FUSE-123
```

### Enabling the Jira extension in an existing project (e.g. fuse)

If you already have a project set up with this submodule, you just need to pull the latest rules — the Jira extension comes in automatically.

**Step 1 — Pull the latest rules into your project**
```bash
cd /path/to/fuse
git submodule update --remote .ai-dlc
git add .ai-dlc
git commit -m "Update AI-DLC rules — add Jira extension"
git push
```

**Step 2 — Set up the Atlassian MCP on your machine (once per machine)**
```bash
claude mcp add --transport sse atlassian https://mcp.atlassian.com/v1/sse
```
Restart `claude` and run `/mcp` to confirm `atlassian` is listed.

**Step 3 — Test it in fuse**

In Claude Code inside your fuse project:
```
Using AI-DLC, implement https://clearlink.atlassian.net/browse/FUSE-123
```
Claude will fetch the ticket, map it to the Inception phase, confirm its understanding, then proceed into design and planning.

**Step 4 — Teammates**

Each teammate runs the MCP setup command once on their own machine:
```bash
claude mcp add --transport sse atlassian https://mcp.atlassian.com/v1/sse
```
The rules themselves come in automatically via the submodule — no manual file copying needed.

### Coming soon
- Auto-transition ticket status when Construction phase starts
- Post the generated plan back to the Jira ticket as a comment

---

## Resources

| Resource | Link |
|---|---|
| AI-DLC Source Repo | [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows) |
| AI-DLC Methodology Blog | [AWS Blog](https://aws.amazon.com/blogs/devops/ai-driven-development-life-cycle/) |
| Claude Code Docs | [claude.ai/code](https://claude.ai/code) |
| Atlassian MCP | [mcp.atlassian.com](https://mcp.atlassian.com) |