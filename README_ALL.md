# SIESTA DFT TOOLKIT: Fe₃O₄, ZIF-67, Cu₂O, composite, tetracycline

## STATUS
A reproducible **starter toolkit and model-builder**, **NOT** a universally validated ready-to-run production study. No pseudopotentials, experimental structures or actual energies are included. The magnetic order, adsorption protonation, surface termination, interface and chosen band paths **cannot be deduced** from chemical formulas alone. There is **no automatic chemically validated Fe₃O₄@ZIF-67@Cu₂O geometry**.

## What is included
- `scripts/make_structure.py`: structural FDF from ASE-readable CIF/POSCAR/XYZ.
- `scripts/inspect_structure.py`: formula, cell, nearest-distance check.
- `scripts/make_stage.py`: stage template plus structural blocks.
- `scripts/make_mesh_series.py`: MeshCutoff single-point series.
- `scripts/make_kgrid_series.py`: k-grid single-point series.
- `scripts/make_slab.py`: raw crystalline slab candidates.
- `scripts/make_interface_trial.py`: **combine two already matched/identical lateral cells** into trial interface. Refuses mismatched cells. Does not optimize chemical registry.
- `scripts/place_tc_trial.py`: place TC geometry above slab. Requires independently prepared TC geometry and manual selection of several conformers/sites/orientations.
- `scripts/make_spin_template.py`: **review-only placeholders** for Fe/Co magnetic moments.
- `scripts/collect_results.py`: collect useful output fragments, not a definitive convergence test.
- `templates/`: 6 calculation-stage templates, originally for Cu₂O. Must adjust spin, k-grid, XC, DFT+U and dispersion for other materials.
- `templates_materials/`: clearly marked material-specific REVIEW templates.
- `examples/submit.slurm`: adapt to actual HPC cluster.

## Full scientific workflow
| Model | Use | Selection/validation |
|---|---|---|
| Bulk Cu₂O | lattice, DOS, PDOS, band gap | Cuprite, Cu:O=2:1, 6-atom primitive often sufficient; convergence |
| Bulk Fe₃O₄ | magnetic/electronic benchmark | inverse spinel; tetrahedral and octahedral Fe opposite initial spin; try multiple magnetic/charge orders; DFT+U validation |
| Bulk ZIF-67 | framework geometry and electronic states | correct Co/2-methylimidazolate composition; check H, disorder, solvent; spin states and vdW |
| Bulk supercell | defects, substitution, nontrivial spin patterns | use only if physics requires |
| Slab Fe₃O₄ (001)/(111) | external surface/adsorption | investigate reconstruction, oxygen chemical potential and termination |
| Slab Cu₂O (111)/(100) | external surface/adsorption | check Cu/O termination, polarity, slab thickness, lateral separation, vacuum |
| ZIF-67 terminated surface/cluster | adsorption at accessible Co/ligand sites | physically terminate broken linkers, expose plausible sites; TC cannot automatically access internal pore |
| Fe₃O₄/ZIF-67 and ZIF-67/Cu₂O | contact energetics and charge distribution | match cells without excessive strain or construct justified cluster |
| Three-component trial | local interface hypothesis | only after morphology and two-component interfaces established |
| TC isolated/adsorbed | conformers, protonation, binding, PDOS | aqueous pH/solvation, multiple poses, BSSE, reference geometries |
| Reaction intermediate | mechanistic degradation | experimental pathways, ROS and reaction barriers; SCF/PDOS alone do not prove photocatalysis |

## First commands (after `conda activate <your-environment>` and `pip install ase numpy` if needed)
```bash
python scripts/inspect_structure.py Cu2O.cif
python scripts/inspect_structure.py Fe3O4.cif
python scripts/inspect_structure.py ZIF67.cif
python scripts/make_structure.py Cu2O.cif -o Cu2O_structure.fdf
python scripts/make_stage.py --structure Cu2O_structure.fdf --template templates/01_single_point.fdf -o Cu2O_singlepoint.fdf --label Cu2O_SP
python scripts/make_mesh_series.py Cu2O_singlepoint.fdf
python scripts/make_kgrid_series.py Cu2O_singlepoint.fdf
```
Proceed with geometry optimization only after numerical convergence. Rebuild all subsequent FDF structural blocks from **optimized geometry** rather than starting CIF. Maintain pseudopotential XC compatibility.

## Creating trial slabs
```bash
python scripts/make_slab.py Cu2O_conventional.cif --hkl 1 1 1 --layers 6 --vacuum 18 --repeat 2 2 -o Cu2O_111_trial.vasp
```
Do not assume it picked the correct stable Cu-rich or O-rich termination. Inspect and compare surface energies/convergences. For ZIF-67, a generic surface cut can create chemically invalid unsaturated linkers and must not be used without repair.

## Combine two PREMATCHED trial slabs
```bash
python scripts/make_interface_trial.py Fe3O4_matched.vasp ZIF67_matched.vasp --gap 2.8 --vacuum 18 -o Fe3O4_ZIF67_trial.xyz
```
Both slab files must already share identical lateral cell vectors, determined by separate lattice-matching and justified strain. Assemble Fe₃O₄/ZIF-67 and ZIF-67/Cu₂O first. Adding all three is an optional *model hypothesis*, not an automatic representation of a synthesized core-shell composite.

## Put TC near a surface
```bash
python scripts/place_tc_trial.py Cu2O_111_trial.vasp TC_optimized.xyz --height 3.0 --xy 0.5 0.5 -o TC_Cu2O_trial.xyz
python scripts/make_structure.py TC_Cu2O_trial.xyz -o TC_Cu2O_structure.fdf
```
The output is a *trial pose*. Sample multiple adsorption sites, molecular orientations, protonation states and consider solution effects. Check atom overlaps and lateral periodic image effects. The extended XYZ ASE writer stores periodic cell; verify when reading.

## Computing adsorption energy
E_ads = E(surface+TC) - E(surface) - E(TC), using compatible XC, basis, k sampling, charge/spin and fragment reference conditions. Evaluate counterpoise BSSE with localized basis. Gas-phase adsorption energy is not an aqueous adsorption free energy.

## Slurm and DOS/bands
Copy `examples/submit.slurm` into the selected job directory, rename a verified input `run.fdf`, then use `sbatch submit.slurm` **only** after adapting modules and cluster policy. Final SCF → DOS/PDOS (denser k mesh) and band-structure (validated reciprocal k path). Bands in `templates/06_bands.fdf` are specific to a simple cubic axis representation and should not be reused uncritically. SIESTA pseudopotentials for Fe, Co, Cu, O, N, C, H must be obtained separately and tested.

## Documentation
- https://docs.siesta-project.org/projects/siesta/en/stable/
- https://ase-lib.org/ase/build/surface.html
- https://ase-lib.org/ase/io/io.html
