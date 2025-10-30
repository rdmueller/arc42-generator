# Development Plan: arc42-generator (refactor-implement branch)

*Generated on 2025-10-30 by Vibe Feature MCP*
*Workflow: [epcc](https://mrsimpson.github.io/responsible-vibe-mcp/workflows/epcc)*

## Goal
Refactor the arc42-generator project to remove Gradle as the build system and replace it with an alternative solution that:
- Maintains all current functionality (template generation, format conversion, distribution packaging)
- Is simpler to understand and maintain
- Reduces complexity for contributors
- Remains cross-platform compatible

## Explore

### Phase Entrance Criteria
*This is the initial phase - no entrance criteria needed*

### Tasks
- [ ] Document current Gradle responsibilities and features used
- [ ] Analyze dependencies on Gradle-specific features (plugins, task system, etc.)
- [ ] Research alternative build tools (Make, Python, shell scripts, etc.)
- [ ] Evaluate complexity of dynamic subproject generation without Gradle
- [ ] Identify technical constraints and requirements
- [ ] Review existing documentation for insights

### Completed
- [x] Created development plan file
- [x] Read comprehensive project documentation (Solution Strategy, Building Blocks, Concepts)
- [x] Understood current architecture and build pipeline
- [x] Read build.gradle, buildconfig.groovy, settings.gradle, subBuild.gradle
- [x] Documented current Gradle responsibilities and features used
- [x] Analyzed dependencies on Gradle-specific features (plugins, task system, dynamic subprojects)
- [x] Researched alternative build tools (Make, Python, shell scripts, Groovy standalone)
- [x] Evaluated complexity of dynamic subproject generation without Gradle
- [x] Identified technical constraints and requirements

## Plan

### Phase Entrance Criteria
- [x] All Gradle features and responsibilities are documented
- [x] Alternative build tools have been evaluated with pros/cons
- [x] Technical feasibility of refactoring has been assessed
- [x] Key architectural decisions identified (e.g., keep dynamic behavior vs. static configuration)
- [x] User confirms understanding of scope and constraints

### New File Structure

```
arc42-generator/
├── build.groovy                    # Main entry point (replaces ./gradlew)
├── buildconfig.groovy              # UNCHANGED - existing config
├── lib/                            # New: Groovy helper classes
│   ├── Templates.groovy           # Phase 1: Golden Master filtering
│   ├── Discovery.groovy           # Phase 2: Target discovery
│   ├── Converter.groovy           # Phase 3: Format conversion
│   ├── Packager.groovy            # Phase 4: Distribution creation
│   └── VersionLoader.groovy       # Utility: Load version.properties
├── build-arc42.sh                  # UPDATE: Call groovy instead of gradle
└── [Keep existing]
    ├── arc42-template/             # Submodule - unchanged
    └── buildconfig.groovy          # Config - unchanged
```

### Architecture Design

#### Component: build.groovy (Main Orchestrator)
**Responsibility**: Entry point, command parsing, phase orchestration

**Interface**:
```groovy
#!/usr/bin/env groovy
// Usage: groovy build.groovy [command] [options]
// Commands: generate, convert, package, all, clean
```

**Key Functions**:
- Parse CLI arguments
- Load buildconfig.groovy using ConfigSlurper
- Instantiate helper classes
- Orchestrate build phases
- Handle errors and provide clear output

#### Component: lib/Templates.groovy
**Responsibility**: Phase 1 - Generate template variants from Golden Master

**Key Methods**:
```groovy
class Templates {
    void createFromGoldenMaster(List<String> languages)
    private void generateTemplateStyle(String lang, String style, List features)
    private void copyCommonFiles(String language)
    private void copyImages(String language, String style)
}
```

**Implementation Notes**:
- Reuse exact regex patterns from build.gradle:88-94
- Copy logic from build.gradle:56-120
- Output to: build/src_gen/{LANG}/asciidoc/{STYLE}/

#### Component: lib/Discovery.groovy
**Responsibility**: Phase 2 - Scan generated templates, discover build targets

**Key Methods**:
```groovy
class Discovery {
    List<Map> scanGeneratedTemplates()
    Map loadVersionProperties(String language)
}
```

**Returns**: List of maps with structure:
```groovy
[
    [language: 'EN', style: 'plain', path: 'build/src_gen/EN/asciidoc/plain', version: [...]],
    [language: 'EN', style: 'with-help', path: '...', version: [...]],
    // ...
]
```

**Implementation Notes**:
- Scan build/src_gen/ for directories containing 'src' subfolder
- Extract language/style from path (same logic as settings.gradle:41-42)
- Load version.properties for each language

#### Component: lib/Converter.groovy
**Responsibility**: Phase 3 - Convert templates to all output formats

**Dependencies**:
- AsciidoctorJ library (via @Grab or manual classpath)
- Pandoc CLI tool (external)

**Key Methods**:
```groovy
class Converter {
    void convertAllTargets(List<Map> targets, boolean parallel = true)
    private void convertTarget(Map target)
    private void generateHTML(Map target)
    private File generateDocBook(Map target)
    private void convertWithPandoc(File docbookFile, String format, Map target)
}
```

**Parallel Execution Strategy**:
```groovy
import groovyx.gpars.GParsPool

// Option 1: GPars (if available)
GParsPool.withPool(8) {
    targets.eachParallel { target ->
        convertTarget(target)
    }
}

// Option 2: Java ExecutorService (fallback)
import java.util.concurrent.*
def executor = Executors.newFixedThreadPool(8)
def futures = targets.collect { target ->
    executor.submit({ convertTarget(target) } as Callable)
}
futures*.get()
executor.shutdown()
```

**AsciidoctorJ Integration**:
```groovy
@Grab(group='org.asciidoctor', module='asciidoctorj', version='2.5.10')
@Grab(group='org.asciidoctor', module='asciidoctorj-api', version='2.5.10')

import org.asciidoctor.*

class Converter {
    Asciidoctor asciidoctor

    Converter() {
        this.asciidoctor = Asciidoctor.Factory.create()
    }

    void generateHTML(Map target) {
        def options = Options.builder()
            .backend('html5')
            .toDir(new File("build/${target.language}/html/${target.style}"))
            .safe(SafeMode.UNSAFE)
            .attributes([
                'toc': 'left',
                'doctype': 'book',
                'icons': 'font',
                'sectlink': true,
                'sectanchors': true,
                'numbered': true,
                'revnumber': target.version.revnumber,
                'revdate': target.version.revdate,
                'revremark': target.version.revremark,
                'imagesdir': 'images'
            ])
            .build()

        asciidoctor.convertFile(
            new File("${target.path}/src/arc42-template.adoc"),
            options
        )
    }
}
```

#### Component: lib/Packager.groovy
**Responsibility**: Phase 4 - Create ZIP distributions

**Key Methods**:
```groovy
class Packager {
    void createDistributions(List<Map> targets)
    private void createZip(String language, String style, String format)
}
```

**Implementation**:
```groovy
import java.util.zip.*

void createZip(String lang, String style, String format) {
    def zipName = "arc42-template-${lang}-${style}-${format}.zip"
    def outputPath = "arc42-template/dist/${zipName}"
    def sourcePath = "build/${lang}/${format}/${style}"

    new File("arc42-template/dist").mkdirs()

    def zipFile = new ZipOutputStream(new FileOutputStream(outputPath))
    new File(sourcePath).eachFileRecurse { file ->
        if (file.isFile()) {
            def entryName = file.path - sourcePath - '/'
            zipFile.putNextEntry(new ZipEntry(entryName))
            file.withInputStream { zipFile << it }
            zipFile.closeEntry()
        }
    }
    zipFile.close()
}
```

### Migration Strategy

#### Phase 1: Proof of Concept (Low Risk)
**Goal**: Validate approach with minimal impact

**Tasks**:
1. Create lib/ directory structure
2. Implement Templates.groovy (Phase 1 only)
3. Create minimal build.groovy that only runs Phase 1
4. Test: Compare output of `groovy build.groovy generate` vs `./gradlew createTemplatesFromGoldenMaster`
5. Validation: Diff build/src_gen/ contents between Gradle and Groovy runs

**Success Criteria**:
- Generated .adoc files are byte-for-byte identical
- All languages and styles generated correctly
- Common files and images copied correctly

**Rollback**: Delete lib/ and build.groovy, continue using Gradle

#### Phase 2: Core Conversion (Medium Risk)
**Goal**: Implement full conversion pipeline

**Tasks**:
1. Implement Discovery.groovy
2. Implement Converter.groovy with AsciidoctorJ
3. Set up @Grab dependencies or manual JAR management
4. Implement Pandoc integration
5. Implement parallel execution
6. Update build.groovy to run all phases
7. Test with single language (EN) and single format (HTML)
8. Expand to all languages and formats

**Success Criteria**:
- HTML output matches Gradle version
- DocBook intermediate files match
- All 15+ formats generate successfully
- Parallel execution works (faster than sequential)

**Validation Approach**:
```bash
# Generate with both systems
./gradlew clean createTemplatesFromGoldenMaster arc42
groovy build.groovy clean all

# Compare outputs
diff -r build-gradle/ build-groovy/
```

**Rollback**: Keep Gradle files, use Groovy as alternative

#### Phase 3: Packaging & Cleanup (Low Risk)
**Goal**: Complete migration and remove Gradle

**Tasks**:
1. Implement Packager.groovy
2. Test ZIP creation
3. Update build-arc42.sh to use Groovy
4. Update documentation (CLAUDE.md, README.md, docs/)
5. Create migration guide for contributors
6. Remove Gradle files (build.gradle, settings.gradle, subBuild.gradle, gradlew, gradle/)
7. Update .gitignore if needed

**Success Criteria**:
- Distribution ZIPs match Gradle version
- build-arc42.sh works end-to-end
- Documentation accurate
- No Gradle references remain

### Risk Analysis & Mitigation

#### Risk 1: AsciidoctorJ API Differences
**Impact**: Medium | **Likelihood**: Medium

**Description**: AsciidoctorJ API might not support all Asciidoctor Gradle plugin features

**Mitigation**:
- Study AsciidoctorJ documentation thoroughly during implementation
- Test with small samples first
- Compare HTML output carefully
- Fall back to CLI if needed: `"asciidoctor -b html5 ...".execute()`

#### Risk 2: Parallel Execution Issues
**Impact**: Low | **Likelihood**: Medium

**Description**: Race conditions or resource contention with 20+ parallel conversions

**Mitigation**:
- Start with sequential execution, add parallelization later
- Use thread-safe operations (separate output directories per target)
- Test with different thread pool sizes
- Add error handling per-target (one failure doesn't crash all)

#### Risk 3: Cross-Platform Compatibility
**Impact**: Medium | **Likelihood**: Low

**Description**: Groovy scripts might behave differently on Windows vs Linux

**Mitigation**:
- Use File.separator instead of hardcoded '/'
- Test on Windows (if possible)
- Use ProcessBuilder for better cross-platform subprocess control
- Document platform-specific requirements

#### Risk 4: Performance Degradation
**Impact**: Low | **Likelihood**: Low

**Description**: Groovy scripts might be slower than Gradle

**Mitigation**:
- Profile both implementations
- Optimize hot paths (regex, file I/O)
- Parallel execution should compensate
- Accept slightly slower builds if necessary (simplicity > speed)

#### Risk 5: Dependency Management
**Impact**: Medium | **Likelihood**: Low

**Description**: @Grab might not work in all environments, manual JARs cumbersome

**Mitigation**:
- Document both approaches (@Grab and manual classpath)
- Provide pre-downloaded JARs in repository if needed
- Test @Grab in CI/CD environment
- Fall back to CLI tools if library integration fails

### Testing & Validation Strategy

#### Unit-Level Testing
- Test Templates.groovy regex independently with sample AsciiDoc
- Test Discovery.groovy with mock directory structure
- Test Packager.groovy ZIP creation

#### Integration Testing
- Run both Gradle and Groovy pipelines in parallel
- Binary diff of outputs:
  - build/src_gen/ structure and contents
  - build/{LANG}/{FORMAT}/ outputs
  - arc42-template/dist/*.zip files
- Validate ZIP contents can be extracted and used

#### Acceptance Criteria
1. All 4 languages generate successfully
2. All 15+ formats convert without errors
3. Distribution ZIPs byte-identical (or content-identical)
4. Build time comparable (within 20% of Gradle)
5. Error messages clear and helpful
6. Documentation accurate

### Tasks

#### Planning Tasks
- [ ] Review and refine this detailed plan
- [ ] Identify any missing edge cases
- [ ] Get user approval for approach

### Completed
*None yet*

## Code

### Phase Entrance Criteria
- [ ] Detailed refactoring plan has been created and approved
- [ ] Alternative build system has been selected
- [ ] Migration strategy is documented
- [ ] Risk mitigation approach is defined
- [ ] User confirms plan and is ready to proceed

### Tasks

#### Migration Phase 1: Proof of Concept
- [ ] Create lib/ directory structure
- [ ] Implement language auto-discovery in Templates.groovy (scan arc42-template/ for [A-Z]{2} dirs)
- [ ] Implement lib/Templates.groovy with Golden Master filtering logic
- [ ] Port regex patterns from build.gradle:88-94
- [ ] Implement copyCommonFiles() method
- [ ] Implement copyImages() method with style-specific logic
- [ ] Create minimal build.groovy entry point for 'generate' command
- [ ] Test with single language (EN)
- [ ] Test with all discovered languages (should auto-detect DE, EN, FR, CZ)
- [ ] Validate: Diff build/src_gen/ against Gradle output
- [ ] Fix any discrepancies
- [ ] Verify auto-discovery works (add/remove language dir and see it reflected)

#### Migration Phase 2: Core Conversion
- [ ] Implement lib/Discovery.groovy for target scanning
- [ ] Implement version.properties loading
- [ ] Set up AsciidoctorJ dependencies (@Grab or manual)
- [ ] Implement lib/Converter.groovy skeleton
- [ ] Implement generateHTML() with AsciidoctorJ
- [ ] Test HTML generation for EN:plain
- [ ] Implement generateDocBook() with AsciidoctorJ
- [ ] Implement convertWithPandoc() for DOCX format
- [ ] Test DOCX generation for EN:plain
- [ ] Implement remaining format conversions (Markdown, EPUB, etc.)
- [ ] Handle multi-page formats (markdownMP, mkdocsMP, etc.)
- [ ] Implement copyImages() logic for each format
- [ ] Add parallel execution with GParsPool or ExecutorService
- [ ] Test with all formats for single language
- [ ] Test with all languages and formats
- [ ] Validate: Diff all outputs against Gradle
- [ ] Fix any format-specific issues

#### Migration Phase 3: Packaging & Finalization
- [ ] Implement lib/Packager.groovy for ZIP creation
- [ ] Test ZIP creation for single combination
- [ ] Test ZIP creation for all combinations
- [ ] Validate ZIP contents match Gradle version
- [ ] Update build.groovy to support 'package' and 'all' commands
- [ ] Add 'clean' command to remove build/ directory
- [ ] Add error handling and user-friendly messages
- [ ] Add progress indicators for long-running operations
- [ ] Update build-arc42.sh to call groovy instead of gradle
- [ ] Test full pipeline with build-arc42.sh

#### Documentation Updates
- [ ] Update CLAUDE.md with new build system instructions
- [ ] Update README.md if it exists
- [ ] Update docs/arc42/04-solution-strategy.md (Decision 2 now obsolete)
- [ ] Update docs/arc42/05-building-blocks.md (remove Gradle components)
- [ ] Create migration guide for contributors
- [ ] Document Groovy installation requirements
- [ ] Document new build commands

#### Cleanup
- [ ] Remove build.gradle
- [ ] Remove settings.gradle
- [ ] Remove subBuild.gradle
- [ ] Remove gradlew and gradlew.bat
- [ ] Remove gradle/ directory
- [ ] Update .gitignore (remove Gradle entries, add Groovy if needed)
- [ ] Test that project still builds correctly
- [ ] Final validation: Full build produces correct outputs

### Completed
*None yet*

## Commit

### Phase Entrance Criteria
- [ ] All refactoring implementation is complete
- [ ] Build system works equivalently to Gradle version
- [ ] Tests pass (or equivalent validation performed)
- [ ] Documentation updated to reflect new build system
- [ ] User confirms implementation is acceptable

### Tasks
- [ ] *To be added when this phase becomes active*

### Completed
*None yet*

## Key Decisions

### Decision 1: Use Groovy as Standalone Scripting Language
**Date**: 2025-10-30
**Status**: Approved by user (maintainer: Ralf D. Müller)

**Decision**: Replace Gradle build system with standalone Groovy scripts that orchestrate the same build pipeline.

**Context from Maintainer** (Q18 in open-questions.md):
> "Gradle will be removed in the future" - Ralf D. Müller

**Rationale**:
- **Explicitly planned by maintainer**: Gradle removal already on roadmap
- Team already has Groovy expertise (existing build scripts are Groovy)
- Zero learning curve compared to Python/Make/other alternatives
- `buildconfig.groovy` can remain unchanged
- AsciidoctorJ (Java library) can be used directly instead of CLI subprocess calls
- Same regex patterns and string manipulation capabilities
- Simpler mental model - explicit flow vs. Gradle's lifecycle phases
- Lighter weight execution (no Gradle daemon/wrapper overhead)
- **Internal tool only**: User base is arc42-team only, allows simpler approach

**Why Gradle Was Chosen Originally** (Q5):
> "I was used to gradle and wanted to get a deeper understanding. So I used it for this project." - Ralf D. Müller

This confirms Gradle wasn't chosen for technical superiority, making replacement appropriate.

**Alternatives Considered**:
1. **Python + Jinja2**: More common scripting language, but requires learning new ecosystem
2. **GNU Make**: Standard tool but poor handling of dynamic targets and complex on Windows
3. **Bash scripts**: Too simple for complex logic, cross-platform issues
4. **Keep Gradle**: Rejected - maintainer explicitly plans to remove it

**Trade-offs**:
- Lose automatic incremental builds (acceptable - can rebuild quickly)
- Lose built-in parallel execution (mitigated with GPars or Java ExecutorService)
- Lose Gradle IDE integration (acceptable - simpler debugging)
- Gain explicit control and simpler architecture

### Decision 2: Restore Auto-Discovery of Languages
**Date**: 2025-10-30
**Status**: Confirmed by maintainer

**Decision**: Remove hardcoded language list and restore automatic discovery from directory structure.

**Context from Maintainer** (Q4):
> "This was added four weeks ago when we did a live deployment. So we restricted the list only to the language for which the template has been changed. Can be removed again."

**Implementation**: In Discovery.groovy, scan `arc42-template/` for directories matching `/^[A-Z]{2}$/` pattern.

**Benefits**:
- Adding new languages requires only adding files (no code changes)
- Eliminates maintenance of hardcoded list
- Returns to original design intent (build.gradle:36-39 has auto-discovery code)

### Key Technical Findings

**Current Gradle Features Used**:
1. Build orchestration with task dependencies
2. Dynamic subproject generation (settings.gradle scanning build/src_gen/)
3. Asciidoctor Gradle plugin for AsciiDoc conversion
4. Parallel execution of 20+ subprojects
5. ConfigSlurper for configuration
6. Exec tasks for Pandoc

**Replacement Strategy**:
1. Explicit function calls instead of task dependencies
2. Simple directory scanning + list of (language, style) tuples
3. AsciidoctorJ library (direct Java API calls)
4. GParsPool or ThreadPoolExecutor for parallelization
5. Keep ConfigSlurper (works in standalone Groovy)
6. ProcessBuilder or "cmd".execute() for Pandoc

## Notes

### Plan Summary

This refactoring plan replaces Gradle with standalone Groovy scripts while maintaining 100% functional compatibility. The approach:

**Architecture**:
- 4 Groovy classes in lib/ (Templates, Discovery, Converter, Packager)
- 1 main orchestrator (build.groovy)
- Keep buildconfig.groovy unchanged
- Use AsciidoctorJ library directly instead of Gradle plugin

**Key Benefits**:
- Explicit control flow (no Gradle lifecycle magic)
- Same language (Groovy) - zero learning curve
- Lighter weight (no Gradle daemon/wrapper)
- Easier to understand and debug

**Migration Approach**:
- 3-phase migration with validation at each step
- Run Gradle and Groovy in parallel initially
- Binary diff outputs to ensure correctness
- Rollback possible at each phase

**Timeline Estimate**:
- Phase 1 (Proof of Concept): 1-2 days
- Phase 2 (Core Conversion): 3-5 days
- Phase 3 (Packaging & Cleanup): 1-2 days
- Total: ~1-2 weeks with testing and validation

### Key Insights from Maintainer

**Project Context** (from open-questions.md):
- Created by Ralf D. Müller in 2014
- Sole maintainer: Ralf D. Müller
- **User base: arc42-team only** (internal tool, not public)
- Updates only when needed
- **Gradle will be removed in the future** (confirmed by maintainer)

**Important Finding**: Hardcoded Language List (Q4)
> "This was added four weeks ago when we did a live deployment. So we restricted the list only to the language for which the template has been changed. Can be removed again."

**Action**: Our refactoring should **restore auto-discovery** of languages instead of keeping the hardcoded list in build.gradle:41.

### Open Questions

1. **Groovy Installation**: Should we bundle a Groovy distribution or require system installation?
   - Option A: Require `sdk install groovy` or system package manager
   - Option B: Include groovy-all JAR and provide wrapper script
   - Recommendation: Option A (simpler, arc42-team likely has SDKMAN)
   - **Context**: User base is arc42-team only, can assume technical environment

2. **@Grab vs Manual JARs**: How should we manage AsciidoctorJ dependency?
   - Option A: Use @Grab (automatic download)
   - Option B: Include JARs in lib/dependencies/
   - Recommendation: Try @Grab first, document manual as fallback

3. **Parallel Execution Library**: GPars or standard Java?
   - Option A: GPars (Groovy-native, cleaner syntax)
   - Option B: Java ExecutorService (no extra dependency)
   - Recommendation: Try GPars with @Grab, fallback to ExecutorService

4. **Error Handling Strategy**: How verbose should error messages be?
   - Should we continue on errors (build as much as possible)?
   - Should we fail fast (stop on first error)?
   - Recommendation: Fail fast for Phase 1, continue-on-error for Phase 3 conversions

### Additional Considerations

**Performance**:
- Groovy scripts may have startup overhead vs compiled Gradle
- Parallel execution should compensate
- First run will be slower due to @Grab dependency resolution
- Subsequent runs should be similar to Gradle

**Maintenance**:
- Code will be more explicit and easier to understand
- No Gradle version upgrades to manage
- AsciidoctorJ updates will be manual version bumps
- Consider pinning versions in @Grab for stability

**Testing Strategy**:
- Keep Gradle files temporarily during migration
- Run both systems and diff outputs
- Focus on EN:plain initially (smallest test case)
- Expand to all languages once one works

**Rollback Plan**:
- Phase 1: Simply delete lib/ and build.groovy
- Phase 2: Keep Gradle files as backup
- Phase 3: Git branch allows reverting before push

**Documentation Impact**:
- CLAUDE.md will need significant updates
- Architecture docs need Gradle sections removed
- New "Getting Started" needs Groovy installation steps
- Migration guide for existing contributors

---
*This plan is maintained by the LLM. Tool responses provide guidance on which section to focus on and what tasks to work on.*
