# circRNA-energy-minimization-gromacs-workflow
Energy minimization workflow for circular RNA (circRNA) structures using GROMACS with AMBER14SB-OL15 corrections and topology adaptation.
# 🧬 Circular RNA (circRNA) Energy Minimization Workflow using GROMACS

---

## 📌 Overview

This repository provides a **step-by-step workflow** for performing energy minimization of a circular RNA (circRNA) structure using GROMACS.

The initial 3D structure was obtained from the **3dRNA/DNA web server developed by Xiao Lab**, described in:

> Xiao Lab – 3dRNA/DNA Server  
> DOI: https://doi.org/10.1016/j.jmb.2022.167452  

Due to computational and physical limitations, **current classical force fields are not inherently designed to handle covalently closed circular RNA molecules (circRNAs)**.

Because of this limitation:

- The topology file generated from the Xiao Lab server was opened as a text file.
- Terminal phosphate groups preventing proper force field recognition were manually removed.
- This allowed the force field to properly initialize minimization.

To improve RNA structural accuracy, the most recent corrected RNA-compatible force field was downloaded:

```bash
wget https://ftp.gromacs.org/pub/contrib/forcefields/amber14sb_OL15.ff_corrected-Na-cation-params.tar.gz
```

### Why this force field?

The **AMBER14SB + OL15 corrections** include:

- Improved RNA backbone torsion parameters  
- Better χ torsion treatment  
- Corrected sodium ion parameters  
- Improved structural stability for nucleic acids  

---

## 🧠 Simplification Strategy

To maintain reproducibility and clarity:

- All workflows assume a single standardized input file:
  
```
circRNA.pdb
```

- Default water model: TIP3P  
- Ion naming: NA / CL  
- Physiological salt concentration: 0.15 M  

---

# 🚀 Step-by-Step Workflow

---

## 🔹 Step 1 — Generate Topology

```bash
gmx pdb2gmx -f circRNA.pdb -o circRNA_processed.gro -ignh -water tip3p
```

### What this step does

- Converts `circRNA.pdb` into GROMACS format  
- Generates topology file (`topol.top`)  
- `-ignh` ignores existing hydrogens (they are regenerated properly)  
- Applies selected RNA-compatible force field  
- Defines TIP3P water model  

This prepares the circular RNA structure for simulation.

---

## 🔹 Step 2 — Define the Simulation Box

```bash
gmx editconf -f circRNA_processed.gro -o circRNA_box.gro -c -d 1.0 -bt cubic
```

### What this step does

- Centers the circRNA in the box  
- Adds 1.0 nm distance from molecule to box edges  
- Creates a cubic simulation box  

This avoids artificial periodic boundary interactions.

---

## 🔹 Step 3 — Solvate the System

```bash
gmx solvate -cp circRNA_box.gro -cs spc216.gro -o circRNA_solv.gro -p topol.top
```

### What this step does

- Fills the box with water molecules  
- Updates topology file  

RNA molecules require explicit solvent for structural stability.

---

## 🔹 Step 4 — Create Energy Minimization Parameters

Generate minimization file automatically:

```bash
cat << EOF > minim.mdp
integrator      = steep
emtol           = 1000.0
emstep          = 0.01
nsteps          = 50000
nstlist         = 10
cutoff-scheme   = Verlet
ns-type         = grid
coulombtype     = PME
rcoulomb        = 1.0
rvdw            = 1.0
pbc             = xyz
constraints     = none
EOF
```

### What these parameters control

- Steepest descent minimization  
- Convergence tolerance  
- Maximum number of steps  
- Particle Mesh Ewald electrostatics  
- Periodic boundary conditions  
- No bond constraints (important for RNA flexibility)  

---

## 🔹 Step 5 — Add Ions (Neutralization + Physiological Salt)

### Preprocess:

```bash
gmx grompp -f minim.mdp -c circRNA_solv.gro -p topol.top -o ions.tpr -maxwarn 1
```

### Add ions:

```bash
gmx genion -s ions.tpr -o circRNA_ions.gro -p topol.top -pname NA -nname CL -neutral -conc 0.15
```

When prompted, select:

```
SOL
```

### What this step does

- Neutralizes total system charge  
- Adds Na⁺ and Cl⁻ ions  
- Sets physiological salt concentration (0.15 M)  

This mimics biological ionic conditions.

---

## 🔹 Step 6 — Run Energy Minimization

### Generate binary input:

```bash
gmx grompp -f minim.mdp -c circRNA_ions.gro -p topol.top -o em.tpr -maxwarn 1
```

### Run minimization:

```bash
gmx mdrun -v -deffnm em -ntmpi 1
```

### Output files generated

- `em.gro`  
- `em.edr`  
- `em.log`  

This performs steepest descent energy minimization.

---

## 🔹 Step 7 — Extract Minimized circRNA Structure

```bash
gmx trjconv -s em.tpr -f em.gro -o circRNA_min.pdb
```

Select:

```
RNA
```

### Final minimized structure

```
circRNA_min.pdb
```

Ready for structural analysis or molecular dynamics.

---

# 📊 Extract Potential Energy

```bash
gmx energy -f em.edr -o potential.xvg
```

Select:

```
Potential
```

Press Enter, then type:

```
0
```

---

## Optional: Convert to CSV

```bash
grep -v '^[@#]' potential.xvg | tr -s ' ' ',' > potential.csv
```

This removes headers and converts the file into a comma-separated format for plotting.

---

# 📈 Expected Energy Profile

A successful minimization should show:

- Rapid decrease in potential energy (steric clash correction)  
- Stabilization plateau  

---

# 📌 Final Output

- `circRNA_min.pdb`  
- `potential.xvg`  
- `potential.csv`  
- `em.log`  

---

# ⚠️ Important Technical Note

Current classical force fields are not inherently parameterized for **covalently closed circular RNAs**.

Manual topology adjustment (removal of terminal phosphates preventing correct connectivity recognition) was necessary to allow proper force field initialization.

This workflow represents a **computationally adapted strategy** for circRNA minimization under classical MD limitations.

---

# 👨‍🔬 Author

Luis Ernesto Castañeda Mota  
Department of Biochemistry  
Cinvestav-IPN  

---

# 📚 How to Cite This Workflow

If you use this protocol in academic work, please cite:

Castañeda-Mota, L.E.  
*circRNA Energy Minimization Workflow using GROMACS.*  
GitHub Repository (2026).

Additionally cite:

Xiao Lab – 3dRNA/DNA Server  
https://doi.org/10.1016/j.jmb.2022.167452  

---

# 🔬 Reproducibility Statement

This workflow has been simplified and standardized for reproducibility:

- Single input file: circRNA.pdb  
- Explicit force field specification  
- Fixed ionic concentration  
- Defined minimization parameters  

It is designed for computational reproducibility and educational purposes.

---

# 📜 License

MIT License
