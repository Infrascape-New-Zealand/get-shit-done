# Decimal Phase Calculation

Calculate the next decimal phase number for urgent insertions.

## Calculating Next Decimal

To find the next decimal phase after a given phase, scan the `context/phases/` directory for existing decimal phases:

```bash
# Get next decimal phase after phase 6
# Scan context/phases/ for directories matching "06.*"
AFTER_PHASE="06"
EXISTING=$(ls -d context/phases/${AFTER_PHASE}.*-* 2>/dev/null | grep -oP "${AFTER_PHASE}\.\K[0-9]+" | sort -n | tail -1)
if [ -z "$EXISTING" ]; then
  DECIMAL_PHASE="${AFTER_PHASE}.1"
else
  DECIMAL_PHASE="${AFTER_PHASE}.$((EXISTING + 1))"
fi
```

Output structure:
```json
{
  "found": true,
  "base_phase": "06",
  "next": "06.1",
  "existing": []
}
```

With existing decimals:
```json
{
  "found": true,
  "base_phase": "06",
  "next": "06.3",
  "existing": ["06.1", "06.2"]
}
```

## Extract Values

```bash
# Scan for existing decimal phases and compute next
AFTER_PHASE="06"
EXISTING=$(ls -d context/phases/${AFTER_PHASE}.*-* 2>/dev/null | grep -oP "${AFTER_PHASE}\.\K[0-9]+" | sort -n | tail -1)
if [ -z "$EXISTING" ]; then
  DECIMAL_PHASE="${AFTER_PHASE}.1"
else
  DECIMAL_PHASE="${AFTER_PHASE}.$((EXISTING + 1))"
fi
BASE_PHASE="${AFTER_PHASE}"
```

## Examples

| Existing Phases | Next Phase |
|-----------------|------------|
| 06 only | 06.1 |
| 06, 06.1 | 06.2 |
| 06, 06.1, 06.2 | 06.3 |
| 06, 06.1, 06.3 (gap) | 06.4 |

## Directory Naming

Decimal phase directories use the full decimal number:

```bash
SLUG=$(echo "$DESCRIPTION" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
PHASE_DIR="context/phases/${DECIMAL_PHASE}-${SLUG}"
mkdir -p "$PHASE_DIR"
```

Example: `context/phases/06.1-fix-critical-auth-bug/`
