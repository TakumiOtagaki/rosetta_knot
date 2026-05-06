# RNA Denovo with Knot Penalty Documentation

This folder contains documentation about the RNAknotDetector integration with Rosetta's FARFAR2 RNA fragment assembly method.

## Key Findings

### Program Reference
- **rna_denovo**: The main executable for RNA de novo structure prediction
  - Full name: Fragment Assembly of RNA with Full Atom Refinement (FARFAR)
  - FARFAR2 is the improved version (2020+) of the same `rna_denovo` binary
  - Details: [01_program_reference.md](01_program_reference.md)

### Knot Penalty Integration
- Knot penalty has been **explicitly integrated into FARFAR2's Monte Carlo assembly loop**
- Controlled via environment variables
- Implemented in: `source/src/protocols/rna/denovo/RNA_FragmentMonteCarlo.cc`
- Details: [02_knot_penalty_integration.md](02_knot_penalty_integration.md)

### Build Instructions
- Selective build is possible (no need to build everything)
- Required: `rna_denovo` and `extract_pdbs`
- Details: [03_build_guide.md](03_build_guide.md)

## Quick Reference

### Programs & Binaries
| Program | Purpose | Binary Location |
|---------|---------|-----------------|
| **rna_denovo** | RNA structure prediction | `source/src/apps/public/rna/rna_denovo.cc` |
| **FARFAR2** | Optimized version of rna_denovo (same binary) | Same as rna_denovo |
| **extract_pdbs** | Convert silent files to PDB | `source/src/apps/public/extract_pdbs.cc` |

### Environment Variables
```bash
# Enable knot penalty with weight
export ROSETTA_KNOT_PENALTY_WEIGHT=0.5

# Optional: detailed evaluation logging
export ROSETTA_KNOT_EVAL_LOG=/path/to/log.txt
```

### Build Commands
```bash
cd source

# Build rna_denovo (FARFAR2)
python3 scons.py -j 4 mode=release bin/rna_denovo.macosclangrelease

# Build silent-to-PDB converter
uv run scons.py -j 4 mode=release bin/extract_pdbs.macosclangrelease

# Build both together
uv run scons.py -j 4 mode=release bin/rna_denovo.macosclangrelease bin/extract_pdbs.macosclangrelease
```
