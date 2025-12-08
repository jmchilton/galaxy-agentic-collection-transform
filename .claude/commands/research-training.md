Research Galaxy Training Network materials on dataset collections and create RESEARCH_SUMMARY_TRAINING.md.

Read PROBLEM_AND_GOAL.md to understand the context and objectives.

IMPORTANT: Do NOT read previous versions of RESEARCH_SUMMARY_TRAINING.md from artifacts/research/v*/. Generate fresh research from source materials to enable unbiased comparison between research cycles.

Fetch and summarize these Galaxy Training Network tutorials:
1. https://training.galaxyproject.org/training-material/topics/galaxy-interface/tutorials/collections/tutorial.html
2. https://training.galaxyproject.org/training-material/topics/galaxy-interface/tutorials/upload-rules/tutorial.html
3. https://training.galaxyproject.org/training-material/topics/galaxy-interface/tutorials/upload-rules-advanced/tutorial.html

Focus on:
- Collection creation and manipulation techniques
- Upload rules and their applications
- Best practices for reproducible collection operations
- How these techniques support workflow-based analyses

Output: Create/update artifacts/research/v<N>/RESEARCH_SUMMARY_TRAINING.md with comprehensive summary.

Version detection:
1. List directories in artifacts/research/ matching v*
2. Find latest version (highest number)
3. If RESEARCH_SUMMARY_TRAINING.md exists in latest version → create v<N+1>
4. If not exists → use latest version (continuing cycle)
5. Create version directory if needed, write output there
