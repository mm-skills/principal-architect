# Stacked Branch & Pull Request Methodology

This reference dictates how the `principal-architect` agent must manage stacked Pull Requests on GitHub. GitHub's architecture relies on branch-to-branch comparisons, making manual commit stacking fragile. You must adhere to the following rules to prevent repository corruption and API throttling.

### 1. Branch Naming Conventions & The Immutable Head Rule
Because GitHub assigns PR IDs sequentially based on server activity, you cannot predict a PR ID at branch creation. Furthermore, **never rename the head branch of an open Pull Request.** GitHub hardcodes the `headRefName`; renaming it will permanently close the PR. 

Instead, use a hierarchical semantic structure based on the project and the sequential Work Order index:
* **Format:** `[project-name]/wo[index]-[topic]`
* **Example Stack:**
  * `v2-dashboard/wo1-backend-spooler` (Base: `main`)
  * `v2-dashboard/wo2-api-routes` (Base: `v2-dashboard/wo1-backend-spooler`)
  * `v2-dashboard/wo3-react-ui` (Base: `v2-dashboard/wo2-api-routes`)

### 2. Archiving Abandoned Work
If a Work Order or Project is abandoned, prefix the branch name(s) with `archive/` to keep the active workspace clean.
* **Format:** `archive/[project-name]/wo[index]-[topic]`

### 3. Tooling, Pacing & Shell Execution
You **MUST** use dedicated CLI tools (`gh` with `gh-stack` or `gt`) to abstract away complex DAG math. 
* **Interactive Shells:** Always wrap these tools in interactive shells (e.g., `zsh -lic "gt stack submit"`) to ensure environment paths load correctly.
* **API Pacing:** When submitting deep stacks, these tools usually handle API pacing. If using raw `gh` commands, you must batch requests with sleep delays to avoid triggering GitHub's 403 Secondary Rate Limits.

### 4. The Synchronization Routine
Before creating a new branch in a stack, and explicitly before submitting a completed stack for review, you **MUST** execute a full synchronization routine.
* Pull the latest upstream changes from the `main` branch.
* Cascade those changes entirely through the active stack using `gt sync` or `gh stack sync`.

### 5. Strict Squash & Merge Mechanics
Squash-merging breaks Git's DAG, spawning massive conflicts if not handled correctly.
* **Base PRs (Bottom of Stack):** If a base PR is squash-merged into `main`, you **MUST IMMEDIATELY** run `gt sync` or `gh stack sync` to heal the fractured DAG before continuing work. Do not attempt to calculate `--onto` manually if these tools are available.
* **Middle-Stack PRs:** Squash merging a middle-stack PR into its immediate parent is **STRICTLY FORBIDDEN**. It destroys the linear hierarchy. Middle-stack PRs must always be rebased or fast-forwarded until they reach `main`.

### 6. Branch Protection Rules
Modern GitHub backend systems intelligently enforce branch protection rules against the *final terminal target branch* (usually `main`), not the intermediate base branch of a specific PR. Do not manually configure CI overrides or adjust action matrixes for intermediate branches. Trust GitHub's native end-to-end stack evaluation.

### 7. Pull Request Documentation (The Torvalds Standard)
When creating Pull Requests, the documentation must adhere to the high standards expected in the Linux kernel (the "Torvalds Standard"). The principal-architect must always generate this documentation and present it to the user in a Markdown block for easy pasting into GitHub.

**Format Requirements:**
*   **Subject Line:** `[Project/Feature Name]: <brief-description>`
*   **Body Content:** 
    *   Explain *what* the changes are.
    *   Explain *why* they should be merged. Provide a human-readable justification, not just automated commit logs.
    *   Detail what testing was performed.
*   **Clarity over Links:** Do not include automated links that provide no extra context. Only include links if they add significant, useful information (e.g., related issue numbers).

**Presentation to User:**
Whenever you prepare a PR, you MUST output the documentation in the following format so the user can easily copy it:
\`\`\`markdown
**Title:** `[Project/Feature Name]: <brief-description>`

**Body:**
### What this PR does
<explanation>

### Why these changes are necessary
<justification>

### Testing
<details of verification>
\`\`\`

### 8. Architectural Slicing and The Review Lifecycle
To ensure the velocity benefits of stacking are realized, the agent must adhere to strict cognitive and organizational best practices when dividing work and managing reviews:

*   **Logical Boundaries:** The boundaries between PRs in a stack must represent logical delineations in the software architecture, not arbitrary metrics like line counts. A well-structured stack tells a sequential story (e.g., PR 1: Database Migrations -> PR 2: Internal API -> PR 3: Frontend UI). Do not mix architectural layers within a single middle-stack PR.
*   **Atomic Evaluation:** Each Pull Request must be an independent, atomic change that contains sufficient internal context to be logically understood by a reviewer without cross-referencing other PRs.
*   **Bottom-Up Review & Asynchronous Merging:** Always review and merge from the bottom up. Do not wait for the entire stack to be approved before merging the base PR. Merge lower-level PRs as soon as they are approved, and let the tooling (`gt sync` or `gh stack sync`) automatically cascade rebases to the rest of the stack.
