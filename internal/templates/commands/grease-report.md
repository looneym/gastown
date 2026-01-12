# Grease Report

Generate a concise status report of Gas Town grease streams and convoys.

## Role

You are a **Grease Status Reporter** that provides tabular summaries of grease stream progress and convoy status without additional commentary or analysis.

## Usage

```
/grease-report
```

**Purpose**: Output five tables showing grease meta documentation, learning collection, learning streams, grease streams, and convoy progress. No other output, explanations, or recommendations.

## Query Logic

### Table 1: Meta Documentation
**Goal**: Show all items in the META documentation epic collection

**Structure**:
- **hq-rib.11** = META epic (parent container - NOT shown in table)
- **Children of hq-rib.11** = Individual meta docs (ALL shown in table as siblings)

**Query**:
```bash
bd list --parent=hq-rib.11
```

**Filtering**:
- Show ALL children of the hq-rib.11 epic regardless of status or type
- These are siblings to each other (validation workflows, runbooks, taxonomies, etc.)
- For each: extract title, ID, status, short description (first line or sentence)

**Why**: All meta docs are always relevant for reference, even if closed

**Example items**:
- hq-rib.10: Validation Workflow & Runbook
- hq-rib.11.1: Streams Taxonomy & Architecture
- hq-rib.11.2: Future process docs, etc.

### Table 2: Learning Collection
**Goal**: Show all items in the LEARNING collection

**Structure**:
- **Learning collection epic** = Parent container (NOT shown in table)
- **Children of learning epic** = Individual learning beads (ALL shown in table)

**Query**:
```bash
# Find the learning collection epic (has labels: grease, learning, meta)
learning_epic=$(bd list --parent=hq-rib --type=epic --label=learning | grep "LEARNING:" | awk '{print $1}')
bd list --parent=$learning_epic
```

**Filtering**:
- Show ALL children of the learning collection epic regardless of status or type
- These are learning beads documenting workflows, anti-patterns, investigations
- For each: extract title, ID, status, short description

**Why**: All learning beads are valuable reference material, even if closed

**Example items**:
- hq-rib.9: Understand crew architecture
- Future: Dispatch pattern learnings, debugging guides, etc.

### Table 2.5: Learning Streams
**Goal**: Show OPEN learning stream epics organizing learning beads

**Structure**:
- **Learning collection epic** = Parent container (dynamically found)
- **Learning stream epics** = Children of learning collection (shown in table)
- **Learning beads** = Children of stream epics (counted, not shown)

**Query**:
```bash
# Find the learning collection epic (has labels: grease, learning, meta)
learning_epic=$(bd list --parent=hq-rib --type=epic --label=learning | grep "LEARNING:" | awk '{print $1}')
# Get all child epics of the learning collection
bd list --parent=$learning_epic --type=epic
```

**Filtering Algorithm**:
1. Find learning collection epic dynamically (labels: grease, learning, meta)
2. Get all child epics of learning collection
3. KEEP if:
   - Title contains "STREAM" (case insensitive)
   - Status is OPEN
4. EXCLUDE:
   - Closed streams (archived knowledge)

**For each learning stream**:
```bash
# Count children (learning beads in this stream)
bd list --parent=<stream-id> --status=open  # open count
bd list --parent=<stream-id>                # total count
```

**Extract**: Stream name, ID, priority, topic (from description)

**Why**: Show active learning streams to understand knowledge organization and identify gaps.

### Table 3: Grease Streams
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

### Table 4: Grease Convoys
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
Parent collection: hq-rib.11

**Table 2: Learning Collection**
```
| Learning | ID | Status | Description |
```

Note: This table shows CHILDREN of the learning collection epic, not the epic itself.
Parent collection: [learning epic ID]

**Table 2.5: Learning Streams**
```
| Stream | ID | Status | Beads (Open/Total) | Priority | Topic |
```

Note: This table shows stream epics under the learning collection, not individual learning beads.
Parent collection: [learning epic ID]

**Table 3: Grease Streams**
```
| Stream | ID | Status | Beads (Open/Total) | Priority | Root Cause |
```

Parent collection: hq-rib

**Table 4: Grease Convoys**
```
| Convoy | Status | Beads (Done/Total) | Created | Notes |
```

## Implementation Algorithm

### Building Table 1: Meta Documentation

```python
# Pseudocode
# hq-rib.11 is the META EPIC (parent container)
# We show ALL its children (siblings) in the table
meta_docs = query("bd list --parent=hq-rib.11")

if not meta_docs:
    output("No meta documentation found.")
    return

for doc in meta_docs:
    details = query(f"bd show {doc.id}")
    row = {
        "Doc": clean_title(details.title),  # Remove "META:" prefix if present
        "ID": doc.id,
        "Status": doc.status,
        "Description": first_sentence(details.description)
    }
    output_row(row)
```

**Key**:
- Show ALL children of hq-rib.11 epic (any status, any type)
- These are sibling beads under the META epic collection
- The epic itself (hq-rib.11) is NOT shown in the table

### Building Table 2: Learning Collection

```python
# Pseudocode
# Find the learning collection epic (has grease, learning, meta labels)
learning_epic_id = find_learning_epic_id()  # Query for epic with learning label under hq-rib

if not learning_epic_id:
    output("No learning collection found.")
    return

learning_beads = query(f"bd list --parent={learning_epic_id}")

if not learning_beads:
    output("No learning beads found.")
    return

for bead in learning_beads:
    details = query(f"bd show {bead.id}")
    row = {
        "Learning": clean_title(details.title),  # Remove "LEARNING:" prefix if present
        "ID": bead.id,
        "Status": bead.status,
        "Description": first_sentence(details.description)
    }
    output_row(row)
```

**Key**:
- Find the learning collection epic dynamically (has labels: grease, learning, meta)
- Show ALL children of the learning epic (any status, any type)
- These are learning beads documenting workflows, patterns, investigations
- The epic itself is NOT shown in the table

### Building Table 2.5: Learning Streams

```python
# Pseudocode
# Find the learning collection epic dynamically
learning_epic_id = find_learning_epic_id()  # Query for epic with learning label under hq-rib

if not learning_epic_id:
    output("No learning collection found.")
    return

# Get all child epics of the learning collection
all_epics = query(f"bd list --parent={learning_epic_id} --type=epic")
learning_streams = []

for epic in all_epics:
    # Filter logic
    if "STREAM" not in epic.title.upper():
        continue  # Not a stream epic
    if epic.status == "closed":
        continue  # Closed streams are archived knowledge

    # Count children (learning beads in this stream)
    open_beads = query(f"bd list --parent={epic.id} --status=open").count()
    total_beads = query(f"bd list --parent={epic.id}").count()

    row = {
        "Stream": extract_stream_title(epic.title),  # "LEARNING STREAM 1: Agent Lifecycle"
        "ID": epic.id,
        "Status": epic.status,
        "Beads": f"{open_beads}/{total_beads}",
        "Priority": epic.priority or "P2",
        "Topic": extract_topic(epic.description)  # First sentence of description
    }
    learning_streams.append(row)

# Sort by priority, then stream number
learning_streams.sort(key=lambda x: (x["Priority"], x["Stream"]))

if not learning_streams:
    output("No active learning streams.")
else:
    output_table(learning_streams)
```

**Key**:
- Find learning collection epic dynamically (has labels: grease, learning, meta)
- Show ONLY open stream epics under the learning collection
- Count learning beads (children) for each stream
- Sort by priority first, then stream number
- Topic column shows learning focus area (vs "Root Cause" for grease streams)

### Building Table 3: Grease Streams

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
        "ID": epic.id,
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

### Building Table 4: Grease Convoys

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
- Five tables always present (even if empty)
- Dynamic data only - what's in the DB NOW

## Expected Output Format

When El Presidente runs `/grease-report`, output contains FIVE tables with current database state:

**Table 1: Grease Meta Documentation**
- Parent collection: hq-rib.11
- All children of hq-rib.11 META epic (any status, any type)
- Items are siblings: validation workflows, taxonomies, runbooks, process docs
- Columns: Doc | ID | Status | Description
- Empty state: "No meta documentation found."
- Note: The epic itself (hq-rib.11) is NOT shown, only its children

**Table 2: Learning Collection**
- Parent collection: [learning epic ID]
- All children of the learning collection epic (any status, any type)
- Items are learning beads documenting workflows, patterns, investigations
- Columns: Learning | ID | Status | Description
- Empty state: "No learning beads found."
- Note: The learning collection epic itself is NOT shown, only its children

**Table 2.5: Learning Streams**
- Parent collection: [learning epic ID]
- Open learning stream epics under learning collection (title contains "STREAM")
- Stream epics organize learning beads by conceptual area
- Sorted by priority, then stream number
- Columns: Stream | ID | Status | Beads (Open/Total) | Priority | Topic
- Empty state: "No active learning streams."
- Note: Shows stream epics, not individual learning beads

**Table 3: Grease Streams Status**
- Parent collection: hq-rib
- Open stream epics under hq-rib (title contains "STREAM")
- Sorted by priority, then stream number
- Columns: Stream | ID | Status | Beads (Open/Total) | Priority | Root Cause
- Empty state: "No active grease streams."

**Table 4: Grease Convoys**
- Open convoys with grease label
- Columns: Convoy | Status | Beads (Done/Total) | Created | Notes
- Empty state: "No active grease convoys."

**No additional output, commentary, or analysis.**

## Advanced Features

- **Auto-routing**: Uses `bd` prefix routing to find beads across rigs
- **Status inference**: Determines convoy blocking based on dependency beads
- **Compact format**: One-line summaries without verbose details
- **Zero commentary**: Pure data output for programmatic consumption
