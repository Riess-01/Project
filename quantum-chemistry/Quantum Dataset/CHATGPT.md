# Petunjuk ChatGPT — Quantum Data

Dokumen ini adalah konteks wajib bagi ChatGPT/Codex sebelum membaca atau
mengubah subprogram **Quantum Dataset**. Baca juga `../AGENTS.md` dan
`PROGRAM.md`, serta kontrak lintas aplikasi
`../SCIENTIFIC_PROBLEM_AND_SOLUTION.md`.

## Tujuan aplikasi

Quantum Data adalah aplikasi deskripsi, integritas, dan referensi dataset.
Quantum Data menjaga integritas 133.885 file QM9 XYZ dan menerbitkan data
molekul tervalidasi untuk HMCS dan Heuristic-ADAPT Hamiltonian Builder.
Aplikasi ini menghitung **energi referensi eksternal QM9** dari properti dataset:

```text
e_ref_qm9_electronic = U0 - ZPVE
```

Nilai tersebut berasal dari QM9 dan bukan hasil PennyLane, bukan koefisien
Hamiltonian, dan bukan energi ground-state dari Hamiltonian STO-3G/DHF yang
kita bangun sendiri.

Pembangunan, simulasi, dan perbandingan Hamiltonian bukan tanggung jawab
Quantum Data. File atau eksperimen fixed Hamiltonian lama di folder ini adalah
legacy/eksperimen dan tidak boleh dianggap sebagai entity Quantum Data.

## Input dan integritas

Input adalah file top-level `dsgdb9nsd_*.xyz`. Setiap source file harus memiliki:

```text
dataset_release
molecule_id asli dan canonical
relative source path
source SHA-256
byte size
decode/parse status
declared atom count
```

Parser lengkap harus mempertahankan, atau secara eksplisit mencatat sebagai
ignored, komponen QM9 berikut:

```text
15 properti molekul
symbols
coordinates dalam angstrom
partial charge per atom
vibrational frequencies
SMILES dan InChI dari dua sumber yang tersedia
```

Charge netral dan multiplicity singlet adalah kebijakan baseline QM9, bukan
field hasil parsing. Simpan nilainya bersama:

```text
charge = 0
multiplicity = 1
value_source = qm9_baseline_policy
```

Kemudian validasi jumlah elektron dan electron–multiplicity parity.

## Entitas keluaran

```text
RawXYZRecord
QM9PropertyRecord
QM9AncillaryRecord
MoleculeRecord
ValidationReport + ValidationIssue
CompositionProfile
GeometryProfile
ConnectivityProfile
ChemicalMotifProfile
BasisProfile
FailureRecord + QuarantineRecord
DatasetManifest + ArtifactManifest
```

Selain field mentah, profil harus menjabarkan sebanyak mungkin atribut yang
berguna bagi HMCS dan Builder, dengan metode, versi, satuan, status, dan
`value_source` yang jelas:

```text
formula dan komposisi unsur
atom/heavy-atom/electron counts
massa molekul
jarak dan penyebaran geometri
konektivitas, bond order, rings, heteroatoms, dan motif
perkiraan orbital/basis dan fitur kompleksitas yang murah
```

Atribut turunan bukan properti mentah QM9 dan tidak boleh dipresentasikan
seolah-olah dibaca langsung dari XYZ.

`MoleculeRecord` minimum yang diterima Builder:

```text
schema_version
molecule_id
source_sha256
symbols
coordinates
coordinate_unit = angstrom
charge
multiplicity
state_value_source
validation_status
```

## Alur wajib

```text
Discover files
→ raw bytes/hash audit record
→ decode
→ strict parse
→ normalize units and identity
→ apply charge/multiplicity policy
→ accumulate safe validation issues
→ build optional profiles
→ persist success and explicit failures
→ QC gates
→ publish versioned handoff
```

Target penerimaan dataset adalah:

```text
discovered = 133885
success + explicit failure = 133885
unique canonical molecule IDs = 133885
database integrity = ok
foreign key violations = 0
```

Satu record rusak tidak boleh menghilang dan tidak boleh menggagalkan audit raw
record seluruh dataset. Bedakan raw-ingestion completeness dari publishable
validated subset.

## Energi yang tidak boleh tertukar

| Nama | Asal | Makna |
|---|---|---|
| `qm9_u0` | QM9 | Internal energy at 0 K pada protokol QM9 |
| `qm9_zpve` | QM9 | Zero-point vibrational energy |
| `e_ref_qm9_electronic` | `U0-ZPVE` | Reference eksternal untuk evaluasi lintas protokol |
| `qm9_configuration_energy` | Builder | Reproduksi protokol QM9 untuk Gate A |
| `fixed_hamiltonian_energy` | Builder | Energi evaluasi Hamiltonian konfigurasi pembanding |
| `hmcs_hamiltonian_energy` | Builder | Energi evaluasi Hamiltonian konfigurasi HMCS |

Selisih terhadap `e_ref_qm9_electronic` adalah **external protocol discrepancy**,
bukan otomatis galat operator atau galat VQE.

## Batas tanggung jawab

Quantum Data tidak:

- menentukan kategori kompleksitas HMCS;
- menentukan `HamiltonianType`, `ResourceType`, `ResourceBudget`, atau
  `BuilderPolicy`;
- memilih active space atau mapping adaptif;
- membangun, memetakan, menyelesaikan, atau membandingkan Hamiltonian;
- mengarang koefisien Hamiltonian dari U0;
- menyebut surrogate sebagai Hamiltonian elektronik ab-initio;
- menganggap file tanpa failure record sebagai sukses.

## Status implementasi saat ini

- Inventory dan manifest 133.885 telah tersedia.
- Manifest produksi belum sama dengan entity database lengkap.
- `qm9_reference_hamiltonians.sqlite` adalah eksperimen lama, bukan output
  Quantum Data dan bukan artifact selesai.

## Dokumentasi acuan

- PennyLane, Building molecular Hamiltonians:
  https://pennylane.ai/demos/tutorial_quantum_chemistry
- PennyLane, Differentiable Hartree–Fock:
  https://pennylane.ai/demos/tutorial_differentiable_HF
- Dokumentasi internal: `FLOWCHART.md`, `IMPLEMENTATION_BACKLOG.md`, dan
