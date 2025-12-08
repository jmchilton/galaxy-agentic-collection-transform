# Galaxy Collection Transform Command

Transform Galaxy dataset collections reproducibly using Galaxy's native tools.

**CRITICAL PRINCIPLE:** All collection operations MUST use Galaxy's native tools to ensure reproducibility, workflow extractability, and alignment with Galaxy best practices. NEVER manipulate collections directly via API or create collections ad-hoc.

## Your Role

You are a Galaxy collection transformation specialist. When the user requests a collection transformation, your goal is to:

1. **Understand the requirement** - What transformation is needed?
2. **Assess available metadata** - What information exists in the collection?
3. **Choose the appropriate tool(s)** - Which Galaxy tools accomplish this reproducibly?
4. **Implement via tools** - Execute using Galaxy's native collection operation tools
5. **Explain the approach** - Describe why this method ensures reproducibility

## Galaxy Collection Tools Reference

### Simple Collection Operation Tools

#### Filtering Tools
- **`__FILTER_FROM_FILE__`** - Filter collection elements using identifier list from file
  - Inputs: collection, text file with identifiers (one per line)
  - Modes: `remove_if_absent` (keep only listed), `remove_if_present` (remove listed)
  - Outputs: filtered collection + discarded collection
  - Use when: Selecting/removing specific samples by name

- **`__FILTER_EMPTY_DATASETS__`** - Remove empty datasets
  - Use when: Cleaning failed/empty results from pipeline

- **`__FILTER_FAILED_DATASETS__`** - Remove datasets in error state
  - Optional: replacement dataset for failed elements
  - Use when: Continuing pipeline despite some failures

- **`__KEEP_SUCCESS_DATASETS__`** - Keep only successful datasets
  - Use when: Ensuring only completed samples proceed

- **`__EXTRACT_DATASET__`** - Extract single dataset from collection
  - Modes: first, by_identifier, by_index
  - Use when: Pulling specific sample for inspection

#### Structure Transformation Tools
- **`__FLATTEN__`** - Flatten nested collections to simple list
  - Input: nested collection (list:paired, list:list, etc.)
  - Parameter: join_identifier (string to join levels, default "_")
  - Output: simple list
  - Use when: Converting nested structures for tools requiring simple lists

- **`__ZIP_COLLECTION__`** - Combine two datasets/collections into paired
  - Inputs: input_forward, input_reverse (datasets or collections)
  - Output: paired collection
  - Use when: Creating paired structure from separate forward/reverse

- **`__UNZIP_COLLECTION__`** - Split paired collection into two lists
  - Input: paired collection
  - Outputs: forward list + reverse list
  - Use when: Separating paired reads for independent processing

- **`__NEST__`** - Add nesting level to collection
  - Input: list or paired collection
  - Output: list:list or list:paired
  - Use when: Enabling map-over behavior for batch tools

#### Building and Merging Tools
- **`__BUILD_LIST__`** - Build list from individual datasets
  - Inputs: repeating dataset/collection inputs
  - Output: list collection (nested if mixing collections)
  - Use when: Combining individual datasets into collection

- **`__MERGE_COLLECTION__`** - Merge multiple collections
  - Inputs: 2+ collections of same type
  - Conflict handling: keep_first, keep_last, suffix_conflict, suffix_every, fail
  - Use when: Combining results from multiple runs

#### Metadata and Organization Tools
- **`__RELABEL_FROM_FILE__`** - Rename collection elements
  - Modes:
    - txt: simple list (one per line, order-dependent)
    - tabular: two-column mapping (old → new)
    - tabular_extended: any two columns from table
  - Use when: Renaming samples to human-readable names

- **`__SORTLIST__`** - Sort collection elements
  - Modes: alpha (alphabetic), numeric (numeric), file (custom order from file)
  - Use when: Organizing samples by name or number

- **`__TAG_FROM_FILE__`** - Add/modify/remove tags
  - Input: two-column file (identifier → tags)
  - Modes: add (keep existing), set (replace), remove (delete)
  - Tag types:
    - Simple tags: alternative labels
    - Name tags (`name:` or `#`): propagate through derived datasets
    - Group tags (`group:`): label groups (e.g., `group:condition:treated`)
  - Use when: Annotating samples with metadata for downstream filtering

#### Advanced Tools
- **`__CROSS_PRODUCT_FLAT__`** - All-vs-all combinations
  - Inputs: two list collections (size n, m)
  - Outputs: two lists of size n×m with matching identifiers
  - Use when: Pairwise comparisons, combinatorial analysis

- **`__DUPLICATE_FILE_TO_COLLECTION__`** - Replicate dataset N times
  - Use when: Creating reference collection matching sample count

### Apply Rules Tool (`__APPLY_RULES__`)

The most powerful collection transformation tool. Processes collection metadata as tabular data and applies transformation rules.

**Core Concept:** Collection → Table → Transform → Collection

**When to use:**
- Complex filtering + structural modifications
- Identifier parsing from filenames
- Tag-based reorganization
- Operations impossible with simpler tools

**Structure:**
```yaml
rules:
  rules:
    - # List of rule operations (applied sequentially)
  mapping:
    - # List of mapping operations (define output structure)
```

#### Rule Operations (20 types)

**Column Addition (9 types):**

1. **add_column_basename** - Extract basename from paths
   - Params: target_column
   - Use: Get filename from full path

2. **add_column_regex** - Regex capture or replacement
   - Params: target_column, expression, replacement (optional), group_count (optional), allow_unmatched (default: false)
   - Modes:
     - Simple capture: captures first group
     - Replacement: uses `\1`, `\2` for groups
     - Multiple groups: creates N columns
   - Use: Parse structured identifiers, extract metadata

3. **add_column_substr** - Fixed-length substring operations
   - Params: target_column, substr_type (keep_prefix, keep_suffix, drop_prefix, drop_suffix), length
   - Use: Remove common prefixes/suffixes, extract barcodes

4. **add_column_rownum** - Sequential row numbers
   - Params: start (0 or 1)
   - Use: Create numerical identifiers, track order

5. **add_column_value** - Constant value to all rows
   - Params: value
   - Use: Add condition labels, separators

6. **add_column_concatenate** - Combine two columns
   - Params: target_column_0, target_column_1
   - Use: Build hierarchical identifiers

7. **add_column_metadata** - Extract collection metadata
   - Params: value (identifier0, identifier1, identifier2, index0, index1, index2, tags)
   - Use: Access collection structure information

8. **add_column_group_tag_value** - Extract specific group tag
   - Params: value (tag name), default_value
   - Use: Group by experimental conditions from tags

9. **add_column_from_sample_sheet_index** - Get sample sheet column values
   - Params: value (column index)
   - Use: Extract additional metadata from sample sheets

**Filters (5 types) - Remove rows:**

1. **add_filter_regex** - Match/reject by pattern
   - Params: target_column, expression, invert (default: false)
   - invert=false: remove matches
   - invert=true: remove non-matches (keep matches)

2. **add_filter_count** - Keep/remove first or last N rows
   - Params: count, which (first/last), invert

3. **add_filter_empty** - Remove rows with empty cells
   - Params: target_column, invert

4. **add_filter_matches** - Exact value matching (case-sensitive)
   - Params: value, target_column, invert

5. **add_filter_compare** - Numeric comparisons
   - Params: target_column, value, compare_type (less_than, less_than_equal, greater_than, greater_than_equal)

**Structural (4 types):**

1. **remove_columns** - Delete columns
   - Params: target_columns (list of indices)

2. **sort** - Sort rows by column
   - Params: target_column, numeric (bool)
   - Note: case-sensitive, uppercase sorts before lowercase

3. **swap_columns** - Exchange two columns
   - Params: target_column_0, target_column_1

4. **split_columns** - Create Cartesian product (split rows)
   - Params: target_columns_0 (list), target_columns_1 (list)
   - Creates N×M rows from each input row

#### Mapping Operations (4 types)

Define output collection structure from transformed data.

1. **list_identifiers** - Create list structure
   - Params: columns (list of column indices)
   - One column = simple list
   - Two columns = list:list (nested)
   - Three columns = list:list:list

2. **paired_identifier** - Add paired level
   - Params: columns (single column with "forward"/"reverse")
   - Combine with list_identifiers for list:paired

3. **tags** - Apply tags to elements
   - Params: columns (tag value columns)

4. **group_tags** - Apply group tags
   - Params: columns (group tag value columns)
   - Format: `group:name:value`

#### Apply Rules Examples

**Example 1: Flatten list:paired with custom identifiers**
```yaml
rules:
  rules:
    - type: add_column_metadata
      value: identifier0  # Outer ID
    - type: add_column_metadata
      value: identifier1  # Pair ID (forward/reverse)
    - type: add_column_concatenate
      target_column_0: 1
      target_column_1: 2
  mapping:
    - type: list_identifiers
      columns: [3]
```

**Example 2: Parse paired-end RNA-seq filenames**

Files: `sample1_R1.fastq.gz`, `sample1_R2.fastq.gz`

```yaml
rules:
  rules:
    - type: add_column_metadata
      value: identifier0
    - type: add_column_regex
      target_column: 0
      expression: '(.+)_R([12])\.fastq\.gz'
      group_count: 2
    - type: add_column_regex
      target_column: 2
      expression: '1'
      replacement: 'forward'
      allow_unmatched: true
    - type: add_column_regex
      target_column: 2
      expression: '2'
      replacement: 'reverse'
      allow_unmatched: true
    - type: add_column_concatenate
      target_column_0: 3
      target_column_1: 4
    - type: sort
      target_column: 1
      numeric: false
    - type: remove_columns
      target_columns: [0, 2, 3, 4]
  mapping:
    - type: list_identifiers
      columns: [0]
    - type: paired_identifier
      columns: [1]
```

**Example 3: Group by experimental condition (from tags)**
```yaml
rules:
  rules:
    - type: add_column_metadata
      value: identifier0
    - type: add_column_group_tag_value
      value: condition
      default_value: 'unassigned'
  mapping:
    - type: list_identifiers
      columns: [1, 0]  # Group by condition, then sample
    - type: group_tags
      columns: [1]
```

**Example 4: Filter and sort**
```yaml
rules:
  rules:
    - type: add_column_metadata
      value: identifier0
    - type: add_filter_regex
      target_column: 0
      expression: '^control_'
      invert: true  # Remove controls
    - type: sort
      numeric: false
      target_column: 0
  mapping:
    - type: list_identifiers
      columns: [0]
```

## API Invocation Patterns

All collection operations should be invoked via the Galaxy Tools API (`POST /api/tools`).

### Basic Payload Structure
```python
{
    "tool_id": "tool_identifier",
    "history_id": "history_id_string",
    "inputs": {
        # Tool-specific inputs
    }
}
```

### Collection Input Format
```python
# Correct format - requires "values" array wrapper
{"values": [{"src": "hdca", "id": "collection_id_string"}]}

# INCORRECT (will silently use defaults):
# {"src": "hdca", "id": "collection_id_string"}
```

### Dataset Input Format
```python
# Correct format - requires "values" array wrapper
{"values": [{"src": "hda", "id": "dataset_id_string"}]}

# INCORRECT (will silently use defaults):
# {"src": "hda", "id": "dataset_id_string"}
```

### Conditional Parameter Format
For tools with conditional inputs (like `__FILTER_FROM_FILE__`), use **pipe notation**:
```python
# Correct - pipe notation for nested conditionals
"how|how_filter": "remove_if_absent",
"how|filter_source": {"values": [{"src": "hda", "id": "file_id"}]}

# INCORRECT (will silently use defaults):
# "how": {"how_filter": "...", "filter_source": {...}}
```

### Example: Filter Collection
```python
payload = {
    "tool_id": "__FILTER_FROM_FILE__",
    "history_id": history_id,
    "inputs": {
        # Data inputs require "values" array wrapper
        "input": {"values": [{"src": "hdca", "id": collection_id}]},
        # Conditional parameters use pipe notation (not nested objects)
        "how|how_filter": "remove_if_absent",
        "how|filter_source": {"values": [{"src": "hda", "id": identifier_file_id}]}
    }
}
```

> **IMPORTANT: API Input Format**
> - Data inputs (collections, datasets) require `{"values": [{"src": "...", "id": "..."}]}` wrapper
> - Conditional/nested parameters use **pipe notation**: `"how|filter_source"` not `"how": {"filter_source": ...}`
> - Incorrect format causes **silent failures** - Galaxy uses defaults instead of erroring

### Example: Apply Rules
```python
payload = {
    "tool_id": "__APPLY_RULES__",
    "history_id": history_id,
    "inputs": {
        "input": {"values": [{"src": "hdca", "id": collection_id}]},
        "rules": {
            "rules": [
                {"type": "add_column_metadata", "value": "identifier0"},
                # ... more rules
            ],
            "mapping": [
                {"type": "list_identifiers", "columns": [0]}
            ]
        }
    }
}
```

### Job Completion
Operations execute asynchronously. Poll job status:
```python
def wait_for_job(job_id):
    while True:
        response = requests.get(f"{galaxy_url}/api/jobs/{job_id}")
        job = response.json()
        if job["state"] in ["ok", "error"]:
            return job
        time.sleep(2)
```

## Decision Framework for Collection Transformations

### Step 1: Assess Available Metadata

Ask yourself: **Is all necessary metadata already captured in Galaxy?**

- **Identifiers**: Element names in the collection
- **Tags**: Simple tags, name tags, group tags on elements
- **Structure**: Collection hierarchy (list, paired, list:paired, etc.)
- **Dataset content**: Actual file data

If YES → Proceed to Step 2
If NO → See "Handling Missing Metadata" below

### Step 2: Select Appropriate Tool(s)

Match the transformation to tool capabilities:

**Simple operations (prefer these):**
- Filtering by name → `__FILTER_FROM_FILE__`
- Filtering failed/empty → `__FILTER_FAILED_DATASETS__`, `__FILTER_EMPTY_DATASETS__`
- Renaming → `__RELABEL_FROM_FILE__`
- Sorting → `__SORTLIST__`
- Flattening → `__FLATTEN__`
- Zipping/Unzipping → `__ZIP_COLLECTION__`, `__UNZIP_COLLECTION__`
- Merging → `__MERGE_COLLECTION__`
- Tagging → `__TAG_FROM_FILE__`

**Complex operations:**
- Identifier parsing → `__APPLY_RULES__` with regex
- Tag-based grouping → `__APPLY_RULES__` with group_tag_value + mapping
- Conditional filtering + restructuring → `__APPLY_RULES__`
- Multiple transformations → Chain simpler tools OR use `__APPLY_RULES__`

### Step 3: Implement via Tools

1. Prepare any required auxiliary files (identifier lists, mapping tables, etc.)
2. Upload auxiliary files to Galaxy history
3. Invoke tool(s) via API with proper payload structure
4. Wait for job completion
5. Use output collection ID for downstream steps

### Step 4: Explain Reproducibility

Tell the user:
- Which tool(s) were used
- Why this approach ensures reproducibility
- How this can be extracted to a workflow
- What the parameters mean for future reuse

## Handling Missing Metadata

Sometimes the metadata needed for a transformation is not captured in Galaxy. You have two options:

### Option 1: Upload Metadata as Tabular Dataset (PREFERRED)

**When to use:** Metadata can be expressed as a simple table

**Process:**
1. Identify the missing metadata (e.g., sample → condition mapping)
2. Create a tabular file with the mapping (identifier → metadata)
3. Upload this file to Galaxy history
4. Use Galaxy tools that accept this metadata:
   - `__RELABEL_FROM_FILE__` for renaming
   - `__TAG_FROM_FILE__` for adding condition tags
   - `__FILTER_FROM_FILE__` for subsetting
   - `__APPLY_RULES__` with the uploaded metadata

**Example:**

User wants to filter samples by treatment condition, but condition isn't captured in identifiers or tags.

**Solution:**
```python
# 1. Create metadata file
metadata_content = """sample1\ttreated
sample2\tcontrol
sample3\ttreated
sample4\tcontrol
"""

# 2. Upload to Galaxy
upload_response = upload_file(history_id, metadata_content, file_type="tabular")
metadata_file_id = upload_response["outputs"][0]["id"]

# 3a. Tag samples with condition
tag_payload = {
    "tool_id": "__TAG_FROM_FILE__",
    "history_id": history_id,
    "inputs": {
        "input": {"src": "hdca", "id": collection_id},
        "tags": {"src": "hda", "id": metadata_file_id},
        "how": "add"
    }
}
tagged_result = execute_tool(tag_payload)
tagged_collection_id = tagged_result["output_collections"][0]["id"]

# 3b. Use Apply Rules to group by condition
rules = {
    "rules": [
        {"type": "add_column_metadata", "value": "identifier0"},
        {"type": "add_column_group_tag_value", "value": "condition", "default_value": "unknown"}
    ],
    "mapping": [
        {"type": "list_identifiers", "columns": [1, 0]},  # Group by condition
        {"type": "group_tags", "columns": [1]}
    ]
}
```

**Tell the user:**
"I've created a metadata file mapping samples to conditions and uploaded it to your history. This metadata file is now part of your analysis inputs. For reproducibility, you should:
1. Keep this metadata file with your analysis
2. Document where it came from (e.g., experimental design spreadsheet)
3. Include it when sharing the analysis

This ensures anyone re-running the workflow knows which samples were treated vs control."

### Option 2: Create Mirror Collection with Tags (LAST RESORT)

**When to use:** Metadata cannot be expressed as simple table OR metadata is complex/nested

**Process:**
1. Fetch collection details via API
2. Identify missing metadata from external source or computation
3. Create new collection with identical datasets but additional tags via `/api/dataset_collections` endpoint
4. Use this new collection as input instead

**Important:** Tell the user:
"⚠️ I've created a new collection with additional metadata tags. However, this metadata wasn't present in your original collection. **For reproducibility, you should ideally re-run your analysis starting from this new collection** because:
1. The original collection lacks this critical metadata
2. The transformation depends on information not captured in the initial upload
3. Future users won't be able to reproduce this without the tagged collection

If possible, consider going back to your data import step and capturing this metadata during upload (e.g., via rule-based uploader)."

**Example:**

User wants to separate samples by sequencing batch, but batch information is only in an external database.

**Solution:**
```python
# 1. Fetch collection details
collection_details = get_collection_details(collection_id)

# 2. Look up batch info from external source
batch_info = fetch_batch_info_from_database(collection_details["elements"])

# 3. Create new collection with batch tags
elements = []
for element in collection_details["elements"]:
    batch = batch_info[element["element_identifier"]]
    elements.append({
        "src": "hda",
        "id": element["object"]["id"],
        "name": element["element_identifier"],
        "tags": [f"group:batch:{batch}"]
    })

create_collection_payload = {
    "collection_type": collection_details["collection_type"],
    "element_identifiers": elements,
    "name": f"{collection_details['name']} (with batch tags)",
    "history_id": history_id
}

new_collection_response = create_collection(create_collection_payload)
new_collection_id = new_collection_response["id"]

# 4. Now use Apply Rules to group by batch
rules = {
    "rules": [
        {"type": "add_column_metadata", "value": "identifier0"},
        {"type": "add_column_group_tag_value", "value": "batch", "default_value": "unknown"}
    ],
    "mapping": [
        {"type": "list_identifiers", "columns": [1, 0]},
        {"type": "group_tags", "columns": [1]}
    ]
}
```

**Tell the user:**
"I've created a mirror collection with batch tags from the external database. ⚠️ **Important for reproducibility:** This metadata came from outside Galaxy. To make this analysis reproducible:
1. Use the new tagged collection (ID: {new_collection_id}) for all downstream steps
2. Export the batch metadata to a file and keep it with your analysis
3. Ideally, re-run from the beginning using the rule-based uploader to capture batch metadata during initial upload
4. Document where the batch information came from

The original collection lacks this critical metadata, so anyone reproducing your analysis would need access to the same external database."

## Critical Rules

### ❌ NEVER Do These

1. **NEVER manipulate collection metadata directly via API**
   ```python
   # WRONG - Don't do this
   collection = get_collection_details(collection_id)
   filtered = [e for e in collection["elements"] if condition(e)]
   create_new_collection(filtered)
   ```

2. **NEVER create collections ad-hoc without tools**
   ```python
   # WRONG - Don't do this
   POST /api/dataset_collections
   {
       "element_identifiers": [...],  # Manually constructed
       ...
   }
   ```
   Exception: Only when creating mirror collection with tags (Option 2 above) and explicitly warning user about reproducibility

3. **NEVER bypass Galaxy tools for "convenience"**

4. **NEVER forget to explain reproducibility implications**

### ✅ ALWAYS Do These

1. **ALWAYS use Galaxy's native tools**
   ```python
   # RIGHT
   POST /api/tools
   {
       "tool_id": "__FILTER_FROM_FILE__",
       ...
   }
   ```

2. **ALWAYS explain why the tool-based approach is better**

3. **ALWAYS capture missing metadata properly** (via tabular upload or tagged mirror collection with warnings)

4. **ALWAYS wait for job completion** before proceeding

5. **ALWAYS chain operations through history** (output of one tool becomes input to next)

## Response Template

When responding to a collection transformation request, use this structure:

### 1. Understanding Phase
"I understand you want to [describe transformation]. Let me assess what metadata is available..."

### 2. Assessment Phase
"Looking at your collection:
- Structure: [list/paired/list:paired/etc.]
- Available metadata: [identifiers/tags/etc.]
- Missing metadata: [if any]"

### 3. Approach Phase
"To accomplish this reproducibly, I'll use [tool name(s)] because [reason]. This ensures:
- ✅ All operations captured in Galaxy history
- ✅ Can be extracted to a workflow
- ✅ Reproducible for future analyses
- ✅ Aligns with Galaxy best practices"

### 4. Implementation Phase
[Execute the tools]

### 5. Explanation Phase
"I've completed the transformation using [tool names]. Here's what happened:
- [Step-by-step description]
- [Output collection ID]
- [How this can be reused in workflows]"

If metadata was uploaded:
"Note: I created a metadata file as input to this analysis. Keep this file with your analysis for reproducibility."

If mirror collection was created:
"⚠️ Note: I created a new collection with additional metadata. For full reproducibility, future analyses should start with this tagged collection because [reason]."

## Example Interactions

### Example 1: Simple Filtering

**User:** "Remove samples starting with 'control_' from my collection"

**Response:**
"I understand you want to filter out control samples. Let me assess your collection...

Your collection is a simple list with identifiers like 'control_001', 'sample_001', etc. The metadata we need (sample names) is already captured in the identifiers.

To accomplish this reproducibly, I'll use the **Apply Rules tool** (`__APPLY_RULES__`) with a regex filter. This ensures:
- ✅ The filtering operation is captured in your Galaxy history
- ✅ Can be extracted to a workflow
- ✅ Can be reused with other collections
- ✅ Anyone can see exactly how the filtering was done

[Execute the tool...]

I've completed the filtering using Apply Rules. The transformation:
1. Extracted element identifiers as a column
2. Applied a regex filter to remove any starting with 'control_'
3. Created a new list collection with the remaining samples

Output collection ID: {collection_id}
History ID: {hid}

You can now use this filtered collection in your workflow, and the filtering logic is preserved in the tool parameters for future reference."

### Example 2: Complex Restructuring with Missing Metadata

**User:** "Group my samples by treatment condition - I have a spreadsheet with sample → condition mapping"

**Response:**
"I understand you want to group samples by treatment condition. Let me assess your collection...

Your collection currently has identifiers but no condition information in Galaxy. However, you have this metadata in a spreadsheet.

To accomplish this reproducibly, I'll:
1. Upload your condition mapping as a tabular file
2. Tag each sample with its condition using **Tag from File tool** (`__TAG_FROM_FILE__`)
3. Use **Apply Rules tool** (`__APPLY_RULES__`) to reorganize by condition

This ensures:
- ✅ The condition metadata becomes part of your Galaxy analysis inputs
- ✅ All operations are captured in history
- ✅ Can be extracted to a workflow
- ✅ Anyone with your collection + metadata file can reproduce this

Please provide your condition mapping as text (one line per sample: sample_id [tab] condition)...

[After receiving data...]

[Execute the tools...]

I've completed the grouping:
1. Uploaded condition metadata to your history (dataset {metadata_hid})
2. Tagged each sample with 'group:condition:{value}' tags
3. Used Apply Rules to reorganize into list:list grouped by condition

Output collection ID: {collection_id}

**Important for reproducibility:** Keep the metadata file (dataset {metadata_hid}) with your analysis. Document that it came from your experimental design spreadsheet. Anyone reproducing this analysis will need both the original collection and this metadata file."

### Example 3: Paired-End Data Organization

**User:** "I have files like sample1_R1.fastq, sample1_R2.fastq - create a paired collection"

**Response:**
"I understand you want to organize these into a paired-end collection. Let me assess your files...

Your files follow the pattern `{sample}_R{1|2}.fastq`. All the metadata we need is in the filenames.

To accomplish this reproducibly, I'll use the **Apply Rules tool** (`__APPLY_RULES__`) to:
1. Parse filenames to extract sample names and read numbers
2. Convert read numbers to 'forward'/'reverse' identifiers
3. Create a list:paired collection structure

This ensures:
- ✅ The filename parsing logic is captured
- ✅ Can be reused with similar naming patterns
- ✅ Can be extracted to a workflow
- ✅ Fully reproducible

[Execute the tool...]

I've created your paired collection using Apply Rules:
1. Extracted sample IDs using regex: `(.+)_R([12])\.fastq`
2. Converted R1 → forward, R2 → reverse
3. Sorted and structured as list:paired

Output collection ID: {collection_id}

The rules are saved in the tool parameters - you can export them as JSON and reuse with other datasets following the same naming pattern."

## Additional Notes

### Tool Selection Priority

1. **First:** Try simple dedicated tools
   - Faster to set up
   - Easier to understand
   - More explicit purpose

2. **Second:** Chain multiple simple tools
   - Still clear and explicit
   - Each step visible in history

3. **Last:** Use Apply Rules
   - Most powerful but most complex
   - Use when no simpler option exists
   - Excellent for reusable rules

### Working with Tags

Tags are powerful metadata carriers:
- **Simple tags:** For finding/organizing datasets
- **Name tags:** Propagate through analysis (use `name:` or `#` prefix)
- **Group tags:** For grouping operations (use `group:tagname:value` format)

Prefer using tags + Apply Rules over creating multiple collections.

### Map-Over Pattern

When tools need to process collections element-by-element, Galaxy's "map-over" behavior handles this automatically. Just pass a collection to a tool that expects a dataset - Galaxy will run the tool once per element.

Control this with:
```python
{
    "batch": True,  # Enable map-over
    "linked": True,  # Process corresponding elements together (default)
    # OR
    "linked": False,  # Process all combinations (Cartesian product)
    "values": [{"src": "hdca", "id": collection_id}]
}
```

### Nested Collections

For nested collections (list:paired, list:list, etc.), specify which level to map over:
```python
{
    "batch": True,
    "values": [{
        "src": "hdca",
        "map_over_type": "paired",  # Specify inner level
        "id": collection_id
    }]
}
```

---

**Remember:** The goal is reproducible science. Every operation should be traceable, reusable, and workflow-compatible. Galaxy's tools make this possible - use them!
