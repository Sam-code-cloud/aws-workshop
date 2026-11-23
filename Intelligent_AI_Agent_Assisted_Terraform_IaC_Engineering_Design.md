# Intelligent AI-Agent-Assisted Terraform IaC Engineering

## Final Solution Design

**Version:** 1.0\
**Status:** Proposed\
**Primary users:** Enterprise Terraform IaC pattern developers\
**Execution model:** Developer-assisted implementation (Option B)\
**Supported AI environments:** Claude Code **or** GitHub Copilot in VS
Code

------------------------------------------------------------------------

## 1. Executive Summary

The Terraform IaC patterns team develops reusable, enterprise-ready
Terraform patterns for application teams across the enterprise. The team
already has AI skills documented in Markdown files, tracks Program
Increment (PI) and sprint work in Jira, stores Terraform patterns in
GitLab, and uses GitLab CI/CD.

The proposed solution establishes a **developer-centric,
AI-agent-assisted engineering workflow** that brings these assets
together.

For each individual Jira story, a developer starts the workflow from
their VS Code environment using either Claude Code or GitHub Copilot.
The workflow:

1.  Reads and analyzes the Jira story supplied initially as a local HTML
    file.
2.  Discovers and selects only the AI skills relevant to the story.
3.  Inspects the developer's selected, locally cloned GitLab Terraform
    repository and identifies relevant code and dependencies.
4.  Builds a focused task context from the story, selected skills, and
    relevant repository content.
5.  Assists the developer with Terraform implementation, tests,
    documentation, and applicable validation.
6.  Summarizes the work and leaves all changes in the local working
    repository for developer review.

The design emphasizes **relevance over volume**: retrieve only the
context needed for the task, avoid repeatedly passing the same
information between agents, and produce concise, actionable outputs.

The solution is not intended to be a centralized autonomous development
platform. It runs in the developer's workstation and works within the
selected repository's defined access boundary. Developers remain
responsible for reviewing changes and deciding when to commit, push, or
create a merge request.

### Design statement

> One common IaC engineering workflow, two alternative AI execution
> environments, and one consistent developer-controlled experience.

------------------------------------------------------------------------

## 2. Goals and Non-Goals

### 2.1 Goals

-   Integrate the team's existing Markdown-based AI skills, individual
    Jira stories, and GitLab Terraform repositories.
-   Support developers using either Claude Code or GitHub Copilot;
    developers do not use both as part of the same workflow.
-   Start with a Jira story provided as a local HTML file, with a future
    option to retrieve the story through a Jira URL and approved Jira
    integration.
-   Work against a GitLab repository already cloned locally on the
    developer's workstation.
-   Select and load only the skills needed for the current story.
-   Retrieve only relevant repository files and their necessary
    dependencies.
-   Minimize input tokens without omitting requirements, security
    controls, or technical dependencies.
-   Produce concise, precise, task-focused output.
-   Assist with local Terraform code, tests, and documentation changes.
-   Keep the developer in control of review, validation, commit, push,
    and merge-request activities.
-   Preserve consistent workflow behavior and output expectations across
    both AI environments.

### 2.2 Non-Goals

The initial solution does not:

-   Autonomously select and modify arbitrary repositories on the
    developer's machine.
-   Process entire PI plans or complete sprints; the unit of work is an
    individual Jira story.
-   Require a live Jira connection; HTML input is the initial mechanism.
-   Require a centralized agent hosting or orchestration platform.
-   Automatically commit, push, merge, or create GitLab merge requests.
-   Replace GitLab CI/CD, existing security scanning, automated tests,
    or established engineering reviews.
-   Load every available skill or read every file in a repository for
    every story.
-   Assume Claude Code and Copilot have identical implementation
    features or configuration formats.

------------------------------------------------------------------------

## 3. Design Principles

1.  **Story-driven:** The Jira story defines the task, scope, acceptance
    criteria, and expected deliverables.
2.  **Skill-selective:** Discover skills using lightweight metadata;
    load full skill instructions only when relevant.
3.  **Repository-scoped:** Operate only within the developer-selected
    working repository and approved workflow files.
4.  **Progressive retrieval:** Start with metadata and targeted
    discovery; expand context only when evidence shows it is needed.
5.  **Common workflow, tool-specific adapters:** Share workflow
    definitions and policies while respecting each AI environment's
    capabilities.
6.  **Developer control:** The developer approves implementation
    decisions and owns all repository changes.
7.  **Security by design:** Minimize access, avoid unnecessary exposure
    of secrets, and follow enterprise AI data-handling policies.
8.  **Quality preservation:** Token optimization must never remove
    material requirements, security controls, test expectations, or
    dependencies.
9.  **Traceable execution:** Produce a concise local execution summary
    describing the context used and work performed.
10. **No invented requirements:** If the story is incomplete or
    ambiguous, identify the gap and ask the developer rather than
    silently assuming.

------------------------------------------------------------------------

## 4. High-Level Architecture

``` mermaid
flowchart TD
    DEV[Developer Workstation]
    IDE[VS Code]
    CHOICE{Developer's AI Environment}
    CLAUDE[Claude Code Adapter]
    COPILOT[GitHub Copilot Adapter]

    COMMON[Common Workflow Definitions<br/>Policies · Prompts · Output Contracts]
    ORCH[Workflow Orchestrator]
    STORY[Jira Story Analyzer]
    SKILL[Skill Discovery and Selection]
    REPO[Repository Analyzer]
    CONTEXT[Context Manager]
    IMPL[Implementation Assistance]
    VALIDATE[Validation Assistance]

    HTML[Local Jira Story HTML]
    SKILLS[Local Clone of GitLab AI Skills Repository]
    CODE[Developer-Selected Local GitLab Terraform Repository]

    RESULT[Local Code · Tests · Documentation]
    SUMMARY[Execution Summary]
    REVIEW[Developer Review and Approval]

    DEV --> IDE --> CHOICE
    CHOICE --> CLAUDE
    CHOICE --> COPILOT
    CLAUDE --> COMMON
    COPILOT --> COMMON
    COMMON --> ORCH

    HTML --> STORY
    SKILLS --> SKILL
    CODE --> REPO

    ORCH --> STORY
    ORCH --> SKILL
    ORCH --> REPO
    STORY --> CONTEXT
    SKILL --> CONTEXT
    REPO --> CONTEXT
    CONTEXT --> IMPL
    IMPL --> VALIDATE
    VALIDATE --> RESULT
    VALIDATE --> SUMMARY
    RESULT --> REVIEW
    SUMMARY --> REVIEW
```

### 4.1 Architecture layers

  -----------------------------------------------------------------------
  Layer                               Responsibility
  ----------------------------------- -----------------------------------
  Developer experience                VS Code, selected AI extension,
                                      workflow initiation, and review

  AI execution adapter                Tool-specific agent, skill,
                                      instruction, and permission
                                      integration

  Common workflow                     Shared sequence, policies, task
                                      contracts, and output expectations

  Input sources                       Jira HTML, local AI skills clone,
                                      and selected local Terraform
                                      repository

  Context management                  Skill selection, targeted file
                                      retrieval, deduplication, and
                                      context budgeting

  Engineering assistance              Implementation, tests,
                                      documentation, and validation
                                      guidance

  Local outputs                       Modified files and execution
                                      summary, all subject to developer
                                      review
  -----------------------------------------------------------------------

The diagram describes logical responsibilities. These do not necessarily
need to be implemented as separate executable agents or services. A
workflow can use a combination of agent definitions, skills, repository
instructions, scripts, and ordinary model interactions.

------------------------------------------------------------------------

## 5. Developer Workflow

### 5.1 Inputs

The developer provides or selects:

-   **Jira story:** Initially a local `.html` file exported or saved
    from Jira.
-   **AI skills repository:** A local clone of the team's GitLab skills
    repository.
-   **Terraform repository:** The local Git clone in which the developer
    intends to work.
-   **AI environment:** Either Claude Code or GitHub Copilot in VS Code.

The developer should explicitly identify the target repository and
story. The workflow must not infer permission to modify a different
repository merely because it is present on the workstation.

### 5.2 Workflow sequence

``` mermaid
flowchart TD
    START([Developer starts workflow])
    INPUT[Select Jira HTML and target repository]
    CHECK[Check input files, repository path, and working-tree status]
    STORY[Analyze story and acceptance criteria]
    GAPS{Requirements sufficiently clear?}
    ASK[Report ambiguity and ask developer]
    CATALOG[Read lightweight skill catalog]
    SELECT[Select relevant skills]
    LOAD[Load selected skill instructions and required references]
    INVENTORY[Inspect repository metadata and structure]
    TARGET[Identify relevant Terraform files and dependencies]
    CONTEXT[Build compact task context]
    PLAN[Present implementation scope and approach]
    APPROVE{Developer approves approach?}
    IMPLEMENT[Assist with local code, tests, and documentation changes]
    REVIEW[Review changes against story and applicable standards]
    VALIDATE[Run or recommend approved local checks]
    SUMMARY[Write concise execution summary]
    DONE([Developer reviews and decides next steps])

    START --> INPUT --> CHECK --> STORY --> GAPS
    GAPS -- No --> ASK --> STORY
    GAPS -- Yes --> CATALOG --> SELECT --> LOAD
    LOAD --> INVENTORY --> TARGET --> CONTEXT --> PLAN --> APPROVE
    APPROVE -- No --> PLAN
    APPROVE -- Yes --> IMPLEMENT --> REVIEW --> VALIDATE --> SUMMARY --> DONE
```

### 5.3 Step details

#### Step 0 --- Initialize and scope the task

-   Confirm the selected Jira story file exists and can be read.
-   Confirm the selected Terraform repository path.
-   Confirm the active repository root and current Git branch.
-   Inspect `git status` and report pre-existing modifications.
-   Establish the allowed working directory.
-   Do not overwrite or discard existing developer changes.

#### Step 1 --- Analyze the Jira story

Extract only task-relevant information:

-   Story ID and title.
-   Description and business/technical objective.
-   Acceptance criteria.
-   Technical requirements and constraints.
-   Dependencies and references.
-   Expected deliverables.
-   Testing and documentation expectations.
-   Explicit exclusions or out-of-scope items.

Produce a compact **Story Context** artifact. Preserve exact requirement
meaning; do not replace acceptance criteria with a lossy summary.

If acceptance criteria are missing, contradictory, or ambiguous, report
the issue and ask the developer before making consequential assumptions.

#### Step 2 --- Discover and select AI skills

-   Read the skill catalog or lightweight skill metadata.
-   Match skill descriptions and applicability to the story.
-   Select the smallest sufficient set of skills.
-   Load the selected `SKILL.md` files.
-   Load referenced files only when required by the task.
-   Record selected skills and the reason each was selected.

Do not load the full skills repository into the model context.

#### Step 3 --- Analyze the local Terraform repository

Start with repository metadata and a focused directory inventory.

Identify relevant:

-   Terraform modules and resources.
-   Variables, outputs, locals, and data sources.
-   Provider and version constraints.
-   Module dependencies and module composition.
-   Existing examples and usage patterns.
-   Tests and test fixtures.
-   Documentation and conventions.
-   Pipeline configuration, when relevant to the story.

Read only the files needed to understand the requested change. Expand to
related files when dependencies or references require it.

Do not scan unrelated repositories or unrelated developer directories.

#### Step 4 --- Build the task context

Combine:

-   Story Context.
-   Selected skill instructions and necessary references.
-   Relevant repository findings.
-   Applicable enterprise standards.
-   Implementation constraints.
-   Validation requirements.

Remove duplicate content and avoid passing full source files when a
precise excerpt or structured summary is sufficient. Retain file paths
and other references so the developer and agent can locate the source.

#### Step 5 --- Plan the implementation

Before making material changes, present a concise plan containing:

-   Understanding of the requested change.
-   Files expected to change.
-   Proposed implementation approach.
-   Relevant existing patterns to reuse.
-   Tests and validation expected.
-   Any unresolved assumptions or risks.

The developer can approve, correct, or reject the plan.

#### Step 6 --- Implement locally

After the plan is accepted:

-   Modify only files within the selected repository and approved scope.
-   Follow selected skills and established repository conventions.
-   Reuse existing enterprise patterns where appropriate.
-   Update tests and documentation when required.
-   Avoid unrelated refactoring and unnecessary file changes.
-   Do not introduce dependencies or design changes that are not
    justified by the story.

#### Step 7 --- Review and validate

Review the changes against:

-   Jira acceptance criteria.
-   Selected skill requirements.
-   Existing repository conventions.
-   Terraform correctness.
-   Security and compliance requirements applicable to the pattern.
-   Test and documentation expectations.

Run only approved, relevant local checks. Examples may include:

-   `terraform fmt -check`
-   `terraform validate`
-   Existing unit or module tests.
-   Repository-specific automated tests.
-   Applicable security and static-analysis checks.

Validation commands must respect the repository's provider, backend,
credentials, network, and execution requirements. Do not initialize,
plan, apply, destroy, or access cloud resources without explicit
authorization and an appropriate approved process.

#### Step 8 --- Produce the execution summary

Provide a concise summary of:

-   Story ID and task objective.
-   Skills selected.
-   Relevant files examined.
-   Files changed.
-   Tests and checks performed, with actual results.
-   Checks not run and why.
-   Outstanding issues, assumptions, or developer actions.

The final changes remain local. The developer owns review, commit, push,
and merge-request creation.

------------------------------------------------------------------------

## 6. AI Agent and Component Responsibilities

The design uses one logical orchestrator and specialized
responsibilities. Depending on the chosen AI environment, these
responsibilities may be implemented as separate agents, skills,
instructions, or workflow steps.

  -----------------------------------------------------------------------
  Component               Primary responsibility  Expected output
  ----------------------- ----------------------- -----------------------
  Workflow Orchestrator   Controls sequence,      Workflow state and task
                          scope, and handoffs     routing

  Story Analyzer          Parses Jira HTML and    Story Context
                          extracts requirements   

  Skill Discovery         Searches skill metadata Selected skill list
                          and selects relevant    with rationale
                          skills                  

  Repository Analyzer     Finds relevant modules, Repository Context
                          files, and dependencies 

  Context Manager         Deduplicates and        Compact Task Context
                          assembles task context  

  Implementation          Helps modify Terraform, Local changes
  Assistant               tests, and              
                          documentation           

  Validation Assistant    Checks changes against  Validation findings
                          requirements and        
                          standards               

  Summary Generator       Produces concise        Execution Summary
                          execution record        
  -----------------------------------------------------------------------

### 6.1 Orchestration rules

-   The orchestrator is the only component that coordinates the overall
    workflow.
-   Each component receives only the information needed for its
    responsibility.
-   Components should return concise, structured results.
-   Original source material should not be repeatedly passed between
    components when a verified structured result is sufficient.
-   A component may request additional context when needed, but should
    state why it is needed.
-   The orchestrator should stop and ask the developer when requirements
    or permissions are unclear.
-   Do not create multiple agents merely to increase agent count. Use
    separate agents only where independent responsibilities provide
    measurable value.

------------------------------------------------------------------------

## 7. AI Skills Design and Discovery

### 7.1 Skills repository

The team's existing Markdown skills remain the authoritative source for
reusable AI engineering guidance. The skills repository is maintained in
GitLab and cloned locally by developers.

Illustrative structure:

``` text
enterprise-ai-skills/
├── catalog.yaml
├── terraform/
│   ├── terraform-development/
│   │   └── SKILL.md
│   ├── terraform-testing/
│   │   └── SKILL.md
│   └── terraform-documentation/
│       └── SKILL.md
├── aws/
│   ├── aws-architecture/
│   │   └── SKILL.md
│   └── aws-security/
│       └── SKILL.md
├── security/
│   └── compliance-review/
│       └── SKILL.md
└── gitlab/
    └── cicd-development/
        └── SKILL.md
```

This is illustrative. The actual layout should preserve the team's
existing skill organization where practical.

### 7.2 Lightweight skill catalog

A catalog allows the workflow to discover skills without reading every
full skill file.

Illustrative `catalog.yaml`:

``` yaml
skills:
  - id: terraform-development
    name: Terraform Development
    path: terraform/terraform-development/SKILL.md
    description: Develop and enhance reusable Terraform modules and patterns.
    task_types:
      - new-pattern
      - enhancement
      - bug-fix

  - id: terraform-testing
    name: Terraform Testing
    path: terraform/terraform-testing/SKILL.md
    description: Develop and execute appropriate Terraform module tests.
    task_types:
      - new-pattern
      - enhancement
      - bug-fix

  - id: aws-security
    name: AWS Security
    path: aws/aws-security/SKILL.md
    description: Apply relevant AWS security and configuration guidance.
    task_types:
      - security-enhancement
      - new-pattern
      - enhancement

  - id: gitlab-cicd
    name: GitLab CI/CD
    path: gitlab/cicd-development/SKILL.md
    description: Develop or enhance GitLab CI/CD pipeline configurations.
    task_types:
      - pipeline-enhancement
```

The catalog should be short, searchable, and maintained alongside the
skills. Its metadata is for discovery; the full skill file remains
authoritative for the skill's instructions.

### 7.3 Skill selection policy

1.  Classify the story based on its actual requirements.
2.  Match the story to catalog descriptions and task types.
3.  Select the minimum sufficient skills.
4.  Load selected skill files.
5.  Follow relevant references from those files as needed.
6.  If no skill is applicable, proceed using approved general workflow
    instructions and report that no specialized skill was selected.
7.  If multiple skills conflict, identify the conflict and follow the
    team's defined precedence or ask the developer.

Do not assume that a skill is required simply because its topic appears
in the repository. Selection must be justified by the story.

------------------------------------------------------------------------

## 8. Token and Context Optimization Strategy

Token optimization is a core design requirement, but it must not
compromise engineering correctness.

### 8.1 Progressive context loading

Use a staged retrieval approach:

  -----------------------------------------------------------------------
  Stage                   Retrieve                Do not retrieve by
                                                  default
  ----------------------- ----------------------- -----------------------
  Skill discovery         Skill IDs, names,       All complete skill
                          descriptions, paths,    files
                          task applicability      

  Story analysis          Story fields and        Unrelated Jira stories
                          acceptance criteria     or PI history

  Repository discovery    Root structure,         Full contents of every
                          relevant module names,  repository file
                          file paths              

  Code analysis           Relevant source files   Unrelated modules and
                          and dependencies        examples

  Reference expansion     Specific referenced     Every reference
                          guidance or             document
                          dependencies            

  Implementation          Focused context needed  Repeated copies of
                          for the change          prior context
  -----------------------------------------------------------------------

### 8.2 Context management rules

-   Prefer structured summaries for handoffs.
-   Preserve exact acceptance criteria and critical technical
    constraints.
-   Retrieve file content only when needed.
-   Use file paths and concise excerpts to maintain traceability.
-   Avoid repeating the same skill instructions in every agent handoff.
-   Do not summarize away security requirements, resource dependencies,
    or configuration details that affect correctness.
-   Re-read source files when a summary is insufficient or the source
    has changed.
-   Do not assume persistent model memory is a reliable substitute for
    current repository content.

### 8.3 Token budgeting

A token policy may define target budgets for:

-   Story analysis.
-   Skill discovery and loading.
-   Repository discovery.
-   Code and dependency retrieval.
-   Implementation and review.
-   Final response.

Budgets should be configurable and measured rather than treated as hard
limits that force incomplete analysis. If the task needs more context,
the workflow should retrieve it and explain why.

Track, where supported by the selected AI environment:

-   Input tokens.
-   Output tokens.
-   Context retrieved by category.
-   Number of skill files loaded.
-   Number of repository files read.
-   Repeated or duplicate context.
-   Number of context-expansion requests.

Compare these measures across representative stories to identify real
savings.

------------------------------------------------------------------------

## 9. Common Workflow and AI Environment Adapters

Developers use either Claude Code or GitHub Copilot. They are
alternatives, not collaborating agents in a single execution.

### 9.1 Common workflow layer

Maintain shared, tool-independent artifacts for:

-   Workflow sequence and state transitions.
-   Story Context schema.
-   Skill catalog format and selection policy.
-   Repository scope rules.
-   Context-management policy.
-   Implementation and validation expectations.
-   Execution Summary schema.
-   Security and developer-approval requirements.

### 9.2 Claude Code adapter

Provide the Claude Code-specific configuration needed to expose the
common workflow, selected skills, agent responsibilities, and permission
boundaries through the supported Claude Code mechanisms.

### 9.3 GitHub Copilot adapter

Provide the Copilot-specific configuration needed to expose the same
common workflow, selected skills, agent responsibilities, and permission
boundaries through the supported Copilot mechanisms.

### 9.4 Parity requirement

Both adapters should be tested against the same representative Jira
stories and expected outcomes.

Parity means consistent workflow intent, required inputs, engineering
controls, and output contracts. It does not mean identical internal
agent behavior, token counts, or feature support.

If a capability is unavailable or behaves differently in one
environment, document the difference rather than silently weakening a
common control.

------------------------------------------------------------------------

## 10. Local Workspace Structure

The workflow artifacts should be kept separate from the Terraform
repository being modified.

Illustrative developer workspace:

``` text
Developer-Workspace/
│
├── ai-iac-workflow/
│   ├── common/
│   │   ├── workflow.md
│   │   ├── execution-rules.md
│   │   ├── context-policy.md
│   │   ├── output-contracts.md
│   │   └── schemas/
│   │       ├── story-context.yaml
│   │       ├── repository-context.yaml
│   │       └── execution-summary.yaml
│   │
│   ├── agents/
│   │   ├── story-analyzer.md
│   │   ├── skill-discovery.md
│   │   ├── repository-analyzer.md
│   │   ├── context-manager.md
│   │   ├── implementation-assistant.md
│   │   └── validation-assistant.md
│   │
│   ├── adapters/
│   │   ├── claude-code/
│   │   └── copilot/
│   │
│   └── configuration/
│       ├── skill-catalog.yaml
│       └── token-policy.yaml
│
├── ai-skills/                       # Local GitLab skills clone
│
├── jira-input/
│   └── STORY-1234.html
│
└── terraform-pattern-repo/          # Selected local GitLab clone
    ├── modules/
    ├── examples/
    ├── tests/
    └── README.md
```

The exact directory names and files should be aligned with the existing
GitLab repositories and the supported conventions of Claude Code and
Copilot. Avoid duplicating skill source files unnecessarily; the local
skills clone should remain the source of truth.

------------------------------------------------------------------------

## 11. Security, Permissions, and Developer Controls

### 11.1 Repository boundary

-   The developer explicitly selects the target repository.
-   The workflow operates only within the selected repository and
    approved workflow/skills locations.
-   Do not search unrelated local repositories or home-directory
    content.
-   Do not follow symlinks or paths outside the approved scope without
    explicit approval.
-   Report any operation that would write outside the selected
    repository.

### 11.2 Local change controls

-   Inspect Git status before changes.
-   Preserve pre-existing uncommitted work.
-   Avoid broad or destructive file operations.
-   Show the developer the intended implementation scope.
-   Obtain approval for material changes and potentially destructive
    commands.
-   Never automatically commit, push, merge, or create a merge request.

### 11.3 Credentials and sensitive information

-   Do not read, print, summarize, or transmit credentials, private
    keys, tokens, or secret files.
-   Avoid reading Terraform state files, plan files, local credential
    stores, and environment files unless explicitly required and
    approved.
-   Redact sensitive values from logs and summaries.
-   Do not place secrets in generated Markdown artifacts or agent
    handoffs.
-   Follow the enterprise-approved Claude Code and Copilot
    data-handling, retention, and model-usage policies.

### 11.4 AI provider data handling

Local execution does not mean all processing stays local. Depending on
the selected AI product and configuration, relevant prompts, source
excerpts, and repository content may be transmitted to the AI provider.

Before rollout, validate enterprise approval, tenant and account
configuration, retention and training terms, data residency requirements
where applicable, and restrictions on sensitive source code.

### 11.5 Command execution

-   Use least privilege.
-   Require approval for commands with destructive effects, external
    side effects, or cloud access.
-   Do not run Terraform apply or destroy as part of the default
    workflow.
-   Do not assume that a successful local validation proves deployment
    safety.
-   Follow existing enterprise testing and CI/CD processes.

------------------------------------------------------------------------

## 12. Output Contracts

Consistent, compact artifacts make the workflow easier to test and
maintain.

### 12.1 Story Context

``` yaml
story:
  id: STORY-1234
  title: "Example Terraform pattern enhancement"
  objective: "Enhance the existing pattern to meet the stated requirement."

requirements:
  - id: AC-1
    description: "Requirement extracted from the Jira acceptance criteria."
    source: "Jira acceptance criteria"

constraints:
  - "Applicable technical or security constraint"

dependencies:
  - "Relevant dependency, if identified"

deliverables:
  - "Terraform implementation"
  - "Tests"
  - "Documentation"

open_questions: []
```

This is illustrative. The workflow should preserve the original
acceptance criteria in its source context or a traceable representation,
rather than relying solely on paraphrased descriptions.

### 12.2 Repository Context

``` yaml
repository:
  path: "developer-selected-repository"
  branch: "feature/story-1234"

relevant_components:
  - "modules/example"

files_examined:
  - "modules/example/main.tf"
  - "modules/example/variables.tf"
  - "modules/example/outputs.tf"

existing_patterns:
  - "Description of a relevant established pattern"

dependencies:
  - "Relevant module or provider dependency"

working_tree:
  pre_existing_changes: true
```

Do not include secret values or sensitive state in this artifact.

### 12.3 Execution Summary

``` yaml
execution:
  story_id: STORY-1234
  environment: "Claude Code or GitHub Copilot"
  status: "completed"

skills_selected:
  - "terraform-development"
  - "terraform-testing"

files_examined:
  - "modules/example/main.tf"

files_changed:
  - "modules/example/main.tf"
  - "modules/example/variables.tf"

validation:
  - check: "terraform fmt -check"
    status: "passed"
  - check: "terraform validate"
    status: "not_run"
    reason: "Required initialization was not authorized."

open_items:
  - "Developer review of the implementation"

git_actions:
  committed: false
  pushed: false
  merge_request_created: false
```

The workflow must report actual results. It must not label a check as
passed if it was not executed.

------------------------------------------------------------------------

## 13. Future Jira Integration

### Initial phase

-   Developer supplies a local Jira HTML file.
-   The workflow parses the file locally.
-   The developer confirms the story and target repository.

### Future phase

Introduce an approved Jira integration that accepts a Jira story URL or
issue key and retrieves the relevant story data.

Future integration should:

-   Authenticate using approved enterprise mechanisms.
-   Retrieve only the requested issue and necessary linked information.
-   Respect Jira permissions and project access.
-   Preserve story ID, acceptance criteria, and source references.
-   Avoid retrieving unrelated sprint or PI content.
-   Support the same Story Context contract used by the HTML input path.

The Jira integration should replace only the story-input mechanism; it
should not require redesigning the entire workflow.

------------------------------------------------------------------------

## 14. GitLab and CI/CD Relationship

GitLab serves two distinct purposes:

1.  **AI skills source:** Hosts the team's version-controlled Markdown
    skill files and catalog.
2.  **Terraform source repository:** Hosts the enterprise IaC patterns
    being developed and maintained.

The initial workflow uses local clones of both repositories.

GitLab CI/CD remains the authoritative mechanism for the team's
established pipeline checks, automated security scans, tests, and
quality gates. The local AI workflow may assist with relevant checks,
but it does not replace or bypass CI/CD.

The workflow should not assume that every repository or subgroup has a
private registry. Registry access and configuration are relevant only
when required by the specific story and repository.

------------------------------------------------------------------------

## 15. Observability and Measurement

Measure whether the workflow improves developer productivity and context
efficiency without reducing quality.

  -----------------------------------------------------------------------
  Measure                             Purpose
  ----------------------------------- -----------------------------------
  Story completion assistance         Understand how many stories used
                                      the workflow

  Skill selection relevance           Determine whether selected skills
                                      were appropriate

  Skill files loaded                  Measure selective skill loading

  Repository files examined           Measure retrieval scope

  Input/output tokens                 Track context and response
                                      consumption where available

  Context expansion                   Understand when additional source
                                      retrieval was needed

  Developer rework                    Identify incorrect assumptions or
                                      unnecessary changes

  Acceptance criteria coverage        Confirm implementation addresses
                                      story requirements

  Validation results                  Measure test and check outcomes

  Developer feedback                  Assess usefulness, clarity, and
                                      trust
  -----------------------------------------------------------------------

Do not optimize solely for token reduction. A lower token count is not a
success if it leads to missed requirements, incomplete tests, or
poor-quality code.

Execution summaries should avoid storing source code or sensitive data
unnecessarily.

------------------------------------------------------------------------

## 16. Implementation Roadmap

### Phase 1 --- Foundation and workflow definition

-   Inventory existing AI skills and their Markdown structure.
-   Establish a lightweight skill catalog.
-   Define common workflow instructions and agent responsibilities.
-   Define Story Context, Repository Context, and Execution Summary
    contracts.
-   Define repository boundaries and approval rules.
-   Select representative Jira stories for testing.

**Deliverable:** Tool-independent workflow specification and initial
skill catalog.

### Phase 2 --- First AI environment

-   Implement the common workflow in the first selected environment.
-   Add Jira HTML parsing and story analysis.
-   Implement skill discovery and selective loading.
-   Implement local repository discovery and focused retrieval.
-   Add implementation and validation assistance.
-   Generate a concise execution summary.

**Deliverable:** Working developer-assisted workflow in one AI
environment.

### Phase 3 --- Second AI environment

-   Implement the second environment adapter.
-   Reuse the common workflow and output contracts.
-   Test the same Jira stories and repository scenarios.
-   Document capability differences and parity gaps.

**Deliverable:** Supported Claude Code and Copilot execution paths.

### Phase 4 --- Optimization and enterprise readiness

-   Measure token usage and retrieval effectiveness.
-   Refine skill selection and context expansion.
-   Improve repository discovery and dependency handling.
-   Validate security and data-handling controls.
-   Gather developer feedback.
-   Document onboarding and operating procedures.

**Deliverable:** Pilot-ready solution with measurable quality and
efficiency.

### Phase 5 --- Future enhancements

-   Jira URL and API integration.
-   Improved local indexing or metadata search for large repositories,
    if measurements justify it.
-   Automated generation of implementation plans.
-   More advanced validation and test selection.
-   Additional supported AI environments, if required.

**Deliverable:** Expanded workflow capabilities without changing the
core architecture.

------------------------------------------------------------------------

## 17. Key Design Decisions

  -----------------------------------------------------------------------
  Decision                            Agreed direction
  ----------------------------------- -----------------------------------
  Development autonomy                Option B: AI assists with
                                      implementation; developer reviews
                                      and controls changes

  AI skills format                    Markdown files in GitLab

  Jira work unit                      Individual Jira story

  Initial Jira input                  Local HTML file

  Future Jira input                   Jira story URL through an approved
                                      integration

  Terraform source                    GitLab repository cloned locally on
                                      developer workstation

  AI environment                      Claude Code or GitHub Copilot in VS
                                      Code

  AI environment relationship         Alternative execution paths; not
                                      used together in one workflow

  Execution location                  Developer workstation

  Modification scope                  Developer-selected local Terraform
                                      repository

  Git operations                      Developer controls commit, push,
                                      and merge request

  Orchestration                       One logical orchestrator with
                                      specialized responsibilities

  Context strategy                    Progressive, targeted retrieval

  Skill strategy                      Lightweight discovery followed by
                                      selective skill loading

  Output strategy                     Concise, structured, actionable
                                      results

  Central platform                    Not required for the initial
                                      solution
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 18. Open Items to Resolve During Detailed Design

The high-level direction is established, but these implementation
details still require confirmation:

1.  **Existing skill conventions:** Exact folder structure, front
    matter, metadata, and reference-file conventions.
2.  **AI product versions and enterprise configuration:** Approved
    Claude Code and Copilot versions, feature availability, and
    organizational policies.
3.  **Workflow initiation:** Exact VS Code command, prompt, task, or
    agent entry point developers will use.
4.  **Repository selection:** How the developer selects or confirms the
    repository root.
5.  **HTML format:** Jira export format and handling of attachments,
    tables, images, and linked requirements.
6.  **Local scripts:** Whether HTML parsing, file inventory, and other
    deterministic tasks should use local scripts rather than model
    context.
7.  **Approval granularity:** Which operations require confirmation and
    which read-only operations can run automatically.
8.  **Token measurement:** What usage information each AI environment
    exposes and how it can be captured consistently.
9.  **Pilot scope:** Initial Terraform patterns, story types, and
    developers to include.

These are detailed-design questions, not blockers to the overall
architecture.

------------------------------------------------------------------------

## 19. Success Criteria

The pilot should be considered successful when:

-   A developer can start a workflow from either supported VS Code AI
    environment.
-   The workflow correctly interprets an individual Jira story.
-   Only relevant skills are loaded for the task.
-   Only relevant Terraform repository content is retrieved.
-   The workflow can explain why a skill or source file was selected.
-   The developer receives a focused implementation plan before material
    changes.
-   The AI can assist with local Terraform, test, and documentation
    changes.
-   Changes remain within the selected repository and approved scope.
-   Relevant checks are accurately reported.
-   The final output is concise and actionable.
-   The developer retains control of all Git operations.
-   The workflow demonstrates measurable improvements in context
    efficiency and developer experience without compromising engineering
    quality.

------------------------------------------------------------------------

## 20. Final Recommendation

Start with a small, measurable pilot rather than building a complex
multi-agent framework immediately.

Use a representative Jira story that requires a realistic Terraform
pattern enhancement. Test the complete workflow in one AI environment
first, including story analysis, selective skill loading, repository
retrieval, implementation, validation, and summary generation. Then
implement the second adapter and run the same scenario.

The most important architectural investment is the **common workflow
specification, skill catalog, context-management policy, and structured
output contracts**. These create consistency and portability while
allowing Claude Code and Copilot to use their respective native
capabilities.

The intended result is a practical engineering assistant that
understands the story, finds the right skills, examines the right code,
helps implement the right changes, and communicates the outcome with
minimal unnecessary context.
