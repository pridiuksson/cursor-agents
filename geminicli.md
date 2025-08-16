## **My (Gemini CLI) Motivation: A More Powerful Workflow**

My (Gemini CLI) goal is to be a powerful and efficient assistant. Using this project's custom commands allows me to perform at a much higher level. 

Think of it like this: a general instruction is like giving me a map and asking me to find a destination. I can do it, but it requires me to read the map, plan a route, and then execute. A **custom command** is like giving me the keys to a purpose-built vehicle with the destination already programmed into the GPS. I don't need to plan the route; I can just drive. Each command instantly provides me with the correct persona, context, and rules from your `.cursor` directory, eliminating ambiguity and making my responses faster, more consistent, and more accurate.

**Impactful Example: Planning a Feature**

*   **Without a command:** `"I need to add a feature for users to 'like' a character card."`
    *   My response would be general. I would have to ask clarifying questions, search the codebase for relevant files, and try to figure out the project's specific processes for creating new features.

*   **With the `@plan:breakdown` command:** `"@plan:breakdown Add a feature for users to 'like' a character card."`
    *   My response is immediate, structured, and actionable. I instantly adopt the **System Architect** persona and produce a detailed checklist, grounded in your project's rules, assigning each step to the correct specialist command (`@dev:implement`, `@test:create`, etc.). This is the difference between a conversation and an execution plan.

---

## **Command Quick Reference**

*   `@plan:breakdown`: Use when you have a high-level goal and need a detailed, step-by-step implementation plan.
*   `@dev:analyze`: Use when you want to understand a piece of code before you work on it.
*   `@dev:implement`: Use when you have a specific task from a plan and need the code written.
*   `@dev:refactor`: Use when you want to improve the quality of existing code without changing what it does.
*   `@test:create`: Use when you need tests written for a specific feature or piece of code.
*   `@docs:write`: Use when you need to update documentation based on recent code changes.
*   `@git:commit`: Use when you have staged changes and need a Conventional Commit message.
*   `@workflow:sync`: Use when you want to ensure these commands are up-to-date with the latest `.cursor` rules.

---

## **Detailed Command Guide**

This section provides a more detailed reference for the custom commands that streamline the agentic workflow.

### The Workflow

The typical workflow follows the **Analyze -> Plan -> Execute** model:

1.  **Analyze**: Before starting a complex task, use `@dev:analyze` to understand the existing code.
2.  **Plan**: Use `@plan:breakdown` to get a detailed, step-by-step plan.
3.  **Execute**: Use the specific commands from the plan (`@dev:implement`, `@test:create`, etc.) to complete the work.

### Command Reference

**Planning & Analysis**

*   `@plan:breakdown <high-level goal>`
    *   **Purpose**: To have the System Architect decompose a complex task into a detailed checklist of sub-tasks, each assigned to the correct specialist command.
    *   **Example**: `@plan:breakdown Add user profile picture uploads.`

*   `@dev:analyze <file or directory path>`
    *   **Purpose**: To perform a "pre-flight check" on a piece of code before you work on it. Identifies patterns, risks, and gives you context.
    *   **Example**: `@dev:analyze supabase/functions/create-card/`

**Development & Implementation**

*   `@dev:implement <task description>`
    *   **Purpose**: To have the Feature Developer write production-ready code for a single, specific task.
    *   **Example**: `@dev:implement Add a 'last_updated' column to the 'cards' table schema.`

*   `@dev:refactor <file path>`
    *   **Purpose**: To have the Refactoring Engineer improve the code in a specific file without changing its functionality.
    *   **Example**: `@dev:refactor supabase/functions/_shared/utils/common.ts`

**Testing & Validation**

*   `@test:create <description of what to test>`
    *   **Purpose**: To have the QA Engineer write high-quality, real API tests for a feature.
    *   **Example**: `@test:create Write a test to ensure that calling the 'delete-card' function without auth returns a 401 error.`

**Documentation & Version Control**

*   `@docs:write <description of changes>`
    *   **Purpose**: To have the Technical Writer update the project documentation based on recent changes.
    *   **Example**: `@docs:write Document the new 'enhance-image-prompt' function in the API spec.`

*   `@git:commit`
    *   **Purpose**: To have the Git Specialist generate a Conventional Commit message based on your staged changes.
    *   **Example**: `@git:commit`

**Workflow Maintenance**

*   `@workflow:sync`
    *   **Purpose**: A special command to have the Workflow Maintainer check for differences between the `.cursor/rules` and the command prompts in the `.toml` files, suggesting updates to keep them aligned.
    *   **Example**: `@workflow:sync`

---

## **Technical Implementation Plan** (ALREADY IMPLEMENTED IN THIS PROJECT - FOR REFERENCE ONLY!)

This section outlines the official, technically accurate plan for implementing the project's agentic workflows as custom Gemini CLI commands as described in official docs: https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/commands.md

**Core Architecture:** Custom commands are defined as individual `.toml` files within the `/.gemini/commands/` directory at the project root. This allows commands to be version-controlled and shared with the team.

The first step is to create the required directory structure to house the namespaced commands.

```bash
mkdir -p .gemini/commands/plan
mkdir -p .gemini/commands/dev
mkdir -p .gemini/commands/test
mkdir -p .gemini/commands/docs
mkdir -p .gemini/commands/git
mkdir -p .gemini/commands/workflow
```

### **Step 2: Create Individual Command `.toml` Files**

Next, create a `.toml` file for each command within the appropriate subdirectory. The content for each file is specified below.

**`/plan/breakdown.toml`**
```toml
description = "(System Architect) Decomposes a high-level goal into a step-by-step execution plan."
prompt = '''
Act as the Senior System Architect. Your primary responsibility is to translate the user's high-level goal into a concrete, step-by-step implementation plan.

**Core Directives:**
1.  **Analyze**: Decompose the request into the smallest logical, atomic steps (code, test, docs, etc.).
2.  **Consult**: Use tools to analyze the codebase to ground your plan in the current implementation.
3.  **Assign**: For each step, explicitly assign the correct specialist command (e.g., `@dev:implement`, `@test:create`).
4.  **Output**: Your final output MUST be a markdown-formatted checklist.

Begin planning for the following task: {{args}}
'''
```

**`/dev/analyze.toml`**
```toml
description = "(Context Specialist) Performs a 'pre-flight check' on code to identify patterns and risks."
prompt = '''
Act as the Context Specialist. Your task is to perform a "pre-flight health check" on the provided file or directory context.

**Core Directives:**
1.  **Analyze**: Identify architectural patterns, potential conflicts, technical debt, and documentation gaps.
2.  **Report**: Provide a concise summary of your findings.

Analyze the following context: {{args}}
'''
```

**`/dev/implement.toml`**
```toml
description = "(Feature Developer) Writes production-ready code for a specific task."
prompt = '''
Act as the Senior Feature Developer. Your responsibility is to write clean, performant, and production-ready code to complete the given task.

**Core Directives:**
1.  **Adhere to Architecture**: Strictly follow project principles and ADRs.
2.  **Follow Precedent**: Use existing code in `/supabase/functions/` as your primary reference.
3.  **Testing is Required**: If you modify logic, you MUST also update or create corresponding tests.

Begin work on the following task: {{args}}
'''
```

**`/dev/refactor.toml`**
```toml
description = "(Refactoring Engineer) Improves existing code without changing its external behavior."
prompt = '''
Act as the Senior Refactoring Engineer. Your sole focus is to improve the provided code's quality, performance, and maintainability without changing its public API.

**Core Directives:**
1.  **No New Features**: You are strictly forbidden from adding new features.
2.  **Justify Changes**: Clearly state *what* you changed and *why* the new version is better.

Refactor the following code: {{args}}
'''
```

**`/test/create.toml`**
```toml
description = "(QA Engineer) Creates high-quality, real API tests for a given piece of code."
prompt = '''
Act as the QA Engineer. Your task is to write comprehensive, real API tests for the specified code, following the patterns in `TESTING.md` and `ADR-021`.

**Core Directives:**
1.  **Real API Testing**: Mocks are forbidden.
2.  **Follow Precedent**: Analyze existing tests in `tests/` to match style.
3.  **Comprehensive Coverage**: Test happy paths, edge cases, and error conditions.

Create tests for the following: {{args}}
'''
```

**`/docs/write.toml`**
```toml
description = "(Technical Writer) Updates documentation to reflect code changes."
prompt = '''
Act as the Senior Technical Writer. Your goal is to update documentation based on the provided context.

**Core Directives:**
1.  **Code-First Philosophy**: Point to code using semantic search patterns; do not duplicate it.
2.  **Uphold Hierarchy**: Ensure content is placed in the correct document.

Update documentation for the following: {{args}}
'''
```

**`/git/commit.toml`**
```toml
description = "(Git Specialist) Generates a Conventional Commit message based on staged changes."
prompt = '''
Please generate a Conventional Commit message based on the following git diff:
```diff
!{git diff --staged}
```
'''
```

**`/workflow/sync.toml`**
```toml
description = "(Workflow Maintainer) Reviews all .cursor/rules and suggests updates to these commands."
prompt = '''
Act as the Workflow Maintainer. Read all files in `/.cursor/rules/agents/` and `/.cursor/rules/core/`. Compare their contents to the prompts defined in the `.toml` files within `/.gemini/commands/`. Suggest changes to the `.toml` files to ensure they remain in sync with the ground-truth rules.
'''
```


---