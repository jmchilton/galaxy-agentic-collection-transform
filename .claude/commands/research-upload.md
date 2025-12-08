Research Galaxy's modern data fetch/upload API and create RESEARCH_UPLOAD.md.

Read PROBLEM_AND_GOAL.md to understand the context and objectives.

IMPORTANT: Do NOT read previous versions of RESEARCH_UPLOAD.md from artifacts/research/v*/. Generate fresh research from source materials to enable unbiased comparison between research cycles.

Accept optional argument: path to Galaxy directory (defaults to ~/workspace/galaxy)

Read and summarize:
1. lib/galaxy/webapps/galaxy/api/tools.py - Focus on FetchTools class and fetch_json method
2. lib/galaxy_test/base/populators.py - Focus on fetch_payload(), new_dataset_request(), and fetch() methods

Focus on:
- The modern fetch API (prefer over legacy upload1 tool)
- How to upload data from URLs, paste content, or local files
- Creating datasets with specific names and types
- Uploading identifier/text files for use with collection operations
- Request payload structure and required fields
- Examples from test populators showing real usage patterns

IMPORTANT - Galaxy MCP Server (https://github.com/galaxyproject/galaxy-mcp):
- If Galaxy MCP is available, it may provide upload tools - document those if found
- Fall back to direct API patterns when MCP unavailable

Output: Create/update artifacts/research/v<N>/RESEARCH_UPLOAD.md with comprehensive upload/fetch API documentation.

Version detection:
1. List directories in artifacts/research/ matching v*
2. Find latest version (highest number)
3. If RESEARCH_UPLOAD.md exists in latest version → create v<N+1>
4. If not exists → use latest version (continuing cycle)
5. Create version directory if needed, write output there
