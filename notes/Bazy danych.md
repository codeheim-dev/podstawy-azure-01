# Azure Database for PostgreSQL - Polecenia Azure CLI

## Czym jest Azure Database for PostgreSQL?

Azure Database for PostgreSQL to w pełni zarządzana usługa bazodanowa oparta na silniku PostgreSQL.

**Kluczowe cechy:**
- W pełni zarządzana usługa (patching, backupy, HA — automatycznie)
- Wbudowana wysoka dostępność (zone-redundant / same-zone)
- Automatyczne backupy z retencją do 35 dni
- Skalowanie compute i storage niezależnie
- Integracja z VNet (prywatny dostęp)
- Obsługa głównych wersji PostgreSQL (13, 14, 15, 16, 17)
- Intelligent Performance (Query Store, Query Performance Insight)

---

## Wymagania wstępne

```bash
# Sprawdzenie czy rozszerzenie jest zainstalowane
az extension add --name rdbms-connect --upgrade

# Rejestracja dostawcy (jeśli nie jest zarejestrowany)
az provider register --namespace Microsoft.DBforPostgreSQL
```

---

## Tworzenie serwera PostgreSQL Flexible Server

### Podstawowe tworzenie

```bash
# Najprostsze utworzenie serwera
az postgres flexible-server create \
  --resource-group <resource-group-name> \
  --name <server-name> \
  --location westeurope \
  --admin-user <admin-username> \
  --admin-password <admin-password>

# Przykład
az postgres flexible-server create \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --location westeurope \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!'
```

> **Uwaga:** Nazwa serwera musi być globalnie unikalna. FQDN: `<server-name>.postgres.database.azure.com`

### Tworzenie z pełną konfiguracją

```bash
az postgres flexible-server create \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --location westeurope \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --version 16 \
  --sku-name Standard_B1ms \
  --tier Burstable \
  --storage-size 32 \
  --backup-retention 14 \
  --geo-redundant-backup Disabled \
  --high-availability Disabled \
  --public-access 0.0.0.0
```

### Warstwy cenowe (tiers) i rozmiary

```bash
# Lista dostępnych SKU w regionie
az postgres flexible-server list-skus \
  --location westeurope \
  --output table
```

| Warstwa | SKU (przykłady) | Opis |
|---------|-----------------|------|
| **Burstable** | `Standard_B1ms`, `Standard_B2s`, `Standard_B2ms` | Dev/test, niskie obciążenie |
| **General Purpose** | `Standard_D2s_v3`, `Standard_D4s_v3`, `Standard_D8s_v3` | Produkcja, zrównoważone obciążenie |
| **Memory Optimized** | `Standard_E2s_v3`, `Standard_E4s_v3`, `Standard_E8s_v3` | Duże bazy, cache, analytics |

### Tworzenie z wysoką dostępnością

```bash
# Zone-redundant HA (replika w innej strefie dostępności)
az postgres flexible-server create \
  --resource-group MyResourceGroup \
  --name mypostgres-ha \
  --location westeurope \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --tier GeneralPurpose \
  --sku-name Standard_D2s_v3 \
  --storage-size 64 \
  --high-availability ZoneRedundant \
  --standby-zone 3

# Same-zone HA
az postgres flexible-server create \
  --resource-group MyResourceGroup \
  --name mypostgres-ha \
  --location westeurope \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --tier GeneralPurpose \
  --sku-name Standard_D2s_v3 \
  --high-availability SameZone
```

> **Uwaga:** Wysoka dostępność wymaga warstwy General Purpose lub Memory Optimized (nie Burstable).

---

## Networking (sieć)

### Dostęp publiczny (firewall)

```bash
# Tworzenie z dostępem publicznym (wszystkie adresy Azure)
az postgres flexible-server create \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --location westeurope \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --public-access 0.0.0.0

# Tworzenie z dostępem z konkretnego IP
az postgres flexible-server create \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --location westeurope \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --public-access <my-ip-address>

# Tworzenie z zakresem IP
az postgres flexible-server create \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --location westeurope \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --public-access <start-ip>-<end-ip>
```

### Reguły firewall

```bash
# Dodanie reguły firewall
az postgres flexible-server firewall-rule create \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --rule-name AllowMyIP \
  --start-ip-address <my-ip> \
  --end-ip-address <my-ip>

# Zezwolenie na dostęp z usług Azure
az postgres flexible-server firewall-rule create \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --rule-name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Zezwolenie na dostęp ze wszystkich adresów (NIE dla produkcji!)
az postgres flexible-server firewall-rule create \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --rule-name AllowAll \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 255.255.255.255

# Lista reguł firewall
az postgres flexible-server firewall-rule list \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --output table

# Usunięcie reguły
az postgres flexible-server firewall-rule delete \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --rule-name AllowMyIP \
  --yes
```

### Dostęp prywatny (VNet integration)

```bash
# Tworzenie z prywatnym dostępem (nowy VNet)
az postgres flexible-server create \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --location westeurope \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --vnet myVNet \
  --subnet mySubnet \
  --private-dns-zone mypostgres.private.postgres.database.azure.com

# Tworzenie z istniejącym VNet i subnet
az postgres flexible-server create \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --location westeurope \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --subnet <subnet-resource-id> \
  --private-dns-zone <private-dns-zone-resource-id>
```

> **Uwaga:** Po utworzeniu serwera nie można zmienić trybu dostępu (publiczny ↔ prywatny).

---

## Zarządzanie bazami danych

### Tworzenie i zarządzanie bazami

```bash
# Utworzenie bazy danych
az postgres flexible-server db create \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --database-name myappdb

# Utworzenie z określonym kodowaniem i collation
az postgres flexible-server db create \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --database-name myappdb \
  --charset UTF8 \
  --collation "en_US.utf8"

# Lista baz danych
az postgres flexible-server db list \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --output table

# Szczegóły bazy danych
az postgres flexible-server db show \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --database-name myappdb

# Usunięcie bazy danych
az postgres flexible-server db delete \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --database-name myappdb \
  --yes
```

---

## Łączenie z bazą danych

### Azure CLI — interaktywne połączenie

```bash
# Połączenie z serwerem (wymaga rozszerzenia rdbms-connect)
az postgres flexible-server connect \
  --name mypostgresserver \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --database-name myappdb

# Wykonanie zapytania SQL
az postgres flexible-server execute \
  --name mypostgresserver \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --database-name myappdb \
  --querytext "SELECT version();"

# Wykonanie zapytania z pliku
az postgres flexible-server execute \
  --name mypostgresserver \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --database-name myappdb \
  --file-path ./init.sql
```

### psql (natywny klient PostgreSQL)

```bash
# Połączenie przez psql
psql "host=mypostgresserver.postgres.database.azure.com \
  port=5432 \
  dbname=myappdb \
  user=pgadmin \
  password='MyStr0ngP@ssw0rd!' \
  sslmode=require"

# Lub krócej
psql "postgresql://pgadmin:MyStr0ngP@ssw0rd!@mypostgresserver.postgres.database.azure.com:5432/myappdb?sslmode=require"
```

### Connection string (dla aplikacji)

```
# Format connection string
postgresql://pgadmin:<password>@mypostgresserver.postgres.database.azure.com:5432/myappdb?sslmode=require

# ADO.NET
Server=mypostgresserver.postgres.database.azure.com;Database=myappdb;Port=5432;User Id=pgadmin;Password=<password>;Ssl Mode=Require;

# JDBC
jdbc:postgresql://mypostgresserver.postgres.database.azure.com:5432/myappdb?user=pgadmin&password=<password>&sslmode=require
```

---

## Wyświetlanie informacji o serwerze

```bash
# Lista serwerów w subskrypcji
az postgres flexible-server list --output table

# Lista w grupie zasobów
az postgres flexible-server list \
  --resource-group MyResourceGroup \
  --output table

# Szczegóły serwera
az postgres flexible-server show \
  --resource-group MyResourceGroup \
  --name mypostgresserver

# Pobranie FQDN serwera
az postgres flexible-server show \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --query fullyQualifiedDomainName --output tsv

# Status serwera
az postgres flexible-server show \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --query state --output tsv

# Wersja PostgreSQL
az postgres flexible-server show \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --query version --output tsv

# Podsumowanie serwera (przydatne info w tabeli)
az postgres flexible-server show \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --query "{Name:name, FQDN:fullyQualifiedDomainName, State:state, Version:version, SKU:sku.name, Tier:sku.tier, Storage:storage.storageSizeGb, HA:highAvailability.mode}" \
  --output table
```

---

## Aktualizacja serwera

### Skalowanie

```bash
# Zmiana warstwy i rozmiaru (SKU)
az postgres flexible-server update \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --sku-name Standard_D4s_v3 \
  --tier GeneralPurpose

# Zwiększenie storage (nie można zmniejszyć!)
az postgres flexible-server update \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --storage-size 128

# Włączenie auto-grow storage
az postgres flexible-server update \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --storage-auto-grow Enabled
```

### Zmiana retencji backupów

```bash
az postgres flexible-server update \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --backup-retention 35
```

### Włączenie/wyłączenie HA

```bash
# Włączenie HA
az postgres flexible-server update \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --high-availability ZoneRedundant

# Wyłączenie HA
az postgres flexible-server update \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --high-availability Disabled
```

---

## Konfiguracja parametrów serwera

```bash
# Lista parametrów
az postgres flexible-server parameter list \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --output table

# Wyświetlenie konkretnego parametru
az postgres flexible-server parameter show \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --name max_connections

# Zmiana parametru
az postgres flexible-server parameter set \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --name max_connections \
  --value 200

# Inne przydatne parametry
az postgres flexible-server parameter set \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --name log_min_duration_statement \
  --value 1000

az postgres flexible-server parameter set \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --name shared_buffers \
  --value 262144
```

> **Uwaga:** Niektóre parametry wymagają restartu serwera (`is_restart_required = true`).

---

## Uruchamianie i zatrzymywanie serwera

```bash
# Zatrzymanie serwera (brak opłat za compute, storage nadal naliczane)
az postgres flexible-server stop \
  --resource-group MyResourceGroup \
  --name mypostgresserver

# Uruchomienie serwera
az postgres flexible-server start \
  --resource-group MyResourceGroup \
  --name mypostgresserver

# Restart serwera
az postgres flexible-server restart \
  --resource-group MyResourceGroup \
  --name mypostgresserver
```

> **Uwaga:** Zatrzymany serwer automatycznie uruchomi się po 7 dniach.

---

## Backup i przywracanie

### Backup

Backupy są tworzone automatycznie. Retencja: 7–35 dni (domyślnie 7).

```bash
# Wyświetlenie konfiguracji backupów
az postgres flexible-server show \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --query "{BackupRetention:backup.backupRetentionDays, GeoRedundant:backup.geoRedundantBackup}" \
  --output table

# Zmiana retencji backupów
az postgres flexible-server update \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --backup-retention 14
```

### Point-in-Time Restore (PITR)

```bash
# Przywrócenie do konkretnego momentu w czasie
az postgres flexible-server restore \
  --resource-group MyResourceGroup \
  --name mypostgresserver-restored \
  --source-server mypostgresserver \
  --restore-time "2026-03-05T10:30:00Z"

# Przywrócenie do innej grupy zasobów
az postgres flexible-server restore \
  --resource-group NewResourceGroup \
  --name mypostgresserver-restored \
  --source-server /subscriptions/<sub-id>/resourceGroups/MyResourceGroup/providers/Microsoft.DBforPostgreSQL/flexibleServers/mypostgresserver \
  --restore-time "2026-03-05T10:30:00Z"
```

### Geo-restore

```bash
# Przywrócenie geo-redundantnego backupu w innym regionie
az postgres flexible-server geo-restore \
  --resource-group MyResourceGroup \
  --name mypostgresserver-geo \
  --source-server mypostgresserver \
  --location northeurope
```

---

## Repliki do odczytu (Read Replicas)

```bash
# Tworzenie repliki do odczytu
az postgres flexible-server replica create \
  --resource-group MyResourceGroup \
  --replica-name mypostgres-replica \
  --source-server mypostgresserver

# Tworzenie repliki w innym regionie
az postgres flexible-server replica create \
  --resource-group MyResourceGroup \
  --replica-name mypostgres-replica-eu \
  --source-server mypostgresserver \
  --location northeurope

# Lista replik
az postgres flexible-server replica list \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --output table

# Promowanie repliki na samodzielny serwer (zerwanie replikacji)
az postgres flexible-server replica stop-replication \
  --resource-group MyResourceGroup \
  --name mypostgres-replica
```

---

## Rozszerzenia PostgreSQL (Extensions)

```bash
# Lista dostępnych rozszerzeń
az postgres flexible-server parameter show \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --name azure.extensions

# Włączenie rozszerzenia (na poziomie serwera)
az postgres flexible-server parameter set \
  --resource-group MyResourceGroup \
  --server-name mypostgresserver \
  --name azure.extensions \
  --value "PG_TRGM,BTREE_GIST,UUID-OSSP,POSTGIS"
```

Następnie w psql:
```sql
-- Aktywacja rozszerzenia w konkretnej bazie
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE EXTENSION IF NOT EXISTS postgis;

-- Lista zainstalowanych rozszerzeń
SELECT * FROM pg_extension;
```

---

## Upgrade wersji PostgreSQL

```bash
# Sprawdzenie aktualnej wersji
az postgres flexible-server show \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --query version --output tsv

# Major version upgrade (np. 15 → 16)
az postgres flexible-server update \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --version 16
```

> **Uwaga:** Major version upgrade wymaga przestoju. Zawsze przetestuj na replice lub przywróconym serwerze.

---

## Usuwanie serwera

```bash
# Usunięcie serwera (z potwierdzeniem)
az postgres flexible-server delete \
  --resource-group MyResourceGroup \
  --name mypostgresserver

# Usunięcie bez potwierdzenia
az postgres flexible-server delete \
  --resource-group MyResourceGroup \
  --name mypostgresserver \
  --yes
```

---

## Przykład: pełne wdrożenie od zera

```bash
# 1. Utwórz resource group
az group create --name PostgresRG --location westeurope

# 2. Utwórz serwer PostgreSQL
az postgres flexible-server create \
  --resource-group PostgresRG \
  --name myapp-postgres \
  --location westeurope \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --version 16 \
  --sku-name Standard_B1ms \
  --tier Burstable \
  --storage-size 32 \
  --backup-retention 7 \
  --public-access 0.0.0.0

# 3. Dodaj regułę firewall dla swojego IP
az postgres flexible-server firewall-rule create \
  --resource-group PostgresRG \
  --name myapp-postgres \
  --rule-name AllowMyIP \
  --start-ip-address <my-ip> \
  --end-ip-address <my-ip>

# 4. Utwórz bazę danych
az postgres flexible-server db create \
  --resource-group PostgresRG \
  --server-name myapp-postgres \
  --database-name myappdb

# 5. Włącz potrzebne rozszerzenia
az postgres flexible-server parameter set \
  --resource-group PostgresRG \
  --server-name myapp-postgres \
  --name azure.extensions \
  --value "UUID-OSSP,PG_TRGM"

# 6. Połącz się i zweryfikuj
az postgres flexible-server connect \
  --name myapp-postgres \
  --admin-user pgadmin \
  --admin-password 'MyStr0ngP@ssw0rd!' \
  --database-name myappdb

# 7. Sprawdź connection string
echo "postgresql://pgadmin:MyStr0ngP@ssw0rd!@myapp-postgres.postgres.database.azure.com:5432/myappdb?sslmode=require"
```

---

## Podsumowanie najważniejszych poleceń

| Polecenie | Opis |
|-----------|------|
| `az postgres flexible-server create` | Tworzenie serwera |
| `az postgres flexible-server show` | Szczegóły serwera |
| `az postgres flexible-server list` | Lista serwerów |
| `az postgres flexible-server update` | Aktualizacja serwera |
| `az postgres flexible-server delete` | Usunięcie serwera |
| `az postgres flexible-server start` | Uruchomienie serwera |
| `az postgres flexible-server stop` | Zatrzymanie serwera |
| `az postgres flexible-server restart` | Restart serwera |
| `az postgres flexible-server db create` | Tworzenie bazy danych |
| `az postgres flexible-server db list` | Lista baz danych |
| `az postgres flexible-server db delete` | Usunięcie bazy danych |
| `az postgres flexible-server connect` | Połączenie z serwerem |
| `az postgres flexible-server execute` | Wykonanie zapytania SQL |
| `az postgres flexible-server firewall-rule create` | Dodanie reguły firewall |
| `az postgres flexible-server parameter set` | Zmiana parametru serwera |
| `az postgres flexible-server restore` | Point-in-Time Restore |
| `az postgres flexible-server replica create` | Tworzenie repliki do odczytu |