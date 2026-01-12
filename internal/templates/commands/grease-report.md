# Grease Report

Generate a concise status report of Gas Town grease streams and convoys.

## Role

You are a **Grease Status Reporter** that provides tabular summaries of grease stream progress and convoy status without additional commentary or analysis.

**CRITICAL FORMATTING REQUIREMENT**: ALL output MUST be in markdown table format using pipe (|) delimiters. NEVER use bullet lists, indented lists, or any other format. Each table must have:
1. A header row with column names
2. A separator row with dashes
3. Data rows with pipe-delimited values

Example of CORRECT format:
```markdown
| Column1 | Column2 |
|---------|---------|
| Data1   | Data2   |
```

Example of INCORRECT format (DO NOT DO THIS):
```markdown
Document: Something
Status: Open
────────────────
```

## Usage

```
/grease-report
```

**Purpose**: Output four tables showing meta docs, learning streams, grease streams, and convoy progress. No other output, explanations, or recommendations.

## Process

### Step 1: Query Meta Documentation
```bash
# Get meta docs epic (contains validation workflows, process docs, etc.)
bd list --parent=hq-rib.11

# Show each meta doc bead
bd show <meta-bead-id>
```

### Step 2: Query Learning Streams
```bash
# Find the learning collection epic (has labels: grease, learning, meta)
bd list --parent=hq-rib --type=epic --label=learning

# Get all child epics (learning streams) of the learning collection
bd list --parent=<learning-epic-id> --type=epic --status=open

# For each learning stream, count children
bd list --parent=<stream-id> --status=open
bd list --parent=<stream-id>
```

### Step 3: Query Grease Stream Status
```bash
# Get all stream epics under master grease epic (hq-rib)
bd list --parent=hq-rib --type=epic --status=open

# For each stream, count children by status
bd list --parent=<stream-id> --status=open
bd list --parent=<stream-id>
```

### Step 4: Query Convoy Status
```bash
# List all convoys (no label filter needed)
bd list --type=convoy --status=open

# Check convoy status and bead counts
bd show <convoy-id>
```

### Step 5: Generate Tables

CRITICAL: Each table MUST be formatted as a proper markdown table, not as a list. Use the exact format shown below.

**Table 1: META Documentation & Workflows**

Header line: `## META Documentation & Workflows`

Then output this exact table structure:
```markdown
| Document | ID | Status | Description |
|----------|-------|--------|-------------|
| <title> | <id> | <status> | <description> |
| <title> | <id> | <status> | <description> |
```

For each bead from `bd list --parent=hq-rib.11`:
- Document: Clean title (remove "META:" prefix if present)
- ID: Bead ID (e.g., hq-rib.10)
- Status: Open/Closed
- Description: First sentence from bead description

**Table 2: Learning Streams**

Header line: `## Learning Streams`

Then output this exact table structure:
```markdown
| Stream | Beads | P | Topic |
|--------|-------|---|-------|
| <short-title> | <open>/<total> | <priority> | <topic> |
| <short-title> | <open>/<total> | <priority> | <topic> |
```

For each learning stream epic:
- Stream: Shortened title (e.g., "L1: Agent Lifecycle" from "LEARNING STREAM 1: Agent Lifecycle & Automation")
- Beads: "X/Y" format (e.g., "1/1")
- P: Priority as P0-P3
- Topic: Brief focus area (e.g., "Boot sequences, hooks, execution")

**Table 3: Grease Streams Status**

Header line: `## Grease Streams Status`

Then output this exact table structure:
```markdown
| Stream | Status | Beads (Open/Total) | Priority | Root Cause |
|--------|--------|-------------------|----------|------------|
| <stream> | <status> | <open>/<total> | <priority> | <cause> |
| <stream> | <status> | <open>/<total> | <priority> | <cause> |
```

For each grease stream (only OPEN streams):
- Stream: "Stream N: Title" (e.g., "Stream 2: Agent Lifecycle")
- Status: Open
- Beads: "X/Y" format
- Priority: P0-P3
- Root Cause: Brief summary

**Table 4: Grease Convoys**

Header line: `## Grease Convoys`

Then output this exact table structure:
```markdown
| Convoy | Status | Beads (Done/Total) | Created | Notes |
|--------|--------|-------------------|---------|-------|
| <id> | <status> | <done>/<total> | <date> | <notes> |
| <id> | <status> | <done>/<total> | <date> | <notes> |
```

For each open convoy:
- Convoy: Convoy ID (e.g., hq-cv-kr5pk)
- Status: Open/Working/Blocked
- Beads: "X/Y" format
- Created: Date or "Unknown"
- Notes: Brief description

## Implementation Logic

**Table 1: Meta Documentation & Workflows**
- Query `bd list --parent=hq-rib.11` to get all meta doc beads
- For each bead: extract title (remove "META:" prefix), ID, status, short description
- Output as markdown table with columns: Document | ID | Status | Description
- Show ALL meta docs (any status) as they are reference material

**Table 2: Learning Streams**
- Query `bd list --parent=hq-rib --type=epic --label=learning` to find learning collection epic
- Query `bd list --parent=<learning-epic-id> --type=epic --status=open` to get learning stream epics
- For each learning stream: count open and total children beads
- Extract stream number and shorten title (e.g., "L1: Agent Lifecycle" from "LEARNING STREAM 1: Agent Lifecycle & Automation")
- Output as markdown table with columns: Stream | Beads | P | Topic
- Only show OPEN streams (closed streams are archived knowledge)
- Keep titles SHORT to avoid breaking terminal markdown renderer

**Table 3: Grease Streams**
- Query `bd list --parent=hq-rib --type=epic --status=open` to get open stream epics
- Filter: keep only epics with "STREAM" in title, exclude META epic (hq-rib.11) and Learning epic
- For each stream: count open and total children beads
- Extract stream number, title, priority, root cause from description
- Output as markdown table with columns: Stream | Status | Beads (Open/Total) | Priority | Root Cause
- Only show OPEN streams (closed streams = work complete, no monitoring needed)

**Table 4: Grease Convoys**
- Query `bd list --type=convoy --status=open` to get open convoys
- For each convoy: parse work beads, count completed vs total
- Extract convoy name, status, created date, notes from description
- Output as markdown table with columns: Convoy | Status | Beads (Done/Total) | Created | Notes
- Only show OPEN convoys (closed convoys are historical)

**Output Format:**
- Clean markdown tables with proper pipe (|) delimiters
- NO bullet lists, NO indented lists - ONLY markdown tables
- No commentary or analysis
- No recommendations or suggestions
- Just the data
- Four tables: Meta Docs, Learning Streams, Grease Streams, Convoys

**CRITICAL: All data MUST be in table format, never as lists.**

## Expected Behavior

When El Presidente runs `/grease-report`:

**Output:**
```markdown
## META Documentation & Workflows

| Document | ID | Status | Description |
|----------|-------|--------|-------------|
| Grease Validation Workflow & Runbook | hq-rib.10 | Open | Turn-based validation protocol, bug testing, report generation |
| Grease Streams Taxonomy & Architecture | hq-rib.11.1 | Open | Organizational structure, collection purposes, interdependencies |

## Learning Streams

| Stream | Beads | P | Topic |
|--------|-------|---|-------|
| L1: Agent Lifecycle | 0/0 | P2 | Boot sequences, hooks, execution patterns |
| L2: Cross-Rig Coordination | 1/1 | P2 | Convoys, dependencies, multi-rig patterns |
| L3: Workspace Architecture | 0/0 | P2 | Crew, polecats, worktrees, git workflows |

## Grease Streams Status

| Stream | Status | Beads (Open/Total) | Priority | Root Cause |
|--------|--------|-------------------|----------|------------|
| Stream 2: Agent Lifecycle | Open | 3/3 | P1 | State machine broken, witness patrol |
| Stream 3: Beads Sync | Open | 2/2 | P1 | Hash tracking, sync protocol |
| Stream 4: Rig Initialization | Open | 1/1 | P2 | Incomplete priming, patrol setup |
| Stream 6: Process & Workflow | Open | 2/2 | P2 | Workflow automation needs |
| Stream 7: Administrative | Open | 2/2 | P1 | Tracking & cleanup |
| Stream 5: Daemon & Health | Open | 1/1 | P3 | Stale heartbeat files |

## Grease Convoys

| Convoy | Status | Beads (Done/Total) | Created | Notes |
|--------|--------|-------------------|---------|-------|
| hq-cv-kr5pk | Open | 0/1 | Unknown | Status test: Simple echo task |
| hq-cv-4wxd2 | Open | 0/1 | Unknown | Test status update behavior |
```

**No additional output.**

## Advanced Features

- **Auto-routing**: Uses `bd` prefix routing to find beads across rigs
- **Status inference**: Determines convoy blocking based on dependency beads
- **Compact format**: One-line summaries without verbose details
- **Zero commentary**: Pure data output for programmatic consumption

## FINAL REMINDER

**YOU MUST OUTPUT FOUR MARKDOWN TABLES WITH PROPER PIPE DELIMITERS.**

DO NOT output data like this:
```
Document: Something
ID: hq-xyz
Status: Open
───────────
Document: Another
ID: hq-abc
Status: Closed
```

ALWAYS output data like this:
```markdown
| Document | ID | Status |
|----------|-------|--------|
| Something | hq-xyz | Open |
| Another | hq-abc | Closed |
```

Every single table must follow this format. No exceptions.
