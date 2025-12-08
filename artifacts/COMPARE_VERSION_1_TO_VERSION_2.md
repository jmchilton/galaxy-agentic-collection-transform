# Comparison: V1 (Sonnet) vs V2 (Opus) Command Versions

## Overview

This document compares two versions of the `galaxy-transform-collection.md` slash command:
- **V1**: Generated with Sonnet model using v1 research documents (`artifacts/old/galaxy-transform-collection.md`)
- **V2**: Generated with Opus model using v2 research documents (`artifacts/command/galaxy-transform-collection.md`)

The comparison is informed by real-world failures documented in [LOG.md](https://github.com/nekrut/claude-projects/blob/914ce3b450351ccd2bcbf4e632cb1926601b64c6/rnaseq/johns-collection-test/LOG.md).

---

## Structural Differences

| Aspect | V1 (Sonnet/old) | V2 (Opus/new) |
|--------|-----------------|---------------|
| Length | ~830 lines | ~450 lines |
| Organization | Role + Tools + Decision Framework + Examples | Core Principle + Decision Framework + Strategy Tiers + Reference |
| Tool coverage | 26 tools listed inline | 20 tools in table format |
| Apply Rules depth | 20 rule types with detailed descriptions | 15 rule types in compact tables |
| Strategy structure | 2-tier (Option 1: Upload metadata, Option 2: Mirror collection) | 4-tier (A: Simple Tools → B: Apply Rules → C: Upload Metadata → D: Mirror Collection) |

---

## Real-World Failures from LOG.md

The LOG.md documented several failures when using the V1 command:

1. **Wrong input structure** - No `values` wrapper caused tool to use defaults, producing empty collections
2. **Empty job outputs** - `/api/jobs/{id}/outputs` returned empty for collection operations
3. **Finding collection outputs** - Required `?full=true` parameter not documented
4. **Pipe notation** - Conditional parameters needed `how|filter_source` format

---

## Detailed Comparison by Pitfall

### 1. Input Format Requirements (THE CRITICAL PITFALL)

The LOG.md failure was caused by using wrong input structure - missing `values` wrapper.

**V1 (Sonnet):**
```python
# Shows only simple format
{"src": "hdca", "id": "collection_id_string"}
```
The `values` wrapper pattern appears buried in "Map-Over Pattern" section at line ~805, not prominently featured.

**V2 (Opus):**
```python
# Explicitly shows multiple patterns upfront in API Usage Reference:

# Collection input:
{"src": "hdca", "id": "encoded_collection_id"}

# Batch/map-over collection:
{
    "batch": true,
    "values": [{"src": "hdca", "id": "collection_id"}]
}

# Conditional parameters use pipe notation:
{"how|filter_source": {"src": "hda", "id": "file_id"}}
```

**Assessment:** V2 is better - explicitly documents the `values` wrapper pattern earlier and more prominently. However, neither version explicitly shows the FILTER_FROM_FILE input structure that caused the failure.

---

### 2. Job Output Discovery (`full=true` requirement)

The LOG.md failure: `/api/jobs/{id}/outputs` returned empty for collection operations.

**V1:** No mention of `full=true` parameter.

**V2:** Documents this in RESEARCH_API (which informed the command), but the command itself only shows:
```python
# Wait for Job Completion
while True:
    job = GET /api/jobs/{job_id}
    ...
```
Does NOT include `?full=true` guidance in the command itself.

**Assessment:** Gap in both versions - neither explicitly warns about this gotcha.

---

### 3. Pipe Notation for Conditional Parameters

The LOG.md showed issues with nested conditional structures.

**V1:** Example shows nested structure without explanation:
```python
"how": {
    "how_filter": "remove_if_absent",
    "filter_source": {...}
}
```

**V2:** Explicitly warns about pipe notation:
```python
# Conditional parameters use pipe notation:
{"how|filter_source": {"src": "hda", "id": "file_id"}}
```

**Assessment:** V2 is better - explicitly calls out this pattern.

---

## Pitfall Coverage Summary

| Pitfall | V1 Coverage | V2 Coverage | Will V2 Fix It? |
|---------|-------------|-------------|-----------------|
| Wrong input structure (no `values` wrapper) | Buried at line ~805 | More prominent in API section | **Partially** - better placement but still not explicit for FILTER_FROM_FILE |
| `/api/jobs/{id}/outputs` returns empty | Not covered | Not in command (only in research) | **No** - same gap |
| Finding collection outputs | Not explicit | Not explicit | **No** - same gap |
| Pipe notation for conditionals | Implicit in example | Explicit warning | **Yes** - clear improvement |

---

## Strengths and Weaknesses

### V1 Strengths
- More comprehensive tool documentation (26 tools)
- More detailed Apply Rules examples
- Better "Response Template" guidance for Claude's output format
- More thorough "Critical Rules" section with explicit DO/DON'T
- Longer examples with step-by-step explanations

### V1 Weaknesses
- Critical API patterns buried deep in document
- `values` wrapper not prominent
- Missing `full=true` job query guidance
- No explicit strategy hierarchy
- Verbose - may exceed context limits in some scenarios

### V2 Strengths
- Cleaner 4-tier strategy framework (A→B→C→D)
- API patterns more prominent
- Explicit pipe notation warning
- More concise (less cognitive load)
- Anti-patterns section explicitly lists what NOT to do
- Tables for quick reference

### V2 Weaknesses
- Less detailed tool documentation
- Fewer Apply Rules examples
- Less prescriptive output formatting guidance
- Still missing `full=true` guidance
- May lack detail for edge cases

---

## Performance Predictions

### V1 Expected Behavior
- Claude will have detailed tool knowledge but may miss API format requirements
- Likely to produce well-structured responses with clear explanations
- May still fail on `values` wrapper requirement (buried documentation)
- Will not know about `full=true` for job queries

### V2 Expected Behavior
- Claude will follow clearer strategy hierarchy
- More likely to use correct pipe notation
- Better awareness of `values` wrapper (more prominent)
- Still likely to fail on job output retrieval
- May produce less detailed explanations

---

## Verdict

**V2 is marginally better** for the specific failures in LOG.md due to:
- More prominent API pattern documentation
- Explicit pipe notation warning
- Cleaner strategy hierarchy
- Concise format reduces cognitive overload

However, **neither version fully addresses the root cause** - the `values` wrapper requirement for filter tools and the `full=true` job query need explicit, prominent examples.

---

## Recommendations to Fix Remaining Gaps

To truly address the LOG.md failures, either version would need:

### 1. Explicit FILTER_FROM_FILE Example

Add a prominent, correct example:
```python
# FILTER_FROM_FILE - CORRECT INPUT STRUCTURE
inputs = {
    "input": {"values": [{"src": "hdca", "id": collection_id}]},
    "how|filter_source": {"batch": True, "values": [{"src": "hda", "id": filter_file_id}]}
}
```

### 2. Job Output Retrieval Guidance

Add explicit warning:
```python
# CRITICAL: Collection operations may return empty from /outputs endpoint
# Always use full=true:
job = GET /api/jobs/{job_id}?full=true
collection_id = job["output_collections"][0]["id"]
```

### 3. Silent Failure Warning

Add prominent warning (from RESEARCH_API but not in either command):
> **WARNING:** Galaxy API may silently use defaults when input format is incorrect.
> - Missing `values` wrapper → Tool may run with no input
> - Wrong conditional syntax → Defaults used instead
> - Invalid parameter names → Ignored
>
> **Always verify** tool execution by checking output collection contents, not just job status.

### 4. Common Pitfalls Section

Add dedicated section at the top of either command:
```markdown
## Common Pitfalls (READ FIRST)

1. **Data inputs require `values` wrapper** for most tools
2. **Use `?full=true`** when querying job status for collection outputs
3. **Conditionals use pipe notation**: `how|filter_source`, not nested objects
4. **Galaxy fails silently** - always verify output contents
```

---

## Conclusion

V2 represents an incremental improvement over V1, particularly in API pattern prominence and strategy organization. However, the critical failures from LOG.md would likely still occur with either version because:

1. The specific `values` wrapper requirement for FILTER_FROM_FILE is not explicitly shown
2. The `full=true` requirement for job queries is documented in research but not in the command
3. Neither version has a "Common Pitfalls" section that surfaces these issues prominently

A V3 should incorporate the recommendations above, potentially combining V1's detailed examples with V2's cleaner structure and adding explicit pitfall documentation.
