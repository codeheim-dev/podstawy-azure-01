# Storage Account

## Podstawowe informacje

Storage Account to usługa Azure do przechowywania danych w chmurze. Oferuje różne typy pamięci masowej:
- **Blob Storage** - przechowywanie obiektów binarnych (pliki, obrazy, filmy)
- **File Storage** - udziały plików SMB
- **Queue Storage** - kolejki komunikatów
- **Table Storage** - dane NoSQL

## Azure CLI - Podstawowe komendy

### Tworzenie Storage Account

```bash
# Tworzenie Storage Account
az storage account create \
  --name <nazwa-storage-account> \
  --resource-group <nazwa-grupy-zasobów> \
  --location <region> \
  --sku Standard_LRS \
  --kind StorageV2

# Przykład
az storage account create \
  --name mystorageaccount123 \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2
```

**Dostępne SKU:**
- `Standard_LRS` - Locally Redundant Storage
- `Standard_GRS` - Geo-Redundant Storage
- `Standard_RAGRS` - Read-Access Geo-Redundant Storage
- `Standard_ZRS` - Zone-Redundant Storage
- `Premium_LRS` - Premium Locally Redundant Storage

### Zarządzanie Storage Account

```bash
# Listowanie Storage Accounts
az storage account list --resource-group <nazwa-grupy-zasobów>
az storage account list --output table

# Szczegóły Storage Account
az storage account show \
  --name <nazwa-storage-account> \
  --resource-group <nazwa-grupy-zasobów>

# Pobranie kluczy dostępu
az storage account keys list \
  --account-name <nazwa-storage-account> \
  --resource-group <nazwa-grupy-zasobów>

# Pobranie connection string
az storage account show-connection-string \
  --name <nazwa-storage-account> \
  --resource-group <nazwa-grupy-zasobów>

# Aktualizacja Storage Account
az storage account update \
  --name <nazwa-storage-account> \
  --resource-group <nazwa-grupy-zasobów> \
  --sku Standard_GRS

# Usunięcie Storage Account
az storage account delete \
  --name <nazwa-storage-account> \
  --resource-group <nazwa-grupy-zasobów>
```

### Zarządzanie kontenerami (Blob Storage)

```bash
# Tworzenie kontenera
az storage container create \
  --name <nazwa-kontenera> \
  --account-name <nazwa-storage-account> \
  --account-key <klucz-dostępu>

# Lub z użyciem connection string
az storage container create \
  --name <nazwa-kontenera> \
  --connection-string "<connection-string>"

# Listowanie kontenerów
az storage container list \
  --account-name <nazwa-storage-account> \
  --account-key <klucz-dostępu>

# Usunięcie kontenera
az storage container delete \
  --name <nazwa-kontenera> \
  --account-name <nazwa-storage-account>
```

### Operacje na plikach (Blob)

```bash
# Upload pliku do kontenera
az storage blob upload \
  --container-name <nazwa-kontenera> \
  --file <ścieżka-do-pliku> \
  --name <nazwa-bloba> \
  --account-name <nazwa-storage-account>

# Przykład
az storage blob upload \
  --container-name mycontainer \
  --file ./local-file.txt \
  --name remote-file.txt \
  --account-name mystorageaccount123

# Listowanie blobów w kontenerze
az storage blob list \
  --container-name <nazwa-kontenera> \
  --account-name <nazwa-storage-account> \
  --output table

# Pobieranie pliku
az storage blob download \
  --container-name <nazwa-kontenera> \
  --name <nazwa-bloba> \
  --file <ścieżka-lokalna> \
  --account-name <nazwa-storage-account>

# Usunięcie bloba
az storage blob delete \
  --container-name <nazwa-kontenera> \
  --name <nazwa-bloba> \
  --account-name <nazwa-storage-account>

# Kopiowanie bloba
az storage blob copy start \
  --source-container <źródłowy-kontener> \
  --source-blob <źródłowy-blob> \
  --destination-container <docelowy-kontener> \
  --destination-blob <docelowy-blob> \
  --account-name <nazwa-storage-account>
```

### Zarządzanie uprawnieniami (SAS Token)

```bash
# Generowanie SAS token dla kontenera
az storage container generate-sas \
  --name <nazwa-kontenera> \
  --account-name <nazwa-storage-account> \
  --account-key <klucz-dostępu> \
  --permissions rwdl \
  --expiry 2026-12-31T23:59:59Z

# Generowanie SAS token dla bloba
az storage blob generate-sas \
  --container-name <nazwa-kontenera> \
  --name <nazwa-bloba> \
  --account-name <nazwa-storage-account> \
  --account-key <klucz-dostępu> \
  --permissions r \
  --expiry 2026-12-31T23:59:59Z
```

**Uprawnienia SAS:**
- `r` - read (odczyt)
- `w` - write (zapis)
- `d` - delete (usuwanie)
- `l` - list (listowanie)

### File Share (Azure Files)

```bash
# Tworzenie file share
az storage share create \
  --name <nazwa-share> \
  --account-name <nazwa-storage-account> \
  --quota 5

# Listowanie file shares
az storage share list \
  --account-name <nazwa-storage-account> \
  --output table

# Upload pliku do file share
az storage file upload \
  --share-name <nazwa-share> \
  --source <ścieżka-do-pliku> \
  --account-name <nazwa-storage-account>

# Usunięcie file share
az storage share delete \
  --name <nazwa-share> \
  --account-name <nazwa-storage-account>
```