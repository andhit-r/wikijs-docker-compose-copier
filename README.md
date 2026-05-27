# wikijs-copier

Copier template untuk mendeploy **Wiki.js 2** dengan PostgreSQL, diakses via **Traefik** sebagai reverse proxy.

## Prasyarat

- [Copier](https://copier.readthedocs.io/) ≥ 9.0
- Docker & Docker Compose
- Traefik sudah berjalan dengan network eksternal yang dikonfigurasi

## Penggunaan

```bash
# Dari path lokal
copier copy /path/to/wikijs-copier ./my-wiki

# Dari Git repository
copier copy gh:your-org/wikijs-copier ./my-wiki
```

### Prompt yang akan ditanyakan

| Prompt | Keterangan | Default |
|---|---|---|
| `project_name` | Nama project, prefix container & Traefik router | `wikijs` |
| `traefik_network` | Nama Docker network eksternal Traefik | `inverseproxy_shared` |
| `domain` | Domain untuk Wiki.js (Host rule Traefik) | — |
| `traefik_entrypoint` | Entrypoint Traefik HTTPS | `https` |
| `traefik_certresolver` | Cert resolver Traefik (Let's Encrypt) | `letsencrypt` |
| `db_name` | Nama database PostgreSQL | `wikijs` |
| `db_user` | Username database PostgreSQL | `wikijs` |
| `db_pass` | Password database PostgreSQL *(secret)* | — |
| `tz` | Timezone container | `Asia/Jakarta` |

## Setelah Generate

1. Masuk ke direktori output:
   ```bash
   cd my-wiki
   ```

2. Pastikan network Traefik sudah ada di server:
   ```bash
   docker network ls | grep <traefik_network>
   ```

4. Deploy di server:
   ```bash
   docker compose up -d
   ```

## Struktur Output

```
my-wiki/
├── docker-compose.yml   # Compose file dengan konfigurasi yang sudah dirender
├── .env                 # File env siap pakai dengan nilai dari prompt
└── .gitignore           # Mengabaikan .env
```

## Arsitektur

- **Wiki.js** terhubung ke dua network: `shared` (Traefik) dan `default` (internal)
- **PostgreSQL** hanya di network `default` — tidak pernah terekspos ke Traefik
- **TLS** ditangani oleh Traefik via Let's Encrypt

## Update Template

```bash
copier update
```

## Struktur Template

```
wikijs-copier/
├── copier.yml              # Konfigurasi & pertanyaan Copier
├── README.md               # Dokumentasi ini
└── template/
    ├── docker-compose.yml.jinja
    ├── .env.jinja
    └── .gitignore
```
