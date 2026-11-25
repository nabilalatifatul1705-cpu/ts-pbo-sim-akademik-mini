# SIM Akademik Mini — Modul PMB, Modul KRS & Perancangan UML

Aplikasi web sistem informasi akademik mini berbasis **FastAPI** untuk mata kuliah **Pemrograman Berorientasi Objek (PBO)** yang menyatukan perancangan UML, modul **Penerimaan Mahasiswa Baru (PMB)**, dan modul **Kartu Rencana Studi (KRS)** dalam satu alur terintegrasi.

## Alur Utama Sistem
1. Calon mahasiswa mendaftar melalui endpoint PMB dengan validasi input dan status awal `pending`[cite: 5, 9, 10].
2. Admin menyetujui pendaftaran dan sistem menerbitkan **NIM** otomatis dengan format `YYKKNNNN` (2 digit tahun, 2 digit kode prodi, 4 digit nomor urut) yang aman dari *race condition* serta *idempotent*[cite: 6, 10].
3. Record `Mahasiswa` terbentuk otomatis sehingga NIM langsung dapat digunakan untuk mengisi, memvalidasi, dan mengajukan KRS ke Dosen PA[cite: 5, 10, 11].

## Tech Stack
* **Backend:** FastAPI, Uvicorn
* **Database & ORM:** SQLite, SQLAlchemy 2.0
* **Validasi:** Pydantic v2
* **Pengujian:** `pytest`, `pytest-cov`, `hypothesis`
* **Diagram & Dokumentasi:** Mermaid[cite: 2]

## Arsitektur & Desain OOP
* **Perancangan UML (`diagrams/`):** `class-diagram.mmd`, `usecase-diagram.mmd`, `sequence-pmb.mmd`, dan `sequence-krs.mmd`[cite: 2].
* **Prinsip SOLID & Design Pattern:**
  * **Inheritance & Hierarki Domain:** Abstract `User` (`CalonMahasiswa`, `Mahasiswa`, `Dosen`, `Admin`), komposisi (`KRS`–`Enrollment`, `Course`–`Schedule`), dan agregasi (`ProgramStudi`–mahasiswa)[cite: 2, 3, 8].
  * **Chain of Responsibility:** Validasi KRS berjalan berurutan (`SKSValidator` maks 24 SKS -> `DuplicateValidator` -> `PrerequisiteValidator` nilai minimal C -> `ConflictValidator` jadwal bentrok) dan berhenti pada kegagalan pertama[cite: 7, 11, 12].
  * **State Machine:** Transisi status KRS (`DRAFT`, `SUBMITTED`, `REVISION`, `APPROVED`) dikontrol terpusat pada `state.py` dengan penolakan transisi tidak sah (`HTTP 409`)[cite: 3, 8, 12].
  * **Dependency Inversion & Service Layer:** Pemisahan logika bisnis pada `KRSManager` dan abstraksi `Notifier` / `Validator`[cite: 7, 8, 11].

## Struktur Folder (Sesuaikan dengan Repo)
```text
├── app/
│   ├── main.py
│   ├── models.py
│   ├── nim.py
│   ├── routers/
│   │   ├── pmb.py
│   │   └── krs.py
│   └── krs/
│       ├── service.py
│       ├── validators.py
│       ├── state.py
│       └── notify.py
├── diagrams/
├── migrations/
│   └── 001_init.py
├── docs/
│   └── ADR-architecture-decisions.pdf
├── prompts/
│   └── ai-usage-log.md
└── requirements.txt
