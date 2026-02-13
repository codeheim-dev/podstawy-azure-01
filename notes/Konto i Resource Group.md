# Azure CLI - Podstawowe polecenia

## Instalacja i weryfikacja

```bash
# Sprawdzenie wersji Azure CLI
az --version

# Aktualizacja Azure CLI
az upgrade
```

## Logowanie

```bash
# Logowanie interaktywne (otwiera przeglądarkę)
az login

# Logowanie z konkretnym użytkownikiem
az login -u <email>

# Logowanie z Service Principal
az login --service-principal -u <app-id> -p <password-or-cert> --tenant <tenant-id>

# Wyświetlenie aktualnie zalogowanego użytkownika
az account show

# Wylogowanie
az logout
```

## Zarządzanie subskrypcjami

```bash
# Lista wszystkich subskrypcji
az account list --output table

# Wyświetlenie aktywnej subskrypcji
az account show

# Ustawienie domyślnej subskrypcji
az account set --subscription <subscription-id>
az account set --subscription "<subscription-name>"

# Wyświetlenie szczegółów subskrypcji w formacie JSON
az account show --output json
```

## Resource Groups

### Tworzenie Resource Group

```bash
# Utworzenie nowej grupy zasobów
az group create --name <resource-group-name> --location <location>

# Przykład
az group create --name MyResourceGroup --location westeurope
```

### Wyświetlanie Resource Groups

```bash
# Lista wszystkich grup zasobów
az group list --output table

# Szczegóły konkretnej grupy zasobów
az group show --name <resource-group-name>

# Lista grup zasobów w formacie JSON
az group list --output json
```

### Aktualizacja Resource Group

```bash
# Dodanie tagów do grupy zasobów
az group update --name <resource-group-name> --tags Environment=Dev Project=Demo

# Aktualizacja tagów
az group update --name <resource-group-name> --set tags.Owner=Admin
```

### Usuwanie Resource Group

```bash
# Usunięcie grupy zasobów (usuwa wszystkie zasoby w grupie)
az group delete --name <resource-group-name>

# Usunięcie bez potwierdzenia
az group delete --name <resource-group-name> --yes --no-wait
```

### Dodatkowe operacje na Resource Groups

```bash
# Lista zasobów w grupie zasobów
az resource list --resource-group <resource-group-name> --output table

# Eksport template grupy zasobów
az group export --name <resource-group-name>

# Sprawdzenie, czy grupa zasobów istnieje
az group exists --name <resource-group-name>
```

## Popularne lokalizacje w Azure

```bash
# Lista wszystkich dostępnych lokalizacji
az account list-locations --output table

# Popularne lokalizacje:
# - westeurope (Europa Zachodnia)
# - northeurope (Europa Północna)
# - eastus (US East)
# - westus (US West)
```

## Formaty wyjścia

```bash
# Table (tabela - czytelny format)
az <command> --output table

# JSON (domyślny format)
az <command> --output json

# YAML
az <command> --output yaml

# TSV (Tab-Separated Values)
az <command> --output tsv

# Ustawienie domyślnego formatu wyjścia
az configure --defaults output=table
```

## Przydatne wskazówki

- Użyj `--help` z dowolnym poleceniem, aby zobaczyć szczegółową pomoc
- Użyj `--debug` aby zobaczyć szczegółowe informacje o wykonywaniu polecenia
- Użyj `--verbose` dla dodatkowych informacji podczas wykonywania
- Użyj `--query` z JMESPath do filtrowania wyników JSON

```bash
# Przykład użycia query
az group list --query "[?location=='westeurope']" --output table

# Wyświetlenie tylko nazw grup zasobów
az group list --query "[].name" --output tsv
```
