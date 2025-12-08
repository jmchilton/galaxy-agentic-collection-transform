# Galaxy Data Upload/Fetch API Research Summary

## Overview

Galaxy provides two upload methods:
1. **Fetch API** (`/api/tools/fetch`) - Modern, preferred method
2. **Upload1 Tool** (`/api/tools` with `upload1`) - Legacy method

**Always prefer the Fetch API** for new implementations.

**Sources:**
- `lib/galaxy/webapps/galaxy/api/tools.py` - FetchTools class
- `lib/galaxy/schema/fetch_data.py` - Payload models
- `lib/galaxy_test/base/populators.py` - Test usage patterns

---

## 1. Fetch API Endpoint

**POST** `/api/tools/fetch`

### Basic Payload Structure

```json
{
    "history_id": "encoded_history_id",
    "targets": [
        {
            "destination": {"type": "hdas"},
            "elements": [...]
        }
    ]
}
```

### Response Structure

```json
{
    "outputs": [
        {"id": "...", "name": "...", "hid": 1}
    ],
    "output_collections": [
        {"id": "...", "name": "...", "hid": 2, "collection_type": "list"}
    ],
    "jobs": [
        {"id": "...", "state": "queued"}
    ]
}
```

---

## 2. Destination Types

### HDA Destination (individual datasets)

```json
{"destination": {"type": "hdas"}}
```

### HDCA Destination (collections)

```json
{
    "destination": {"type": "hdca"},
    "collection_type": "list",
    "name": "My Collection"
}
```

### Library Folder Destination

```json
{
    "destination": {
        "type": "library_folder",
        "library_folder_id": "encoded_folder_id"
    }
}
```

---

## 3. Data Source Types (`src`)

### Pasted Content

Upload text content directly:

```json
{
    "src": "pasted",
    "paste_content": "line1\nline2\nline3",
    "name": "my_data.txt",
    "ext": "txt"
}
```

### URL

Fetch from remote URL:

```json
{
    "src": "url",
    "url": "https://example.com/data.fastq.gz",
    "name": "sample.fastq.gz",
    "ext": "fastqsanger.gz"
}
```

### Files (multipart upload)

Upload local files via form data:

```json
{
    "src": "files",
    "name": "uploaded_file.txt",
    "ext": "txt"
}
```

Files are attached as `__files` dictionary with keys like `files_0|file_data`.

### FTP Import

Import from user's FTP directory:

```json
{
    "src": "ftp_import",
    "ftp_path": "path/to/file.txt",
    "name": "ftp_file.txt",
    "ext": "txt"
}
```

### Server Directory (admin)

Import from server filesystem:

```json
{
    "src": "server_dir",
    "server_dir": "/path/on/server/file.txt",
    "link_data_only": false
}
```

### Path (admin/testing)

Direct filesystem path:

```json
{
    "src": "path",
    "path": "/absolute/path/to/file.txt"
}
```

---

## 4. Element Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `name` | string | auto | Dataset name |
| `ext` | string | `"auto"` | File type/extension |
| `dbkey` | string | `"?"` | Genome build |
| `tags` | list | null | Tags to apply |
| `info` | string | null | Dataset info |
| `to_posix_lines` | bool | false | Convert line endings |
| `space_to_tab` | bool | false | Convert spaces to tabs |
| `auto_decompress` | bool | false | Auto-decompress archives |
| `deferred` | bool | false | Create deferred dataset (don't download yet) |

### Hash Verification

```json
{
    "src": "url",
    "url": "...",
    "MD5": "d41d8cd98f00b204e9800998ecf8427e",
    "SHA256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
}
```

Or using the `hashes` array:

```json
{
    "hashes": [
        {"hash_function": "MD5", "hash_value": "d41d8cd98f00b204e9800998ecf8427e"},
        {"hash_function": "SHA-256", "hash_value": "..."}
    ]
}
```

---

## 5. Creating Datasets

### Single Dataset from Pasted Content

```python
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdas"},
        "elements": [{
            "src": "pasted",
            "paste_content": "sample1\nsample2\nsample3",
            "name": "sample_list.txt",
            "ext": "txt"
        }]
    }]
}
response = POST /api/tools/fetch, json=payload
```

### Single Dataset from URL

```python
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdas"},
        "elements": [{
            "src": "url",
            "url": "https://example.com/data.txt",
            "name": "remote_data.txt",
            "ext": "txt"
        }]
    }]
}
```

### Multiple Datasets

```python
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdas"},
        "elements": [
            {"src": "pasted", "paste_content": "data1", "name": "file1.txt", "ext": "txt"},
            {"src": "pasted", "paste_content": "data2", "name": "file2.txt", "ext": "txt"},
            {"src": "url", "url": "https://example.com/file3.txt", "name": "file3.txt", "ext": "txt"}
        ]
    }]
}
```

---

## 6. Creating Collections

### Simple List Collection

```python
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdca"},
        "collection_type": "list",
        "name": "My Sample List",
        "elements": [
            {"src": "pasted", "paste_content": "sample1_data", "name": "sample1", "ext": "txt"},
            {"src": "pasted", "paste_content": "sample2_data", "name": "sample2", "ext": "txt"},
            {"src": "pasted", "paste_content": "sample3_data", "name": "sample3", "ext": "txt"}
        ]
    }]
}
```

### Paired Collection

```python
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdca"},
        "collection_type": "paired",
        "name": "Paired Sample",
        "elements": [
            {"src": "pasted", "paste_content": "forward_reads", "name": "forward", "ext": "fastqsanger"},
            {"src": "pasted", "paste_content": "reverse_reads", "name": "reverse", "ext": "fastqsanger"}
        ]
    }]
}
```

### List:Paired Collection

```python
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdca"},
        "collection_type": "list:paired",
        "name": "Paired Samples",
        "elements": [
            {
                "name": "sample1",
                "elements": [
                    {"src": "pasted", "paste_content": "s1_forward", "name": "forward", "ext": "txt"},
                    {"src": "pasted", "paste_content": "s1_reverse", "name": "reverse", "ext": "txt"}
                ]
            },
            {
                "name": "sample2",
                "elements": [
                    {"src": "pasted", "paste_content": "s2_forward", "name": "forward", "ext": "txt"},
                    {"src": "pasted", "paste_content": "s2_reverse", "name": "reverse", "ext": "txt"}
                ]
            }
        ]
    }]
}
```

### Collection with Tags

```python
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdca"},
        "collection_type": "list",
        "name": "Tagged Samples",
        "tags": ["project:myproject", "batch:1"],
        "elements": [
            {
                "src": "pasted",
                "paste_content": "data1",
                "name": "sample1",
                "ext": "txt",
                "tags": ["group:treatment:control", "replicate:1"]
            },
            {
                "src": "pasted",
                "paste_content": "data2",
                "name": "sample2",
                "ext": "txt",
                "tags": ["group:treatment:treated", "replicate:1"]
            }
        ]
    }]
}
```

---

## 7. Uploading Identifier Files for Collection Operations

Many collection operations use identifier files. Here's how to create them:

### Filter Identifiers File

```python
# Create file with identifiers to keep/remove
identifiers = "sample1\nsample3\nsample5"
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdas"},
        "elements": [{
            "src": "pasted",
            "paste_content": identifiers,
            "name": "keep_samples.txt",
            "ext": "txt"
        }]
    }]
}
```

### Relabeling Map File (tabular)

```python
# Two-column: old_name<tab>new_name
mapping = "old_sample1\tnew_sample1\nold_sample2\tnew_sample2"
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdas"},
        "elements": [{
            "src": "pasted",
            "paste_content": mapping,
            "name": "relabel_map.txt",
            "ext": "tabular"
        }]
    }]
}
```

### Sort Order File

```python
# One identifier per line in desired order
order = "sample3\nsample1\nsample2"
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdas"},
        "elements": [{
            "src": "pasted",
            "paste_content": order,
            "name": "sort_order.txt",
            "ext": "txt"
        }]
    }]
}
```

### Tag Assignment File (tabular)

```python
# identifier<tab>tag1<tab>tag2...
tags = "sample1\tgroup:treatment:control\ntreatment:treated"
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdas"},
        "elements": [{
            "src": "pasted",
            "paste_content": tags,
            "name": "tag_assignments.txt",
            "ext": "tabular"
        }]
    }]
}
```

---

## 8. Advanced: Archive/Directory Upload

### From Archive

Upload and expand an archive:

```python
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdas"},
        "elements": [{
            "src": "pasted",
            "paste_content": "",  # Empty, actual data from files
            "ext": "directory",
            "extra_files": {
                "items_from": "archive",
                "src": "files",
                "fuzzy_root": False
            }
        }]
    }],
    "__files": {"files_0|file_data": archive_file_handle}
}
```

### Items From (batch upload from directory structure)

```python
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdca"},
        "collection_type": "list",
        "src": "path",
        "path": "/server/path/to/directory",
        "items_from": "directory"
    }]
}
```

---

## 9. Deferred Datasets

Create dataset references without downloading:

```python
payload = {
    "history_id": history_id,
    "targets": [{
        "destination": {"type": "hdas"},
        "elements": [{
            "src": "url",
            "url": "https://example.com/large_file.bam",
            "name": "deferred.bam",
            "ext": "bam",
            "deferred": True
        }]
    }]
}
```

Deferred datasets are downloaded on-demand when accessed by tools.

---

## 10. Python Helper Pattern

From test populators, a clean helper for creating datasets:

```python
def fetch_payload(history_id, content, file_type="txt", name="Dataset", **kwds):
    element = {
        "ext": file_type,
        "name": name,
        "auto_decompress": kwds.get("auto_decompress", False),
    }

    if hasattr(content, "read"):
        # File-like object
        element["src"] = "files"
        files = {"files_0|file_data": content}
    elif content and "://" in content:
        # URL
        element["src"] = "url"
        element["url"] = content
        files = {}
    else:
        # Pasted content
        element["src"] = "pasted"
        element["paste_content"] = content
        files = {}

    return {
        "history_id": history_id,
        "targets": [{
            "destination": {"type": "hdas"},
            "elements": [element]
        }],
        "__files": files
    }
```

---

## 11. Galaxy MCP Server

If Galaxy MCP is available (https://github.com/galaxyproject/galaxy-mcp), it may provide upload tools:

- Check for `upload_file` or similar MCP tools
- MCP handles authentication and simplifies payloads
- Falls back to direct API when MCP unavailable

**MCP Advantages:**
- Simplified interface
- Built-in authentication
- May provide streaming for large files

---

## 12. Wait for Upload Completion

```python
response = POST /api/tools/fetch, json=payload
job_id = response["jobs"][0]["id"]

# Poll for completion
while True:
    job = GET /api/jobs/{job_id}
    if job["state"] in ["ok", "error"]:
        break
    time.sleep(2)

# Get outputs
if job["state"] == "ok":
    outputs = response["outputs"]  # or output_collections
```

---

## 13. Key Patterns for Agentic Use

1. **Always use Fetch API** over upload1 tool
2. **Pasted content** for small text data (identifier files, etc.)
3. **URL source** for remote data
4. **Set explicit ext** when auto-detection might fail
5. **Use tags** for metadata that drives collection operations
6. **Nest elements** for complex collection types
7. **Wait for job completion** before using uploaded data
8. **Use deferred** for large files accessed conditionally

---

## 14. Common File Types

| Extension | Description |
|-----------|-------------|
| `txt` | Plain text |
| `tabular` | Tab-separated values |
| `csv` | Comma-separated values |
| `fastqsanger` | FASTQ (Sanger quality) |
| `fastqsanger.gz` | Compressed FASTQ |
| `fasta` | FASTA sequences |
| `bam` | Binary alignment |
| `vcf` | Variant call format |
| `bed` | BED intervals |
| `auto` | Auto-detect (default) |

---

## 15. Error Handling

Check response status and job state:

```python
response = POST /api/tools/fetch, json=payload
if response.status_code != 200:
    error = response.json()
    # Handle error

# Even with 200, job may fail
job = GET /api/jobs/{job_id}?full=true
if job["state"] == "error":
    stderr = job.get("stderr", "")
    # Handle job failure
```
