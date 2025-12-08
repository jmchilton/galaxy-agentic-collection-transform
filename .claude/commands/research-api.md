Research Galaxy's tools API and create RESEARCH_API.md.

Read PROBLEM_AND_GOAL.md to understand the context and objectives.

IMPORTANT: Do NOT read previous versions of RESEARCH_API.md from artifacts/research/v*/. Generate fresh research from source materials to enable unbiased comparison between research cycles.

Accept optional argument: path to Galaxy directory (defaults to ~/workspace/galaxy)

Read and summarize:
1. lib/galaxy/webapps/galaxy/api/tools.py - The tools API implementation
2. lib/galaxy_test/api/test_tools.py - API test examples
3. lib/galaxy/webapps/galaxy/api/histories.py - History API (listing, lookup, contents)
4. lib/galaxy/webapps/galaxy/api/jobs.py - Jobs API (status, outputs)
5. https://training.galaxyproject.org/training-material/topics/dev/tutorials/bioblend-api/slides-plain.html - BioBlend API tutorial (if high-level slides without code examples, note "conceptual only" and move on)

Focus on:
- How to invoke tools via the API
- Collection handling in tool API calls
- Input/output formats for collections
- Parameter passing and configuration
- Examples from tests showing real usage patterns
- History API: listing, getting contents, looking up by ID
- Jobs API: checking status, retrieving outputs (especially collection outputs)

IMPORTANT - Galaxy MCP Server (https://github.com/galaxyproject/galaxy-mcp):
- If Galaxy MCP is available, prefer it over direct API calls
- MCP tools: `run_tool`, `get_histories`, `list_history_ids`, `get_history_contents`, `get_job_details`, etc.
- MCP handles input format complexity internally (no need for `values` wrapper)
- Check for MCP availability before falling back to direct API

IMPORTANT - History ID Usage:
- Prefer working with history IDs directly when possible
- If history name/slug lookup is needed: `/api/histories?slug=name` or `?search=partial`
- Galaxy UI requires JavaScript - WebFetch/scraping won't work, must use API

IMPORTANT - Job Output Discovery:
- `/api/jobs/{id}/outputs` may return EMPTY for collection operations
- Use `/api/jobs/{id}?full=true` to get `output_collections` field

IMPORTANT - API Input Format Requirements (from real-world testing):
- Data inputs (collections/datasets) require `{"values": [{"src": "...", "id": "..."}]}` wrapper
- Conditional parameters use pipe notation: `"how|filter_source"` not nested `"how": {"filter_source": ...}`
- Incorrect format causes SILENT failures - Galaxy uses defaults without error
- Verify format by checking `/api/tools/{tool_id}/build` endpoint response structure

IMPORTANT - Map-Over Collection Patterns:
- Document batch processing with `{"batch": true, "values": [...]}`
- Linked vs unlinked: `linked: true` processes corresponding elements, `linked: false` produces cartesian product
- `map_over_type` for nested collections - e.g., map over "paired" within list:paired
- Show examples for each pattern from test_tools.py

IMPORTANT - Include Complete Pipeline Example:
- Show multi-step tool chaining: execute tool → extract output collection ID → use as next input
- Include job polling/waiting between steps
- Demonstrate proper error checking

IMPORTANT - Tool ID Quick Reference:
- Include table mapping common operations to tool IDs
- Categories: filtering, structure transformation, metadata, combination
- Enables fast lookup without cross-referencing other docs

Output: Create/update artifacts/research/v<N>/RESEARCH_API.md including:
1. API endpoint documentation with input format requirements
2. Map-over collection patterns (batch, linked/unlinked, map_over_type)
3. Complete multi-step pipeline example with job waiting
4. Tool ID quick reference table

Version detection:
1. List directories in artifacts/research/ matching v*
2. Find latest version (highest number)
3. If RESEARCH_API.md exists in latest version → create v<N+1>
4. If not exists → use latest version (continuing cycle)
5. Create version directory if needed, write output there
