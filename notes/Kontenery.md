# Azure Container Services

## Azure Container Instances (ACI)

Azure Container Instances to najprostszy i najszybszy sposób uruchamiania kontenerów w Azure, bez konieczności zarządzania maszynami wirtualnymi.

### Tworzenie Container Instance

```bash
# Podstawowe utworzenie kontenera
az container create \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --image <image-name> \
  --dns-name-label <dns-name> \
  --ports 80

# Przykład z nginx
az container create \
  --resource-group MyResourceGroup \
  --name mynginx \
  --image nginx \
  --dns-name-label mynginx-demo \
  --ports 80

# Kontener z określonym CPU i pamięcią
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --cpu 1 \
  --memory 1 \
  --dns-name-label myapp-demo \
  --ports 80
```

### Tworzenie z rejestrem prywatnym (ACR)

```bash
# Utworzenie kontenera z Azure Container Registry
az container create \
  --resource-group MyResourceGroup \
  --name myapp \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-login-server myregistry.azurecr.io \
  --registry-username <username> \
  --registry-password <password> \
  --dns-name-label myapp-demo \
  --ports 80

# Lub z użyciem managed identity
az container create \
  --resource-group MyResourceGroup \
  --name myapp \
  --image myregistry.azurecr.io/myapp:v1 \
  --acr-identity <managed-identity-id> \
  --ports 80
```

### Zmienne środowiskowe i sekrety

```bash
# Kontener ze zmiennymi środowiskowymi
az container create \
  --resource-group MyResourceGroup \
  --name myapp \
  --image myimage \
  --environment-variables 'KEY1=value1' 'KEY2=value2' \
  --ports 80

# Kontener z bezpiecznymi zmiennymi (secure environment variables)
az container create \
  --resource-group MyResourceGroup \
  --name myapp \
  --image myimage \
  --environment-variables 'PUBLIC_VAR=value1' \
  --secure-environment-variables 'DB_PASSWORD=secretpass' 'API_KEY=secretkey' \
  --ports 80
```

### Montowanie wolumenów

```bash
# Utworzenie z Azure File Share
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image myimage \
  --azure-file-volume-account-name <storage-account> \
  --azure-file-volume-account-key <storage-key> \
  --azure-file-volume-share-name <share-name> \
  --azure-file-volume-mount-path /data \
  --ports 80

# Pustry katalog (emptyDir)
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image myimage \
  --command-line "/bin/sh -c 'echo hello > /mnt/empty/hello.txt && sleep 3600'" \
  --environment-variables EmptyDirVolume=/mnt/empty
```

### Zarządzanie Container Instances

```bash
# Lista wszystkich kontenerów
az container list --output table

# Lista w konkretnej grupie zasobów
az container list --resource-group <resource-group-name> --output table

# Szczegóły kontenera
az container show --resource-group <resource-group-name> --name <container-name>

# Status kontenera
az container show --resource-group <resource-group-name> --name <container-name> \
  --query instanceView.state

# Logi kontenera
az container logs --resource-group <resource-group-name> --name <container-name>

# Logi w czasie rzeczywistym (follow)
az container attach --resource-group <resource-group-name> --name <container-name>

# Wykonanie komendy w kontenerze
az container exec --resource-group <resource-group-name> --name <container-name> \
  --exec-command "/bin/bash"
```

### Restart i usuwanie

```bash
# Restart kontenera
az container restart --resource-group <resource-group-name> --name <container-name>

# Zatrzymanie kontenera
az container stop --resource-group <resource-group-name> --name <container-name>

# Uruchomienie kontenera
az container start --resource-group <resource-group-name> --name <container-name>

# Usunięcie kontenera
az container delete --resource-group <resource-group-name> --name <container-name> --yes
```

## Azure Container Apps

Azure Container Apps to zarządzana usługa do uruchamiania mikrousług i konteneryzowanych aplikacji w środowisku serverless.

### Instalacja rozszerzenia (zależne od wersji Azure CLI)

```bash
# Instalacja rozszerzenia Container Apps
az extension add --name containerapp --upgrade

# Rejestracja dostawcy
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights
```

### Tworzenie środowiska Container Apps

```bash
# Utworzenie środowiska
az containerapp env create \
  --name <environment-name> \
  --resource-group <resource-group-name> \
  --location westeurope

# Przykład
az containerapp env create \
  --name mycontainerenv \
  --resource-group MyResourceGroup \
  --location westeurope
```

### Tworzenie Container App

```bash
# Podstawowa aplikacja kontenerowa
az containerapp create \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --environment <environment-name> \
  --image <image-name> \
  --target-port 80 \
  --ingress external \
  --query properties.configuration.ingress.fqdn

# Przykład z nginx
az containerapp create \
  --name mynginxapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image nginx \
  --target-port 80 \
  --ingress external \
  --query properties.configuration.ingress.fqdn

# Aplikacja z określonymi zasobami
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

### Tworzenie z Azure Container Registry

```bash
# Container App z ACR
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

# Z managed identity
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-identity system \
  --target-port 80 \
  --ingress external
```

### Zmienne środowiskowe i sekrety

```bash
# Dodanie zmiennych środowiskowych
az containerapp create \
  --name myapp \
  --resource-group MyResourceGroup \
  --environment mycontainerenv \
  --image myimage \
  --env-vars "API_URL=https://api.example.com" "LOG_LEVEL=info" \
  --target-port 80 \
  --ingress external

# Dodanie sekretów
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

### Aktualizacja Container App

```bash
# Aktualizacja obrazu
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --image myregistry.azurecr.io/myapp:v2

# Aktualizacja replicas
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --min-replicas 2 \
  --max-replicas 20

# Aktualizacja zmiennych środowiskowych
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --set-env-vars "NEW_VAR=value"

# Usunięcie zmiennej
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --remove-env-vars "OLD_VAR"
```

### Skalowanie

```bash
# Konfiguracja autoskalowania na podstawie HTTP
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

# Skalowanie na podstawie CPU
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --scale-rule-name cpu-scale \
  --scale-rule-type cpu \
  --scale-rule-metadata "type=Utilization" "value=75"

# Skalowanie na podstawie Azure Queue
az containerapp update \
  --name myapp \
  --resource-group MyResourceGroup \
  --scale-rule-name queue-scale \
  --scale-rule-type azure-queue \
  --scale-rule-metadata "queueName=myqueue" "queueLength=5" \
  --scale-rule-auth "connection=connection-string"
```

### Wersjonowanie i rewizje

```bash
# Lista rewizji
az containerapp revision list \
  --name myapp \
  --resource-group MyResourceGroup \
  --output table

# Aktywacja konkretnej rewizji
az containerapp revision activate \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision <revision-name>

# Dezaktywacja rewizji
az containerapp revision deactivate \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision <revision-name>

# Konfiguracja traffic splitting
az containerapp ingress traffic set \
  --name myapp \
  --resource-group MyResourceGroup \
  --revision-weight <revision-1>=80 <revision-2>=20
```

### Zarządzanie Container Apps

```bash
# Lista wszystkich aplikacji
az containerapp list --output table

# Lista w grupie zasobów
az containerapp list --resource-group <resource-group-name> --output table

# Szczegóły aplikacji
az containerapp show --name <app-name> --resource-group <resource-group-name>

# Logi aplikacji
az containerapp logs show \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --follow

# Wyświetlenie URL aplikacji
az containerapp show \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --query properties.configuration.ingress.fqdn

# Restart aplikacji
az containerapp revision restart \
  --name <app-name> \
  --resource-group <resource-group-name> \
  --revision <revision-name>
```

### Usuwanie

```bash
# Usunięcie Container App
az containerapp delete --name <app-name> --resource-group <resource-group-name> --yes

# Usunięcie środowiska
az containerapp env delete --name <environment-name> --resource-group <resource-group-name> --yes
```

## Azure Container Registry (ACR)

### Tworzenie i zarządzanie rejestrem

```bash
# Utworzenie Container Registry
az acr create \
  --resource-group MyResourceGroup \
  --name myregistry \
  --sku Basic

# SKU: Basic, Standard, Premium

# Logowanie do rejestru
az acr login --name myregistry

# Włączenie admin user (nie zalecane w produkcji)
az acr update --name myregistry --admin-enabled true

# Pobranie poświadczeń
az acr credential show --name myregistry
```

### Operacje na obrazach

```bash
# Lista repozytoriów
az acr repository list --name myregistry --output table

# Lista tagów w repozytorium
az acr repository show-tags --name myregistry --repository myapp --output table

# Usunięcie obrazu
az acr repository delete --name myregistry --image myapp:v1 --yes

# Push obrazu do ACR
docker tag myapp:latest myregistry.azurecr.io/myapp:v1
docker push myregistry.azurecr.io/myapp:v1

# Pull obrazu z ACR
docker pull myregistry.azurecr.io/myapp:v1
```

### Zadania ACR (ACR Tasks)

```bash
# Szybkie zbudowanie obrazu w chmurze
az acr build \
  --registry myregistry \
  --image myapp:v1 \
  --file Dockerfile \
  .

# Utworzenie zadania build na commit
az acr task create \
  --registry myregistry \
  --name buildtask \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/myuser/myrepo.git \
  --file Dockerfile \
  --git-access-token <token>

# Ręczne uruchomienie zadania
az acr task run --registry myregistry --name buildtask

# Lista zadań
az acr task list --registry myregistry --output table
```

## Porównanie: ACI vs Container Apps

| Cecha | ACI | Container Apps |
|-------|-----|----------------|
| **Przypadek użycia** | Szybkie zadania, testy, batch jobs | Mikrousługi, web apps, API |
| **Skalowanie** | Manualne | Automatyczne (event-driven, HTTP) |
| **Networking** | Publiczny IP lub VNet | Zaawansowane (ingress, traffic splitting) |
| **Cena** | Za sekundę użycia | Za sekundę użycia + consumption |
| **Zarządzanie** | Prosty lifecycle | Rewizje, wersjonowanie |
| **Ingress** | Podstawowy | Zaawansowany (HTTP/HTTPS, traffic split) |
| **Sekrety** | Secure env vars | Managed secrets |

## Przydatne wskazówki

### ACI
- Idealny do krótkotrwałych zadań i testów
- Szybkie uruchomienie (sekundy)
- Płacisz tylko za rzeczywiste użycie
- Użyj Container Groups dla aplikacji wielokontenerowych (sidecar pattern)
- Montuj Azure Files dla trwałego storage

### Container Apps
- Idealny do mikrousług i aplikacji webowych
- Automatyczne skalowanie (w tym do zera)
- Wbudowane HTTPS i zarządzanie certyfikatami
- Traffic splitting dla strategii wdrożeniowych (blue-green, canary)
- Wbudowana obsługa Dapr dla mikrousług
- Użyj rewizji dla łatwego rollback

### ACR
- Zawsze używaj Premium SKU dla produkcji (geo-replication, webhooks)
- Używaj managed identity zamiast admin credentials
- Konfiguruj webhook dla CI/CD
- Włącz skanowanie podatności (Defender for Cloud)
- Regularnie czyść nieużywane obrazy
