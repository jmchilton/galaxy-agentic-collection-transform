Research Galaxy's apply rules DSL in detail and create RESEARCH_APPLY_RULES.md.

Read PROBLEM_AND_GOAL.md to understand the context and objectives.

IMPORTANT: Do NOT read previous versions of RESEARCH_APPLY_RULES.md from artifacts/research/v*/. Generate fresh research from source materials to enable unbiased comparison between research cycles.

Fetch and analyze:
1. https://github.com/galaxyproject/galaxy/pull/5819 - The PR introducing apply rules
2. https://github.com/galaxyproject/galaxy/blob/dev/lib/galaxy/util/rules_dsl_spec.yml - The rules DSL specification

Provide detailed documentation on:
- Available rule operations
- Rule syntax and structure
- How to compose rules
- Use cases for each rule type
- Examples from the PR and spec
- How rules transform collections

This is critical documentation - the apply rules DSL is a powerful tool for collection manipulation and needs thorough understanding.

## Required Sections

The output MUST include these sections with substantive content:

### 1. Core Concept: Data + Sources Model
- Explain how collections are converted to tabular representation
- Show the `data` and `sources` structure with concrete example
- Explain the processing flow: Collection → Table → Rules → Mapping → Collection

### 2. Rule Operations (Comprehensive)
For each rule type, include:
- Purpose and parameters
- **Concrete transformation example** showing before/after data state
- Use cases

### 3. Step-by-Step Transformation Trace
Include AT LEAST ONE complex example showing **every intermediate step**:
```
Initial state → After rule 1 → After rule 2 → ... → Final mapping → Result collection
```
This teaches how rules compose together. Show actual data arrays at each step.

### 4. Rule Composition Patterns
Document reusable patterns with names:
- Pattern: Extract and Flatten
- Pattern: Group by Tag
- Pattern: Parse Filename Structure
- Pattern: Create Paired from Separate Lists
Each pattern should have complete rules + mapping + input/output description.

### 5. Common Pitfalls
MUST include warnings about:
- **Column Indices Shift** after remove_columns (indices change!)
- **Invert Logic Confusion** in filters (what invert: true/false actually means)
- **Regex Escaping** (special characters need escaping)
- **Case Sensitivity** in filters
- **Empty Results** when all rows filtered out

### 6. Best Practices
Include guidance on:
- Plan Column Layout (sketch before writing)
- Test Incrementally (add one rule at a time)
- Use allow_unmatched Carefully (fail fast vs silent empty)
- Remove Intermediate Columns before mapping
- Validate with Filters (ensure data quality)

### 7. When to Use / When NOT to Use
**Critical for the project goal of preferring simpler tools first:**
- When to use Apply Rules (complex restructuring, tag-based grouping, regex extraction)
- When NOT to use (simple operations that dedicated tools handle better)
- Include comparison table: Operation | Simple Tool | When to use Apply Rules instead

### 8. Real-World Use Cases
Include at least 3 complete examples from bioinformatics:
- Paired-End RNA-seq (sample_R1.fastq, sample_R2.fastq → list:paired)
- Filter by sample type (remove controls)
- Group by experimental condition (treatment tags)

### 9. Collection Type Determination
Show how output collection type is computed from mapping:
- list_identifiers columns → list nesting depth
- paired_identifier → adds :paired
- Include the actual logic/code if possible

## Style Guidance

- Include "Key Insights" annotations explaining *why* not just *what*
- Show complete transformation traces for complex examples
- Emphasize when simpler tools are preferable (supports project goal)
- Include common regex patterns for bioinformatics filenames

Output: Create/update artifacts/research/v<N>/RESEARCH_APPLY_RULES.md with very detailed rules DSL documentation.

Version detection:
1. List directories in artifacts/research/ matching v*
2. Find latest version (highest number)
3. If RESEARCH_APPLY_RULES.md exists in latest version → create v<N+1>
4. If not exists → use latest version (continuing cycle)
5. Create version directory if needed, write output there
