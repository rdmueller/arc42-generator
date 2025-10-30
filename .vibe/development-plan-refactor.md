# Development Plan: arc42-generator (refactor branch)

*Generated on 2025-10-30 by Vibe Feature MCP*
*Workflow: [epcc](https://mrsimpson.github.io/responsible-vibe-mcp/workflows/epcc)*

## Goal
Create comprehensive mental model documentation for the arc42-generator project following Peter Naur's "Programming as Theory Building" principles, using arc42 as the structural framework. The goal is to enable new senior developers to understand not just WHAT the system does, but WHY it exists and HOW to think about it.

## Explore

### Phase Entrance Criteria:
- [x] Initial phase - no prerequisites

### Tasks
*All exploration tasks completed*

### Completed
- [x] Created development plan file
- [x] Analyze codebase structure and build system architecture
- [x] Identify core architectural decisions and their rationale
- [x] Extract implicit knowledge from code patterns
- [x] Understand the Golden Master concept and template generation flow
- [x] Map out the dynamic Gradle subproject generation system
- [x] Identify stakeholders and quality goals
- [x] Document unwritten rules and team conventions

## Plan

### Phase Entrance Criteria:
- [x] Core architectural patterns are understood
- [x] Key design decisions have been identified
- [x] System boundaries and dependencies are mapped
- [x] Implicit knowledge has been extracted from code
- [x] Stakeholders and quality goals are documented

### Tasks
*All planning tasks completed*

### Completed
- [x] Define documentation structure and file organization
- [x] Plan arc42 section content outlines (7 sections + 8 ADRs)
- [x] Identify ADR topics from 8 core architectural decisions
- [x] Design mental model concept hierarchy (5 must-understand concepts)
- [x] Plan onboarding journey structure (4-week path with validation)
- [x] Create LLM knowledge graph structure (concepts, antipatterns, common mistakes)
- [x] List open questions and assumptions to document (7 categories)

### Documentation Structure Plan

```
docs/
├── arc42/
│   ├── 01-introduction.md          # Vision, quality goals, stakeholders, roadmap
│   ├── 03-context.md                # System boundaries, interfaces, neighbor systems
│   ├── 04-solution-strategy.md     # Top 5 architecture decisions with rationale
│   ├── 05-building-blocks.md       # Component landscape and dependencies
│   ├── 08-concepts.md               # ⭐ Mental model map, core metaphor, key concepts
│   ├── 09-decisions/
│   │   ├── README.md                # ADR timeline and evolution phases
│   │   ├── ADR-001-golden-master-pattern.md
│   │   ├── ADR-002-dynamic-subproject-generation.md
│   │   ├── ADR-003-two-stage-conversion-pipeline.md
│   │   ├── ADR-004-feature-flag-system.md
│   │   ├── ADR-005-submodule-architecture.md
│   │   ├── ADR-006-pandoc-as-converter.md
│   │   ├── ADR-007-version-per-language.md
│   │   └── ADR-008-template-style-variants.md
│   └── 11-risks.md                  # Technical risks, constraints, post-mortems
│
├── onboarding/
│   ├── journey-map.md               # 4-week learning path with validation
│   ├── development-workflow.md     # Feature lifecycle, review checklist
│   ├── team-structure.md            # Knowledge map, code ownership
│   └── common-issues.md             # Troubleshooting patterns
│
├── llm/
│   ├── knowledge-graph.md           # Structured concepts for LLM consumption
│   └── antipatterns.md              # What NOT to do with examples
│
└── open-questions.md                # Missing info, ambiguities, assumptions
```

### Key Content Areas (from exploration)

**Core Metaphor: Template Compiler System**
- Golden Master = source code
- Feature flags = preprocessor directives
- Template variants = compiled outputs
- Languages = build targets
- Pandoc = backend compiler

**Top 5 Architectural Decisions for 04-solution-strategy.md:**
1. Golden Master Pattern → Single source of truth with feature flags
2. Dynamic Subproject Generation → Convention over configuration approach
3. Two-Stage Conversion Pipeline → AsciiDoc → DocBook → Target format
4. Submodule Architecture → Content separation from build logic
5. Pandoc as Universal Converter → Leverage existing tool vs. custom solution

**Must-Understand Concepts for 08-concepts.md:**
1. Golden Master Pattern (why, impact, common mistakes)
2. Dynamic Subproject Generation (chicken-and-egg problem)
3. Feature Flag System (regex-based filtering)
4. Gradle Build Lifecycle (settings.gradle timing)
5. Two-Stage Conversion (DocBook as intermediate)

**ADR Topics (8 total):**
- ADR-001: Golden Master Pattern
- ADR-002: Dynamic Subproject Generation
- ADR-003: Two-Stage Conversion Pipeline
- ADR-004: Feature Flag System
- ADR-005: Submodule Architecture
- ADR-006: Pandoc as Universal Converter
- ADR-007: Version Per Language
- ADR-008: Template Style Variants

**Onboarding Journey (4 weeks):**
- Week 1: Overview - Understand problem space (300+ files → 10 masters), run full build
- Week 2: Fundamentals - Golden Master, feature flags, single language build
- Week 3: Deep dive - Dynamic subprojects, Pandoc integration, add new format
- Week 4: Independence - Add new language, troubleshoot, contribute

**Common Antipatterns:**
1. Editing generated files in build/ directory
2. Running arc42 before createTemplatesFromGoldenMaster
3. Modifying subBuild.gradle directly instead of template
4. Forgetting git submodule update
5. Adding language without updating hardcoded list in build.gradle:41

### Open Questions to Address in Documentation

1. **Historical Context**
   - When was the project created? By whom?
   - What prompted the shift from manual maintenance to generator?
   - Any major architectural refactorings or pivots?
   - Why was req42-framework added as a second template option?

2. **Decision Rationale**
   - Why Gradle over other build tools (Maven, Make)?
   - Why DocBook as intermediate format vs. direct Pandoc AsciiDoc support?
   - Why hardcode language list when auto-discovery code exists?
   - Why regex-based feature removal vs. AsciiDoc preprocessing?
   - Why separate submodules vs. monorepo?

3. **Current State & Roadmap**
   - What's the current version numbering scheme? (9.0 observed)
   - Active maintenance status? Community size?
   - Known issues or limitations?
   - Planned features or improvements?
   - Why is publish/ pointing to rdmueller's fork vs. arc42 org?

4. **Team Structure & Processes**
   - Who has commit access to arc42-template submodule?
   - How are new language contributions reviewed?
   - Release process and cadence?
   - How are breaking changes in arc42 template handled?
   - CI/CD setup (GitHub Actions, etc.)?

5. **Technical Constraints**
   - Why Pandoc 3.7.0.2 specifically? Any known issues with other versions?
   - Minimum supported Gradle version?
   - Java version requirements and testing?
   - Windows/macOS support status?

6. **Failed Experiments (to discover)**
   - Git history mentions "fix common folder" - what went wrong?
   - Why were multi-page formats complex to implement?
   - Any rejected output formats?
   - Alternative approaches tried before current architecture?

7. **Assumptions Made for Documentation**
   - Assuming users have basic Gradle knowledge
   - Assuming familiarity with arc42 template usage
   - Assuming submodule model is understood
   - Need to verify if examples feature (role="arc42example") was ever implemented

## Code

### Phase Entrance Criteria:
- [x] Documentation structure is planned
- [x] Content outline for each arc42 section is defined
- [x] ADR topics are identified
- [x] Mental model concepts are prioritized
- [x] Onboarding journey is mapped out

### Tasks
- [ ] Create documentation directory structure (docs/arc42, docs/onboarding, docs/llm)
- [ ] Write arc42/01-introduction.md (vision, quality goals, stakeholders)
- [ ] Write arc42/03-context.md (system boundaries, interfaces)
- [ ] Write arc42/04-solution-strategy.md (top 5 decisions with rationale)
- [ ] Write arc42/05-building-blocks.md (components and dependencies)
- [ ] Write arc42/08-concepts.md (mental model map, core metaphor, 5 key concepts)
- [ ] Write 8 ADRs in arc42/09-decisions/ directory
- [ ] Write arc42/09-decisions/README.md (ADR timeline)
- [ ] Write arc42/11-risks.md (technical risks, constraints)
- [ ] Write onboarding/journey-map.md (4-week learning path)
- [ ] Write onboarding/development-workflow.md (feature lifecycle, checklists)
- [ ] Write onboarding/team-structure.md (knowledge map, ownership)
- [ ] Write onboarding/common-issues.md (troubleshooting patterns)
- [ ] Write llm/knowledge-graph.md (structured concepts for LLMs)
- [ ] Write llm/antipatterns.md (what NOT to do)
- [ ] Write docs/open-questions.md (missing info, assumptions)

### Completed
*None yet*

## Commit

### Phase Entrance Criteria:
- [ ] All arc42 documentation sections are complete
- [ ] ADRs are written for key decisions
- [ ] Mental model map and concepts are documented
- [ ] Onboarding materials are created
- [ ] LLM knowledge graph is structured
- [ ] Open questions document is complete

### Tasks
- [ ] *To be added when this phase becomes active*

### Completed
*None yet*

## Key Decisions

### Core Architectural Decisions Identified

1. **Golden Master Pattern**: Single-source-of-truth approach using AsciiDoc with feature flags
2. **Dynamic Gradle Subproject Generation**: Build system creates subprojects at runtime based on scanned directories
3. **Two-Stage Conversion Pipeline**: AsciiDoc → DocBook XML → Target Format (for most formats)
4. **Feature Flag System**: Using AsciiDoc `ifdef::arc42help[]` and `[role="arc42help"]` attributes
5. **Submodule Architecture**: Template content separate from generator logic
6. **Pandoc as Universal Converter**: External tool dependency for format transformations
7. **Version Per Language**: Each language has independent version.properties file
8. **Template Style Variants**: "plain" (no help) and "with-help" (includes explanatory text)

### Stakeholders Identified

- **Users**: Software architects/developers documenting systems with arc42
- **Contributors**: Community members improving formats, toolchains, content
- **Founders**: Peter Hruschka & Gernot Starke (ensure conceptual integrity)
- **Maintainers**: Those managing the build system and generator
- **Downstream Users**: Consumers of generated ZIP distributions

### Quality Goals (from requirements doc)

1. **Agnostic**: Works across processes, technologies, domains, system sizes
2. **Easily Usable**: Clear docs, examples, multiple formats/languages, low toolchain requirements
3. **Flexible**: Adaptable to user needs, works with different toolchains

## Notes

### System Boundaries

**IN SCOPE:**
- Template generation and format conversion
- Feature flag processing (help text, examples)
- Multi-language support
- Distribution packaging
- Version management per language

**OUT OF SCOPE:**
- Template content itself (maintained in arc42-template submodule)
- Actual documentation of systems (that's what users do with the templates)
- Safety-critical system requirements
- Formal technical documentation generation

### Implicit Knowledge Discovered

1. **Why two submodules?**: arc42-template (main templates) + req42-framework (requirements variant)
   - switchToreq42.sh script swaps between them
2. **Gradle command shortcuts**: Can abbreviate tasks as long as unambiguous (`cTFGM` for createTemplatesFromGoldenMaster)
3. **Image handling varies by format**: Some formats need images/ folder, others embed
4. **Language list hardcoded**: Despite auto-discovery code, languages manually specified in build.gradle:41
5. **Multi-page variants**: Some formats (markdownMP, mkdocsMP) split into multiple files
6. **Version properties drive document metadata**: revnumber, revdate, revremark substituted into templates
7. **Docker/Gitpod support**: Standardized dev environments with Pandoc pre-installed
8. **GitHub Pages publishing**: Automated through publish/ subproject (currently pointing to rdmueller's fork)

### Technical Debt & Historical Context

- JCenter deprecation addressed (commented out in publish/build.gradle)
- Gradle API modernization: Switched from leftShift operator to doLast
- Archive naming updated to use `.set()` for Gradle 7+ compatibility
- Russian language special handling: Uses fontenc=T1,T2A for LaTeX
- Pandoc version pinned to 3.7.0.2 for consistency

### Unwritten Rules Observed

1. **Always run createTemplatesFromGoldenMaster first**: Other tasks depend on generated structure
2. **settings.gradle dynamically includes subprojects**: Must run after template generation
3. **Common files copied per language**: Ensures each language build is self-contained
4. **Placeholder replacement in subBuild.gradle**: %LANG%, %TYPE%, %REVNUMBER%, etc. substituted
5. **Distribution ZIPs committed to submodule**: Final artifacts pushed back to arc42-template repo
6. **Build dir is ephemeral**: Always regenerated, never committed

### Build Flow Deep Dive

**Phase 1: createTemplatesFromGoldenMaster**
```
1. Scan arc42-template/ for language directories (DE, EN, FR, CZ, etc.)
2. For each language:
   a. Copy common files (styles, images)
   b. Load version.properties (revnumber, revdate, revremark)
   c. For each template style (plain, with-help):
      - Calculate features to remove (allFeatures - featuresWanted)
      - For each .adoc file:
        * Read file content
        * Remove unwanted features using regex:
          - Remove role blocks: /(?ms)\[role="arc42FEATURE"\][ \r\n]+[*]{4}.*?[*]{4}/
          - Remove ifdef/endif if help removed
        * Write filtered content to build/src_gen/{LANG}/asciidoc/{STYLE}/
   d. Copy appropriate images (only logo for plain, logo+examples for with-help)

Result: build/src_gen/{LANG}/asciidoc/{STYLE}/src/*.adoc
```

**Phase 2: Dynamic Subproject Creation (settings.gradle)**
```
1. Parse buildconfig.groovy
2. Scan build/src_gen/ recursively
3. For each "src/" directory found:
   a. Extract language and docFormat from path
   b. Load version.properties for that language
   c. Copy subBuild.gradle → build.gradle with substitutions:
      - %LANG% → language code
      - %TYPE% → template style (plain/with-help)
      - %REVNUMBER%, %REVDATE%, %REVREMARK% → from version.properties
   d. Register subproject: include("{LANG}:{STYLE}")
   e. Set projectDir to the containing folder

Result: Subprojects like :EN:plain, :DE:with-help, etc.
```

**Phase 3: arc42 Task (runs in each subproject)**
```
For each subproject (:LANG:STYLE):
1. copyAsciidoc → Copy generated AsciiDoc to build/{LANG}/asciidoc/
2. generateHTML → AsciiDoc → HTML5 directly (Asciidoctor plugin)
3. generateDocbook → AsciiDoc → DocBook XML (Asciidoctor plugin)
4. generateDocbookMP → Same but multi-page (splits sections)
5. For each target format (markdown, docx, epub, latex, etc.):
   a. If multi-page: Process each XML file separately
   b. Run pandoc: pandoc -r docbook -t FORMAT -o OUTPUT INPUT
6. copyImages → Copy images/ folder where needed

Result: build/{LANG}/{FORMAT}/{STYLE}/*.{ext}
```

**Phase 4: createDistribution**
```
For each combination of (language × templateStyle × format):
1. Create ZIP task dynamically
2. Archive contents from build/{LANG}/{FORMAT}/{STYLE}/
3. Name: arc42-template-{LANG}-{STYLE}-{FORMAT}.zip
4. Place in: arc42-template/dist/

Result: arc42-template/dist/*.zip ready for publishing
```

### The "Theory" Behind This System (Peter Naur's perspective)

**Core Metaphor**: This is a **Template Compiler** system
- The Golden Master is the "source code"
- Feature flags are "preprocessor directives"
- Each template variant is a "compiled output"
- Languages are "build targets"
- Pandoc is the "backend compiler"

**Why This Architecture?**
1. **Single Source of Truth (DRY)**: Maintain one master, generate many outputs
2. **Separation of Concerns**: Content (submodule) vs. Build Logic (generator)
3. **Convention Over Configuration**: Directory structure drives build process
4. **Late Binding**: Subprojects created dynamically based on what exists
5. **Fail-Safe Approach**: Each language build is self-contained (copied common files)

**What Problem Does It Solve?**
Before this generator, maintaining arc42 templates across:
- 10+ languages
- 15+ output formats
- 2+ style variants
= 300+ files to maintain manually!

This system reduces it to ~10 Golden Master files per language that generate everything.

**The Hidden Complexity**:
- Gradle's build lifecycle requires settings.gradle to run AFTER createTemplatesFromGoldenMaster
  but BEFORE any other tasks can see the subprojects
- This creates a "chicken-and-egg" problem solved by making settings.gradle conditional
  (only includes subprojects if build/src_gen/ exists)
- First run: createTemplatesFromGoldenMaster creates structure
- Subsequent runs: settings.gradle discovers structure and enables all other tasks

---
*This plan is maintained by the LLM. Tool responses provide guidance on which section to focus on and what tasks to work on.*
