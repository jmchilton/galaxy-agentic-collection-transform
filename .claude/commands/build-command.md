You should "think hard" about this command.

Read PROBLEM_AND_GOAL.md to understand the problem and then ingest the research from artifacts/research/<latest_version>/RESEARCH_* documents to understand the research you've already done to address the problem.

(Find latest version by listing artifacts/research/v* directories and selecting highest number)

Create a slash command called galaxy-transform-collection.md in artifacts/command/ that implements the goal. The generated command should be self contained and not depend on access to Galaxy or the contents of this repository.

The generated slash command should contain all the relevant research to transform collections in Galaxy reproducibly. The generated slash command should take in a prompt
for how to modify the collection and do then do the transformation.

Here is some additional context for how the generated slash command should work ideally:

If there is sufficient metadata in the datasets and collections the Claude agent knows about - it should prefer using collection operation tools first and the apply rules second. If Claude deduces some metadata that is not captured in current Galaxy - Claude has a few options but should not resort to just creating collections ad-hoc using the /api/dataset_collections or via the data fetch API. Claude should create a mapping to an existing collection's identifiers to the relevant metadata as a tabular dataset and upload it to Galaxy. That metadata can then be used with Galaxy existing tools to accomplish the required collection creation. Claude should describe what it is doing and how this metadata should be considered input to the analysis for the analysis to be reproducible within Galaxy.  

If the Claude cannot capture the metadata with a simple table - Claude can also consider recreating a mirror source collection using the same datasets in the API but attaching additional metadata via tagging. This metadata can be used from the apply rules tools or certain collection operations. If Claude goes this route - it should inform the user ideally the analysis would be re-run with this collection as input because important metadata was not captured initially that is required to complete the desired analysis. 

After you've created this Claude command - create a summary document (artifacts/summary.md) that reflect on what research was the most relevant and what research was the least,
what are some open question you have that would improve the command, and what are some suggestions you'd make to improve the generation process.
