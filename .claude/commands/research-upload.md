Research Galaxy's modern data fetch/upload API and create RESEARCH_UPLOAD.md.

Read PROBLEM_AND_GOAL.md to understand the context and objectives.

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

Output: Create/update RESEARCH_UPLOAD.md with comprehensive upload/fetch API documentation.
