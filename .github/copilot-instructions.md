# Copilot Instructions — wikijs-copier

## Konteks Project

Project ini adalah **Copier template** untuk mendeploy Wiki.js 2 dengan PostgreSQL, diekspos via Traefik sebagai reverse proxy. Template di-generate ke direktori target dengan `copier copy`.

## Struktur Repository

```
wikijs-copier/
├── copier.yml                      # Definisi variabel & prompt interaktif
├── README.md
├── .github/
│   └── workflows/
│       └── test.yml                # CI: render + validasi template
└── template/                       # _subdirectory Copier
    ├── docker-compose.yml.jinja
    ├── .env.jinja
    └── .gitignore
```

## Aturan Copier

- `_subdirectory: template` — semua file template ada di dalam folder `template/`
- `_min_copier_version: "9.0"`
- File berekstensi `.jinja` akan di-render Jinja2 dan ekstensinya dihapus di output
- File tanpa ekstensi `.jinja` disalin apa adanya (static)

## Variabel Template

| Variabel | Type | Secret | Default |
|---|---|---|---|
| `project_name` | str | — | `wikijs` |
| `traefik_network` | str | — | `inverseproxy_shared` |
| `domain` | str | — | — |
| `traefik_entrypoint` | str (choices) | — | `https` |
| `traefik_certresolver` | str | — | `letsencrypt` |
| `db_name` | str | — | `wikijs` |
| `db_user` | str | — | `wikijs` |
| `db_pass` | str | **ya** | — |
| `tz` | str | — | `Asia/Jakarta` |

## Konvensi Template (Jinja2)

- Variabel Copier: `{{ variable_name }}` → diganti saat render
- Variabel Docker Compose (dari `.env`): `${VAR_NAME}` → **dibiarkan apa adanya**, bukan variabel Copier
- Jangan pernah menukar konvensi keduanya

## Aturan docker-compose.yml.jinja

- `project_name` digunakan sebagai prefix: container name, volume name, dan nama router/service Traefik
- `traefik_network` dipakai di dua tempat: `networks.shared.name` dan label `traefik.docker.network`
- `domain` hanya ada di `.env.jinja` sebagai `DOMAIN={{ domain }}` — di compose tetap `${DOMAIN}`
- Service `db` **tidak** terhubung ke network `shared`, hanya ke `default` (tidak pernah diekspos Traefik)
- Selalu ada `healthcheck` pada service `db` dan `depends_on: condition: service_healthy` pada service `wiki`
- Gunakan image spesifik untuk database: `postgres:15-alpine` (bukan `latest`)
- Wiki.js menggunakan `requarks/wiki:2` (major tag) — intentional

## Aturan Traefik Labels

Label Traefik selalu menggunakan pola:
```
traefik.http.routers.{{ project_name }}.*
traefik.http.services.{{ project_name }}.*
```

Entrypoint dan certresolver adalah variabel Copier (`{{ traefik_entrypoint }}`, `{{ traefik_certresolver }}`).

## Aturan copier.yml

- Semua variabel wajib punya `help` yang jelas dalam Bahasa Indonesia
- Variabel yang tidak boleh kosong wajib punya `validator`
- `db_pass` wajib `secret: true` dan minimal 8 karakter
- Pilihan `traefik_entrypoint` dibatasi dengan `choices: [https, websecure]`

## CI / GitHub Actions

- Workflow di `.github/workflows/test.yml` berjalan pada push & PR ke `master`
- Test: render dengan `--defaults --overwrite --data ...`, lalu validasi file output dengan `test -f` dan `grep -q`
- `docker-compose.yml` syntax divalidasi dengan `python -c "import yaml; yaml.safe_load(...)"`
- Jika ada variabel baru ditambahkan di `copier.yml`, tambahkan `--data` yang sesuai di workflow dan tambahkan `grep -q` untuk memvalidasi nilainya

## Deployment

Template ini di-deploy ke server remote. Jangan jalankan `docker compose` atau `copier copy` secara langsung — berikan perintah kepada user untuk dijalankan manual.

## Branch conventions

- Selalu gunakan branch **`master`** sebagai branch utama.
- Jangan membuat branch baru kecuali diminta secara eksplisit.

## Commit conventions

- **Bahasa**: selalu gunakan Bahasa Indonesia untuk pesan commit.
- **Format**: `<tipe>: <ringkasan singkat>` diikuti body jika perlu.
- **Tipe yang digunakan**:
  | Tipe | Kapan dipakai |
  |---|---|
  | `feat` | Menambah fitur atau variabel template baru |
  | `fix` | Memperbaiki bug pada template atau test |
  | `test` | Menambah atau memperbaiki test |
  | `refactor` | Perubahan struktur tanpa mengubah perilaku |
  | `docs` | Perubahan README, copilot-instructions, komentar |
  | `ci` | Perubahan workflow GitHub Actions |
  | `chore` | Pembaruan dependensi, konfigurasi alat bantu |
- **Body** (opsional): jelaskan *apa yang berubah* dan *mengapa*, bukan *bagaimana*. Sertakan nama file atau variabel yang terdampak agar mudah ditelusuri dari log.

Contoh commit yang baik:
```
feat: tambah variabel traefik_network sebagai prompt Copier

Menambah variabel traefik_network di copier.yml agar nama Docker network
eksternal dapat dikonfigurasi saat generate. Template docker-compose.yml.jinja
diperbarui untuk menggunakan {{ traefik_network }} di networks.shared.name
dan label traefik.docker.network.
```
