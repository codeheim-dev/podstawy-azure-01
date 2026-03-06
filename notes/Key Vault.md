# Azure Key Vault - Polecenia Azure CLI

## Czym jest Azure Key Vault?

Azure Key Vault to zarządzana usługa do bezpiecznego przechowywania i kontrolowania dostępu do sekretów, kluczy kryptograficznych i certyfikatów. Centralizuje zarządzanie wrażliwymi danymi i eliminuje potrzebę przechowywania ich w kodzie aplikacji.

**Kluczowe cechy:**
- Przechowywanie sekretów (connection strings, hasła, tokeny API)
- Zarządzanie kluczami kryptograficznymi (RSA, EC)
- Zarządzanie certyfikatami SSL/TLS (tworzenie, import, automatyczne odnawianie)
- Integracja z Managed Identity (bezhasłowy dostęp z usług Azure)
- Logowanie i audyt każdego dostępu (Azure Monitor, Log Analytics)
- Soft-delete i purge protection (ochrona przed przypadkowym usunięciem)
- Kontrola dostępu przez RBAC lub Access Policies
- Dwie warstwy: **Standard** (software-backed) i **Premium** (HSM-backed)

---

## Tworzenie Key Vault

### Podstawowe tworzenie

```bash
# Utworzenie Key Vault
az keyvault create \
  --resource-group <resource-group-name> \
  --name <vault-name> \
  --location westeurope

# Przykład
az keyvault create \
  --resource-group MyResourceGroup \
  --name mykeyvault2026 \
  --location westeurope
```

> **Uwaga:** Nazwa Key Vault musi być globalnie unikalna (3–24 znaki, litery, cyfry i myślniki).

### Tworzenie z pełną konfiguracją

```bash
az keyvault create \
  --resource-group MyResourceGroup \
  --name mykeyvault2026 \
  --location westeurope \
  --sku premium \
  --enable-rbac-authorization true \
  --enable-soft-delete true \
  --soft-delete-retention-days 90 \
  --enable-purge-protection true \
  --retention-days 90
```

### Tworzenie z dostępem z sieci wirtualnej

```bash
# Key Vault z ograniczeniami sieciowymi
az keyvault create \
  --resource-group MyResourceGroup \
  --name mykeyvault2026 \
  --location westeurope \
  --default-action Deny \
  --bypass AzureServices
```

---

## Zarządzanie Key Vault

### Wyświetlanie informacji

```bash
# Lista Key Vaults w subskrypcji
az keyvault list --output table

# Lista w grupie zasobów
az keyvault list --resource-group <resource-group-name> --output table

# Szczegóły Key Vault
az keyvault show \
  --name <vault-name> \
  --resource-group <resource-group-name>

# Pobranie URI Key Vault
az keyvault show \
  --name <vault-name> \
  --query properties.vaultUri --output tsv
```

### Aktualizacja Key Vault

```bash
# Włączenie RBAC
az keyvault update \
  --name mykeyvault2026 \
  --resource-group MyResourceGroup \
  --enable-rbac-authorization true

# Włączenie purge protection (nieodwracalne!)
az keyvault update \
  --name mykeyvault2026 \
  --resource-group MyResourceGroup \
  --enable-purge-protection true

# Zmiana domyślnej akcji sieciowej
az keyvault update \
  --name mykeyvault2026 \
  --resource-group MyResourceGroup \
  --default-action Deny
```

### Usuwanie Key Vault

```bash
# Usunięcie Key Vault (soft-delete — trafia do kosza)
az keyvault delete \
  --name <vault-name> \
  --resource-group <resource-group-name>

# Lista usuniętych (soft-deleted) Key Vaults
az keyvault list-deleted --output table

# Przywrócenie usuniętego Key Vault
az keyvault recover --name <vault-name>

# Trwałe usunięcie (purge) — wymaga wyłączonego purge protection
az keyvault purge --name <vault-name>
```

---

## Sekrety (Secrets)

Sekrety to dowolne dane tekstowe: hasła, connection strings, tokeny API, klucze licencyjne itp.

### Tworzenie i ustawianie sekretów

```bash
# Ustawienie sekretu
az keyvault secret set \
  --vault-name <vault-name> \
  --name <secret-name> \
  --value <secret-value>

# Przykłady
az keyvault secret set \
  --vault-name mykeyvault2026 \
  --name DatabasePassword \
  --value 'MyStr0ngP@ssw0rd!'

az keyvault secret set \
  --vault-name mykeyvault2026 \
  --name ConnectionString \
  --value 'Server=myserver.database.windows.net;Database=mydb;User=admin;Password=secret'

az keyvault secret set \
  --vault-name mykeyvault2026 \
  --name ApiKey \
  --value 'sk-abc123def456'
```

### Sekret z datą wygaśnięcia

```bash
# Sekret z datą wygaśnięcia i aktywacji
az keyvault secret set \
  --vault-name mykeyvault2026 \
  --name TemporaryToken \
  --value 'temp-token-value' \
  --expires '2026-12-31T23:59:59Z' \
  --not-before '2026-03-06T00:00:00Z'
```

### Sekret z pliku

```bash
# Ustawienie sekretu z zawartości pliku
az keyvault secret set \
  --vault-name mykeyvault2026 \
  --name SSHPrivateKey \
  --file ~/.ssh/id_rsa

# Sekret z pliku z określonym kodowaniem
az keyvault secret set \
  --vault-name mykeyvault2026 \
  --name CertificatePEM \
  --file ./cert.pem \
  --encoding utf-8
```

### Sekret z tagami

```bash
az keyvault secret set \
  --vault-name mykeyvault2026 \
  --name DatabasePassword \
  --value 'MyStr0ngP@ssw0rd!' \
  --tags environment=production team=backend
```

### Pobieranie sekretów

```bash
# Pobranie wartości sekretu
az keyvault secret show \
  --vault-name <vault-name> \
  --name <secret-name> \
  --query value --output tsv

# Pełne szczegóły sekretu
az keyvault secret show \
  --vault-name <vault-name> \
  --name <secret-name>

# Pobranie konkretnej wersji
az keyvault secret show \
  --vault-name <vault-name> \
  --name <secret-name> \
  --version <version-id>

# Lista wszystkich sekretów
az keyvault secret list \
  --vault-name <vault-name> \
  --output table

# Lista wersji sekretu
az keyvault secret list-versions \
  --vault-name <vault-name> \
  --name <secret-name> \
  --output table
```

### Aktualizacja i usuwanie sekretów

```bash
# Aktualizacja atrybutów sekretu (bez zmiany wartości)
az keyvault secret set-attributes \
  --vault-name mykeyvault2026 \
  --name DatabasePassword \
  --expires '2027-01-01T00:00:00Z' \
  --tags environment=staging

# Wyłączenie sekretu (bez usuwania)
az keyvault secret set-attributes \
  --vault-name mykeyvault2026 \
  --name OldApiKey \
  --enabled false

# Usunięcie sekretu (soft-delete)
az keyvault secret delete \
  --vault-name <vault-name> \
  --name <secret-name>

# Lista usuniętych sekretów
az keyvault secret list-deleted \
  --vault-name <vault-name> \
  --output table

# Przywrócenie usuniętego sekretu
az keyvault secret recover \
  --vault-name <vault-name> \
  --name <secret-name>

# Trwałe usunięcie (purge)
az keyvault secret purge \
  --vault-name <vault-name> \
  --name <secret-name>
```

### Backup i restore sekretów

```bash
# Backup sekretu do pliku
az keyvault secret backup \
  --vault-name <vault-name> \
  --name <secret-name> \
  --file secret-backup.blob

# Przywrócenie sekretu z backupu
az keyvault secret restore \
  --vault-name <vault-name> \
  --file secret-backup.blob
```

---

## Klucze kryptograficzne (Keys)

Klucze służą do szyfrowania, podpisywania i opakowywania (wrapping) danych.

### Tworzenie kluczy

```bash
# Klucz RSA (domyślny)
az keyvault key create \
  --vault-name <vault-name> \
  --name <key-name> \
  --kty RSA \
  --size 2048

# Klucz RSA 4096-bit
az keyvault key create \
  --vault-name mykeyvault2026 \
  --name MyEncryptionKey \
  --kty RSA \
  --size 4096

# Klucz EC (Elliptic Curve)
az keyvault key create \
  --vault-name mykeyvault2026 \
  --name MySigningKey \
  --kty EC \
  --curve P-256

# Klucz z dozwolonymi operacjami
az keyvault key create \
  --vault-name mykeyvault2026 \
  --name MyKey \
  --kty RSA \
  --size 2048 \
  --ops encrypt decrypt sign verify wrapKey unwrapKey

# Klucz chroniony przez HSM (wymaga Premium SKU)
az keyvault key create \
  --vault-name mykeyvault2026 \
  --name MyHSMKey \
  --kty RSA-HSM \
  --size 2048
```

### Import klucza

```bash
# Import klucza z pliku PEM
az keyvault key import \
  --vault-name mykeyvault2026 \
  --name ImportedKey \
  --pem-file ./private-key.pem

# Import klucza z pliku PEM z hasłem
az keyvault key import \
  --vault-name mykeyvault2026 \
  --name ImportedKey \
  --pem-file ./encrypted-key.pem \
  --pem-password 'keypassword'
```

### Zarządzanie kluczami

```bash
# Lista kluczy
az keyvault key list \
  --vault-name <vault-name> \
  --output table

# Szczegóły klucza
az keyvault key show \
  --vault-name <vault-name> \
  --name <key-name>

# Lista wersji klucza
az keyvault key list-versions \
  --vault-name <vault-name> \
  --name <key-name> \
  --output table

# Rotacja klucza (tworzenie nowej wersji)
az keyvault key rotate \
  --vault-name <vault-name> \
  --name <key-name>

# Wyłączenie klucza
az keyvault key set-attributes \
  --vault-name <vault-name> \
  --name <key-name> \
  --enabled false

# Usunięcie klucza (soft-delete)
az keyvault key delete \
  --vault-name <vault-name> \
  --name <key-name>

# Przywrócenie klucza
az keyvault key recover \
  --vault-name <vault-name> \
  --name <key-name>

# Backup klucza
az keyvault key backup \
  --vault-name <vault-name> \
  --name <key-name> \
  --file key-backup.blob

# Restore klucza
az keyvault key restore \
  --vault-name <vault-name> \
  --file key-backup.blob
```

### Polityka rotacji kluczy

```bash
# Ustawienie automatycznej rotacji co 90 dni
az keyvault key rotation-policy update \
  --vault-name mykeyvault2026 \
  --name MyKey \
  --value ./rotation-policy.json
```

Przykładowy `rotation-policy.json`:
```json
{
  "lifetimeActions": [
    {
      "trigger": {
        "timeAfterCreate": "P90D"
      },
      "action": {
        "type": "Rotate"
      }
    },
    {
      "trigger": {
        "timeBeforeExpiry": "P30D"
      },
      "action": {
        "type": "Notify"
      }
    }
  ],
  "attributes": {
    "expiryTime": "P1Y"
  }
}
```

---

## Certyfikaty (Certificates)

Key Vault umożliwia tworzenie, import i automatyczne odnawianie certyfikatów SSL/TLS.

### Tworzenie certyfikatu (self-signed)

```bash
# Tworzenie self-signed certyfikatu
az keyvault certificate create \
  --vault-name <vault-name> \
  --name <cert-name> \
  --policy "$(az keyvault certificate get-default-policy)"

# Tworzenie z niestandardową polityką
az keyvault certificate create \
  --vault-name mykeyvault2026 \
  --name MyCert \
  --policy @cert-policy.json
```

Przykładowy `cert-policy.json`:
```json
{
  "issuerParameters": {
    "name": "Self"
  },
  "keyProperties": {
    "exportable": true,
    "keySize": 2048,
    "keyType": "RSA",
    "reuseKey": true
  },
  "x509CertificateProperties": {
    "subject": "CN=myapp.example.com",
    "subjectAlternativeNames": {
      "dnsNames": ["myapp.example.com", "www.myapp.example.com"]
    },
    "validityInMonths": 12
  },
  "lifetimeActions": [
    {
      "trigger": {
        "daysBeforeExpiry": 30
      },
      "action": {
        "actionType": "AutoRenew"
      }
    }
  ]
}
```

### Import certyfikatu

```bash
# Import certyfikatu PFX
az keyvault certificate import \
  --vault-name <vault-name> \
  --name <cert-name> \
  --file ./certificate.pfx \
  --password <pfx-password>

# Import certyfikatu PEM
az keyvault certificate import \
  --vault-name <vault-name> \
  --name <cert-name> \
  --file ./certificate.pem
```

### Zarządzanie certyfikatami

```bash
# Lista certyfikatów
az keyvault certificate list \
  --vault-name <vault-name> \
  --output table

# Szczegóły certyfikatu
az keyvault certificate show \
  --vault-name <vault-name> \
  --name <cert-name>

# Pobranie certyfikatu (Base64)
az keyvault certificate show \
  --vault-name <vault-name> \
  --name <cert-name> \
  --query cer --output tsv

# Pobranie i zapis do pliku PFX
az keyvault secret show \
  --vault-name <vault-name> \
  --name <cert-name> \
  --query value --output tsv | base64 -d > cert.pfx

# Lista wersji certyfikatu
az keyvault certificate list-versions \
  --vault-name <vault-name> \
  --name <cert-name> \
  --output table

# Usunięcie certyfikatu
az keyvault certificate delete \
  --vault-name <vault-name> \
  --name <cert-name>

# Przywrócenie certyfikatu
az keyvault certificate recover \
  --vault-name <vault-name> \
  --name <cert-name>

# Backup certyfikatu
az keyvault certificate backup \
  --vault-name <vault-name> \
  --name <cert-name> \
  --file cert-backup.blob

# Restore certyfikatu
az keyvault certificate restore \
  --vault-name <vault-name> \
  --file cert-backup.blob
```

---

## Kontrola dostępu

### RBAC (zalecane)

```bash
# Włączenie RBAC na Key Vault
az keyvault update \
  --name mykeyvault2026 \
  --resource-group MyResourceGroup \
  --enable-rbac-authorization true

# Przypisanie roli Key Vault Secrets User (odczyt sekretów)
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <principal-id-or-email> \
  --scope /subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault-name>

# Przypisanie roli Key Vault Secrets Officer (odczyt + zapis sekretów)
az role assignment create \
  --role "Key Vault Secrets Officer" \
  --assignee <principal-id-or-email> \
  --scope /subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault-name>

# Przypisanie roli Key Vault Administrator (pełny dostęp)
az role assignment create \
  --role "Key Vault Administrator" \
  --assignee <principal-id-or-email> \
  --scope /subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault-name>
```

**Wbudowane role Key Vault:**

| Rola | Opis |
|------|------|
| `Key Vault Administrator` | Pełny dostęp do wszystkich obiektów |
| `Key Vault Secrets User` | Odczyt sekretów |
| `Key Vault Secrets Officer` | Zarządzanie sekretami |
| `Key Vault Crypto User` | Operacje kryptograficzne (szyfrowanie, podpisywanie) |
| `Key Vault Crypto Officer` | Zarządzanie kluczami |
| `Key Vault Certificates Officer` | Zarządzanie certyfikatami |
| `Key Vault Reader` | Odczyt metadanych (bez wartości) |

### Access Policies (starszy model)

```bash
# Nadanie uprawnień do sekretów
az keyvault set-policy \
  --name mykeyvault2026 \
  --object-id <principal-object-id> \
  --secret-permissions get list set delete

# Nadanie uprawnień do kluczy
az keyvault set-policy \
  --name mykeyvault2026 \
  --object-id <principal-object-id> \
  --key-permissions get list create delete encrypt decrypt

# Nadanie uprawnień do certyfikatów
az keyvault set-policy \
  --name mykeyvault2026 \
  --object-id <principal-object-id> \
  --certificate-permissions get list create delete

# Nadanie uprawnień dla Service Principal (App Registration)
az keyvault set-policy \
  --name mykeyvault2026 \
  --spn <app-id> \
  --secret-permissions get list

# Usunięcie uprawnień
az keyvault delete-policy \
  --name mykeyvault2026 \
  --object-id <principal-object-id>
```

---

## Networking (reguły sieciowe)

```bash
# Ustawienie domyślnej akcji na Deny
az keyvault update \
  --name mykeyvault2026 \
  --resource-group MyResourceGroup \
  --default-action Deny

# Dodanie reguły dla IP
az keyvault network-rule add \
  --name mykeyvault2026 \
  --ip-address <ip-address-or-cidr>

# Dodanie reguły dla subnetu VNet
az keyvault network-rule add \
  --name mykeyvault2026 \
  --subnet <subnet-resource-id>

# Zezwolenie na bypass dla usług Azure
az keyvault update \
  --name mykeyvault2026 \
  --resource-group MyResourceGroup \
  --bypass AzureServices

# Lista reguł sieciowych
az keyvault network-rule list \
  --name mykeyvault2026

# Usunięcie reguły IP
az keyvault network-rule remove \
  --name mykeyvault2026 \
  --ip-address <ip-address-or-cidr>

# Przywrócenie domyślnej akcji na Allow
az keyvault update \
  --name mykeyvault2026 \
  --resource-group MyResourceGroup \
  --default-action Allow
```

### Private Endpoint

```bash
# Tworzenie Private Endpoint dla Key Vault
az network private-endpoint create \
  --resource-group MyResourceGroup \
  --name mykeyvault-pe \
  --vnet-name myVNet \
  --subnet mySubnet \
  --private-connection-resource-id $(az keyvault show --name mykeyvault2026 --query id --output tsv) \
  --group-id vault \
  --connection-name mykeyvault-connection
```

---

## Integracja z usługami Azure

### Managed Identity — dostęp bez hasła

```bash
# 1. Włącz system-assigned identity na VM / App Service / Container App
az vm identity assign --resource-group MyRG --name MyVM

# 2. Nadaj uprawnienia do Key Vault (RBAC)
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <vm-principal-id> \
  --scope /subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/mykeyvault2026

# 2. Lub Access Policy
az keyvault set-policy \
  --name mykeyvault2026 \
  --object-id <vm-principal-id> \
  --secret-permissions get list
```

### App Service — Key Vault References

```bash
# Ustawienie app setting z referencją do Key Vault
az webapp config appsettings set \
  --resource-group MyResourceGroup \
  --name mywebapp \
  --settings "DatabasePassword=@Microsoft.KeyVault(VaultName=mykeyvault2026;SecretName=DatabasePassword)"

# Format reference:
# @Microsoft.KeyVault(VaultName=<vault>;SecretName=<secret>)
# @Microsoft.KeyVault(VaultName=<vault>;SecretName=<secret>;SecretVersion=<version>)
# @Microsoft.KeyVault(SecretUri=https://<vault>.vault.azure.net/secrets/<secret>)
```

### Container Apps — sekret z Key Vault

```bash
az containerapp secret set \
  --name myapp \
  --resource-group MyResourceGroup \
  --secrets "db-password=keyvaultref:https://mykeyvault2026.vault.azure.net/secrets/DatabasePassword,identityref:system"
```

---

## Diagnostyka i monitorowanie

```bash
# Włączenie logowania diagnostycznego
az monitor diagnostic-settings create \
  --name keyvault-diagnostics \
  --resource $(az keyvault show --name mykeyvault2026 --query id --output tsv) \
  --workspace <log-analytics-workspace-id> \
  --logs '[{"category":"AuditEvent","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'

# Sprawdzenie ustawień diagnostycznych
az monitor diagnostic-settings list \
  --resource $(az keyvault show --name mykeyvault2026 --query id --output tsv) \
  --output table
```

---

## Przykład: pełne wdrożenie od zera

```bash
# 1. Utwórz resource group
az group create --name KeyVaultRG --location westeurope

# 2. Utwórz Key Vault z RBAC
az keyvault create \
  --resource-group KeyVaultRG \
  --name myapp-kv-2026 \
  --location westeurope \
  --enable-rbac-authorization true \
  --enable-soft-delete true \
  --soft-delete-retention-days 90 \
  --enable-purge-protection true

# 3. Przypisz sobie rolę administratora
az role assignment create \
  --role "Key Vault Administrator" \
  --assignee $(az ad signed-in-user show --query id --output tsv) \
  --scope $(az keyvault show --name myapp-kv-2026 --query id --output tsv)

# 4. Dodaj sekrety
az keyvault secret set --vault-name myapp-kv-2026 --name DbPassword --value 'P@ssw0rd123!'
az keyvault secret set --vault-name myapp-kv-2026 --name ApiKey --value 'sk-abc123'
az keyvault secret set --vault-name myapp-kv-2026 --name ConnectionString \
  --value 'Server=mydb.postgres.database.azure.com;Database=myapp;User=admin;Password=secret'

# 5. Utwórz klucz szyfrujący
az keyvault key create --vault-name myapp-kv-2026 --name DataEncryptionKey --kty RSA --size 2048

# 6. Utwórz self-signed certyfikat
az keyvault certificate create \
  --vault-name myapp-kv-2026 \
  --name AppCert \
  --policy "$(az keyvault certificate get-default-policy)"

# 7. Zweryfikuj zawartość
az keyvault secret list --vault-name myapp-kv-2026 --output table
az keyvault key list --vault-name myapp-kv-2026 --output table
az keyvault certificate list --vault-name myapp-kv-2026 --output table

# 8. Pobierz sekret
az keyvault secret show --vault-name myapp-kv-2026 --name DbPassword --query value --output tsv
```

---

## Podsumowanie najważniejszych poleceń

| Polecenie | Opis |
|-----------|------|
| **Key Vault** | |
| `az keyvault create` | Tworzenie Key Vault |
| `az keyvault show` | Szczegóły Key Vault |
| `az keyvault list` | Lista Key Vaults |
| `az keyvault update` | Aktualizacja Key Vault |
| `az keyvault delete` | Usunięcie Key Vault (soft-delete) |
| `az keyvault recover` | Przywrócenie Key Vault |
| `az keyvault purge` | Trwałe usunięcie Key Vault |
| **Sekrety** | |
| `az keyvault secret set` | Ustawienie sekretu |
| `az keyvault secret show` | Pobranie sekretu |
| `az keyvault secret list` | Lista sekretów |
| `az keyvault secret delete` | Usunięcie sekretu |
| `az keyvault secret recover` | Przywrócenie sekretu |
| `az keyvault secret backup` | Backup sekretu |
| **Klucze** | |
| `az keyvault key create` | Tworzenie klucza |
| `az keyvault key show` | Szczegóły klucza |
| `az keyvault key list` | Lista kluczy |
| `az keyvault key rotate` | Rotacja klucza |
| `az keyvault key delete` | Usunięcie klucza |
| **Certyfikaty** | |
| `az keyvault certificate create` | Tworzenie certyfikatu |
| `az keyvault certificate import` | Import certyfikatu |
| `az keyvault certificate show` | Szczegóły certyfikatu |
| `az keyvault certificate list` | Lista certyfikatów |
| `az keyvault certificate delete` | Usunięcie certyfikatu |
| **Dostęp** | |
| `az keyvault set-policy` | Ustawienie access policy |
| `az keyvault delete-policy` | Usunięcie access policy |
| `az keyvault network-rule add` | Dodanie reguły sieciowej |
| `az role assignment create` | Przypisanie roli RBAC |
