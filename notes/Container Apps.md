# Azure Container Apps - Polecenia Azure CLI

## Czym jest Azure Container Apps?

Azure Container Apps to w pełni zarządzana usługa serverless do uruchamiania konteneryzowanych aplikacji. Oparta jest na Kubernetes (KEDA, Dapr, Envoy), ale nie wymaga bezpośredniego zarządzania klastrem. Idealna do mikrousług, API, aplikacji webowych i zadań sterowanych zdarzeniami (event-driven).

**Kluczowe cechy:**
- Automatyczne skalowanie (w tym do zera replik)
- Wbudowane HTTPS i zarządzanie certyfikatami TLS
- Traffic splitting między rewizjami (blue-green, canary)
- Wbudowana obsługa Dapr (Distributed Application Runtime)
- Zarządzanie sekretami
- Rewizje i wersjonowanie aplikacji
- Różne typy wyzwalaczy skalowania (HTTP, CPU, kolejki, CRON, custom)
- Consumption i Dedicated plan

---

## Wymagania wstępne

### Instalacja rozszerzenia i rejestracja dostawców

```bash
# Instalacja/aktualizacja rozszerzenia Container Apps
az extension add --name containerapp --upgrade

# Rejestracja wymaganych dostawców zasobów
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights

# Sprawdzenie statusu rejestracji
az provider show --namespace Microsoft.App --query "registrationState" --output tsv
az provider show --namespace Microsoft.OperationalInsights --query "registrationState" --output tsv
```

---

## Środowisko (Container Apps Environment)

Środowisko to logiczna granica, w której działają aplikacje kontenerowe. Aplikacje w tym samym środowisku współdzielą sieć wirtualną i workspace Log Analytics.

### Tworzenie środowiska

```bash
# Podstawowe środowisko
az containerapp env create \
  --name <environment-name> \
  --resource-group <resource-group-name> \
  --location westeurope

# Środowisko z istniejącym Log Analytics workspace
az containerapp env create \
  --name mycontainerenv \
  --resource-group MyResourceGroup \
  --location westeurope \
  --logs-workspace-id <log-analytics-workspace-id> \
  --logs-workspace-key <log-analytics-workspace-key>
```

### Środowisko z siecią wirtualną (VNet)

```bash
# Tworzenie środowiska z istniejącym VNet (internal - brak dostępu z internetu)
az containerapp env create \
  --name mycontainerenv \
  --resource-group MyResourceGroup \
  --location westeurope \
  --infrastructure-subnet-resource-id <subnet-resource-id> \
  --internal-only true

# Środowisko z VNet i dostępem zewnętrznym
az containerapp env create \
  --name mycontainerenv \
  --resource-group MyResourceGroup \
  --location westeurope \
  --infrastructure-subnet-resource-id <subnet-resource-id> \
  --internal-only false
```

> **Uwaga:** Subnet musi mieć minimalnie /23 CIDR range i być delegowany do `Microsoft.App/environments`.

### Zarządzanie środowiskiem

```bash
# Lista środowisk
az containerapp env list --output table

# Lista w grupie zasobów
az containerapp env list --resource-group <resource-group-name> --output table

# Szczegóły środowiska
az containerapp env show \
  --name <environment-name> \
  --resource-group <resource-group-name>

# Sprawdzenie domyślnej domeny środowiska
az containerapp env show \
  --name <environment-name> \
  --resource-group <resource-group-name> \
  --query properties.defaultDomain --output tsv

# Usunięcie środowiska
az containerapp env delete \
  --name <environment-name> \
  --resource-group <resource-group-name> \
  --yes
```

---

## Tworzenie Container App

### Podstawowe tworzenie

```bash
# Najprostsza aplikacja
az containerapp create \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --environment <environment-name> \
  --image <image-name> \
  --target-port 80 \
  --ingress external

# Przykład z nginx
az containerapp create \
  --name mynginxapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image nginx:latest \
  --target-port 80 \
  --ingress external \
  --query properties.configuration.ingress.fqdn
```

### Określenie zasobów

```bash
# Aplikacja z CPU, pamięcią i replikami
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage:latest \
  --target-port 8080 \
  --ingress external \
  --cpu 0.5 \
  --memory 1.0Gi \
  --min-replicas 1 \
  --max-replicas 10
```

> **Dostępne kombinacje CPU/RAM (Consumption plan):**
> | CPU (cores) | Pamięć (Gi) |
> |-------------|-------------|
> | 0.25 | 0.5 |
> | 0.5 | 1.0 |
> | 0.75 | 1.5 |
> | 1.0 | 2.0 |
> | 1.25 | 2.5 |
> | 1.5 | 3.0 |
> | 1.75 | 3.5 |
> | 2.0 | 4.0 |

---

## Ingress (ruch przychodzący)

Ingress kontroluje, w jaki sposób ruch trafia do aplikacji.

### Typy ingress

```bash
# External - dostęp z internetu i z wewnątrz środowiska
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage \
  --target-port 80 \
  --ingress external

# Internal - dostęp tylko z wewnątrz środowiska
az containerapp create \
  --name mybackend \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image mybackend:latest \
  --target-port 8080 \
  --ingress internal
```

### Zarządzanie ingress

```bash
# Włączenie ingress na istniejącej aplikacji
az containerapp ingress enable \
  --name myapp \
  --resource-group MyResourceGroup \
  --type external \
  --target-port 80 \
  --transport auto

# Wyłączenie ingress
az containerapp ingress disable \
  --name myapp \
  --resource-group MyResourceGroup

# Wyświetlenie konfiguracji ingress
az containerapp ingress show \
  --name myapp \
  --resource-group MyResourceGroup
```

### Transport protocol

```bash
# Auto (HTTP/1.1 i HTTP/2)
az containerapp ingress enable \
  --name myapp \
  --resource-group MyResourceGroup \
  --type external \
  --target-port 80 \
  --transport auto

# HTTP/2 only
az containerapp ingress enable \
  --name myapp \
  --resource-group MyResourceGroup \
  --type external \
  --target-port 80 \
  --transport http2

# TCP (dla non-HTTP workloads)
az containerapp ingress enable \
  --name myapp \
  --resource-group MyResourceGroup \
  --type external \
  --target-port 9000 \
  --transport tcp \
  --exposed-port 9000
```

### CORS (Cross-Origin Resource Sharing)

```bash
# Konfiguracja CORS
az containerapp ingress cors enable \
  --name myapp \
  --resource-group MyResourceGroup \
  --allowed-origins "https://example.com" "https://www.example.com" \
  --allowed-methods "GET" "POST" "PUT" \
  --allowed-headers "Content-Type" "Authorization" \
  --max-age 3600

# Wyłączenie CORS
az containerapp ingress cors disable \
  --name myapp \
  --resource-group MyResourceGroup
```

---

## Rejestr kontenerów (Container Registry)

### Azure Container Registry (ACR)

```bash
# Tworzenie z poświadczeniami
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-server myregistry.azurecr.io \
  --registry-username <username> \
  --registry-password <password> \
  --target-port 80 \
  --ingress external

# Z system-assigned managed identity (zalecane)
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-identity system \
  --target-port 80 \
  --ingress external

# Z user-assigned managed identity
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-identity <managed-identity-resource-id> \
  --target-port 80 \
  --ingress external
```

### Prywatny rejestr Docker

```bash
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image registry.example.com/myapp:v1 \
  --registry-server registry.example.com \
  --registry-username <username> \
  --registry-password <password> \
  --target-port 80 \
  --ingress external
```

---

## Zmienne środowiskowe i sekrety

### Zmienne środowiskowe

```bash
# Tworzenie z env vars
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage \
  --env-vars "API_URL=https://api.example.com" "LOG_LEVEL=info" "NODE_ENV=production" \
  --target-port 80 \
  --ingress external

# Aktualizacja zmiennych środowiskowych
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --set-env-vars "NEW_VAR=value" "LOG_LEVEL=debug"

# Usunięcie zmiennych środowiskowych
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --remove-env-vars "OLD_VAR" "DEPRECATED_VAR"
```

### Sekrety

Sekrety są przechowywane na poziomie aplikacji i mogą być używane przez zmienne środowiskowe lub montowane jako wolumeny.

```bash
# Tworzenie z sekretami
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage \
  --secrets "db-password=SuperSecret123" "api-key=MyApiKey456" \
  --env-vars "DB_PASSWORD=secretref:db-password" "API_KEY=secretref:api-key" \
  --target-port 80 \
  --ingress external
```

### Zarządzanie sekretami

```bash
# Lista sekretów
az containerapp secret list \
  --name myapp \
  --resource-group MyResourceGroup \
  --output table

# Wyświetlenie wartości sekretu
az containerapp secret show \
  --name myapp \
  --resource-group MyResourceGroup \
  --secret-name db-password

# Dodanie / aktualizacja sekretu
az containerapp secret set \
  --name myapp \
  --resource-group MyResourceGroup \
  --secrets "new-secret=NewValue123"

# Usunięcie sekretu
az containerapp secret remove \
  --name myapp \
  --resource-group MyResourceGroup \
  --secret-names "old-secret"
```

### Sekrety z Key Vault

```bash
# Sekret odwołujący się do Azure Key Vault
az containerapp secret set \
  --name myapp \
  --resource-group MyResourceGroup \
  --secrets "kv-secret=keyvaultref:<key-vault-secret-uri>,identityref:<managed-identity-resource-id>"
```

---

## Skalowanie (Autoscaling)

Container Apps wspiera automatyczne skalowanie na podstawie różnych wyzwalaczy. Oparte na KEDA (Kubernetes Event-Driven Autoscaling).

### Skalowanie HTTP

```bash
# Skalowanie na podstawie liczby równoczesnych żądań HTTP
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage \
  --target-port 80 \
  --ingress external \
  --min-replicas 0 \
  --max-replicas 10 \
  --scale-rule-name http-scale \
  --scale-rule-type http \
  --scale-rule-http-concurrency 100
```

> **Uwaga:** Skalowanie do zera (`--min-replicas 0`) oznacza, że przy braku ruchu nie ma żadnych instancji (cold start przy pierwszym żądaniu).

### Skalowanie CPU/Memory

```bash
# Na podstawie użycia CPU
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --scale-rule-name cpu-scale \
  --scale-rule-type cpu \
  --scale-rule-metadata "type=Utilization" "value=75"

# Na podstawie użycia pamięci
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --scale-rule-name memory-scale \
  --scale-rule-type memory \
  --scale-rule-metadata "type=Utilization" "value=70"
```

### Skalowanie na podstawie kolejki (Azure Queue)

```bash
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --scale-rule-name queue-scale \
  --scale-rule-type azure-queue \
  --scale-rule-metadata "queueName=myqueue" "queueLength=5" \
  --scale-rule-auth "connection=connection-string-secret"
```

### Skalowanie na podstawie Azure Service Bus

```bash
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --scale-rule-name servicebus-scale \
  --scale-rule-type azure-servicebus \
  --scale-rule-metadata "queueName=myqueue" "messageCount=10" \
  --scale-rule-auth "connection=sb-connection-secret"
```

### Manualne skalowanie

```bash
# Ustawienie stałej liczby replik
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --min-replicas 3 \
  --max-replicas 3
```

---

## Rewizje (Revisions)

Rewizja to niezmienna (immutable) wersja aplikacji. Każda zmiana konfiguracji kontenera tworzy nową rewizję.

### Tryby rewizji

```bash
# Single revision mode (domyślny) - tylko jedna aktywna rewizja
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage:v1 \
  --target-port 80 \
  --ingress external \
  --revision-suffix v1

# Multiple revision mode - wiele aktywnych rewizji jednocześnie
az containerapp revision set-mode \
  --name myapp \
  --resource-group MyResourceGroup \
  --mode multiple
```

### Zarządzanie rewizjami

```bash
# Lista wszystkich rewizji
az containerapp revision list \
  --name myapp \
  --resource-group MyResourceGroup \
  --output table

# Szczegóły rewizji
az containerapp revision show \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision <revision-name>

# Aktywacja rewizji
az containerapp revision activate \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision <revision-name>

# Dezaktywacja rewizji
az containerapp revision deactivate \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision <revision-name>

# Restart rewizji
az containerapp revision restart \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision <revision-name>

# Kopiowanie rewizji (tworzenie nowej z kopii)
az containerapp revision copy \
  --name myapp \
  --resource-group MyResourceGroup \
  --image myimage:v2 \
  --revision-suffix v2
```

### Nadawanie nazw rewizji

```bash
# Tworzenie z sufiksem rewizji
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage:v1 \
  --target-port 80 \
  --ingress external \
  --revision-suffix release-v1

# Aktualizacja z nowym sufiksem
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --image myimage:v2 \
  --revision-suffix release-v2
```

> **Format nazwy rewizji:** `<app-name>--<suffix>`, np. `myapp--release-v1`

---

## Traffic Splitting

Podział ruchu między rewizjami — kluczowy mechanizm dla strategii wdrożeniowych.

### Konfiguracja podziału ruchu

```bash
# Podział 80/20 między dwie rewizje
az containerapp ingress traffic set \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision-weight myapp--v1=80 myapp--v2=20

# 100% ruchu na najnowszą rewizję
az containerapp ingress traffic set \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision-weight latest=100

# Wyświetlenie aktualnego podziału ruchu
az containerapp ingress traffic show \
  --name myapp \
  --resource-group MyResourceGroup
```

### Strategie wdrożeniowe

```bash
# Blue-Green: przełączenie całego ruchu na nową rewizję
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --image myimage:v2 \
  --revision-suffix green

az containerapp ingress traffic set \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision-weight myapp--green=100

# Canary: stopniowe zwiększanie ruchu
az containerapp ingress traffic set \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision-weight myapp--v1=90 myapp--v2=10

# Zwiększenie do 50/50
az containerapp ingress traffic set \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision-weight myapp--v1=50 myapp--v2=50

# Pełne przełączenie
az containerapp ingress traffic set \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision-weight myapp--v2=100
```

### Etykiety rewizji (Labels)

Etykiety pozwalają odwoływać się do rewizji przez nazwy logiczne i testować je pod dedykowanym URL.

```bash
# Przypisanie etykiety do rewizji
az containerapp revision label add \
  --name myapp \
  --resource-group MyResourceGroup \
  --label staging \
  --revision myapp--v2

# Ruch do etykiety: myapp---staging.<default-domain>

# Usunięcie etykiety
az containerapp revision label remove \
  --name myapp \
  --resource-group MyResourceGroup \
  --label staging

# Podział ruchu z użyciem etykiet
az containerapp ingress traffic set \
  --name myapp \
  --resource-group MyResourceGroup \
  --label-weight production=80 staging=20
```

---

## Aktualizacja Container App

### Aktualizacja obrazu

```bash
# Zmiana obrazu (tworzy nową rewizję)
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --image myregistry.azurecr.io/myapp:v2

# Zmiana obrazu z nowym sufiksem rewizji
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --image myregistry.azurecr.io/myapp:v3 \
  --revision-suffix v3
```

### Aktualizacja zasobów i replik

```bash
# Zmiana limitów CPU i pamięci
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --cpu 1.0 \
  --memory 2.0Gi

# Zmiana limitów skalowania
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --min-replicas 2 \
  --max-replicas 20
```

---

## Dapr (Distributed Application Runtime)

Dapr zapewnia budulce (building blocks) do budowania mikrousług: service invocation, state management, pub/sub i inne.

### Włączenie Dapr

```bash
# Tworzenie aplikacji z Dapr
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage \
  --target-port 80 \
  --ingress internal \
  --dapr-enabled true \
  --dapr-app-id myapp \
  --dapr-app-port 80 \
  --dapr-app-protocol http

# Aktualizacja konfiguracji Dapr
az containerapp dapr enable \
  --name myapp \
  --resource-group MyResourceGroup \
  --dapr-app-id myapp \
  --dapr-app-port 80 \
  --dapr-app-protocol grpc

# Wyłączenie Dapr
az containerapp dapr disable \
  --name myapp \
  --resource-group MyResourceGroup
```

### Komponenty Dapr

```bash
# Lista komponentów Dapr w środowisku
az containerapp env dapr-component list \
  --name mycontainerenv \
  --resource-group MyResourceGroup \
  --output table

# Tworzenie komponentu z pliku YAML
az containerapp env dapr-component set \
  --name mycontainerenv \
  --resource-group MyResourceGroup \
  --dapr-component-name statestore \
  --yaml statestore.yaml

# Usunięcie komponentu
az containerapp env dapr-component remove \
  --name mycontainerenv \
  --resource-group MyResourceGroup \
  --dapr-component-name statestore
```

---

## Container Apps Jobs

Jobs to zadania, które uruchamiają się na żądanie, wg harmonogramu lub wyzwalane zdarzeniami — w odróżnieniu od zwykłych aplikacji, nie działają ciągle.

### Typy jobów

```bash
# Manual job — uruchamiany ręcznie
az containerapp job create \
  --name myjob \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage:latest \
  --trigger-type Manual \
  --replica-timeout 300 \
  --replica-retry-limit 1 \
  --cpu 0.5 \
  --memory 1.0Gi

# Schedule job — cron schedule
az containerapp job create \
  --name myschedulejob \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage:latest \
  --trigger-type Schedule \
  --cron-expression "0 */6 * * *" \
  --replica-timeout 600 \
  --cpu 0.25 \
  --memory 0.5Gi

# Event-driven job — wyzwalane zdarzeniami (np. kolejka)
az containerapp job create \
  --name myeventjob \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage:latest \
  --trigger-type Event \
  --min-executions 0 \
  --max-executions 5 \
  --scale-rule-name queue-trigger \
  --scale-rule-type azure-queue \
  --scale-rule-metadata "queueName=jobs" "queueLength=1" \
  --scale-rule-auth "connection=queue-connection-secret" \
  --secrets "queue-connection-secret=<connection-string>" \
  --cpu 0.5 \
  --memory 1.0Gi
```

### Zarządzanie jobami

```bash
# Ręczne uruchomienie joba
az containerapp job start \
  --name myjob \
  --resource-group MyResourceGroup

# Lista wykonań
az containerapp job execution list \
  --name myjob \
  --resource-group MyResourceGroup \
  --output table

# Szczegóły joba
az containerapp job show \
  --name myjob \
  --resource-group MyResourceGroup

# Usunięcie joba
az containerapp job delete \
  --name myjob \
  --resource-group MyResourceGroup \
  --yes
```

---

## Custom Domains i certyfikaty

### Dodawanie domeny niestandardowej

```bash
# Dodanie custom domain
az containerapp hostname add \
  --name myapp \
  --resource-group MyResourceGroup \
  --hostname myapp.example.com

# Lista hostnames
az containerapp hostname list \
  --name myapp \
  --resource-group MyResourceGroup \
  --output table

# Usunięcie hostname
az containerapp hostname delete \
  --name myapp \
  --resource-group MyResourceGroup \
  --hostname myapp.example.com --yes
```

### Certyfikaty SSL/TLS

```bash
# Upload certyfikatu do środowiska
az containerapp env certificate upload \
  --name mycontainerenv \
  --resource-group MyResourceGroup \
  --certificate-file ./mycert.pfx \
  --password <pfx-password>

# Lista certyfikatów
az containerapp env certificate list \
  --name mycontainerenv \
  --resource-group MyResourceGroup \
  --output table

# Powiązanie certyfikatu z hostname
az containerapp hostname bind \
  --name myapp \
  --resource-group MyResourceGroup \
  --hostname myapp.example.com \
  --environment mycontainerenv \
  --certificate <certificate-id>

# Managed certificate (automatyczny certyfikat)
az containerapp hostname bind \
  --name myapp \
  --resource-group MyResourceGroup \
  --hostname myapp.example.com \
  --environment mycontainerenv \
  --validation-method CNAME
```

---

## Zarządzanie Container Apps

### Wyświetlanie informacji

```bash
# Lista wszystkich aplikacji w subskrypcji
az containerapp list --output table

# Lista w grupie zasobów
az containerapp list --resource-group <resource-group-name> --output table

# Szczegóły aplikacji
az containerapp show \
  --name <app-name> \
  --resource-group <resource-group-name>

# URL aplikacji (FQDN)
az containerapp show \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --query properties.configuration.ingress.fqdn --output tsv

# Pobranie outbound IP
az containerapp show \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --query "properties.outboundIpAddresses" --output tsv
```

### Logi

```bash
# Logi systemowe
az containerapp logs show \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --type system

# Logi konsoli (stdout/stderr)
az containerapp logs show \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --type console

# Śledzenie logów w czasie rzeczywistym
az containerapp logs show \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --type console \
  --follow

# Logi konkretnej rewizji
az containerapp logs show \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --revision <revision-name> \
  --type console

# Logi konkretnej repliki
az containerapp logs show \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --replica <replica-name> \
  --type console
```

### Repliki i instancje

```bash
# Lista replik
az containerapp replica list \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --revision <revision-name> \
  --output table

# Liczba replik
az containerapp replica count \
  --name <app-name> \
  --resource-group <resource-group-name>
```

### Diagnostyka

```bash
# Exec do kontenera (interaktywna sesja)
az containerapp exec \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --command "/bin/sh"

# Debug console
az containerapp debug \
  --name <app-name> \
  --resource-group <resource-group-name>
```

---

## Managed Identity

```bash
# Przypisanie system-assigned managed identity
az containerapp identity assign \
  --name myapp \
  --resource-group MyResourceGroup \
  --system-assigned

# Przypisanie user-assigned managed identity
az containerapp identity assign \
  --name myapp \
  --resource-group MyResourceGroup \
  --user-assigned <identity-resource-id>

# Wyświetlenie przypisanych tożsamości
az containerapp identity show \
  --name myapp \
  --resource-group MyResourceGroup

# Usunięcie system-assigned identity
az containerapp identity remove \
  --name myapp \
  --resource-group MyResourceGroup \
  --system-assigned

# Usunięcie user-assigned identity
az containerapp identity remove \
  --name myapp \
  --resource-group MyResourceGroup \
  --user-assigned <identity-resource-id>
```

---

## Usuwanie zasobów

```bash
# Usunięcie Container App
az containerapp delete \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --yes

# Usunięcie środowiska (usunie wszystkie aplikacje w nim)
az containerapp env delete \
  --name <environment-name> \
  --resource-group <resource-group-name> \
  --yes
```

---

## Wdrożenie z YAML

### Przykładowy plik `containerapp.yaml`

```yaml
properties:
  managedEnvironmentId: /subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.App/managedEnvironments/<env-name>
  configuration:
    activeRevisionsMode: Multiple
    ingress:
      external: true
      targetPort: 80
      transport: auto
      traffic:
        - revisionName: myapp--v1
          weight: 80
        - revisionName: myapp--v2
          weight: 20
    secrets:
      - name: db-password
        value: SuperSecret123
    registries:
      - server: myregistry.azurecr.io
        identity: system
  template:
    revisionSuffix: v2
    containers:
      - image: myregistry.azurecr.io/myapp:v2
        name: myapp
        resources:
          cpu: 0.5
          memory: 1Gi
        env:
          - name: APP_ENV
            value: production
          - name: DB_PASSWORD
            secretRef: db-password
    scale:
      minReplicas: 1
      maxReplicas: 10
      rules:
        - name: http-rule
          http:
            metadata:
              concurrentRequests: "100"
```

### Wdrożenie / aktualizacja z YAML

```bash
# Tworzenie z YAML
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --yaml containerapp.yaml

# Aktualizacja z YAML
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --yaml containerapp.yaml
```

---

## Porównanie: Container Apps vs ACI vs AKS

| Cecha | Container Apps | ACI | AKS |
|-------|---------------|-----|-----|
| **Przypadek użycia** | Mikrousługi, web apps, API | Batch jobs, testy, proste kontenery | Pełna orkiestracja Kubernetes |
| **Skalowanie** | Automatyczne (KEDA) | Manualne | Automatyczne (HPA, KEDA) |
| **Zarządzanie** | Serverless (zero infra) | Minimalne | Pełne zarządzanie klastrem |
| **Networking** | Ingress, VNet, traffic split | Publiczny IP / VNet | Pełna kontrola sieci K8s |
| **Koszty** | Pay-per-use / Consumption | Pay-per-second | Klaster + nodes |
| **Dapr** | Wbudowany | Brak | Możliwy (addon) |
| **Rewizje** | Tak (traffic split) | Brak | Deployments / Rollouts |
| **Złożoność** | Niska | Najniższa | Wysoka |

---

## Przydatne wskazówki

- **Skalowanie do zera** — Container Apps pozwalają na `min-replicas 0`, co eliminuje koszty w czasie bezczynności (uwaga na cold start)
- **Revision suffix** — zawsze używaj `--revision-suffix` dla łatwego identyfikowania wersji
- **Multiple revision mode** — włącz dla blue-green / canary deployments
- **Managed identity** — preferuj nad credentials do ACR i Key Vault
- **YAML** — używaj plików YAML do powtarzalnych wdrożeń i wersjonowania konfiguracji w Git
- **Health probes** — konfiguruj liveness i readiness probes w YAML dla produkcyjnych workloadów
- **Dapr** — wykorzystaj dla komunikacji między mikrousługami (service invocation, pub/sub, state)
- **Log Analytics** — każde środowisko jest zintegrowane z Log Analytics — wykorzystaj KQL do analizy logów

---

## Podsumowanie najważniejszych poleceń

| Polecenie | Opis |
|-----------|------|
| `az containerapp env create` | Tworzenie środowiska |
| `az containerapp create` | Tworzenie aplikacji |
| `az containerapp update` | Aktualizacja aplikacji |
| `az containerapp show` | Szczegóły aplikacji |
| `az containerapp list` | Lista aplikacji |
| `az containerapp delete` | Usunięcie aplikacji |
| `az containerapp logs show` | Wyświetlenie logów |
| `az containerapp exec` | Sesja interaktywna w kontenerze |
| `az containerapp revision list` | Lista rewizji |
| `az containerapp revision copy` | Kopiowanie rewizji |
| `az containerapp ingress traffic set` | Podział ruchu |
| `az containerapp secret set` | Zarządzanie sekretami |
| `az containerapp identity assign` | Przypisanie managed identity |
| `az containerapp job create` | Tworzenie joba |
| `az containerapp hostname add` | Dodanie custom domain |