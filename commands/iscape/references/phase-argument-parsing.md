# Phase Argument Parsing

Parse and normalize phase arguments for commands that operate on phases.

## Extraction

From `$ARGUMENTS`:
- Extract phase number (first numeric argument)
- Extract flags (prefixed with `--`)
- Remaining text is description (for insert/add commands)

## Finding a Phase Directory

Scan the `context/phases/` directory to find a matching phase:

```bash
# Find phase directory by number
PHASE_DIR=$(ls -d context/phases/${PHASE}-* 2>/dev/null | head -1)
```

Returns:
- `directory`: Full path to phase directory
- `phase_number`: Normalized number (e.g., "06", "06.1")
- `phase_name`: Name portion (e.g., "foundation")
- Plans and summaries can be found with: `ls "$PHASE_DIR"/*/PLAN.md` and `ls "$PHASE_DIR"/*/SUMMARY.md`

## Normalization

Zero-pad integer phases to 2 digits. Preserve decimal suffixes.

```bash
# Normalize phase number
if [[ "$PHASE" =~ ^[0-9]+$ ]]; then
  # Integer: 8 -> 08
  PHASE=$(printf "%02d" "$PHASE")
elif [[ "$PHASE" =~ ^([0-9]+)\.([0-9]+)$ ]]; then
  # Decimal: 2.1 -> 02.1
  PHASE=$(printf "%02d.%s" "${BASH_REMATCH[1]}" "${BASH_REMATCH[2]}")
fi
```

## Validation

Validate the phase exists in the roadmap:

```bash
# Check phase exists in ROADMAP.md
if ! grep -qE "Phase ${PHASE}[^0-9]" context/ROADMAP.md 2>/dev/null; then
  echo "ERROR: Phase ${PHASE} not found in roadmap"
  exit 1
fi
```

## Directory Lookup

Find the phase directory by scanning `context/phases/`:

```bash
PHASE_DIR=$(ls -d context/phases/${PHASE}-* 2>/dev/null | head -1)
if [ -z "$PHASE_DIR" ]; then
  echo "ERROR: No directory found for phase ${PHASE}"
  exit 1
fi
```
