# Azure Virtual Machines - Maszyny Wirtualne

## Tworzenie maszyny wirtualnej

### Podstawowe tworzenie VM

```bash
# Utworzenie prostej maszyny wirtualnej Linux
az vm create \
  --resource-group <resource-group-name> \
  --name <vm-name> \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys

# Utworzenie maszyny Windows
az vm create \
  --resource-group <resource-group-name> \
  --name <vm-name> \
  --image Win2022Datacenter \
  --admin-username azureuser \
  --admin-password <password>

# Przykład kompletnego polecenia z parametrami
az vm create \
  --resource-group MyResourceGroup \
  --name MyVM \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --location westeurope \
  --public-ip-sku Standard
```

### Tworzenie VM z dodatkowymi opcjami

```bash
# VM z określonym rozmiarem i dyskiem
az vm create \
  --resource-group MyResourceGroup \
  --name MyVM \
  --image Ubuntu2204 \
  --size Standard_D2s_v3 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --os-disk-size-gb 128 \
  --data-disk-sizes-gb 64 128

# VM bez publicznego IP
az vm create \
  --resource-group MyResourceGroup \
  --name MyVM \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-address ""

# VM z konkretną siecią wirtualną
az vm create \
  --resource-group MyResourceGroup \
  --name MyVM \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --vnet-name MyVNet \
  --subnet MySubnet
```

## Zarządzanie maszynami wirtualnymi

### Uruchamianie i zatrzymywanie VM

```bash
# Uruchomienie maszyny wirtualnej
az vm start --resource-group <resource-group-name> --name <vm-name>

# Zatrzymanie maszyny (nadal naliczane opłaty za storage)
az vm stop --resource-group <resource-group-name> --name <vm-name>

# Dealokacja maszyny (brak opłat za compute)
az vm deallocate --resource-group <resource-group-name> --name <vm-name>

# Restart maszyny
az vm restart --resource-group <resource-group-name> --name <vm-name>
```

### Wyświetlanie informacji o VM

```bash
# Lista wszystkich maszyn wirtualnych w subskrypcji
az vm list --output table

# Lista VM w konkretnej grupie zasobów
az vm list --resource-group <resource-group-name> --output table

# Szczegółowe informacje o konkretnej maszynie
az vm show --resource-group <resource-group-name> --name <vm-name>

# Status maszyny wirtualnej
az vm get-instance-view --resource-group <resource-group-name> --name <vm-name>

# Tylko status zasilania
az vm get-instance-view --resource-group <resource-group-name> --name <vm-name> \
  --query instanceView.statuses[1] --output table
```

### Usuwanie maszyny wirtualnej

```bash
# Usunięcie VM (pozostawia dyski i interfejsy sieciowe)
az vm delete --resource-group <resource-group-name> --name <vm-name>

# Usunięcie z potwierdzeniem
az vm delete --resource-group <resource-group-name> --name <vm-name> --yes

# Usunięcie VM wraz z dyskami
az vm delete --resource-group <resource-group-name> --name <vm-name> --yes
az disk delete --resource-group <resource-group-name> --name <disk-name> --yes
```

## Rozmiary maszyn wirtualnych

```bash
# Lista dostępnych rozmiarów w lokalizacji
az vm list-sizes --location westeurope --output table

# Lista rozmiarów dostępnych dla istniejącej VM
az vm list-vm-resize-options --resource-group <resource-group-name> --name <vm-name> --output table

# Zmiana rozmiaru maszyny wirtualnej
az vm resize --resource-group <resource-group-name> --name <vm-name> --size Standard_D4s_v3
```

### Popularne/Przykładowe rozmiary VM

| Seria | Rozmiar | vCPU | RAM |
|-------|---------|------|-----|
| B | Standard_B1s | 1 | 1 GB |
| B | Standard_B2s | 2 | 4 GB |
| D | Standard_D2s_v3 | 2 | 8 GB |
| D | Standard_D4s_v3 | 4 | 16 GB |
| E | Standard_E2s_v3 | 2 | 16 GB |
| F | Standard_F2s_v2 | 2 | 4 GB |

## Obrazy maszyn wirtualnych

```bash
# Lista popularnych obrazów
az vm image list --output table

# Wyszukiwanie obrazów (np. Ubuntu)
az vm image list --offer Ubuntu --all --output table

# Szczegółowe informacje o obrazie
az vm image show --location westeurope --urn Canonical:0001-com-ubuntu-server-jammy:22_04-lts:latest

# Lista wydawców obrazów
az vm image list-publishers --location westeurope --output table

# Lista ofert wydawcy
az vm image list-offers --location westeurope --publisher Canonical --output table

# Lista SKU dla oferty
az vm image list-skus --location westeurope --publisher Canonical --offer 0001-com-ubuntu-server-jammy --output table
```

### Popularne obrazy

```bash
# Ubuntu 22.04 LTS
--image Ubuntu2204
--image Canonical:0001-com-ubuntu-server-jammy:22_04-lts:latest

# Windows Server 2022
--image Win2022Datacenter
--image MicrosoftWindowsServer:WindowsServer:2022-datacenter:latest

# Debian 11
--image Debian11

# Red Hat Enterprise Linux 9
--image RedHat:RHEL:9-lvm:latest
```

## Dostęp do maszyn wirtualnych

### SSH dla Linux

```bash
# Połączenie SSH (jeśli VM ma publiczny IP)
az vm show --resource-group <resource-group-name> --name <vm-name> -d --query publicIps -o tsv
ssh azureuser@<public-ip>

# Otwarcie portu SSH
az vm open-port --resource-group <resource-group-name> --name <vm-name> --port 22

# Użycie Azure Bastion (bezpieczne połączenie bez publicznego IP)
az network bastion ssh --name <bastion-name> --resource-group <resource-group-name> \
  --target-resource-id <vm-resource-id> --auth-type ssh-key --username azureuser
```

### RDP dla Windows

```bash
# Otwarcie portu RDP
az vm open-port --resource-group <resource-group-name> --name <vm-name> --port 3389

# Pobranie publicznego IP
az vm show --resource-group <resource-group-name> --name <vm-name> -d --query publicIps -o tsv

# Następnie użyj Remote Desktop Connection: mstsc /v:<public-ip>
```

## Dyski

### Zarządzanie dyskami

```bash
# Lista dysków w grupie zasobów
az disk list --resource-group <resource-group-name> --output table

# Utworzenie nowego dysku
az disk create \
  --resource-group <resource-group-name> \
  --name <disk-name> \
  --size-gb 128 \
  --sku Standard_LRS

# Dołączenie dysku do VM
az vm disk attach \
  --resource-group <resource-group-name> \
  --vm-name <vm-name> \
  --name <disk-name>

# Odłączenie dysku
az vm disk detach \
  --resource-group <resource-group-name> \
  --vm-name <vm-name> \
  --name <disk-name>

# Usunięcie dysku
az disk delete --resource-group <resource-group-name> --name <disk-name>
```

### Typy dysków

- **Standard HDD** (`Standard_LRS`) - najtańszy, dla rzadko używanych danych
- **Standard SSD** (`StandardSSD_LRS`) - lepsza wydajność niż HDD
- **Premium SSD** (`Premium_LRS`) - wysoka wydajność dla aplikacji produkcyjnych
- **Ultra Disk** - najwyższa wydajność dla najbardziej wymagających aplikacji

## Rozszerzenia VM

```bash
# Instalacja rozszerzenia Custom Script
az vm extension set \
  --resource-group <resource-group-name> \
  --vm-name <vm-name> \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"fileUris": ["https://example.com/script.sh"],"commandToExecute": "sh script.sh"}'

# Lista rozszerzeń na VM
az vm extension list --resource-group <resource-group-name> --vm-name <vm-name> --output table

# Usunięcie rozszerzenia
az vm extension delete \
  --resource-group <resource-group-name> \
  --vm-name <vm-name> \
  --name customScript
```

## Snapshoty i obrazy

```bash
# Utworzenie snapshota dysku OS
az snapshot create \
  --resource-group <resource-group-name> \
  --name <snapshot-name> \
  --source <disk-id>

# Utworzenie obrazu z VM
az image create \
  --resource-group <resource-group-name> \
  --name <image-name> \
  --source <vm-name>

# Utworzenie VM z obrazu
az vm create \
  --resource-group <resource-group-name> \
  --name <new-vm-name> \
  --image <image-name> \
  --admin-username azureuser \
  --generate-ssh-keys
```

## Grupy dostępności i skalowania

```bash
# Utworzenie grupy dostępności (Availability Set)
az vm availability-set create \
  --resource-group <resource-group-name> \
  --name <availability-set-name> \
  --platform-fault-domain-count 2 \
  --platform-update-domain-count 5

# Utworzenie VM w grupie dostępności
az vm create \
  --resource-group <resource-group-name> \
  --name <vm-name> \
  --image Ubuntu2204 \
  --availability-set <availability-set-name> \
  --admin-username azureuser \
  --generate-ssh-keys
```

## Monitorowanie i diagnostyka

```bash
# Włączenie diagnostyki rozruchu
az vm boot-diagnostics enable \
  --resource-group <resource-group-name> \
  --name <vm-name>

# Pobranie logu rozruchu
az vm boot-diagnostics get-boot-log \
  --resource-group <resource-group-name> \
  --name <vm-name>

# Wyświetlenie metryk VM
az monitor metrics list \
  --resource <vm-resource-id> \
  --metric-names "Percentage CPU" \
  --start-time 2026-02-13T00:00:00Z \
  --end-time 2026-02-13T23:59:59Z
```

## Tagi i metadata

```bash
# Dodanie tagów do VM
az vm update \
  --resource-group <resource-group-name> \
  --name <vm-name> \
  --set tags.Environment=Production tags.Owner=IT

# Wyświetlenie tagów
az vm show \
  --resource-group <resource-group-name> \
  --name <vm-name> \
  --query tags
```

## Przydatne wskazówki

- Zawsze używaj `--generate-ssh-keys` dla Linux VM, aby automatycznie wygenerować klucze SSH
- Użyj `az vm deallocate` zamiast `az vm stop` aby nie płacić za compute
- Rozważ użycie Managed Disks (domyślnie) zamiast Unmanaged Disks
- Dla środowisk produkcyjnych używaj Premium SSD lub wyższej klasy dysków
- Zawsze twórz snapshoty przed ważnymi zmianami
- Używaj grup dostępności lub stref dostępności dla aplikacji krytycznych
- Regularnie aktualizuj system operacyjny i aplikacje na VM
