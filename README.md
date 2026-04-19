# ProtoCubix 🧬

> **Protein-Origin Tectonic Ordered Cubic Assembly**
> Atomistic molecular dynamics of a protein-based cubic superlattice using GROMACS + PyMOL

[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)
[![GROMACS](https://img.shields.io/badge/GROMACS-2023.3-blue)](https://www.gromacs.org/)
[![PyMOL](https://img.shields.io/badge/PyMOL-open--source-green)](https://pymol.org/)
[![Force Field](https://img.shields.io/badge/Force%20Field-AMBER99SB-orange)]()

---

## What is ProtoCubix?

ProtoCubix is a workflow for building and simulating **protein-based cubic superlattices** using atomistic molecular dynamics. It takes a single protein structure (PDB file) and tiles it into a 3D periodic lattice — treating each protein as a structural building block, similar to how atoms arrange in a crystal.

<img width="1688" height="789" alt="pymol" src="https://github.com/user-attachments/assets/1349227f-22da-408e-9d2f-ad956c918cdb" />


The result is a **Simple Cubic supercell** of 27 protein copies (3×3×3), energy-minimized and ready for MD simulation or structural analysis.

```
Single protein  →  Unit cell  →  3×3×3 superlattice  →  Energy minimized
   (1UBQ.pdb)      (boxed)         (lattice.gro)          (em.gro)
```

---

## Modeling Type

This workflow operates at the **Atomistic MD** level — every atom is tracked explicitly using Newton's equations of motion with the AMBER99SB force field.

| Scale | Tool | What's tracked |
|-------|------|----------------|
| Quantum mechanics | Gaussian | electrons |
| **Atomistic MD ← you are here** | **GROMACS** | **every atom** |
| Coarse-grained | MARTINI | bead = residue groups |
| Continuum | ANSYS/FEA | bulk properties |

---

## Requirements

- [GROMACS](https://www.gromacs.org/) ≥ 2020
- [PyMOL](https://pymol.org/) (open-source version works)
- `wget` (for downloading PDB files)

```bash
# Check GROMACS is installed
gmx --version

# Check PyMOL is installed
pymol --version
```

---

## Full Workflow

### Step 1 — Get protein structure

```bash
# Download ubiquitin from RCSB (or replace with your PDB ID)
wget https://files.rcsb.org/download/1UBQ.pdb
```

### Step 2 — Prepare with GROMACS

```bash
# Convert PDB → GROMACS format
# Choose AMBER99SB forcefield (option 1 when prompted)
gmx pdb2gmx -f 1UBQ.pdb -o protein.gro -water spce -ff amber99sb -ignh

# Put protein in a cubic box with 1.2 nm padding
gmx editconf -f protein.gro -o protein_box.gro -bt cubic -d 1.2 -c
```

### Step 3 — Build the lattice

```bash
# KEY COMMAND — replicate into a 3×3×3 supercell (27 protein copies)
gmx genconf -f protein_box.gro -o lattice.gro -nbox 3 3 3

# Convert to PDB for visualization
gmx editconf -f lattice.gro -o lattice.pdb
```

### Step 4 — Energy minimization

Create `minim.mdp`:

```bash
cat > minim.mdp << EOF
integrator      = steep
emtol           = 1000.0
nsteps          = 50000
cutoff-scheme   = Verlet
nstlist         = 10
rcoulomb        = 1.0
rvdw            = 1.0
pbc             = xyz
EOF
```

Run minimization:

```bash
gmx grompp -f minim.mdp -c lattice.gro -p topol.top -o em.tpr -maxwarn 5
gmx mdrun -v -deffnm em

# Convert minimized structure to PDB
gmx editconf -f em.gro -o lattice_final.pdb
```

### Step 5 — Visualize in PyMOL

Launch PyMOL:

```bash
pymol lattice_final.pdb
```

Then type inside the PyMOL command line:

```python
# Color each of the 27 protein copies differently
util.cbc

# Show secondary structure
hide everything
show cartoon

# Render settings
set cartoon_fancy_helices, 1
bg_color white
zoom

# Save high-resolution image
ray 1200, 900
png ProtoCubix.png
```

---

## Lattice Types

`gmx genconf -nbox` builds a **Simple Cubic** lattice by default. Change the repeat counts to vary geometry:

```bash
# Simple Cubic (default)
gmx genconf -f protein_box.gro -o lattice.gro -nbox 3 3 3

# Slab (single layer)
gmx genconf -f protein_box.gro -o lattice.gro -nbox 4 4 1

# Elongated column
gmx genconf -f protein_box.gro -o lattice.gro -nbox 2 2 6
```

For true BCC/FCC topologies, a second shifted copy must be manually overlaid using `gmx editconf -translate`.

---

## Output Files

| File | Description |
|------|-------------|
| `protein.gro` | Prepared single protein |
| `protein_box.gro` | Protein in cubic box |
| `lattice.gro` | 3×3×3 supercell (pre-minimization) |
| `lattice.pdb` | Same, in PDB format |
| `em.gro` | Energy-minimized lattice |
| `lattice_final.pdb` | Final structure for PyMOL |
| `ProtoCubix.png` | Rendered visualization |

---

## PyMOL Quick Reference

```python
show surface          # molecular surface
show sticks           # bond representation
color rainbow, all    # rainbow from N to C terminus
set transparency, 0.5 # transparent surface
bg_color black        # dark background
ray                   # high-quality render
```

---

## Switching Between GROMACS and PyMOL

```bash
# Run PyMOL in background so terminal stays free
pymol &

# Do GROMACS work in the same terminal
gmx mdrun -v -deffnm em
gmx editconf -f em.gro -o lattice_final.pdb

# Reload in PyMOL without restarting
# (type inside PyMOL)
reinitialize
load lattice_final.pdb
```

Or run bash commands directly from inside PyMOL using `!`:

```python
# Inside PyMOL command line
!gmx editconf -f em.gro -o lattice_final.pdb
!ls -lh *.pdb
```

---

## Author

**Akshansh** — [@akshansh11](https://github.com/akshansh11)

---

## License

[![CC BY-NC 4.0](https://licensebuttons.net/l/by-nc/4.0/88x31.png)](https://creativecommons.org/licenses/by-nc/4.0/)

This work is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License](https://creativecommons.org/licenses/by-nc/4.0/).

You are free to:
- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material

Under the following terms:
- **Attribution** — You must give appropriate credit to [@akshansh11](https://github.com/akshansh11)
- **NonCommercial** — You may not use the material for commercial purposes

---

*ProtoCubix — where proteins become crystals.*
