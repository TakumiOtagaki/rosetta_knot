# Build Guide: rna_denovo (FARFAR2) and extract_pdbs

## Prerequisites

### System Requirements
- C++11 or later compiler (GCC 11 confirmed working)
- Linux or macOS
- Python 2 or 3 (for scons build system)
- ~10GB+ disk space for full build

### Dependencies
All required dependencies are included in the Rosetta repository.

### Pre-Build Fix (Required)
Before building, ensure the geometry3d.cpp file has the necessary `<algorithm>` include:

```bash
# Verify the fix is in place
grep "#include <algorithm>" /Users/ootagakitakumi/rosetta_knot/RNAknotDetector/cpp/core/geometry3d.cpp
```

If the output is empty, the file still needs the fix. See the root repository notes for the correction.

---

## Selective Build (Recommended)

### Build Only What You Need

Instead of building the entire Rosetta package, you can build only the RNA-related tools you need.

#### Option 1: Build rna_denovo only
```bash
cd /Users/ootagakitakumi/rosetta_knot/source

uv run scons.py -j 4 mode=release bin/rna_denovo
```

**Time**: ~10-20 minutes (depending on system)
**Output**: `build/src/release/linux/5.14/64/x86/gcc/11/default/rna_denovo`

#### Option 2: Build extract_pdbs only
```bash
cd /Users/ootagakitakumi/rosetta_knot/source

uv run scons.py -j 4 mode=release bin/extract_pdbs
```

**Time**: ~5-10 minutes
**Output**: Silent-to-PDB converter

#### Option 3: Build both together (Recommended)
```bash
cd /Users/ootagakitakumi/rosetta_knot/source

uv run scons.py -j 4 mode=release bin/rna_denovo bin/extract_pdbs
```

---

## Build Parameters Explained

| Parameter | Meaning | Options |
|-----------|---------|---------|
| `-j 4` | Number of parallel compilation jobs | Use number of CPU cores available |
| `mode=release` | Optimization level | `debug`, `release` (default: release) |
| `bin/rna_denovo` | Target binary name | Use `bin/extract_pdbs` for other tools |

### Recommended Settings
- `-j 8` or higher: If you have 8+ CPU cores
- `-j 4`: Safe default for most systems
- `-j 1`: If compilation fails (disable parallelization)

---

## Build Variants

### Debug Build (for development/debugging)
```bash
uv run scons.py -j 4 mode=debug bin/rna_denovo
```

**Pros**: Better debugging symbols, easier to debug crashes
**Cons**: Slower runtime, larger binary

### Release Build (for production runs)
```bash
uv run scons.py -j 4 mode=release bin/rna_denovo
```

**Pros**: Optimized, faster execution
**Cons**: Harder to debug

---

## After Successful Build

### Locate Your Binaries
```bash
# After successful build, binaries will be in:
ls -la build/src/release/linux/5.14/64/x86/gcc/11/default/rna_denovo
ls -la build/src/release/linux/5.14/64/x86/gcc/11/default/extract_pdbs
```

### Create Symlinks (optional but convenient)
```bash
cd /Users/ootagakitakumi/rosetta_knot/source

# Create shortcuts in the bin directory
mkdir -p bin_release
ln -s ../build/src/release/linux/5.14/64/x86/gcc/11/default/rna_denovo bin_release/
ln -s ../build/src/release/linux/5.14/64/x86/gcc/11/default/extract_pdbs bin_release/

# Usage: run like this
./bin_release/rna_denovo [options]
./bin_release/extract_pdbs -in:file:silent [silent_file]
```

---

## Running with Knot Penalty

### Basic Usage
```bash
# Set knot penalty weight
export ROSETTA_KNOT_PENALTY_WEIGHT=0.5

# Run rna_denovo
/path/to/rna_denovo -fasta sequence.fasta -nstruct 10 [other options]
```

### With Detailed Logging
```bash
# Enable knot evaluation logging
export ROSETTA_KNOT_PENALTY_WEIGHT=0.5
export ROSETTA_KNOT_EVAL_LOG=knot_eval.log

# Run prediction
/path/to/rna_denovo -fasta sequence.fasta -nstruct 10

# Check results
cat knot_eval.log
```

### Extract Results to PDB
```bash
# Convert silent output to PDB files
extract_pdbs -in:file:silent output.silent

# Extract specific structures
extract_pdbs -in:file:silent output.silent -tags S_0000 S_0001 S_0002
```

---

## Troubleshooting

### Build Fails with "error: 'sort' is not a member of 'std'"
**Solution**: The geometry3d.cpp file is missing `#include <algorithm>`. 

See root repository notes or run:
```bash
grep "#include <algorithm>" RNAknotDetector/cpp/core/geometry3d.cpp
```

If empty, the pre-build fix hasn't been applied.

### Build Fails with Memory Error
**Solution**: Reduce parallelization
```bash
uv run scons.py -j 1 mode=release bin/rna_denovo
```

### Build Fails with Missing Dependencies
**Solution**: Ensure submodules are initialized
```bash
git submodule update --init --recursive
```

### Slow Build
**Solution**: Increase parallelization (if you have more CPU cores)
```bash
uv run scons.py -j 16 mode=release bin/rna_denovo
```

---

## Version Information

### Build System
- **Rosetta Build Tool**: SCons (Python-based build automation)
- **Language**: C++11
- **Compiler Tested**: GCC 11 (tested on Linux with GCC 11.x.x)

### Package Versions
- **FARFAR2**: 2020+ (integrated into rna_denovo)
- **RNAknotDetector**: Integrated into Rosetta source

---

## Next Steps

1. **Build the tools**: Follow "Option 3" above
2. **Prepare your sequence**: Create FASTA file with RNA sequence
3. **Run prediction**: Use `rna_denovo` with knot penalty enabled
4. **Extract results**: Use `extract_pdbs` to convert silent output to PDB
5. **Analyze**: Examine the K-values and structures

See the main README.md for quick reference commands.
