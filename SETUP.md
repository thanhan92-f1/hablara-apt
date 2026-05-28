# Hablara APT Repository Setup

Anleitung zum Einrichten des `thanhan92-f1/hablara-apt` GitHub-Repositorys.

## 1. GPG Key generieren

```bash
# RSA 4096, kein Ablauf oder 5 Jahre
gpg --full-generate-key

# Name: Hablara APT Signing Key
# Email: apt@hablara.de (oder deine Email)

# Key-ID ermitteln
gpg --list-secret-keys --keyid-format long

# Public Key exportieren (fuer hablara Repo)
gpg --armor --export <KEY_ID> > /path/to/hablara/src-tauri/deb-scripts/gpg-key.asc

# Private Key exportieren (fuer GitHub Secret)
gpg --armor --export-secret-keys <KEY_ID> | pbcopy
```

## 2. GitHub Repository erstellen

1. Erstelle `thanhan92-f1/hablara-apt` auf GitHub (public)
2. Kopiere alle Dateien aus `scripts-dev/apt-repo/` in das neue Repo:
   ```
   hablara-apt/
   ├── conf/
   │   ├── distributions
   │   └── options
   ├── .github/
   │   └── workflows/
   │       └── update-apt.yml
   └── README.md  (optional, Installationsanleitung)
   ```

## 3. GitHub Pages aktivieren

Repository Settings → Pages → Source: "GitHub Actions"

## 4. Secrets konfigurieren

### In `thanhan92-f1/hablara-apt`:

| Secret | Wert |
|--------|------|
| `APT_GPG_PRIVATE_KEY` | Armored GPG Private Key |
| `APT_GPG_PASSPHRASE` | GPG Key Passphrase (optional) |
| `HABLARA_REPO_TOKEN` | PAT mit `repo` Scope (zum Downloaden von hablara Releases) |

### In `thanhan92-f1/hablara`:

| Secret | Wert |
|--------|------|
| `APT_DISPATCH_TOKEN` | PAT mit `repo` Scope (zum Triggern von hablara-apt Workflows) |

**Hinweis:** Ein einzelner PAT mit `repo` Scope fuer beide Repos reicht aus.

## 5. Testen

```bash
# Manueller Trigger via GitHub CLI
gh workflow run update-apt.yml \
  -R thanhan92-f1/hablara-apt \
  --field version=v1.2.5 \
  --field arch=both

# Oder: Test-Release aus dem hablara-Repo triggern
```

## 6. Lokaler Test mit Docker

```bash
docker run -it --rm ubuntu:22.04 bash

# Im Container:
apt-get update && apt-get install -y curl gnupg

curl -fsSL https://thanhan92-f1.github.io/hablara-apt/pubkey.asc \
  | gpg --dearmor -o /usr/share/keyrings/hablara-archive-keyring.gpg

cat > /etc/apt/sources.list.d/hablara.sources <<'EOF'
Types: deb
URIs: https://thanhan92-f1.github.io/hablara-apt
Suites: stable
Components: main
Architectures: amd64 arm64
Signed-By: /usr/share/keyrings/hablara-archive-keyring.gpg
EOF

apt-get update
apt-cache policy hablara
```
