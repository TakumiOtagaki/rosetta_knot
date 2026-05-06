# RNA Programs Reference

## rna_denovo (FARFAR)

### Full Name
Fragment Assembly of RNA with Full Atom Refinement (FARFAR)

### Purpose
De novo and homology-based 3D RNA structure prediction for RNA molecules up to ~250 nucleotides.

### Implementation
- **Main Application**: `source/src/apps/public/rna/rna_denovo.cc`
- **Core Protocol**: `src/protocols/rna/denovo/RNA_DeNovoProtocol.cc`
- **Monte Carlo Loop**: `source/src/protocols/rna/denovo/RNA_FragmentMonteCarlo.cc`

### Key Features
- Fragment assembly using FASTA template libraries
- Full atomic refinement of backbone and side chains
- Base pair step sampling for helix flexibility
- Monte Carlo search with energy-based scoring

---

## FARFAR2

### Status
**Not a separate binary** — It is the improved version of `rna_denovo` released in 2020+.

### Full Name
Fragment Assembly of RNA with Full Atom Refinement (version 2)

### Improvements over FARFAR
1. **Updated Fragment Library**: 2018 library vs. 2009 in original FARFAR
2. **Parameter Optimization**: Stepwise Monte Carlo optimization
3. **New Scoring Function**: Improved energy function
4. **Enhanced Flexibility**: Base pair step sampling

### Key Publication
Watkins, A.M., et al. "FARFAR2: Improved de novo Rosetta prediction of complex global RNA folds." Structure, 2020.

### Usage
Same as `rna_denovo`:
```bash
rna_denovo [options]
```

---

## extract_pdbs

### Full Name
Silent-to-PDB Converter

### Purpose
Converts Rosetta's internal silent file format to standard PDB format.

### Implementation
`source/src/apps/public/extract_pdbs.cc`

### Usage
```bash
extract_pdbs -in:file:silent <silent_file> [-tags tag1 tag2 ...]
```

### Options
- `-in:file:silent <file>`: Input silent file
- `-tags <tag1> <tag2> ...`: Specific pose tags to extract (optional; extracts all if omitted)
- `-out:pdb`: Output directory for PDB files (default: current directory)

### Output
Individual PDB files named according to tags in the silent file.

---

## Related Implementation Files

### Knot Penalty Integration (FARFAR2-specific)
- **Bridge to C++**: `source/src/protocols/rna/denovo/knot/entanglement_bridge.cc`
- **Entanglement Algorithm**: `RNAknotDetector/cpp/core/entanglement.h`
- **Monte Carlo Integration**: `source/src/protocols/rna/denovo/RNA_FragmentMonteCarlo.cc` (lines 114-290)

### Build Fix (Fixed issue)
- **Problem File**: `RNAknotDetector/cpp/core/geometry3d.cpp`
- **Missing Include**: `#include <algorithm>` (required for `std::sort`)
- **Status**: ✅ Fixed
