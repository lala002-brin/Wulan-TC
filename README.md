# Toolkit workflow DFT SIESTA: Cu₂O → permukaan → interface → TC

## Status
Template ini adalah **starter kit**, bukan input produksi tervalidasi. Script Python menulis struktur dari CIF/POSCAR/XYZ dan template FDF. Pseudopotential **tidak** dibuat atau disertakan. FDF untuk Fe₃O₄ dan ZIF-67 memerlukan pengaturan magnetik, functional, basis dan dispersi khusus yang belum ditentukan. Tidak ada hasil DFT yang diklaim.

## Struktur paket
- `scripts/make_structure.py`: konversi struktur ke blok FDF.
- `scripts/make_stage.py`: gabung struktur dan template.
- `scripts/make_mesh_series.py`: seri input MeshCutoff.
- `scripts/make_slab.py`: membuat **kandidat awal** slab dari conventional cell.
- `scripts/collect_results.py`: ringkas baris output, bukan validator final otomatis.
- `templates/*.fdf`: single-point, optimasi fixed/variable cell, final SCF, DOS/PDOS, bands.
- `examples/submit.slurm`: contoh HPC yang wajib disesuaikan.

## Alur lengkap Cu₂O
1. Validasi `Cu2O.cif` dengan ASE: jumlah atom, stoikiometri Cu:O=2:1, sel, koordinasi, occupancy, struktur cuprite.
2. Siapkan `Cu.psml` dan `O.psml`, pseudopotential kompatibel PBE, verifikasi valensi dan nama spesies.
3. Generate struktur:
   ```bash
   python scripts/make_structure.py Cu2O.cif -o structure.fdf
   ```
4. Generate single-point untuk uji MeshCutoff:
   ```bash
   python scripts/make_stage.py --structure structure.fdf --template templates/01_single_point.fdf -o singlepoint.fdf --label Cu2O_TEST
   python scripts/make_mesh_series.py singlepoint.fdf
   ```
   **Perhatian**: output folder `mesh_tests/mesh_*` masing-masing memerlukan pseudopotential Cu dan O. Gunakan satu geometri dan satu set parameter yang sama. Jalankan setiap `run.fdf` sebagai job terpisah. Catat energi total, gaya, dan waktu; ulangi dengan k-grid 4,6,8,10,12 dan uji basis DZP vs lebih besar pada parameter yang sudah stabil. `make_mesh_series.py` hanya otomatis membuat seri MeshCutoff.
5. Setelah parameter numerik konvergen, generate optimasi:
   ```bash
   python scripts/make_stage.py --structure structure.fdf --template templates/02_opt_fixed.fdf -o opt.fdf --label Cu2O_OPT
   ```
   Bila ingin optimasi kisi, gunakan `03_opt_variable.fdf`. Perbarui MeshCutoff, basis dan k-grid hasil konvergensi di template. Periksa hasil SCF, gaya < 0.02 eV/Å, stress untuk variable cell, dan struktur akhir. Jangan menyatakan konvergen jika berhenti karena batas langkah.
6. Ekspor **struktur akhir** ke `optimized.vasp` atau CIF setelah mengecek file `.XV`/`.STRUCT_OUT` dan membacanya dengan alat yang kompatibel. Contoh apabila `Cu2O_OPT.XV` tersedia:
   ```bash
   python -c "from ase.io import read,write; a=read('Cu2O_OPT.XV',format='siesta-xv');write('optimized.vasp',a,format='vasp',direct=True)"
   python scripts/make_structure.py optimized.vasp -o optimized_structure.fdf
   ```
   Catatan: file akhir serta dukungan reader bergantung versi/konfigurasi.
7. Gunakan `optimized_structure.fdf` dengan `04_final_scf.fdf`, `05_dos_pdos.fdf`, dan `06_bands.fdf`. `BandLines` contoh **hanya** cocok bagi sel kubik konvensional dengan koordinat k terkait; jika memakai primitive non-ortogonal, tentukan lintasan baru dengan SeeK-path.
8. Hitung gap dari maksimum valence eigenvalues dan minimum conduction eigenvalues pada sampling k yang cukup, **bukan semata DOS broadened**. Identifikasi direct vs indirect berdasarkan posisi k. PBE biasanya meremehkan gap quasipartikel.

## Contoh perintah tiap tahap
```bash
python scripts/make_stage.py --structure optimized_structure.fdf --template templates/04_final_scf.fdf -o final_scf.fdf --label Cu2O_SCF
python scripts/make_stage.py --structure optimized_structure.fdf --template templates/05_dos_pdos.fdf -o dos.fdf --label Cu2O_DOS
python scripts/make_stage.py --structure optimized_structure.fdf --template templates/06_bands.fdf -o bands.fdf --label Cu2O_BANDS
```
Jalankan sebagai job independen dalam direktori terpisah bersama pseudopotential. `siesta < input.fdf > output.out` hanya untuk node yang diizinkan. Di HPC gunakan `sbatch submit.slurm` setelah menyesuaikan `run.fdf`, modules, partition, account, srun dan resources.

## Kapan memakai bulk, supercell, slab, interface, cluster?
| Model | Kapan dipakai | Kapan tidak cukup |
|---|---|---|
| Bulk primitive/conventional | band gap, band structure, PDOS, lattice, magnetisme | Adsorpsi TC di permukaan luar |
| Bulk supercell | defect, substitusi, magnetic order, penyimpangan lokal | Tidak otomatis meningkatkan gap bulk; mahal |
| Slab surface (hkl) | adsorpsi TC, work function, terminasi permukaan | Tidak mewakili keseluruhan core-shell berpori |
| Supercell lateral slab | TC besar, menghindari interaksi gambar periodik | Tidak diperlukan jika sel awal sudah cukup luas |
| Interface periodik | binding, charge rearrangement, alignment antar komponen | Jika mismatch besar atau kontak amorf/inkohoren |
| Cluster terpotong | skrining interaksi lokal situs Co/Cu/Fe dengan TC | Band structure dan energi permukaan periodik |

## Rencana Fe₃O₄/ZIF-67/Cu₂O/TC
1. Validasi masing-masing bulk: Fe₃O₄ inverse spinel dengan ferrimagnetisme dan uji +U, ZIF-67 framework dengan spin Co serta koreksi dispersi, Cu₂O cuprite.
2. Buat kandidat permukaan berdasarkan morfologi dan terminasi: Fe₃O₄(001)/(111), Cu₂O(111)/(100), dan ZIF-67 sesuai orientasi faset yang benar-benar masuk akal, termasuk dangling bonds/defect/passivation.
3. Periksa konvergensi ketebalan slab, vakum sekitar 15–20 Å sebagai awal, dan ukuran lateral. Hindari mengubah sel vakum selama relaksasi permukaan.
4. Cari antarmuka dengan mismatch rendah. Jika MOF sulit membentuk antarmuka komensurat, gunakan klaster realistis atau model lokal. Hindari strain besar yang tak wajar.
5. Optimasi konformer dan protonasi TC sesuai pH. Skrining karbonil, hidroksil, Cu/Fe/Co terbuka, ikatan hidrogen; model berpelarut untuk interpretasi air.
6. Energi adsorpsi Eads = E(surface+TC) - E(surface) - E(TC), sel/basis/functional konsisten. Evaluasi BSSE counterpoise dan efek solvasi/protonasi.
7. Charge density difference, PDOS, binding, work function, dan jika meneliti degradasi, reaksi perantara/NEB yang berlandaskan eksperimen. Ground-state DFT saja tidak membuktikan S-scheme atau mekanisme fotokatalisis.

## Membuat kandidat slab
```bash
python scripts/make_slab.py Cu2O_conventional.cif --hkl 1 1 1 --layers 6 --vacuum 18 --repeat 2 2 -o Cu2O_111_trial.vasp
```
Input **harus conventional bulk cell yang benar**. Visualisasikan dan evaluasi terminasi, slab polar, serta kemungkinan rekonstruksi sebelum penggunaan fisik. Jangan menganggap keluaran otomatis sebagai permukaan stabil. Untuk adsorpsi gunakan slab dan TC yang telah ditempatkan dengan orientasi fisik yang masuk akal dan optimasi ulang.

## Pemeriksaan output
```bash
python scripts/collect_results.py Cu2O_OPT.out Cu2O_SCF.out
```
Jangan hanya grep satu string. Periksa terminasi sukses, gaya, stress, SCF tiap langkah, dan struktur akhir. Untuk bulk semikonduktor, verifikasi okupansi pita dan k-point sebelum mengekstrak gap.

## Dokumentasi
- https://docs.siesta-project.org/projects/siesta/en/stable/reference/index.html
- https://docs.siesta-project.org/projects/siesta/en/latest/reference/fdf-file.html
- https://ase-lib.org/ase/build/surface.html
- https://docs.siesta-project.org/projects/siesta/en/stable/
