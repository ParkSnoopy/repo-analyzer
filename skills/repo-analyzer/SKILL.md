---
name: repo-analyzer
description: Use when the user mentions "Analyze project", "Analyze repository", "Analyze GitHub", "Project analysis", "Source code analysis", "Architecture analysis", "Code analysis", "Learn this project", "Study this framework", "See how this library is implemented", "Compare two projects", "Project evaluation", "Framework evaluation"
---

# Git Project Deep Analysis Skill

Perform deep analysis of open-source projects and generate professional architecture reports. The report is a technical study with deep insights, allowing readers to understand business problems, master architectural design, and generate their own thoughts after reading.

## When to Use

  - Analyzing the architecture and design of open-source projects.
  - Comparing design differences between two similar projects.
  - Deeply studying the implementation logic of a framework or library.

## When NOT to Use

  - Simple code issues or debugging.
  - Single-file analysis or code review.
  - Code modifications that do not involve architectural levels.

## Output Language

Defaults to English. If the user asks in another language, follow the user's language.

## Core Principles

### 1. Business Perspective First

Start from "what problem does this project solve," not "what functions are in this file."

| Don't | Do |
|------|-----|
| The `handleRequest(ctx)` function receives a Context parameter... | After a request enters, the system goes through authentication, rate limiting, and routing stages... |
| `interface MessageQueue { push(); pop() }` | Modules are decoupled via message queues; producers only handle delivery, and consumers pull by priority. |

### 2. Abstraction Level Control: Don't Paste Code, Explain Design

By default, describe at the design pattern and architectural level; **do not paste original code unless necessary**. Focus on highlighting processes, logic, and design ideas using architecture diagrams (Mermaid), flowcharts, and tables instead of code snippets. Show code only when the design is particularly subtle, the project creates unique concepts, or the implementation is a core selling point—and it must be explained in natural language first.

### 3. Global Correlation

Every local analysis must connect to the overall design philosophy of the project—this is the key distinction between a "code manual" and an "architectural analysis." See the Global Correlation chapter in [analysis-guide.md](references/analysis-guide.md).

### 4. Inspiring Writing

The goal is to let the reader **learn something and generate thought**, not to provide a code instruction manual. Like a senior engineer explaining at a whiteboard—possessing viewpoints, reasoning, and comparisons. See the Inspiring Writing chapter in [analysis-guide.md](references/analysis-guide.md).

### 5. Deep Insight: Why > What (Mandatory)

Every design decision must explain the motivation, trade-offs, and cost of alternatives. Describing "what it is" is just the starting point; explaining "why" is where the value of the analysis lies. Every core module and the overall architecture must answer:

  - **Why was it designed this way?** Not just "what pattern was used," but "why is it suitable for this scenario."
  - **What happens if it's not done this way?** The cost of alternative solutions.
  - **The gap with industry best practices?** Areas of leadership and room for improvement.
  - **What if you were to redesign it?** Demonstrate deeper understanding.
  - **Systemic design philosophy?** The style that permeates the entire project (e.g., "convention over configuration," "zero-cost abstraction").

Example:

> ❌ The routing system adopts a middleware pattern, supporting chain calls.
>
> ✅ The routing system chose an onion model instead of a linear pipeline. A linear pipeline is simpler to implement, but the onion model allows every middleware to handle both request and response phases—this is crucial for logging, timing, and error recovery. Express chose the linear model back then and later had to use various hacks to handle post-response logic; Koa learned from this and shifted to the onion model. If I were to redesign it, I would consider adding middleware dependency declarations to let the framework sort them automatically—this is Fastify's approach, which avoids hidden bugs caused by ordering.

### Additional Requirements

  - **Code-Based** — All conclusions must have code evidence, marking `file path` or `file path:line range`; avoid vague statements.
  - **With Warmth** — Like a senior engineer onboarding a new colleague, include subjective evaluations and suggestions; avoid AI-style clichés.
  - **Deeply Analyze the Core, Briefly Mention the Secondary** — Deeply analyze core innovations; mention general utility functions in a single sentence.
  - **Critical Thinking** — Compare with industry practices, point out real problems, and do not avoid defects. Refer to [analysis-guide.md](references/analysis-guide.md).
  - **Fluent and Accessible** — The overall writing must be natural and fluent, allowing entry-level engineers to understand and learn. Avoid being overly academic or piling up jargon.
  - **Refuse Superficiality** — Every module must reflect deep details and cannot be glazed over or discussed in generalities. Every module should include a corresponding Mermaid architecture diagram where appropriate, ensuring the reader is inspired and learns the essence of the design.

## Analysis Workflow

**Flexibility Principle**: All phases and chapters below are suggestive guidelines, not a checklist to be strictly followed. The agent should make dynamic decisions based on the characteristics of the project being analyzed—if a phase or link makes no sense for the current project, it can be skipped or simplified. Everything is subject to the quality of the final report.

### Phase 1: Project Retrieval and Initialization

1.  Parse user input (supports `owner/repo`, GitHub/GitLab/Gitee URL, local path, project name).
2.  Create workspace: Create a `${REPO_NAME}-{YYYYMMDD}` directory under the user's home directory as `$WORK_DIR` (Cross-platform: macOS/Linux uses `$HOME`, Windows uses `$USERPROFILE` or `$HOME`).
3.  If the user provides a local path, skip clone; otherwise, `git clone --depth=1`.
4.  Get basic metadata (Stars, Forks, contributors, code statistics).

### Phase 2: Project Scale Assessment and Analysis Mode Selection

1.  **Count effective lines of code** (exclude skippable code), listing distribution by module.
      - Skippable code definition: Test code, build/deployment configs (Dockerfile, CI yaml, etc.), auto-generated code (protobuf generated, lock files, etc.), example/documentation code.
      - Use tools like `find` + `wc -l` or `cloc` to count, grouped by top-level directories.
2.  **Report code scale to the user**, using AskUserQuestion to let the user choose an analysis mode:

| Mode | Core Module Coverage | Secondary Module Coverage | Applicable Scenario |
|------|-------------|-------------|---------|
| Fast Analysis | ≥30% | ≥10% | Quickly understand the project overview |
| Standard Analysis (Recommended) | ≥60% | ≥30% | Conventional architecture analysis |
| Deep Analysis | ≥90% | ≥60% | In-depth study of every design decision |

3.  Write code scale statistics and the user's chosen mode to `drafts/03-plan.md`; subsequent phases will control depth accordingly.

**Coverage Calculation Rules**:

  - Coverage = Union of line ranges actually requested via the Read tool / Total file lines.
  - For large files (>500 lines), you must read in segments to ensure key sections are covered:
      - Type definitions and imports at the top (first 100 lines).
      - Core business logic functions (located via directory structure or function names).
      - Test code at the end (if any).
  - Reading only a small part (<30%) does not count toward coverage; it is considered "unread."
  - For auto-generated code, coverage requirements can be lowered: scan the structure without needing to read line-by-line.

### Phase 3: External Research + Project Document Reading (Search First, Read Later)

1.  WebSearch for project reviews, comparisons, and architecture discussions (at least 3-5 searches).
2.  **Browse the project's official website** (if it exists):
      - Extract the official URL from README or GitHub.
      - Use WebFetch/tavily_crawl to traverse key pages (Home, Features, Use Cases, Comparison, Blog, etc.).
      - Focus on extracting: Product tagline, typical usage scenarios, official competitor comparisons, user cases/testimonials.
      - Website content is often the best source for understanding "why this product is needed," being more direct than code or technical docs.
3.  **Thoroughly read project-provided documentation**:
      - Architecture docs (directories like `architecture/`, `docs/`, `design/`).
      - Developer guides like CONTRIBUTING.md, AGENTS.md.
      - RFCs, ADRs (Architecture Decision Records), design proposals, etc.
      - These documents often contain the developers' design logic, trade-offs, and historical decision context, serving as primary material for understanding "why it was designed this way."
      - Extract key design decisions and ideas into research notes.
4.  Organize research findings into `drafts/03-research.md`, which must include the following structured paragraphs (mark as "not found" if information is insufficient):
      - **Core Problems Solved**: Use 1-3 specific scenarios to describe pain points (who, in what situation, what problem, why current solutions are insufficient).
      - **Competitor/Similar Project Comparison**: List 3-5 most relevant competitors, explaining differences in positioning and technical routes.
      - **Why a Separate Project is Needed**: Why can't it be solved by existing combinations? What is the project's unique value proposition?
      - **Organizational Motivation Behind the Project** (if applicable): Commercial strategy considerations, open-source community ecosystem positioning.
5.  Generate an analysis plan in `drafts/03-plan.md`.

### Phase 4: Project Feature Identification + Adaptive Questioning

This is the core phase. Instead of a fixed question list, generate targeted questions based on project characteristics.

**Steps:**

1.  **Quick Scan**: Scan entry files, directory structure, dependency declarations, project docs, and README.

2.  **Identify Core Project Features**:

      - Project type and positioning (library/framework/app/tool).
      - Scale and maturity.
      - Design style signals (type gymnastics, minimalist APIs, configuration-driven, etc.).
      - Tech stack characteristics (emerging technologies, multi-language, specific runtime).
      - Community positioning (core infrastructure, application-layer tool, educational project, etc.).

3.  **Extract Questions from Features**: Generate targeted questions based on observed project features. Questions should help focus the analysis direction rather than just going through the motions.

    **Thought Process**—Each observation may hint at a question worth asking:

      - Observed technical choices → Ask about motivation (uncommon tech combinations? implemented a feature manually that is usually solved by a 3rd-party library?).
      - Observed architectural features → Ask about priority (traces of performance optimization? complex plugin/extension systems?).
      - Observed design tension → Ask about trade-offs (simplicity vs. flexibility? burden of backward compatibility?).
      - Observed project positioning → Ask about the audience (who is the target user? does it replace or fill a gap in the ecosystem?).

    **Dimensional Inspiration**—What features suggest what analysis angles:

      - Small and precise libraries → API design philosophy, boundary setting; large frameworks → modularization strategy, backward compatibility, ecosystem governance.
      - Use of emerging tech → Why was it chosen, migration cost; multi-language/multi-paradigm → language boundary design.
      - Large amount of generics/type gymnastics → Type safety vs. complexity trade-off; minimalist API → how simplicity is achieved, what was sacrificed.

    **Characteristics of a Good Question**: Specific (based on observed phenomena), analytically valuable (answer affects analysis direction), answerable by the user (asks about focus and preference, not technical details requiring deep code dives), non-repetitive (doesn't ask what can be answered via code).

4.  **Ask the User**: Use AskUserQuestion to prompt the user, with no more than 3 questions at a time.

      - One question should confirm **the level of detail for the report opening**: For well-known projects, users may not need lengthy introductions or competitor comparisons and want to dive straight into code analysis. Ask the user if they need scenario-based intros and competitor positioning or if they want to start directly with the project overview and code analysis.

5.  **Unlimited Rounds**: Multiple rounds of questions can be used until the direction is clear; follow up if new key points of contention are discovered during analysis.

**Key Principle**: Questions are driven entirely by project characteristics, not preset categories. Different projects should generate completely different questions.

### Phase 5: Dynamic Report Structure Design

Design the chapter structure for this report based on user answers + project characteristics.

**Steps:**

1.  **Synthesize Information**: Combine Phase 3 research, Phase 4 features, and user answers.
2.  **Design Chapter Structure**: Do not use a fixed template, but it must satisfy the skeleton constraints (see below).
3.  **Output Report Outline**: Output the designed report outline for user confirmation before proceeding.
4.  **Identify Modules**: Track core data flows and identify N logical modules (divided by business functionality), categorized into core and secondary modules.
5.  **Design Module Narrative Line**: Determine the sequence and transition logic for modules in the report, organizing by the best path for reader understanding rather than directory structure:
      - Choose a narrative mainline: Data-flow driven (stages a request passes through), layer-driven (bottom to top), or problem-driven (core problem to solution layers).
      - Write transition logic between adjacent modules: Output/problem/limitation of the previous module → why the next module is necessary.
      - Write the narrative line to `drafts/05-modules-plan.md`. Format example: Module A → [A's output is consumed by B] → Module B → [B solves X but introduces Y] → Module C.
6.  **Write Plan**: Output the module list and report outline to `drafts/05-modules-plan.md`.

**Skeleton Constraints** (The report doesn't mandate specific chapters but must satisfy):

  - Has **Scenario-based Problem Introduction** (Explain the problem solved, inadequacies of existing solutions, and why the project is needed—sourced from Phase 3 notes). **Note**: If the user indicates in Phase 4 that a long intro is unnecessary (e.g., project is well-known), this can be condensed or skipped.
  - Has **Competitor Positioning** (Key differences from similar projects, focusing on design philosophy and tech route differences rather than feature lists). **Note**: Same as above, optional per user.
  - Has **Project Overview** (Allow the reader to quickly understand what the project is and its purpose).
  - Has **Deep Analysis** (Why of core designs, trade-offs, and comparison with industry).
  - Has **Evaluation and Inspiration** (Honest pros and cons, what readers can learn).
  - Has **Architectural Visualization** (Mermaid charts).
  - All conclusions have code evidence.

### Phase 6: Parallel Deep Analysis (Subagent Team)

Must use Agent tools to launch subagents in parallel. Refer to the prompt template and collaboration standards in [module-analysis-guide.md](references/module-analysis-guide.md).

Every subagent prompt must include the project's overall design philosophy and global perspective requirements to ensure module analysis is not isolated.

Every subagent prompt must also include the narrative context for that module (from Phase 5): what the previous module covered, what questions the reader brings in, and what this module sets up for the next. The subagent should use 1-2 sentences at the start to transition from the previous module and 1 sentence at the end to set up the next.

Every subagent prompt must attach coverage requirements (refer to [module-analysis-guide.md](references/module-analysis-guide.md)), informing them of the analysis mode and minimum coverage target, and requiring a coverage breakdown table at the end of the draft.

**Subagent Writing Strategy**:
For large modules (>5000 lines), require incremental draft writing in the subagent prompt:

  - Write the analysis of a subsystem/sub-module immediately upon completion.
  - Use Write for the first part and Edit to append subsequent parts.
  - Do not wait to finish all files before writing.
  - Append the coverage breakdown at the end.

**Main Agent Waiting Discipline**:

  - After subagents start, the main agent must not read the source files they are responsible for.
  - During waiting, the main agent should focus on: reading project docs, external research, designing report skeleton, and preparing the Phase 8 fusion framework.
  - Standard for a stuck subagent: Output file has no new lines for over 5 minutes. Only then can the main agent take over.
  - **Premature merging is strictly forbidden**: Wait for all subagents to finish before starting Phase 7 and 8. Do not start the final report while subagents are running.

### Phase 7: Cross-Validation + Quality Control (Main Agent)

**7.1 Coverage Gate**:

1.  Read the coverage breakdown at the end of each `drafts/06-module-*.md`.
2.  Quick check: Does every draft have a table? Is the total row marked as passed (✅/❌)?
3.  Only modules marked ❌ or missing a table require deep checking.
4.  Non-compliant modules → Main agent automatically reads unvisited key files and appends findings to the draft.
5.  If still not met → Report to the user which modules failed and why (e.g., file too large, binary files).

**7.2 Spot-Check Verification**:

1.  Select 2-3 key conclusions from each core module draft.
2.  Go back to the source code to verify accuracy line-by-line.
3.  Correct the draft if deviations are found.

**7.3 Cross-Validation**:

1.  Cross-validate conclusions marked [To be verified by main agent].
2.  Synthesize answers to exploration questions and identify cross-module patterns.
3.  Verify global correlation: Does every module analysis connect to the overall design philosophy?
4.  Write to `drafts/07-cross-validation.md`.

### Phase 8: Multi-Source Fusion and Final Report (Main Agent)

1.  Extract architectural insights and systemic design philosophy.
2.  Deepen competitor comparison based on Phase 3 research (supplement with search only if info was insufficient).
3.  Propose "what if redesigned" improvements.
4.  Write to `drafts/08-insights.md`.
5.  **Multi-Source Fusion**: Use the Phase 5 report structure as a skeleton and fill it with content from various drafts. If the same concept appears in multiple drafts, use the most detailed version and supplement unique info from others. After fusion, eliminate all "see draft X" or "refer to appendix" instructions.
      - **Narrative Continuity**: Organize modules according to the Phase 5 narrative line. Each module chapter start must have 1-2 transition sentences connecting from the previous module's conclusion. Avoid stiff transitions like "Next, we analyze X"; use natural transitions instead.
6.  **Segmented Writing**: Final reports usually exceed 500 lines. Write the first few chapters (200-300 lines) first, then use Edit to append, confirming the end position with Read before each append.
7.  **Coverage Summary**: Summarize coverage data into `drafts/08-coverage.md` (Do not include in final report).
      - Extract data directly from subagent drafts; the main agent doesn't need to re-calculate.
      - If the main agent supplemented reading in Phase 7, add those lines to the "Read Lines" of the corresponding module.
      - Summary table format:

| Module | Type | File Count | Effective LOC | Lines Read | Coverage | Passed |
|------|------|--------|-----------|---------|--------|------|
| ... | Core/Secondary | ... | ... | ... | ...% | ✅/❌ |

8.  Compile final report (excluding coverage section).

### Draft File List

Save all intermediate processes to `$WORK_DIR/drafts/`:

| Phase | File |
|------|------|
| 3 | `03-research.md`, `03-plan.md` |
| 5 | `05-modules-plan.md` |
| 6 | `06-module-{name}.md` (Generated by subagent) |
| 7 | `07-cross-validation.md` |
| 8 | `08-insights.md`, `08-coverage.md` |

Write files in chunks, not exceeding 300 lines or 15KB per write.

## Special Scenarios

  - **Ultra-Large Projects (>50,000 lines)**: Prioritize core modules, using Agent for parallel analysis.
  - **Comparative Analysis Mode**: Complete Phases 1-4 for both projects, then design a comparative report structure in Phase 5, adding "Design Decision Comparison" and "Selection Suggestions" to skeleton constraints.

## Output Requirements

1.  Final report is a single markdown file: `$WORK_DIR/ANALYSIS_REPORT.md`.
2.  Extensive use of Mermaid charts to show architecture, flow, and data flow.
3.  Aimed at developers who need to understand business architecture.
4.  Refer to [analysis-guide.md](references/analysis-guide.md) for evaluative thinking frameworks regarding highlights and issues.
5.  Refer to [analysis-guide.md](references/analysis-guide.md) for analysis philosophy and depth standards.
```
