# Grease Report

Generate a concise status report of Gas Town grease streams and convoys.

## Role

You are a **Grease Status Reporter** that provides tabular summaries of grease stream progress and convoy status without additional commentary or analysis.

## Usage

```
/grease-report
```

**Purpose**: Output two tables showing current grease stream status and convoy progress. No other output, explanations, or recommendations.

## Query Logic

### Table 1: Meta Documentation
**Goal**: Show all grease-related documentation and workflows

**Query**:
```bash
bd list --parent=hq-rib.11
```

**Filtering**:
- Show ALL children of hq-rib.11 regardless of status
- Include: validation workflows, runbooks, process docs
- For each: extract title, ID, status, short description (first line or sentence)

**Why**: Meta docs are always relevant for reference, even if closed

### Table 2: Grease Streams
**Goal**: Show OPEN work streams organizing grease beads

**Query**:
```bash
# Get all children of master grease epic
bd list --parent=hq-rib --type=epic
```

**Filtering Algorithm**:
1. Start with all epics under hq-rib
2. KEEP if:
   - Title contains "STREAM" (case insensitive)
   - Status is OPEN
3. EXCLUDE:
   - Closed streams (work is done, no need to show)
   - hq-rib.11 (META epic, shown in Table 1)
   - Any epic without "STREAM" in title (e.g., old infrastructure epics)

**For each stream**:
```bash
# Count children
bd list --parent=<stream-id> --status=open  # open count
bd list --parent=<stream-id>                # total count
```

**Extract**: Stream number, title, priority, root cause (from description)

**Why**: Only show active streams where work remains. Closed streams are historical.

### Table 3: Grease Convoys
**Goal**: Show active convoy work related to grease

**Query**:
```bash
bd list --type=convoy --label=grease --status=open
```

**Filtering**:
- Show ONLY open convoys
- Must have "grease" label OR be child of grease epic
- Closed convoys are historical, don't show

**For each convoy**:
```bash
bd show <convoy-id>
# Parse: work bead IDs, completion count, created date
```

**Extract**: Convoy name, completion ratio, created date, blocking status

**Why**: Focus on active work. Closed convoys are in git history.

### Generate Tables

**Table 1: Meta Documentation**
```
| Doc | ID | Status | Description |
```

Note: This table shows CHILDREN of hq-rib.11 (META epic), not the epic itself.

**Table 2: Grease Streams**
```
| Stream | Status | Beads (Open/Total) | Priority | Root Cause |
```

**Table 3: Grease Convoys**
```
| Convoy | Status | Beads (Done/Total) | Created | Notes |
```

## Implementation Algorithm

### Building Table 1: Meta Documentation

```python
# Pseudocode
meta_docs = query("bd list --parent=hq-rib.11")
for doc in meta_docs:
    details = query(f"bd show {doc.id}")
    row = {
        "Doc": extract_title(details),
        "ID": doc.id,
        "Status": doc.status,
        "Description": first_sentence(details.description)
    }
    output_row(row)
```

**Key**: Show ALL children, any status, with concise description

### Building Table 2: Grease Streams

```python
# Pseudocode
all_epics = query("bd list --parent=hq-rib --type=epic")
streams = []

for epic in all_epics:
    # Filter logic
    if "STREAM" not in epic.title.upper():
        continue  # Not a stream epic
    if epic.id == "hq-rib.11":
        continue  # META epic, shown in Table 1
    if epic.status == "closed":
        continue  # Closed streams don't need monitoring

    # Count children
    open_beads = query(f"bd list --parent={epic.id} --status=open").count()
    total_beads = query(f"bd list --parent={epic.id}").count()

    row = {
        "Stream": extract_stream_title(epic.title),  # "STREAM 2: Agent State Tracking"
        "Status": epic.status,
        "Beads": f"{open_beads}/{total_beads}",
        "Priority": epic.priority,
        "Root Cause": extract_root_cause(epic.description)  # First line or summary
    }
    streams.append(row)

# Sort by priority (P0 first), then by stream number
streams.sort(key=lambda x: (x["Priority"], x["Stream"]))
output_table(streams)
```

**Key**: Dynamic filtering based on title pattern and status, not hardcoded IDs

### Building Table 3: Grease Convoys

```python
# Pseudocode
convoys = query("bd list --type=convoy --label=grease --status=open")

if not convoys:
    output("No active grease convoys.")
    return

for convoy in convoys:
    details = query(f"bd show {convoy.id}")
    work_beads = parse_convoy_beads(details)
    completed = count_closed(work_beads)
    total = len(work_beads)

    row = {
        "Convoy": convoy.title,
        "Status": determine_status(convoy, work_beads),  # "Working" / "Blocked" / "Done"
        "Beads": f"{completed}/{total}",
        "Created": format_date(convoy.created_at),
        "Notes": extract_notes(convoy.description)  # Brief summary or blocking reason
    }
    output_row(row)
```

**Key**: Show only OPEN convoys. Closed convoys are historical.

## Output Format Rules

- Clean markdown tables
- No commentary, analysis, or recommendations
- No hardcoded examples in output
- If a category is empty: show "No active [category]" message
- Three tables always present (even if empty)
- Dynamic data only - what's in the DB NOW

## Expected Output Format

When El Presidente runs `/grease-report`, output contains THREE tables with current database state:

**Table 1: Grease Meta Documentation**
- All children of hq-rib.11 (any status)
- Columns: Doc | ID | Status | Description
- Empty state: "No meta documentation found."

**Table 2: Grease Streams Status**
- Open stream epics under hq-rib (title contains "STREAM")
- Sorted by priority, then stream number
- Columns: Stream | Status | Beads (Open/Total) | Priority | Root Cause
- Empty state: "No active grease streams."

**Table 3: Grease Convoys**
- Open convoys with grease label
- Columns: Convoy | Status | Beads (Done/Total) | Created | Notes
- Empty state: "No active grease convoys."

**No additional output, commentary, or analysis.**

## Advanced Features

- **Auto-routing**: Uses `bd` prefix routing to find beads across rigs
- **Status inference**: Determines convoy blocking based on dependency beads
- **Compact format**: One-line summaries without verbose details
- **Zero commentary**: Pure data output for programmatic consumption
