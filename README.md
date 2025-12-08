# Galaxy Agentic Collection Creation

Teaching Claude to transform Galaxy dataset collections reproducibly using Galaxy's native tools.

## Problem

When Claude interacts with Galaxy collections via API, it can directly manipulate collections (filtering, splitting, reorganizing) - but this bypasses Galaxy's tools, losing reproducibility and workflow compatibility. Operations exist only in conversation context, can't be extracted to workflows, and produce stochastic results on re-execution.

## Goal

Create a Claude slash command that prefers Galaxy's native collection operation tools over direct API manipulation, ensuring all transformations are reproducible, workflow-compatible, and aligned with Galaxy best practices.

## Repository Contents

### Core Documents

**[INIT_PROMPT.md](INIT_PROMPT.md)** - Initial prompt that bootstrapped this repository (Dec 2, 2025)
**[PROBLEM_AND_GOAL.md](PROBLEM_AND_GOAL.md)** - Problem statement and success criteria
**[artifacts/command/galaxy-transform-collection.md](artifacts/command/galaxy-transform-collection.md)** - Final slash command
**[artifacts/summary.md](artifacts/summary.md)** - Research reflection and improvement suggestions

### Research Documents (Generation Order)

Generated via custom `/research-*` slash commands to systematically document Galaxy's collection capabilities:

1. **[RESEARCH_SUMMARY_TRAINING.md](RESEARCH_SUMMARY_TRAINING.md)** - Galaxy Training Network materials
   - Collection fundamentals and types
   - Creation techniques (manual, rule-based, Apply Rules)
   - Workflow integration patterns
   - Best practices for reproducibility

2. **[RESEARCH_TOOLS.md](RESEARCH_TOOLS.md)** - Collection operation tools catalog
   - 26 tools from `lib/galaxy/tools/*.xml`
   - Organized by category (filtering, structure transformation, building, metadata, advanced)
   - Each tool: location, purpose, inputs, outputs, parameters, use cases, examples
   - Tool selection guide

3. **[RESEARCH_API.md](RESEARCH_API.md)** - Galaxy tools API usage
   - POST /api/tools endpoint and payload structure
   - Collection/dataset input formats
   - 17 tool invocation examples
   - Map-over patterns (batch processing)
   - Best practices for API-based operations

4. **[RESEARCH_TESTS.md](RESEARCH_TESTS.md)** - Test suite patterns
   - Real examples from `lib/galaxy_test/api/test_tools.py`
   - Building collections for testing
   - Map-over patterns (linked/unlinked/nested)
   - 7 Apply Rules examples from actual tests
   - Testing patterns summary

5. **[RESEARCH_APPLY_RULES.md](RESEARCH_APPLY_RULES.md)** - Apply Rules DSL deep dive
   - Architecture: Collection → Table → Transform → Collection
   - 20 rule operations (9 column addition, 5 filters, 4 structural, 2 other)
   - 4 mapping operations (list_identifiers, paired_identifier, tags, group_tags)
   - Complete examples from `lib/galaxy/util/rules_dsl_spec.yml`
   - 5 composition patterns, best practices, common pitfalls

## Development Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                     RESEARCH PHASE                          │
│                                                             │
│  /research-training          →  RESEARCH_SUMMARY_TRAINING  │
│       ↓                                                     │
│  /research-tools            →  RESEARCH_TOOLS              │
│       ↓                                                     │
│  /research-api              →  RESEARCH_API                │
│       ↓                                                     │
│  /research-tests            →  RESEARCH_TESTS              │
│       ↓                                                     │
│  /research-apply-rules      →  RESEARCH_APPLY_RULES        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  COMMAND GENERATION                         │
│                                                             │
│  /build-command                                             │
│    • Reads all RESEARCH_* docs (~60k tokens)                │
│    • Applies requirements                                   │
│    • Generates galaxy-transform-collection.md               │
│    • Creates summary.md with reflections                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    ITERATION CYCLE                          │
│                                                             │
│  1. Use command → identify gaps                             │
│  2. /research-{topic} → fill gaps                           │
│  3. /build-command → regenerate with new research           │
│  4. Test & validate                                         │
│  5. Repeat                                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Research Phase

Each research command:
1. Read PROBLEM_AND_GOAL.md for context
2. Fetched/read source materials (training docs, Galaxy source code, test suites)
3. Created structured markdown with examples and best practices
4. Output optimized to teach Claude about Galaxy's native approaches

**Key insight:** Research progressed from high-level concepts (training) → tool catalog → API mechanics → real test patterns → detailed DSL specification. This layered approach built comprehensive understanding.

### Command Generation

The `/build-command` slash command:
1. Ingested all 6 research documents (~60k tokens)
2. Applied requirements: self-contained, prefer tools first then Apply Rules, handle missing metadata properly
3. Generated comprehensive slash command with:
   - Complete tool reference (26 tools + Apply Rules)
   - Decision framework (assess metadata → select tools → implement → explain)
   - API invocation patterns
   - Missing metadata strategies (tabular upload preferred, mirror collection as last resort)
   - Example interactions
   - Critical rules (never manipulate directly)

## Using the Command

The generated command is at `artifacts/command/galaxy-transform-collection.md`. To use:

1. **Copy to Claude Code:** Place in `.claude/commands/` directory
2. **Invoke:** `/galaxy-transform-collection "transform description"`
3. **Claude will:**
   - Assess available metadata in collection
   - Select appropriate Galaxy tool(s)
   - Implement via native tools (not direct API)
   - Explain reproducibility benefits

**Example:**
```
/galaxy-transform-collection "Remove control samples and group by treatment condition"
```

Claude will use `__FILTER_FROM_FILE__` and `__APPLY_RULES__` tools, upload any needed metadata files, and explain how the transformation is reproducible.

## Iteration and Improvement

### Current Limitations

- No validation/testing of command effectiveness
- Limited edge case coverage (record collections, deeply nested structures)
- Missing workflow extraction guidance
- No error recovery patterns
- Very long single file (~7500 lines)

### Improvement Strategy

**Phase 1: Validation**
- Test command against real Galaxy instance
- Collect 10-20 actual user transformation requests
- Validate tool selection matches expert recommendations
- Measure success rate

**Phase 2: Refinement**
- Add missing patterns from validation failures
- Include error message catalog with solutions
- Add workflow extraction how-to
- Split into modular sections (reference vs. examples vs. decision logic)

**Phase 3: Enhancement**
- Add tool version handling
- Include performance benchmarks
- Expand edge case coverage
- Add interactive refinement patterns
- Create validation checklist for transformations

**Phase 4: Community**
- Gather user feedback
- Track common modification patterns
- Build example library from real usage
- Version and maintain over time

### How to Iterate

1. **Identify gaps:** Use command, note what's missing/unclear
2. **Research gaps:** Run targeted `/research-*` commands for specific areas
3. **Update command:** Regenerate with `/build-command` incorporating new research
4. **Test:** Validate improvements solve the gap
5. **Document:** Update this README with learnings

> **⚠️ Making Corrections**
>
> Never edit generated artifacts directly (`RESEARCH_*.md`, `artifacts/command/*.md`). Fixes will be lost on regeneration.
>
> Instead, update the **source command** in `.claude/commands/`:
> - API format issues → edit `research-api.md`
> - Tool documentation → edit `research-tools.md`
> - Final command issues → edit `build-command.md` or the relevant research command
>
> Then regenerate: `/research-*` → `/build-command`

**Efficient iteration pattern:**
```bash
# Research specific gap
/research-edge-cases "How does Galaxy handle record collections?"

# Regenerate with all research
/build-command  # Reads all RESEARCH_* + new research

# Test and refine
# ... validate, repeat as needed ...
```

### Metrics for Success

- **Coverage:** % of transformation types handled correctly
- **Tool selection accuracy:** Does Claude choose the right tool?
- **Reproducibility:** Are all operations workflow-extractable?
- **User satisfaction:** Can users accomplish goals without frustration?

## Research Efficiency Notes

**What worked well:**
- Systematic progression (concepts → catalog → API → tests → DSL)
- Direct source material (Galaxy code, tests, training)
- Structured output format (consistent across documents)

**Could improve:**
- Use grep/glob before full file reads to reduce token usage
- Avoid research overlap (tools documented in multiple places)
- Cache common patterns
- Progressive detail loading (summary first, detail on demand)

**Token usage:** ~102k total (95k research + 7k generation)

## Contributing

To extend this work:

1. **Add research:** Create `/research-{topic}` commands for new areas
2. **Test command:** Use with real Galaxy instances, document gaps
3. **Submit issues:** Describe what's missing/incorrect
4. **Propose improvements:** Via PRs to research docs or command

## License

MIT License - see [LICENSE](LICENSE) file for details

## Acknowledgments

- Galaxy Project for comprehensive tools and documentation
- Galaxy Training Network for excellent educational materials
- Galaxy test suite for real-world usage patterns
