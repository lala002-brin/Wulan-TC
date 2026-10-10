# Panduan Teknis DFT untuk Komposit Fe₃O₄/ZIF-67/Cu₂O pada Penghilangan Tetrasiklin

**Fokus:** pemilihan struktur, optimasi, sifat elektronik, antarmuka, adsorpsi tetrasiklin (TC), serta kajian awal jalur degradasi.  
**Perangkat utama:** SIESTA, ASE, spglib, perangkat visualisasi struktur, dan alat pascapemrosesan DOS serta densitas muatan.  
**Status:** rancangan metodologi. Semua ukuran model dan parameter numerik merupakan titik awal yang harus melalui uji konvergensi.

## 1. Pertanyaan penelitian dan luaran DFT

| Pertanyaan | Perhitungan | Luaran |
|---|---|---|
| Bagaimana sifat elektronik komponen? | Optimasi bulk, DOS/PDOS, band structure | Struktur, momen magnetik, keadaan orbital, gap jika terdefinisi |
| Bagaimana interaksi Fe₃O₄, ZIF-67, dan Cu₂O? | Model antarmuka, energi ikat, perbedaan densitas muatan | Stabilitas dan redistribusi elektron |
| Di mana TC beradsorpsi? | Optimasi TC pada situs aktif | Energi adsorpsi, geometri, transfer muatan |
| Bagaimana kemungkinan jalur degradasi TC? | Energi reaksi dan lintasan keadaan transisi terpilih | Energi aktivasi dan perbandingan jalur reaksi |

**Batas interpretasi:** DOS, band gap, dan perbedaan densitas muatan pada keadaan dasar tidak membuktikan mekanisme transfer pembawa muatan saat iluminasi. Analisis tersebut memerlukan bukti eksperimen dan, bila relevan, metode untuk keadaan tereksitasi.

## 2. Arsitektur model: bulk → surface → interface → TC

1. **Bulk:** model periodik tiga dimensi untuk memvalidasi parameter kisi, magnetisme, dan sifat elektronik material.
2. **Surface/slab:** model periodik dua dimensi dengan vakum pada arah normal untuk menguji stabilitas permukaan dan situs adsorpsi.
3. **Interface:** dua material berkontak untuk menganalisis ikatan dan redistribusi muatan.
4. **Adsorpsi TC:** satu atau beberapa konformer dan keadaan protonasi TC pada situs permukaan relevan.
5. **Reaksi:** model reaktan, produk antara, dan keadaan transisi berdasarkan jalur yang memiliki dukungan eksperimen.

Jangan menggunakan supercell besar untuk semua tahapan tanpa alasan fisik. Pilih ukuran minimum yang menghasilkan besaran target konvergen.

## 3. Pemilihan struktur bulk

### 3.1 Fe₃O₄ (magnetit)

- Gunakan struktur *inverse spinel* sebagai titik awal; sel konvensional ideal berisi 56 atom.
- Identifikasi Fe pada subkisi tetrahedral dan oktahedral. Siapkan inisialisasi ferrimagnetik dengan orientasi momen berlawanan antara kedua subkisi.
- Uji beberapa konfigurasi spin, kemungkinan keteraturan muatan, dan distorsi yang relevan. Keadaan Fe₃O₄ pada temperatur rendah tidak selalu terwakili oleh struktur kubik ideal.
- Mulai dengan spin-polarized PBE. Evaluasi DFT+U menggunakan parameter yang tervalidasi untuk jenis proyektor, pseudopotential, dan keadaan oksidasi.
- Luaran: parameter kisi, energi relatif konfigurasi magnetik, momen lokal, DOS/PDOS Fe 3d dan O 2p.

### 3.2 ZIF-67

- Verifikasi bahwa CIF merepresentasikan kerangka Co(2-metilimidazolat)₂ yang tepat. Periksa pelarut, hidrogen, okupansi parsial, dan disorder.
- Gunakan primitive cell yang valid untuk perhitungan bulk. Conventional cell diperlukan bila konfigurasi spin atau disorder tidak kompatibel dengan sel primitive.
- Koreksi dispersi penting untuk mengevaluasi interaksi nonkovalen dan fleksibilitas framework. Verifikasi opsi dispersi pada versi SIESTA yang dipakai.
- Uji pengaturan spin Co dan, jika diperlukan, sensitivitas terhadap parameter U.
- Hindari menganggap rongga internal otomatis dapat diakses TC: ukuran jendela pori, cacat, fleksibilitas, dan akses pelarut perlu dinilai.

### 3.3 Cu₂O

- Mulai dari struktur *cuprite* (Pn-3m), primitive cell berisi 6 atom.
- Optimasi kisi dan hitung DOS/PDOS Cu 3d dan O 2p serta band structure.
- PBE dapat meremehkan gap; bedakan gap fundamental dengan transisi optik. Jangan menetapkan U hanya untuk memaksa kesesuaian dengan gap eksperimen.

| Material | Model bulk awal | Isu utama | Analisis prioritas |
|---|---|---|---|
| Fe₃O₄ | Sel yang mendukung ferrimagnetisme, misalnya 56 atom | Magnetisme dan korelasi Fe 3d | Spin, DOS/PDOS, validasi +U |
| ZIF-67 | Primitive cell tervalidasi | Sel besar, Co, dispersi, pori | Struktur, spin, DOS/PDOS |
| Cu₂O | Primitive cell 6 atom | Perkiraan band gap | Struktur pita, DOS/PDOS |

## 4. Kapan memakai unit cell, supercell, slab, atau klaster?

| Model | Cocok untuk | Tidak cukup untuk |
|---|---|---|
| Unit cell bulk | Struktur dan sifat elektronik intrinsik | Adsorpsi pada permukaan luar |
| Supercell bulk | Defek, dopan, pola spin, gangguan lokal | Permukaan jika tanpa vakum |
| Slab | Energi permukaan, work function, adsorpsi | Seluruh morfologi nanopartikel kompleks |
| Interface periodik | Ikatan dua fase, transfer muatan antarmuka | Antarmuka inkoheren yang dipaksakan menjadi koheren |
| Klaster lokal | Skrining kimia situs aktif pada biaya terbatas | Band structure kristal periodik |

**Aturan keputusan:** hitung bulk dengan unit cell yang cukup; bangun slab hanya untuk persoalan permukaan; perluas sel lateral slab jika salinan TC saling berinteraksi; gunakan klaster lokal bila lattice matching interface periodik menjadi terlalu mahal atau tidak realistis.

## 5. Konstruksi dan validasi surface

### 5.1 Kandidat orientasi awal

| Material | Kandidat orientasi | Kewaspadaan |
|---|---|---|
| Fe₃O₄ | (001), (111) | Terminasi, polaritas, rekonstruksi, magnetisme |
| ZIF-67 | Orientasi yang konsisten dengan habit kristal teramati | Co–N terputus, gugus penutup, rekonstruksi permukaan |
| Cu₂O | (111), (100), (110) | Permukaan kaya Cu/O, polaritas dan muatan permukaan |

Jangan menyamakan orientasi dengan terminasi. Untuk setiap orientasi, identifikasi beberapa terminasi kimia yang masuk akal sebelum memilih konfigurasi final.

### 5.2 Ukuran awal dan konvergensi

- Ketebalan: uji sekitar **4–8 lapisan struktural** sebagai titik awal, lalu tambah hingga sifat target stabil. Definisi satu lapisan mengikuti unit struktural permukaan.
- Vakum: mulai **15–20 Å**, kemudian uji penambahan vakum. Untuk slab asimetris periksa kebutuhan koreksi dipol.
- Sel lateral: bandingkan 1×1, 2×2, atau ukuran yang lebih besar. Untuk TC gunakan ukuran yang mengurangi interaksi antar-citra periodik.
- Pada slab yang merepresentasikan permukaan kristal, fiksasi bagian bawah dapat membantu. Dokumentasikan atom yang difiksasi.
- Jangan mengoptimasi vektor arah vakum seperti kisi bulk.

Untuk slab simetris, stoikiometrik, dan dua muka identik:

$$
\gamma=\frac{E_{\mathrm{slab}}-nE_{\mathrm{bulk}}}{2A}
$$

Untuk terminasi tidak stoikiometrik, energi permukaan memerlukan potensial kimia:

$$
\gamma=\frac{G_{\mathrm{slab}}-\sum_i N_i\mu_i}{2A}
$$

Pembagi $2A$ hanya tepat jika kedua muka setara. Jelaskan lingkungan kimia saat membandingkan terminasi.

## 6. Konstruksi interface komposit

### 6.1 Model yang disarankan

1. **Fe₃O₄/ZIF-67:** identifikasi kontak oksida besi dengan situs ligan atau pusat logam pada tepi framework.
2. **ZIF-67/Cu₂O:** uji beberapa orientasi Cu₂O terhadap gugus permukaan ZIF-67.
3. **Fe₃O₄/ZIF-67/Cu₂O:** bangun hanya setelah struktur dua-interface terkontrol dan bukti morfologi eksperimen mendukung geometri model.

Jika material berupa *core-shell* melengkung, slab planar hanya pendekatan lokal. Jika Cu₂O tersebar sebagai nanopartikel pada ZIF-67, model klaster Cu₂O berjangkar dapat lebih realistis dibanding trilayer planar.

### 6.2 Lattice matching

- Bentuk supercell lateral menggunakan matriks transformasi bilangan bulat.
- Catat regangan tiap material pada dua arah dan sudut sel. Cari kandidat regangan rendah, misalnya sekitar **3–5%** untuk skrining, tanpa menjadikannya batas universal.
- Jika atom menjadi terlalu banyak atau strain berlebihan, pakai model lokal/inkoheren dengan keterbatasan yang dinyatakan jelas.
- Uji beberapa jarak awal dan registri antarmuka, kemudian optimasi posisi atom.

Energi ikat antarmuka dalam model referensi yang konsisten:

$$
E_{\mathrm{bind}}=E_{AB}-E_A-E_B
$$

Untuk antarmuka berbeda luas, laporkan juga energi per luas. Pisahkan kontribusi strain jika komponen dipaksa menyesuaikan kisi.

Perbedaan densitas muatan:

$$
\Delta\rho(\mathbf r)=\rho_{AB}(\mathbf r)-\rho_A(\mathbf r)-\rho_B(\mathbf r)
$$

Hitung densitas masing-masing fragmen pada **geometri beku dan grid serta sel yang sama** dengan gabungan. Analisis akumulasi/deplesi muatan bersama PDOS dan data populasi muatan.

## 7. Menyiapkan tetrasiklin

- Buat beberapa **konformer TC** yang berbeda, lalu optimasi molekul terisolasi.
- Tentukan **keadaan protonasi** dari pH eksperimen. Bentuk kationik, zwitterionik, dan anionik dapat memberi hasil adsorpsi berbeda.
- Jika menghitung ion dalam sel periodik, kendalikan artefak akibat muatan total, latar kompensasi, dan ukuran sel.
- Perlakukan efek pelarut secara eksplisit atau implisit bila tujuan riset adalah adsorpsi dari air. Energi total vakum **bukan** energi bebas adsorpsi dalam air.

### 7.1 Situs adsorpsi kandidat

| Permukaan | Situs dan geometri awal |
|---|---|
| Fe₃O₄ | Gugus O karbonil/hidroksil TC mengarah ke Fe terbuka; orientasi datar dan miring |
| ZIF-67 | Co tidak jenuh atau situs cacat yang dibenarkan; ikatan H dengan ligan |
| Cu₂O | Gugus O TC mengarah ke Cu permukaan; variasi orientasi molekul |
| Komposit | Titik pertemuan dua fase dan situs aktif paling mungkin diakses larutan |

Skrining awal yang layak: **minimal empat geometri berbeda per jenis situs/permukaan**, kemudian tambah sesuai keragaman struktur dan energi. Hindari menyatakan konformer pertama yang konvergen sebagai global minimum.

Energi adsorpsi:

$$
E_{\mathrm{ads}}=E_{\mathrm{surface+TC}}-E_{\mathrm{surface}}-E_{\mathrm{TC}}
$$

Energi negatif berarti adsorpsi menguntungkan relatif terhadap referensi tersebut. Untuk basis orbital lokal SIESTA, **uji basis set superposition error (BSSE)** melalui perhitungan counterpoise atau ghost basis. Laporkan geometri ikatan, PDOS, serta redistribusi muatan. Bandingkan adsorpsi TC dengan kemungkinan kompetisi air jika relevan.

## 8. Kajian degradasi: tidak cukup hanya band gap

- Uji adsorpsi TC terlebih dahulu, kemudian tentukan apakah tujuan eksperimen adalah adsorpsi, fotokatalisis, oksidasi berbantuan PMS/PDS, atau mekanisme gabungan.
- Pilih kandidat spesies reaktif sesuai eksperimen, misalnya •OH atau O₂•⁻. Jangan mengasumsikan keberadaannya tanpa pengujian.
- Jika tersedia, gunakan hasil LC-MS untuk membatasi daftar produk antara dan jalur elementer.
- Optimasi reaktan serta produk, kemudian gunakan NEB atau pencarian transition state yang cocok pada lintasan yang jelas.

$$
E_a=E_{\mathrm{TS}}-E_{\mathrm{reactant}}
$$

Bandingkan juga energi bebas jika ingin mendiskusikan kelayakan reaksi pada temperatur tertentu. Jalur fotokimia yang melibatkan keadaan tereksitasi memerlukan pendekatan tambahan di luar ground-state DFT standar.

## 9. Strategi metode elektronik

### 9.1 Functional dan koreksi

| Tahap | Metode awal | Validasi lanjutan |
|---|---|---|
| Struktur dan skrining | Spin-polarized PBE | Basis, k-point, mesh, spin |
| Fe₃O₄ | PBE+U kandidat | U dan pola spin melalui literatur/sifat eksperimen |
| ZIF-67 | PBE + dispersi | Spin Co, sensitivitas U dan metode dispersi |
| Cu₂O | PBE baseline | Perbandingan pendekatan yang lebih tinggi untuk gap |
| Adsorpsi/interface | Metode dispersi konsisten | BSSE, ukuran sel, vakum, protonasi, pelarut |

Jangan menetapkan nilai U universal untuk Fe, Co, dan Cu. SIESTA dan pseudopotential tertentu memerlukan pengujian orbital/proyektor yang tepat. Pastikan opsi DFT+U dan dispersi sesuai versi perangkat lunak.

### 9.2 Parameter SIESTA untuk skrining awal

| Parameter | Nilai awal yang diuji | Catatan |
|---|---|---|
| `XC.functional` / `XC.authors` | GGA / PBE | Baseline |
| Basis | DZP | Bandingkan dengan basis lebih besar pada model kecil |
| `MeshCutoff` | 250–300 Ry | Uji 300, 400, 500 Ry atau lebih |
| `DM.Tolerance` | 1e-4 | Kencangkan untuk energi final |
| Optimasi gaya | 0,04 eV/Å | Final sekitar 0,02 eV/Å atau lebih ketat bila diperlukan |
| Spin | Terpolarisasi untuk Fe/Co | Inisialisasi momen per situs |
| k-point | Bergantung ukuran dan dimensi sel | Bulk 3D; slab 2D; klaster Gamma |
| Smearing | Sesuaikan logam/semikonduktor | Uji dampak pada energi dan okupansi |

**Contoh kerangka parameter**, belum merupakan file FDF lengkap dan belum mencakup struktur, spesies, pseudopotential, konfigurasi spin spesifik, DFT+U, dispersi, maupun k-point:

```fdf
SystemName Fe3O4_ZIF67_Cu2O
SystemLabel composite_opt

XC.functional GGA
XC.authors PBE
Spin polarized

PAO.BasisSize DZP
MeshCutoff 300 Ry
SolutionMethod diagon

MaxSCFIterations 300
DM.Tolerance 1.d-4
DM.MixingWeight 0.05
DM.NumberPulay 5

MD.TypeOfRun CG
MD.NumCGsteps 300
MD.MaxForceTol 0.04 eV/Ang

WriteForces true
DM.UseSaveDM false
MD.UseSaveXV false
```

### 9.3 Uji konvergensi wajib

1. **MeshCutoff:** periksa perubahan energi relatif, gaya, dan sifat elektronik.
2. **Basis:** DZP dibanding basis yang lebih besar, terutama energi adsorpsi dan momen magnetik.
3. **k-point:** densitas grid hingga energi dan DOS stabil.
4. **Slab:** ketebalan, vakum, dan ukuran lateral.
5. **Spin/U:** uji keadaan magnetik metastabil dan sensitivitas U.
6. **Adsorpsi:** orientasi, protonasi, cakupan adsorbat, BSSE, dan pelarut.

Kriteria kerja awal dapat berupa perubahan energi adsorpsi kurang dari **0,03–0,05 eV** saat parameter numerik ditingkatkan, jika resolusi tersebut memadai untuk membedakan hipotesis. Jangan memakai satu toleransi universal tanpa mempertimbangkan selisih energi kandidat.

## 10. Analisis band alignment dan work function

Untuk slab dengan referensi vakum:

$$
\Phi=E_{\mathrm{vac}}-E_F
$$

Band alignment memerlukan energi acuan yang konsisten, keadaan permukaan yang relevan, serta evaluasi potential lineup pada interface. Perhatikan *surface states*, Fermi-level pinning, dan sifat magnetik Fe₃O₄. PDOS dan muatan keadaan dasar **tidak sendiri** membuktikan mekanisme type-II, Z-scheme, atau S-scheme di bawah cahaya.

## 11. Workflow kerja yang dapat direproduksi

```text
DFT_Fe3O4_ZIF67_Cu2O_TC/
├── 01_bulk/
│   ├── Fe3O4/{convergence,optimization,electronic}/
│   ├── ZIF67/{convergence,optimization,electronic}/
│   └── Cu2O/{convergence,optimization,electronic}/
├── 02_surface/
│   ├── Fe3O4_001/
│   ├── Fe3O4_111/
│   ├── ZIF67_surface/
│   ├── Cu2O_111/
│   └── Cu2O_100/
├── 03_interface/
│   ├── Fe3O4_ZIF67/
│   ├── ZIF67_Cu2O/
│   └── triple_composite/
├── 04_TC/{conformers,protonation_states}/
├── 05_adsorption/{TC_Fe3O4,TC_ZIF67,TC_Cu2O,TC_composite}/
├── 06_reaction/{intermediates,transition_states}/
└── 07_analysis/{DOS_PDOS,band_alignment,charge_density,adsorption_energy,figures}/
```

Simpan versi SIESTA, perintah eksekusi HPC, semua pseudopotential, input FDF, struktur sebelum dan setelah optimasi, parameter U, konfigurasi spin, data energi, serta riwayat uji konvergensi.

## 12. Checklist sebelum publikasi

- [ ] Identitas fase dan komposisi setiap material terverifikasi.
- [ ] Sel bulk dan magnetisme telah divalidasi.
- [ ] Pseudopotential dan basis telah diuji.
- [ ] Parameter mesh serta k-point telah konvergen.
- [ ] Terminasi slab, rekonstruksi, dan kondisi kimia dijelaskan.
- [ ] Model antarmuka sesuai bukti morfologi atau dinyatakan sebagai aproksimasi lokal.
- [ ] Ukuran sel adsorpsi tidak menyebabkan interaksi periodik yang mengganggu.
- [ ] Konformer/protonasi TC sesuai pH yang dipelajari.
- [ ] Dispersi, BSSE, dan pelarut ditangani secara proporsional.
- [ ] Energi adsorpsi menggunakan referensi yang konsisten.
- [ ] Kesimpulan mekanisme fotokatalitik tidak melampaui kemampuan ground-state DFT.
- [ ] Struktur, file input, data energi, dan workflow tersedia untuk reproduksi.

## 13. Sumber rujukan metodologis untuk ditelusuri

1. **SIESTA Documentation.** Manual dan tutorial resmi: https://docs.siesta-project.org/projects/siesta/en/stable/
2. **ASE Documentation.** Pembacaan struktur, supercell, dan slab: https://ase-lib.org/
3. **spglib.** Standardisasi dan deteksi simetri kristal: https://spglib.readthedocs.io/
4. **Materials Project.** Data struktur dan analisis material: https://next-gen.materialsproject.org/
5. **SIESTA publications.** Gunakan artikel metode SIESTA dan dokumentasi versi yang dipakai untuk melaporkan pengaturan implementasi.

> **Kesimpulan:** Gunakan bulk cell untuk validasi intrinsik; slab untuk situs permukaan; interface dua komponen untuk memahami kontak; dan TC pada situs yang terakses pelarut untuk energi adsorpsi. Bangun model tiga komponen setelah bukti struktur dan hasil perhitungan sederhana mendukungnya. Studi degradasi memerlukan hipotesis reaksi dan pengujian tambahan di luar band gap serta DOS.
