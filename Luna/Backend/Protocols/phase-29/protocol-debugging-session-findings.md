# Protocol Debugging Session - Findings and Lessons

Etiquetas: #luna_help_desk 

**Date**: 2025-08-15  
**Issue**: `Phase/question cardinality mismatch 09aa5740-e95a-4dcf-a2bf-0334074b1bd9` error when running `bundle exec rake protocols:phase_29`

## Problem Analysis

### Root Cause

Protocol `09aa5740-e95a-4dcf-a2bf-0334074b1bd9` (Spine Surgery Prevention) had a **cardinality mismatch** between:
- **Phase periods**: 14 phases defined in `phase_periods`
- **Question phases**: Only 12 characters in question `phases` fields

### Validation Logic

The error originates from `app/seeders/protocol_seeder.rb:189-191`:
```ruby
if phase_characters.size != phase_periods.size
  raise "Phase/question cardinality mismatch #{protocol_hash.fetch('id')}"
end
```

This validation ensures each question's `phases` string has exactly as many characters as there are `phase_periods`.

## Investigation Process

### 1. Pattern Analysis Across Protocols

- **Normal protocols**: 4-5 phases with matching character counts
  - Example: `phase_periods: ["3:10", "11:23", "24:37", "38:48"]` → `phases: oo..` (4 chars)
- **Large protocols**: 7-13 phases with matching character counts
  - Example: 12 phases → 12-character phase strings
- **Problematic protocol**: 14 phases → 12-character phase strings ❌

### 2. Comprehensive Protocol Survey

Found only **one protocol** with cardinality mismatch out of 100+ protocols:
- `09aa5740-e95a-4dcf-a2bf-0334074b1bd9` had 14 phases but 12-character question patterns
- All other high-phase protocols (up to 13 phases) had correct matching counts

## Resolution Steps

### 1. Fixed Phase Period Count

**Before**: 14 phases
```yaml
phase_periods: ["0:0","1:1","2:2","3:3","4:4","5:5","6:6","7:7","8:8","9:9","10:10","11:11","12:12","13:13"]
```

**After**: 12 phases (removed `"12:12","13:13"`)
```yaml
phase_periods: ["0:0","1:1","2:2","3:3","4:4","5:5","6:6","7:7","8:8","9:9","10:10","11:11"]
```

### 2. Fixed Individual Question Issue

One question had incorrect phase pattern:
**Before**: `phases: .o..` (4 characters)
**After**: `phases: ..o...o...o.` (12 characters)

## Secondary Issue: Database vs Seeds Mismatch

### Problem

After fixing cardinality, encountered:
```
Protocol ID mismatch: Extras via database: #<Set: {"22b9bd1c-945e-4a52-a5cf-4fdee9e0af20", "6298c2b5-bdbb-4ae7-830a-ca9a438b19e7", "ac39e49d-7fa0-468b-8f9a-be904dac8415", "ad43db53-2476-4934-8027-215d587d1088"}>
```

### Root Cause

Development/staging database contained protocols not present in seed files. This occurs because:
- QA protocols exist in database but not in production seed files
- Development protocols were created during testing
- Previous protocol versions remain in database after seed file removal

### Validation Logic

From `lib/tasks/protocols.rake:360-374`, the `validate_protocol_numbers` method ensures:
- Protocol IDs in seed files exactly match Protocol IDs in database
- This prevents production deployments with mismatched protocol sets

## Key Learnings

### 1. Protocol Structure Requirements

- **Phase periods** define the number of phases in a protocol
- **Question phases** must have exactly one character per phase period
- Each character (`.`, `o`) represents whether a question applies to that phase

### 2. Protocol Validation is Multi-layered

1. **Cardinality validation**: Questions match phase count
2. **Pathway validation**: All referenced pathways exist
3. **Database consistency**: Seeds match database exactly

### 3. Environment Considerations

- Development/staging environments accumulate test data over time
- Production validation is strict to ensure deployment consistency
- The `live_mode` parameter controls whether changes are committed or rolled back

### 4. Debugging Approach

- **Start with the specific error message** - it points to exact validation logic
- **Compare patterns across similar entities** - helped identify this was an isolated issue
- **Understand the business logic** - phase characters map to protocol phases
- **Check environment context** - staging vs production data differences

## Technical Files Involved

### Core Files

- `lib/tasks/protocols.rake` - Main rake task and validation logic
- `app/seeders/protocol_seeder.rb` - Protocol seeding and cardinality validation
- `db/seeds/protocols/09aa5740-e95a-4dcf-a2bf-0334074b1bd9.yml` - Problematic protocol file

### Models (Inferred)

- `Protocol` - Main protocol model
- `ProtocolPhase` - Individual phases within protocols  
- `ClinicalEscalationQuestion` - Questions asked during protocol phases
- `ProtocolEscalationQuestion` - Join table linking questions to specific phases

## Commands Used

```bash
# Run the failing task
bundle exec rake protocols:phase_29

# Search for similar patterns
find db/seeds/protocols -name "*.yml" -exec grep -l "13:13" {} \;

# Count phase periods across protocols
find db/seeds/protocols -name "*.yml" -exec bash -c 'phase_count=$(grep "phase_periods:" "$1" | grep -o ":" | wc -l); if [ $phase_count -gt 6 ]; then echo "$1 has $phase_count phases"; fi' _ {} \;

# Count characters in phase patterns
awk '/phases:/ {gsub(/.*phases: /, ""); gsub(/ *$/, ""); print length($0) ": " $0}' db/seeds/protocols/09aa5740-e95a-4dcf-a2bf-0334074b1bd9.yml
```

## Best Practices Identified

### 1. Protocol Maintenance

- Always validate phase period count matches question phase character count
- Use consistent naming and structure across protocol files
- Maintain clean separation between production and QA protocol sets

### 2. Debugging Process

- Read error messages carefully - they often point to exact validation logic
- Use systematic comparison across similar entities to identify patterns
- Understand the business domain (phases, questions, protocols) before diving into technical details

### 3. Environment Management

- Be aware that staging/dev environments accumulate test data over time
- Production validation is intentionally strict to prevent inconsistent deployments
- Use appropriate flags (like `live_mode`) to control deployment behavior