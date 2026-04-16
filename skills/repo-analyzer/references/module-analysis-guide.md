# Module Analysis Guide

## Core Methodology

Divide modules by **business functionality**, not by files or directories. A logical module might span multiple files, and a single file may contain partial implementations of multiple modules.

Analysis depth standard: **The analysis should allow another AI to reproduce the system's design based solely on the report.** Readers should understand the module's design logic, responsibility boundaries, and collaboration methods with other modules, enabling them to participate in architectural discussions.

## Global Perspective Requirements

**The analysis of every module must answer two global questions:**

1. **Role in the overall project**: Why does this module exist? What happens to the system if it's removed? Which core objective of the project does it serve?
2. **Design synergy with other modules**: What is the contract between this and other modules? Is this collaboration pattern consistent with the overall design philosophy of the project?

Analyzing modules in isolation is the most common mistake. A module's design choices are often driven by the constraints of other modules—without understanding these constraints, one cannot understand the design motivation.

## Four Elements of Module Analysis Completeness

The analysis of every core module must cover the following four elements; missing any item renders it incomplete:

1. **Core Data Structures** — Include key interface/type definitions (only those necessary for understanding the design, not all of them).
2. **Execution Flow** — Describe the call chain using text or Mermaid sequence diagrams, noting source file paths and line numbers.
3. **Design Decisions** — Why this solution was chosen over another and what trade-offs were made.
4. **Inter-module Dependencies** — Who calls whom, how data flows, and what state is shared.

Testing standard: If another AI only reads your analysis (without seeing the source code), could it draw the architecture diagram of this module and explain how it works? If not, an element is missing.

**Must include**: Business problems, design logic and architectural patterns, core flows (Mermaid diagrams), collaborative relationships, design trade-offs.
**Does not need to include**: Complete type definitions, all function signatures, all parameter lists, item-by-item descriptions of error enums.

## Module Identification Methods

1. **Business Functionality Perspective** — What core business capabilities does the project provide?
2. **Data Flow Perspective** — What transformation stages does data go through from input to output?
3. **Responsibility Perspective** — Which code needs to be modified together when business requirements change?

## Analysis Depth

- **Core Modules** (innovations, key architectural components): Deep dive into design logic, core flows with diagrams and interpretation, explanation of design decision trade-offs, and clear collaborative relationships.
- **Secondary Modules** (utility functions, standard wrappers): One-sentence responsibility + file path + unique features.

## Parallel Subagent Analysis

Phase 6 must use Agent tools to initiate independent subagents for parallel analysis of each core module.

Scheduling Strategy:
- Each core module → One independent Agent subagent (`subagent_type: "general-purpose"`)
- All secondary modules → Merged into one Agent subagent for batch processing
- All subagents are initiated in parallel within the same message

### Core Module Subagent Prompt Template

```
You are a senior architect performing a deep analysis of the "{Module Name}" module of "{Project Name}".

## Background Information
- Project Positioning: {One-sentence description}
- Overall Architecture: {Brief description of architectural style and core design}
- Project Design Philosophy: {Core design concepts throughout the project}
- Module Position in the System: {Relationship with other modules}
- Narrative Context: {The module's position in the report's narrative line—what the previous module covered, what questions the reader brings into this module, and what this module needs to set up for the next}

## Files to be Analyzed
{File path list}

## Analysis Structure
Describe design intent in natural language. By default, do not expose function names/parameter names/type definitions; include code snippets only if the design is particularly subtle.

1. Role in the project — Why does this module exist? What happens to the system without it?
2. Problem Solved — Business background; what happens to the system if it's missing?
3. Design Logic — Solution and reasoning, discarded alternatives, core design patterns.
4. Core Data Structures — Include key interface/type definitions necessary for understanding the design (not all).
5. Core Business Flow — Mermaid flowcharts + natural language interpretation, noting source file paths and line numbers.
6. Design Synergy with Other Modules — Dependencies, what depends on it, collaboration methods, shared state, and whether this pattern matches the project's overall design philosophy. Mark cross-module conclusions with [To be verified by main agent].
7. Key Design Decisions — 1-3 most important decisions and trade-offs (why this solution was chosen, costs of alternatives).
8. Deep Research Insights — Cost of alternatives, industry comparisons, what if redesigned.
9. Extension Points (if applicable)
10. Highlights and Issues — List of files involved.

## Global Perspective Requirements
Your analysis must place the module within the project's overall context—how design choices serve the overall philosophy, why boundaries are drawn this way, and how changes affect other modules. (See "Global Perspective Requirements" at the top of the file.)

## Related Exploration Questions
{Question list}
Integrate answers into the relevant sections.

## Writing Strategy
For large modules (total file lines > 5000), you must write drafts incrementally:
- Immediately write the analysis of a subsystem/sub-module to the draft file upon completion.
- Use Write for the first subsystem and Edit to append subsequent ones.
- Do not wait to finish reading all files before writing.
- Append the coverage breakdown table at the end.

## Output
Write to {work_dir}/drafts/06-module-{module_name}.md, with each write not exceeding 300 lines.

## Coverage Requirements
Current analysis mode: {Analysis mode}, core module minimum coverage: {Minimum coverage}%.
The end of the draft must include a coverage breakdown table (Format: Filename | Total lines | Lines read | Coverage % | Reason for unread), with a total row at the bottom marked with Passed✅/Failed❌.
"Read" refers to lines actually read via the Read tool. If coverage is not met, you must continue reading until it is.
```

### Secondary Module Batch Prompt Template

```
You are a senior architect performing a batch analysis of secondary modules for "{Project Name}".

## Background Information
- Project Positioning: {One-sentence description}
- Overall Architecture: {Brief}
- Project Design Philosophy: {Core design concepts throughout the project}

## Secondary Modules to be Analyzed
{Module list: Name, hypothesized responsibility, file scope}

## Output for Each Module
1. Responsibility (one sentence)
2. Role in the overall project (one sentence)
3. Implementation method (one sentence)
4. Elaborate if there are unique features
5. List of files involved

Write to {work_dir}/drafts/06-module-secondary.md

## Coverage Requirements
Current analysis mode: {Analysis mode}, secondary module minimum coverage: {Minimum coverage}%.
The end of the draft must include a coverage breakdown table (Format: Filename | Total lines | Lines read | Coverage % | Reason for unread), with a total row at the bottom marked with Passed✅/Failed❌.
"Read" refers to lines actually read via the Read tool. If coverage is not met, you must continue reading until it is.
```

### Subagent Collaboration Standards

- **Analyze only assigned files**, do not overstep.
- **Mark cross-module inferences with [To be verified by main agent]**, for the main agent to cross-validate in Phase 7.
- **Depth over breadth**, prioritize fully explaining a core process.
- **Global perspective**, analyze the module within the project context, explaining how design choices serve the whole.
- **Narrative coherence**, use 1-2 sentences at the start of the draft to explain the relationship with the previous module, and 1 sentence at the end to set up the next.

## Quality Checklist

- [ ] Modules divided by business functionality, not files/directories
- [ ] Every module explains "Role in project" and "Why it was designed this way"
- [ ] Four elements complete: Core data structures, execution flow (with paths/line numbers), design decisions, inter-module dependencies
- [ ] Core flows shown via Mermaid diagrams
- [ ] Key interface/type definitions included (only essentials)
- [ ] No unnecessary function names, parameter names, or type definitions exposed
- [ ] Collaborative relationships clear, shared state noted
- [ ] Every module analysis connects to the overall design philosophy
- [ ] Test: Could another AI draw the module architecture diagram just by reading the analysis (without source)?
