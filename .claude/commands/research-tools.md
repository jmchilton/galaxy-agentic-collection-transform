Research Galaxy's collection operation tools and create RESEARCH_TOOLS.md.

Read PROBLEM_AND_GOAL.md to understand the context and objectives.

IMPORTANT: Do NOT read previous versions of RESEARCH_TOOLS.md from artifacts/research/v*/. Generate fresh research from source materials to enable unbiased comparison between research cycles.

Accept optional argument: path to Galaxy directory (defaults to ~/workspace/galaxy)

Find all XML tool definitions in lib/galaxy/tools (particularly collection operations) and summarize:
- Available collection operation tools
- What each tool does
- Input/output specifications
- Parameters and configuration options
- Use cases for each tool

Focus on tools relevant to collection manipulation, filtering, splitting, reorganizing, and the apply rules tool.

Output: Create/update artifacts/research/v<N>/RESEARCH_TOOLS.md with comprehensive tool catalog and usage guidance.

Version detection:
1. List directories in artifacts/research/ matching v*
2. Find latest version (highest number)
3. If RESEARCH_TOOLS.md exists in latest version → create v<N+1>
4. If not exists → use latest version (continuing cycle)
5. Create version directory if needed, write output there
