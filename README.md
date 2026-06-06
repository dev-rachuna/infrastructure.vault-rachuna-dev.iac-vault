# <img src=".gitlab/opentofu.png" alt="OpenTofu" height="30"/> Vault Infrastructure as Code

::include{file=.gitlab/badges.md}

---

Repozytorium zarządza konfiguracją HashiCorp Vault za pomocą OpenTofu. Konfiguracja jest zapisana w formacie `.tf.json`, a stan jest przechowywany w backendzie HTTP GitLaba.

## Zakres

| Moduł | Zarządzane zasoby |
|-------|-------------------|
| `auth/` | metoda uwierzytelniania `userpass` |
| `users/` | konta użytkowników i generowane hasła |
| `policies/` | polityki ACL Vault |
| `kv/` | silniki sekretów KV v2: `users` i `dev.rachuna` |
| `audit/` | audit device zapisujący logi do pliku |
| `pki/ca/` | Root CA oraz produkcyjne i testowe Intermediate CA |
| `pki/certs/` | role PKI, certyfikaty i zapis ich danych w KV |

Główny moduł w [`main.tf.json`](main.tf.json) składa powyższe komponenty w jedną konfigurację.

## Wymagania

- OpenTofu `>= 1.10.5`
- HashiCorp Vault dostępny z maszyny uruchamiającej OpenTofu
- `git`, `curl` i `jq`
- token Vault z uprawnieniami wymaganymi do zarządzania zasobami
- token GitLab z dostępem do projektu i zdalnego stanu OpenTofu

Provider Vault jest przypięty do wersji `5.9`, a provider Random do gałęzi `~> 3.7`.

## Konfiguracja środowiska

Przed uruchomieniem skryptów ustaw:

```bash
export VAULT_ADDR="https://vault.rachuna.dev"
export VAULT_TOKEN="hvs.xxx"
export GITLAB_TOKEN="glpat-xxx"

# Opcjonalnie; domyślna wartość to production
export TF_STATE_NAME="production"
```

W środowisku GitLab CI zamiast `GITLAB_TOKEN` można użyć `CI_JOB_TOKEN`. Adres GitLaba można zmienić przez `CI_SERVER_URL`.

## Użycie

```bash
# Inicjalizacja providerów, modułów i backendu GitLab HTTP
./bin/tofu-init

# Podgląd zmian
./bin/tofu-plan

# Zastosowanie zmian; skrypt używa -auto-approve
./bin/tofu-apply
```

Przed zastosowaniem zmian zawsze przejrzyj wynik `./bin/tofu-plan`. Skrypty `plan` i `apply` wykonują ponowną inicjalizację backendu.

## Struktura repozytorium

```text
iac-vault/
├── audit/             # Audit device Vault
├── auth/              # Metody uwierzytelniania
├── bin/               # Skrypty init, plan i apply
├── kv/                # Silniki sekretów KV v2
├── pki/
│   ├── ca/            # Root i Intermediate CA
│   └── certs/         # Role oraz certyfikaty PKI
├── policies/          # Polityki ACL
├── users/             # Konta userpass
├── main.tf.json       # Skład modułów
└── providers.tf.json  # Wersje providerów i backend HTTP
```

## Audit

Moduł `audit/` konfiguruje zapis logów Vault do `/opt/vault/vault-audit.log`. Plik musi istnieć na każdym węźle Vault i być zapisywalny przez proces Vault przed wykonaniem `apply`.

Szczegółowa konfiguracja uprawnień i rotacji logów znajduje się w [`audit/README.md`](audit/README.md).

## Bezpieczeństwo

- Nie commituj tokenów, haseł, plików `.tfvars` ani stanu OpenTofu.
- Przekazuj dane uwierzytelniające wyłącznie przez zmienne środowiskowe lub chronione zmienne CI/CD.
- Traktuj stan OpenTofu jako poufny, ponieważ może zawierać hasła, klucze prywatne i dane certyfikatów.
- Ogranicz dostęp do zdalnego stanu GitLab zgodnie z zasadą najmniejszych uprawnień.
- Provider ma obecnie włączone `skip_tls_verify`; używaj tej konfiguracji wyłącznie w kontrolowanym środowisku.

---

::include{file=.gitlab/footer.md}
