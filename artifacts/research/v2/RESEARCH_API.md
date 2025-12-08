# Galaxy API Research Summary: Tools, Histories, and Jobs

## Overview

This document summarizes Galaxy's REST API for tool execution, history management, and job monitoring - focusing on collection handling patterns discovered from source code analysis.

**Sources:**
- `lib/galaxy/webapps/galaxy/api/tools.py`
- `lib/galaxy/webapps/galaxy/api/histories.py`
- `lib/galaxy/webapps/galaxy/api/history_contents.py`
- `lib/galaxy/webapps/galaxy/api/jobs.py`
- `lib/galaxy_test/api/test_tools.py`

---

## 1. Galaxy MCP Server (Preferred Approach)

**IMPORTANT**: If Galaxy MCP is available, prefer it over direct API calls.

Repository: https://github.com/galaxyproject/galaxy-mcp

### MCP Tools Available
- `run_tool` - Execute Galaxy tools
- `get_histories` - List user histories
- `list_history_ids` - Get history IDs
- `get_history_contents` - List history items
- `get_job_details` - Check job status and outputs

### MCP Advantages
- Handles input format complexity internally (no `values` wrapper needed)
- Simplified authentication
- Built-in error handling

---

## 2. Tools API (`/api/tools`)

### Run Tool Endpoint

**POST** `/api/tools`

```json
{
  "tool_id": "tool_identifier",
  "history_id": "encoded_history_id",
  "inputs": { ... }
}
```

### Tool Response Structure

```json
{
  "outputs": [
    {"id": "...", "name": "...", "hid": 1, "dataset_id": "..."}
  ],
  "output_collections": [
    {"id": "...", "name": "...", "hid": 2, "collection_type": "list"}
  ],
  "implicit_collections": [
    {"id": "...", "name": "...", "hid": 3}
  ],
  "jobs": [
    {"id": "...", "state": "queued", "tool_id": "..."}
  ]
}
```

### Other Tools Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/tools` | GET | List available tools (with optional `q` search) |
| `/api/tools/{id}` | GET | Show tool details (add `io_details=true` for inputs/outputs) |
| `/api/tools/{id}/build` | GET | Get tool form state for a history context |
| `/api/tools/{id}/test_data` | GET | Get tool test case data |

---

## 3. Input Format Requirements (CRITICAL)

### Data Input Format

Data inputs (datasets and collections) require the `values` wrapper:

```json
{
  "input_name": {
    "values": [
      {"src": "hda", "id": "encoded_dataset_id"}
    ]
  }
}
```

**Source types (`src`):**
- `hda` - History Dataset Association (regular dataset)
- `hdca` - History Dataset Collection Association (collection)
- `ldda` - Library Dataset Association

### Collection Input Examples

**Simple collection input:**
```json
{
  "input": {
    "src": "hdca",
    "id": "collection_id"
  }
}
```

**Batch/map-over collection:**
```json
{
  "input": {
    "batch": true,
    "values": [{"src": "hdca", "id": "collection_id"}]
  }
}
```

**Subcollection mapping (e.g., map over paired within list:paired):**
```json
{
  "input": {
    "batch": true,
    "values": [{"src": "hdca", "map_over_type": "paired", "id": "list_paired_collection_id"}]
  }
}
```

### Conditional Parameter Format

Conditional parameters use **pipe notation**, not nesting:

```json
// CORRECT
{
  "how|filter_source": {"src": "hda", "id": "..."}
}

// INCORRECT - will silently fail
{
  "how": {
    "filter_source": {"src": "hda", "id": "..."}
  }
}
```

### Repeated Parameters

```json
{
  "queries_0|input2": {"src": "hda", "id": "..."},
  "queries_1|input2": {"src": "hda", "id": "..."}
}
```

---

## 4. Collection Operation Tool Examples (from test_tools.py)

### Filter Failed Datasets

```python
inputs = {
    "input": {"src": "hdca", "id": failed_hdca_id}
}
response = run_tool("__FILTER_FAILED_DATASETS__", history_id, inputs)
```

### Apply Rules

```python
inputs = {
    "input": {"src": "hdca", "id": hdca_id},
    "rules": {
        "rules": [...],
        "mapping": [...]
    }
}
response = run_tool("__APPLY_RULES__", history_id, inputs)
```

### Unzip Collection

```python
inputs = {
    "input": {"src": "hdca", "id": paired_collection_id}
}
response = run_tool("__UNZIP_COLLECTION__", history_id, inputs)
# Returns two outputs: forward and reverse datasets
```

### Zip Collection (with batch)

```python
inputs = {
    "input_forward": {"batch": True, "values": [{"src": "hdca", "id": list1_id}]},
    "input_reverse": {"batch": True, "values": [{"src": "hdca", "id": list2_id}]}
}
response = run_tool("__ZIP_COLLECTION__", history_id, inputs)
# Creates list:paired from two lists
```

### Sort Collection

```python
inputs = {
    "input": {"src": "hdca", "id": hdca_id}
}
response = run_tool("__SORTLIST__", history_id, inputs)
```

### Build List

```python
inputs = {
    "datasets_0|input": {"src": "hda", "id": dataset_id}
}
response = run_tool("__BUILD_LIST__", history_id, inputs)
```

### Extract Dataset

```python
inputs = {
    "data_collection": {"src": "hdca", "id": collection_id},
    "which": {"which_dataset": "by_index", "index": 0}
}
response = run_tool("__EXTRACT_DATASET__", history_id, inputs)
```

### Filter Collection

```python
inputs = {
    "input": {"values": [{"src": "hdca", "id": hdca_id}]},
    "how|filter_source": {"batch": True, "values": [{"src": "hdca", "id": filter_hdca_id}]}
}
response = run_tool("__FILTER_FROM_FILE__", history_id, inputs)
```

---

## 5. Histories API (`/api/histories`)

### List Histories

**GET** `/api/histories`

Query parameters:
- `search` - Free text search (searches name, annotation, slug, tags)
- `show_own` - Include user's own histories (default: true)
- `show_published` - Include published histories (default: true)
- `show_shared` - Include shared histories (default: false)
- `show_archived` - Include archived histories (default: null/both)
- `sort_by` - Sort field (`update_time`, `name`, etc.)
- `sort_desc` - Sort descending (default: true)
- `limit`, `offset` - Pagination

### Get History

**GET** `/api/histories/{history_id}`

### Create History

**POST** `/api/histories`

```json
{
  "name": "New History"
}
```

---

## 6. History Contents API (`/api/histories/{history_id}/contents`)

### List Contents

**GET** `/api/histories/{history_id}/contents`

Query parameters:
- `types` - Filter by type (`dataset`, `dataset_collection`)
- `deleted` - Show deleted items
- `visible` - Show visible items
- `v=dev` - Use latest API version

### Get Collection Details

**GET** `/api/histories/{history_id}/contents/dataset_collections/{id}`

Returns full collection structure including elements.

Query parameters:
- `fuzzy_count` - For large collections, limit element count

### Typed Endpoints (Preferred)

**GET** `/api/histories/{history_id}/contents/datasets/{id}`
**GET** `/api/histories/{history_id}/contents/dataset_collections/{id}`

---

## 7. Jobs API (`/api/jobs`)

### Show Job

**GET** `/api/jobs/{job_id}`

Query parameters:
- `full=true` - Include stdout, stderr, metrics, output_collections

**CRITICAL**: Use `full=true` to get `output_collections` field!

### Get Job Outputs

**GET** `/api/jobs/{job_id}/outputs`

Returns list of output associations. **Warning**: May return empty for collection operations!

```json
[
  {"name": "output", "dataset": {"id": "...", "src": "hda"}},
  {"name": "output_collection", "dataset_collection": {"id": "...", "src": "hdca"}}
]
```

### Job States

- `new` - Initial state
- `queued` - Waiting for runner
- `running` - Currently executing
- `ok` - Completed successfully
- `error` - Failed
- `paused` - Waiting for input/dependencies
- `deleted` - Cancelled/deleted

### List Jobs

**GET** `/api/jobs`

Query parameters:
- `states` - Filter by state(s)
- `tool_id` - Filter by tool ID(s)
- `history_id` - Filter by history
- `date_range_min`, `date_range_max` - Date filters
- `limit`, `offset` - Pagination

---

## 8. Tool Build Endpoint (Input Discovery)

**GET** `/api/tools/{tool_id}/build?history_id={id}`

Returns tool form structure showing:
- Input parameter definitions
- Current values based on history context
- Options for select parameters

**Use this to verify correct input format** before running tools.

---

## 9. Common Patterns

### Wait for Job Completion

```python
while True:
    job = GET /api/jobs/{job_id}
    if job["state"] in ["ok", "error"]:
        break
    time.sleep(5)
```

### Get Collection Output from Job

```python
# Method 1: From tool response
response = POST /api/tools (run tool)
collection_id = response["output_collections"][0]["id"]

# Method 2: From job (with full=true)
job = GET /api/jobs/{job_id}?full=true
collection_id = job["output_collections"][0]["id"]

# Method 3: From outputs endpoint
outputs = GET /api/jobs/{job_id}/outputs
# Filter for collection associations
```

### Get Collection Details

```python
collection = GET /api/histories/{history_id}/contents/dataset_collections/{collection_id}
elements = collection["elements"]
for element in elements:
    element_id = element["id"]
    element_identifier = element["element_identifier"]
    if element["element_type"] == "hda":
        dataset = element["object"]
    elif element["element_type"] == "dataset_collection":
        nested_collection = element["object"]
```

---

## 10. Map-Over Collection Patterns

### Linked Map-Over (Default)

Process corresponding elements from two collections together:

```python
inputs = {
    "input1": {
        "batch": True,
        "values": [{"src": "hdca", "id": "collection1_id"}]
    },
    "input2": {
        "batch": True,
        "values": [{"src": "hdca", "id": "collection2_id"}]
    }
}
```

**Result:** Element 0 from collection1 paired with element 0 from collection2, etc.

### Unlinked Map-Over (Cartesian Product)

Process all combinations of elements:

```python
inputs = {
    "input1": {
        "batch": True,
        "linked": False,
        "values": [{"src": "hdca", "id": "collection1_id"}]
    },
    "input2": {
        "batch": True,
        "linked": False,
        "values": [{"src": "hdca", "id": "collection2_id"}]
    }
}
```

**Result:** Every element from collection1 paired with every element from collection2.

**Alternative:** Use `__CROSS_PRODUCT_FLAT__` or `__CROSS_PRODUCT_NESTED__` tools.

### Nested Collection Mapping (map_over_type)

Map over inner collection type within nested collections:

```python
# Map over paired elements within list:paired
inputs = {
    "input": {
        "batch": True,
        "values": [{"src": "hdca", "map_over_type": "paired", "id": "list_paired_id"}]
    }
}

# Map over list elements within list:list
inputs = {
    "input": {
        "batch": True,
        "values": [{"src": "hdca", "map_over_type": "list", "id": "list_list_id"}]
    }
}

# Map over list:paired within list:list:paired
inputs = {
    "input": {
        "batch": True,
        "values": [{"src": "hdca", "map_over_type": "list:paired", "id": "llp_id"}]
    }
}
```

---

## 11. Complete Pipeline Example

```python
import requests
import time

galaxy_url = "https://usegalaxy.org"
api_key = "your_api_key"
headers = {"x-api-key": api_key}

def execute_tool(tool_id, history_id, inputs):
    """Execute tool and wait for completion."""
    payload = {
        "tool_id": tool_id,
        "history_id": history_id,
        "inputs": inputs
    }
    response = requests.post(f"{galaxy_url}/api/tools", json=payload, headers=headers)
    result = response.json()

    # Wait for all jobs
    for job in result.get("jobs", []):
        wait_for_job(job["id"])

    return result

def wait_for_job(job_id):
    """Poll job until terminal state."""
    while True:
        response = requests.get(f"{galaxy_url}/api/jobs/{job_id}", headers=headers)
        job = response.json()
        if job["state"] == "ok":
            return True
        elif job["state"] == "error":
            raise Exception(f"Job {job_id} failed: {job.get('stderr', 'Unknown error')}")
        time.sleep(2)

# Example: Filter → Sort → Relabel pipeline
history_id = "abc123"
collection_id = "original_collection"
mapping_file_id = "relabel_mapping_file"

# Step 1: Filter failed datasets
result1 = execute_tool(
    "__FILTER_FAILED_DATASETS__",
    history_id,
    {"input": {"src": "hdca", "id": collection_id}}
)
filtered_id = result1["output_collections"][0]["id"]

# Step 2: Sort alphabetically
result2 = execute_tool(
    "__SORTLIST__",
    history_id,
    {
        "input": {"src": "hdca", "id": filtered_id},
        "sort_type": {"sort_type": "alpha"}
    }
)
sorted_id = result2["output_collections"][0]["id"]

# Step 3: Relabel with mapping file
result3 = execute_tool(
    "__RELABEL_FROM_FILE__",
    history_id,
    {
        "input": {"src": "hdca", "id": sorted_id},
        "how": {
            "how_select": "tabular",
            "labels": {"src": "hda", "id": mapping_file_id}
        }
    }
)
final_collection_id = result3["output_collections"][0]["id"]
print(f"Pipeline complete. Final collection: {final_collection_id}")
```

---

## 12. Silent Failure Warning

Galaxy API may **silently use defaults** when input format is incorrect:

- Missing `values` wrapper → Tool may run with no input
- Wrong conditional syntax → Defaults used instead
- Invalid parameter names → Ignored

**Always verify** tool execution by:
1. Checking job output state
2. Verifying output collection/dataset contents
3. Using `/api/tools/{id}/build` to understand expected format

---

## 13. Tool ID Quick Reference

### Filtering Operations

| Operation | Tool ID |
|-----------|---------|
| Filter by identifier file | `__FILTER_FROM_FILE__` |
| Filter empty datasets | `__FILTER_EMPTY_DATASETS__` |
| Filter failed datasets | `__FILTER_FAILED_DATASETS__` |
| Filter null elements | `__FILTER_NULL__` |
| Keep success only | `__KEEP_SUCCESS_DATASETS__` |
| Extract single dataset | `__EXTRACT_DATASET__` |

### Structure Transformation

| Operation | Tool ID |
|-----------|---------|
| Flatten nested to list | `__FLATTEN__` |
| Add nesting level | `__NEST__` |
| Create paired from two | `__ZIP_COLLECTION__` |
| Split paired to two | `__UNZIP_COLLECTION__` |
| Split paired/unpaired | `__SPLIT_PAIRED_AND_UNPAIRED__` |
| Build list from datasets | `__BUILD_LIST__` |

### Metadata Operations

| Operation | Tool ID |
|-----------|---------|
| Rename identifiers | `__RELABEL_FROM_FILE__` |
| Sort elements | `__SORTLIST__` |
| Add/modify tags | `__TAG_FROM_FILE__` |

### Combination Operations

| Operation | Tool ID |
|-----------|---------|
| Merge collections | `__MERGE_COLLECTION__` |
| Match two collections | `__HARMONIZELISTS__` |
| Cross product (flat) | `__CROSS_PRODUCT_FLAT__` |
| Cross product (nested) | `__CROSS_PRODUCT_NESTED__` |
| Duplicate to collection | `__DUPLICATE_FILE_TO_COLLECTION__` |

### Advanced

| Operation | Tool ID |
|-----------|---------|
| Apply rules (flexible) | `__APPLY_RULES__` |

---

## 14. Key Takeaways for Agentic Use

1. **Prefer Galaxy MCP** when available - simpler interface
2. **Use `values` wrapper** for data inputs
3. **Use pipe notation** for conditionals: `how|filter_source`
4. **Use `full=true`** when getting job details for collection outputs
5. **Check `/api/tools/{id}/build`** to discover correct input format
6. **Watch for silent failures** - verify outputs after tool execution
7. **Poll job state** until terminal state reached
8. **Use typed endpoints** (`/contents/dataset_collections/`) for clarity
9. **Chain tools through history** - output collection ID becomes next input
10. **Use map_over_type** for nested collection operations
