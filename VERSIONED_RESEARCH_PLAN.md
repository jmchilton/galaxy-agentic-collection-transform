# Versioned Research Documents Plan

## Goal

Move generated research documents from `RESEARCH_*.md` (root) to `artifacts/research/<version>/RESEARCH_*.md` with auto-incrementing version numbers.

## Current State

```
/
├── RESEARCH_API.md
├── RESEARCH_APPLY_RULES.md
├── RESEARCH_SUMMARY_TRAINING.md
├── RESEARCH_TESTS.md
├── RESEARCH_TOOLS.md
├── artifacts/
│   ├── command/galaxy-transform-collection.md
│   └── summary.md
└── .claude/commands/
    ├── research-training.md
    ├── research-tools.md
    ├── research-api.md
    ├── research-tests.md
    ├── research-apply-rules.md
    ├── research-upload.md
    └── build-command.md
```

## Target State

```
/
├── artifacts/
│   ├── research/
│   │   ├── v1/
│   │   │   ├── RESEARCH_API.md
│   │   │   ├── RESEARCH_APPLY_RULES.md
│   │   │   ├── RESEARCH_SUMMARY_TRAINING.md
│   │   │   ├── RESEARCH_TESTS.md
│   │   │   └── RESEARCH_TOOLS.md
│   │   └── v2/  (created by next research run)
│   │       ├── RESEARCH_API.md
│   │       ├── RESEARCH_UPLOAD.md  (new)
│   │       └── ...
│   ├── command/galaxy-transform-collection.md
│   └── summary.md
└── .claude/commands/
    └── (updated commands)
```

## Implementation Steps

### Step 1: Move existing research to v1
- [ ] Create `artifacts/research/v1/`
- [ ] Move all `RESEARCH_*.md` files to `artifacts/research/v1/`
- [ ] Git add the moves

### Step 2: Update research commands (6 files)
Each research command needs updated output path with auto-increment logic.

**Files to modify:**
- `.claude/commands/research-training.md`
- `.claude/commands/research-tools.md`
- `.claude/commands/research-api.md`
- `.claude/commands/research-tests.md`
- `.claude/commands/research-apply-rules.md`
- `.claude/commands/research-upload.md`

**Change pattern for each:**
```
Before: Output: Create/update RESEARCH_X.md

After:  Output: Create/update artifacts/research/<next_version>/RESEARCH_X.md

        Version detection:
        - List directories in artifacts/research/ matching v*
        - Find highest number, increment by 1
        - Create new version directory if needed
        - Write to that version
```

### Step 3: Update research-tests.md input paths
This command reads prior research - needs to read from latest version:
```
Before:
1. RESEARCH_TOOLS.md
2. RESEARCH_API.md

After:
1. artifacts/research/<latest_version>/RESEARCH_TOOLS.md
2. artifacts/research/<latest_version>/RESEARCH_API.md
```

### Step 4: Update build-command.md
Needs to read from latest version:
```
Before: ingest the research we've done with RESEARCH_* documents

After:  ingest artifacts/research/<latest_version>/RESEARCH_* documents
```

### Step 5: Update README.md
- Update "Research Documents" section paths
- Update workflow diagram
- Update any file references

## Auto-Increment Version Logic

Commands should include this instruction:
```
Version handling:
1. Check artifacts/research/ for existing version directories (v1, v2, etc.)
2. If running a fresh research cycle, create next version (v1 → v2)
3. If adding to current cycle, use existing latest version
4. Write output to artifacts/research/v<N>/RESEARCH_X.md
```

## Decisions

1. **Version sharing**: Full cycle shares version - first command creates new version, others use it
2. **build-command output**: Keep current single location (not versioned)

## Cycle Detection Logic

To detect whether to create new version or continue existing:
- Find latest version directory (highest v<N>)
- Check if the specific RESEARCH_X.md file already exists in latest version
- If exists → create new version (starting fresh cycle)
- If not exists → use latest version (continuing cycle)

Example flow:
```
v1/ has: RESEARCH_SUMMARY_TRAINING.md, RESEARCH_TOOLS.md, ...

User runs /research-training
→ v1/RESEARCH_SUMMARY_TRAINING.md exists
→ Create v2/, write RESEARCH_SUMMARY_TRAINING.md there

User runs /research-tools
→ v2/RESEARCH_TOOLS.md does NOT exist
→ Write to v2/RESEARCH_TOOLS.md (same cycle)
```

## Files to Modify

| File | Change |
|------|--------|
| `artifacts/research/v1/*` | Create dir, move existing files |
| `.claude/commands/research-training.md` | Output path |
| `.claude/commands/research-tools.md` | Output path |
| `.claude/commands/research-api.md` | Output path |
| `.claude/commands/research-tests.md` | Output + input paths |
| `.claude/commands/research-apply-rules.md` | Output path |
| `.claude/commands/research-upload.md` | Output path |
| `.claude/commands/build-command.md` | Input path |
| `README.md` | Update all references |
