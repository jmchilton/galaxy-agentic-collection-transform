Research Galaxy's collection operation tests and create RESEARCH_TESTS.md.

Read PROBLEM_AND_GOAL.md to understand the context and objectives.

Accept optional argument: path to Galaxy directory (defaults to ~/workspace/galaxy)

IMPORTANT: Do NOT read previous versions of RESEARCH_TESTS.md from artifacts/research/v*/. Generate fresh research from source materials to enable unbiased comparison between research cycles.

NOTE: This command intentionally reads RESEARCH_TOOLS.md and RESEARCH_API.md from the CURRENT cycle (same version directory) as within-cycle dependencies. This is different from reading old versions of itself.

First read from artifacts/research/<latest_version>/:
1. RESEARCH_TOOLS.md - Previously researched tool information (current cycle)
2. RESEARCH_API.md - Previously researched API information (current cycle)

(Find latest version by listing artifacts/research/v* directories)

Use prior research as context - don't re-explain tools, just show test usage patterns.

Then read lib/galaxy_test/api/test_tools.py from Galaxy and summarize:
- Test examples for collection operations
- How apply rules are tested and used
- Real-world usage patterns
- Input/output structures for collection manipulation
- Edge cases and best practices demonstrated in tests

Focus particularly on apply rules examples and collection operation testing patterns.

## Required Sections

The output MUST include these sections with substantive content:

### 1. Map-Over Patterns (Batch Processing)
- Simple map-over with `batch: True`
- **Linked vs Unlinked mapping** - show `linked: False` for Cartesian product
- `map_over_type` for nested collections
- Show output structure differences (e.g., `paired:paired` from unlinked)

### 2. Apply Rules Examples
For EACH example, include:
- The rules JSON structure
- Input collection description
- Output collection description
- **"Rule Explanation" block** explaining what each rule accomplishes and why

Include at minimum: identity transform, add nesting, flatten with concatenation, group by tags, flatten using indices.

### 3. Asynchronous Operations & Best Practices
- `wait_for_history` before operations
- `wait_for_job` for async job completion
- Checking `populated_state` for implicit collections
- Why these matter for reliable operations

### 4. Edge Cases and Error Handling
- Incompatible collection types in linked mapping (paired vs list → error)
- Invalid indices in extract dataset
- Invalid paired identifiers in Apply Rules
- How tests verify error conditions

### 5. Key Takeaways / Testing Philosophy
End with explicit statement that:
- Galaxy's own tests NEVER use direct API manipulation for collection metadata
- All collection ops use native tools even when called via API
- This ensures reproducibility, testability, workflow compatibility

## Style Guidance

- Include "Key Insights" annotations after code examples explaining *why* not just *what*
- Show complete test patterns, not just API calls
- Emphasize patterns that reinforce tool-based over direct-API approach

Output: Create/update artifacts/research/v<N>/RESEARCH_TESTS.md with comprehensive test-based usage examples.

Version detection:
1. List directories in artifacts/research/ matching v*
2. Find latest version (highest number)
3. If RESEARCH_TESTS.md exists in latest version → create v<N+1>
4. If not exists → use latest version (continuing cycle)
5. Create version directory if needed, write output there
