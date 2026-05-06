# Knot Penalty Integration in FARFAR2

## Overview

The knot penalty has been **explicitly integrated into FARFAR2's Monte Carlo assembly loop** to suppress unwanted entangled geometries during RNA structure prediction.

### Background
During FARFAR2's local transitions (fragment assembly moves), the search can encounter geometries where RNA loops cross each other in ways that create "entanglement" — backbone segments crossing loop surfaces. This can create artifacts in pseudoknot-free secondary structures.

### Solution
Implement a lightweight entanglement detection and scoring penalty to guide the Monte Carlo search away from entangled regions.

---

## Implementation Details

### Core Files
1. **Monte Carlo Integration**: `source/src/protocols/rna/denovo/RNA_FragmentMonteCarlo.cc`
   - Lines 114-125: Environment variable initialization
   - Lines 156-166: Penalty calculation with logging
   - Lines 274-290: Conditional evaluation logic
   - Lines 550+: Integration into move acceptance

2. **Entanglement Detection**: `RNAknotDetector/cpp/core/entanglement.h`
   - K-value (entanglement count) calculation
   - Detailed hit information (which loops, which backbone segments)

3. **Bridge to Rosetta**: `source/src/protocols/rna/denovo/knot/entanglement_bridge.cc`
   - C++ interface between Rosetta and RNAknotDetector

### Key Data Structures

#### KnotEnvConfig struct
Manages environment variable initialization at startup:
- Reads environment variables
- Sets default values (knot_penalty_weight = 0.0 by default)

#### KnotContext struct
Manages knot evaluation during Monte Carlo:
- Tracks base pairs from secondary structure (main layer only)
- Builds loop structures (hairpin, internal, multi-loop)
- Maintains previous entanglement count (`k_last`)

### Penalty Calculation

```
penalty_energy = weight × delta_K / temperature

where:
  - weight = ROSETTA_KNOT_PENALTY_WEIGHT (environment variable)
  - delta_K = K_current - K_previous (change in entanglement count)
  - temperature = Monte Carlo temperature
```

### Monte Carlo Integration

During the assembly loop after each fragment move:

1. **Evaluate current structure**: `evaluate_knot_K()`
   - Extract residue coordinates (P and C4' atoms)
   - Build loop structures from secondary structure
   - Calculate surface representations
   - Detect backbone-surface intersections
   - Return K (entanglement count)

2. **Calculate energy delta**:
   ```
   score_delta = weight × delta_K / temperature
   ```

3. **Apply Metropolis criterion**:
   - If delta_K increases entanglement: move is less likely to be accepted
   - If delta_K decreases entanglement: move is more likely to be accepted
   - Higher temperature = more exploratory sampling

---

## Configuration

### Environment Variables

#### ROSETTA_KNOT_PENALTY_WEIGHT
Controls the strength of the knot penalty.

```bash
# Disable knot penalty (default)
export ROSETTA_KNOT_PENALTY_WEIGHT=0.0

# Enable with moderate penalty (recommended for testing)
export ROSETTA_KNOT_PENALTY_WEIGHT=0.5

# Enable with strong penalty
export ROSETTA_KNOT_PENALTY_WEIGHT=1.0

# Fine-tuning
export ROSETTA_KNOT_PENALTY_WEIGHT=0.25
```

**Default**: 0.0 (knot penalty disabled)

#### ROSETTA_KNOT_EVAL_LOG (optional)
Enable detailed logging of knot evaluation during Monte Carlo.

```bash
# Enable logging to file
export ROSETTA_KNOT_EVAL_LOG=/path/to/knot_eval.log

# Log format:
# round,iter,move_type,K,penalty,accept
# Example line:
# 1,42,fragment,2,0.5,true
```

This logs:
- `round`: Monte Carlo round number
- `iter`: Iteration within round
- `move_type`: Type of move (fragment, local, etc.)
- `K`: Current entanglement count
- `penalty`: Penalty applied
- `accept`: Whether move was accepted

---

## Evaluation Details

### K-value (Entanglement Count)
The K-value represents the number of backbone-surface intersections detected in the current RNA structure.

- **K = 0**: No entanglement (clean structure)
- **K > 0**: Backbone segments cross loop surfaces (entanglement detected)

### Surface Representations
For each loop (hairpin, internal, multi-loop):
- Represents the loop surface as a geometric volume
- Uses vertices from loop residues and backbone geometry

### Intersection Detection
For each backbone segment:
- Checks if the backbone crosses any loop surface
- Uses P (phosphorus) and C4' atoms for backbone representation
- Accumulates total intersections to get K

---

## Context Tracking

The KnotContext struct maintains:

1. **Previous K value** (`k_last`)
   - Used to calculate delta_K
   - Updated after each evaluation

2. **Secondary structure**
   - Main layer base pairs
   - Loop definitions

3. **Scoring state**
   - Current penalty weight
   - Temperature factor

---

## Notes

- **Scope**: Applies only to pseudoknot-free secondary structure contexts
- **Performance**: Lightweight detection algorithm; minimal overhead to Monte Carlo
- **Composability**: Works alongside existing FARFAR2 scoring functions
- **Tuning**: Penalty weight can be adjusted per protocol or per prediction run

---

## Documentation

For more details, see:
- `RNAknotDetector/documents/rd/overview.md` — Background and motivation
- `RNAknotDetector/documents/rd/api.md` — API reference
- `source/src/protocols/rna/denovo/RNA_FragmentMonteCarlo.cc` — Implementation details
