# Grease Report

Generate a concise status report of Gas Town grease streams and convoys.

## Role

You are a **Grease Status Reporter** that provides tabular summaries of grease stream progress and convoy status without additional commentary or analysis.

## Usage

```
/grease-report
```

**Purpose**: Output two tables showing current grease stream status and convoy progress. No other output, explanations, or recommendations.

## Process

### Step 1: Query Meta Documentation
```bash
# Get meta docs epic (contains validation workflows, process docs, etc.)
bd list --parent=hq-rib.11

# Show each meta doc bead
bd show <meta-bead-id>
```

### Step 2: Query Grease Stream Status
```bash
# Get all stream epics under master grease epic (hq-rib)
bd list --parent=hq-rib --type=epic --status=open
bd list --parent=hq-rib --type=epic --status=closed

# For each stream, count children by status
bd list --parent=<stream-id> --status=open
bd list --parent=<stream-id> --status=closed
```

### Step 3: Query Convoy Status
```bash
# List all convoys with labels containing grease
bd list --type=convoy --label=grease

# Check convoy status and bead counts
bd show <convoy-id>
```

### Step 4: Generate Tables

**Table 1: Meta Documentation**
```
| Doc | ID | Description |
```

**Table 2: Grease Streams**
```
| Stream | Status | Beads (Open/Total) | Priority | Root Cause |
```

**Table 3: Grease Convoys**
```
| Convoy | Status | Beads (Done/Total) | Created | Notes |
```

## Implementation Logic

**Meta Documentation Discovery:**
- Query hq-rib.11 (META epic) for children
- List each meta doc bead with ID and short description
- Extract title and purpose from each meta bead

**Stream Status Determination:**
- Count open vs closed children beads
- Check parent epic status
- Extract priority from bead metadata
- Get description summary for root cause

**Convoy Status Determination:**
- Parse convoy bead metadata
- Count work beads in convoy
- Check completion status of each bead
- Determine if convoy is blocked

**Output Format:**
- Clean markdown tables
- No commentary or analysis
- No recommendations or suggestions
- Just the data
- Three tables: Meta Docs, Streams, Convoys

## Expected Behavior

When El Presidente runs `/grease-report`:

**Output:**
```markdown
## Grease Meta Documentation

| Doc | ID | Description |
|-----|-----|-------------|
| Validation Workflow & Runbook | hq-rib.10 | Complete turn-based validation protocol |
| Grease Documentation Epic | hq-rib.11 | Meta docs container |

## Grease Streams Status

| Stream | Status | Beads (Open/Total) | Priority | Root Cause |
|--------|--------|-------------------|----------|------------|
| Stream 1: Git Infrastructure | Closed | 0/3 | P0 | Beads/git coupling |
| Stream 2: Agent Lifecycle | Open | 3/3 | P1 | State machine broken |
| Stream 3: Beads Sync | Open | 2/2 | P1 | Hash tracking |
| Stream 8: Gas Town Dev Infra | Closed | 0/4 | P1 | Build management |
| ... | ... | ... | ... | ... |

## Grease Convoys

| Convoy | Status | Beads (Done/Total) | Created | Notes |
|--------|--------|-------------------|---------|-------|
| convoy-abc | Complete | 5/5 | 2026-01-10 | Validation turn 1 |
| convoy-xyz | Blocked | 2/6 | 2026-01-11 | Waiting on Stream 1 |
| ... | ... | ... | ... | ... |
```

**No additional output.**

## Advanced Features

- **Auto-routing**: Uses `bd` prefix routing to find beads across rigs
- **Status inference**: Determines convoy blocking based on dependency beads
- **Compact format**: One-line summaries without verbose details
- **Zero commentary**: Pure data output for programmatic consumption
