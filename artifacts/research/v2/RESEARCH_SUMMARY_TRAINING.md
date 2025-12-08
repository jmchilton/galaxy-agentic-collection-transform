# Galaxy Training Network Research Summary: Dataset Collections

## Overview

This research summarizes Galaxy Training Network (GTN) tutorials on dataset collections, focusing on techniques for reproducible collection creation and manipulation that can be captured in workflows.

**Sources:**
- [Using dataset collections](https://training.galaxyproject.org/training-material/topics/galaxy-interface/tutorials/collections/tutorial.html)
- [Rule Based Uploader](https://training.galaxyproject.org/training-material/topics/galaxy-interface/tutorials/upload-rules/tutorial.html)
- [Advanced Rule Based Uploader](https://training.galaxyproject.org/training-material/topics/galaxy-interface/tutorials/upload-rules-advanced/tutorial.html)

---

## 1. Dataset Collections Fundamentals

### What Are Collections?

Dataset collections bundle multiple datasets into a single entity for batch processing. Instead of manipulating files one-by-one, collections enable simultaneous processing of hundreds or thousands of samples - essential for bioinformatics workflows analyzing large sample cohorts.

### Collection Types

| Type | Structure | Use Case |
|------|-----------|----------|
| **List** | Flat sequence of datasets | Multiple samples, tool outputs |
| **Paired** | Two-layer: sample → forward/reverse | Single paired-end sample |
| **List of Pairs** | Nested: list of paired structures | Multi-sample paired-end data |

### Manual Collection Creation Process

1. Select datasets via history checkboxes
2. Launch collection wizard
3. Filter/pair using text patterns (e.g., `_1` and `_2` identifiers)
4. Auto-pair or manually pair based on filename conventions
5. Name the collection

The wizard auto-suggests pairings from filename patterns.

---

## 2. Collection Operation Tools

Galaxy provides workflow-compatible tools for collection manipulation organized into three categories:

### Element Manipulation

| Tool | Function |
|------|----------|
| **Extract Dataset** | Retrieve specific datasets by position or identifier |
| **Filter Empty** | Remove empty elements |
| **Filter Failed** | Remove datasets in error states |
| **Build List** | Create new collections from datasets or existing collections |
| **Filter Collection** | Remove elements via identifier-based include/exclude lists |
| **Relabel Identifiers** | Rename datasets using mapping files |
| **Sort Collection** | Order alphabetically, numerically, or by custom sort file |
| **Tag Collection** | Apply metadata tags for downstream identification |

### Structure Transformation

| Tool | Function |
|------|----------|
| **Flatten Collection** | Convert nested → flat lists (merges identifiers) |
| **Merge Collections** | Combine collections with conflict resolution |
| **Zip Collection** | Transform separate forward/reverse → paired structure |
| **Unzip Collection** | Separate paired → individual forward/reverse lists |

### Element Combination

| Tool | Function |
|------|----------|
| **Column Join** | Merge elements on specified columns |
| **Collapse Collection** | Concatenate elements with optional identifier prepending |

---

## 3. Rule-Based Uploader

### Why Rules?

> "Manually modifying this metadata is not reproducible" and "Manually modifying this metadata is error prone."

The Rule Builder defines transformations through rules, making uploads:
- **Reproducible**: Galaxy tracks all metadata manipulations
- **Error-resistant**: Systematic rule application minimizes mistakes
- **Scalable**: Handles tens of thousands of datasets

### Core Rule Operations

#### Filter Rules
- **First/Last N Rows**: Strip headers or trailing rows

#### Column Manipulation
- **Regex Splitting**: Extract patterns using capture groups
  - Example: `.*_(\d).fastq.gz` extracts read pair number
- **Column Removal**: Delete unnecessary columns
- **Column Swapping**: Reorder for clarity

#### Row Operations
- **Row Splitting**: Divide rows based on column values (useful for paired-end URLs)

### Column Definitions

Define how Galaxy interprets each column:

| Definition | Purpose |
|------------|---------|
| **Name** | Individual dataset identifier |
| **URL** | Remote file location |
| **List Identifier(s)** | Preserve sample/condition/replicate info through tool mappings |
| **Paired-end Indicator** | Mark forward/reverse (`1`/`2`, `R1`/`R2`, `forward`/`reverse`, `f`/`r`) |
| **Type** | File format (e.g., `fastqsanger.gz`) |

### Data Sources Accepted
- Pasted tabular data
- Galaxy history datasets
- FTP directories (if admin-enabled)

---

## 4. Advanced Rule-Based Techniques

### URL Construction from Accession Data

Build download links dynamically from biological identifiers:

1. Export tabular data from databases (e.g., UniProt)
2. Extract accession identifiers from columns
3. Apply regex to generate URLs via replacement
   - Example: `\0` → `https://www.uniprot.org/uniprot/\0.fasta`

### Building Matched Collections

Create multiple coordinated collections simultaneously:

1. Add columns for datatypes and collection names
2. Build URLs for multiple formats (FASTA, GFF3) from single accession
3. Use column-splitting to duplicate rows per format
4. List identifiers must match across collections

**Use case**: FASTA sequences + GFF3 annotations with identical indexing for tools requiring matched datasets.

### Nested List Architecture

Multi-level hierarchy for complex experimental designs:

**Structure example (SRA data):**
- Outer identifier: Library type (wtPA14, con10, str10, gen8)
- Inner identifier: Run accessions (SRR5363633)

**Regex for grouping**: `([^\d]+)\d+` extracts non-numeric prefixes to create categorical grouping while preserving replicates.

**Assignment**: Multiple list identifier columns establish hierarchy depth. Column assignment order determines nesting.

### Apply Rules to Existing Collections

The **Apply Rules** tool transforms uploaded collections without re-downloading:

**Capabilities:**
- **Structural reorganization**: Extract metadata from identifiers to create nested levels
  - Example: `treated_single_1` → splits into treatment + sequencing type
- **Regex parsing**: `(.*)_(.*)_.*` captures treatment and type info
- **Hierarchy inversion**: Reorder identifier columns to swap nesting levels
- **Filtering**: Apply regex filters (e.g., `.*_single_.*`) to create subset collections
- **Combined transformations**: Filter + restructure simultaneously

---

## 5. Workflow Integration Example

A typical collection lifecycle in a bioinformatics workflow:

1. **Input**: 8 fastq files (4 paired samples) → paired collection
2. **BWA-MEM mapping**: Paired collection → list of BAM files
3. **Lofreq variant calling**: BAM collection → VCF collection
4. **SnpSift extraction**: Maintains structure while reformatting
5. **Collapse Collection**: Merge outputs → single tab-delimited table with sample identifiers

This demonstrates collections streamlining multi-sample pipelines while maintaining sample traceability.

---

## 6. Best Practices for Reproducible Collection Operations

### Naming and Organization
- Use consistent naming conventions (`_1`/`_2` suffixes) for automatic pairing
- Apply metadata tags for flexible downstream filtering/grouping
- Preserve identifiers during operations for sample-to-result traceability

### Tool Selection
- Use Galaxy's collection operation tools over manual manipulation
- Prefer rule-based uploads over spreadsheet editing
- Apply Rules tool for post-upload restructuring (avoids re-downloading)

### Workflow Compatibility
- Flatten nested collections before tools requiring non-nested inputs
- Design collection structure to match workflow requirements
- Save JSON rule sets for reproducibility and reuse across similar datasets

### Reproducibility Principles
- Rules are tracked and reportable by Galaxy
- Avoid manual metadata modifications
- Document transformations through named collections and rule definitions

---

## 7. Key Techniques Summary

| Technique | When to Use | Galaxy Mechanism |
|-----------|-------------|------------------|
| Manual collection creation | Small datasets, simple structures | History checkbox wizard |
| Rule-based upload | Large datasets, complex metadata | Rule Builder |
| URL construction | Database downloads | Regex column manipulation |
| Matched collections | Multi-format parallel workflows | Column splitting + definitions |
| Nested organization | Hierarchical experiments | Multiple list identifiers |
| Post-upload restructuring | Reorganize existing data | Apply Rules tool |
| Collection filtering | Subset creation | Regex-based filters or Filter Collection tool |
| Structure transformation | Change collection type | Zip/Unzip/Flatten tools |

---

## 8. Relevance to Agentic Collection Creation

For an AI assistant helping with Galaxy collections:

1. **Recommend rule-based approaches** over manual manipulation for reproducibility
2. **Suggest appropriate collection tools** based on desired transformation
3. **Guide structure decisions** (flat vs nested) based on workflow requirements
4. **Provide regex patterns** for common identifier extraction scenarios
5. **Emphasize workflow capture** - all operations should be extractable to workflows
6. **Use Apply Rules** for transforming existing collections (no re-upload needed)
7. **Design collection structures** that maintain sample traceability through analysis

The key insight: Galaxy's native collection tools and rule builder provide reproducible, workflow-compatible operations that should be preferred over direct API manipulation.
