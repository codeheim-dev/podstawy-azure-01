# Azure Container Instances (ACI) - Polecenia Azure CLI

## Czym jest Azure Container Instance?

Azure Container Instances (ACI) to najprostszy i najszybszy sposób na uruchomienie kontenera w Azure. Nie wymaga zarządzania maszynami wirtualnymi ani orkiestratorami. Kontenery uruchamiają się w ciągu sekund, a rozliczenie odbywa się za sekundę użycia zasobów (CPU i pamięć).

**Kluczowe cechy:**
- Szybkie uruchomienie kontenerów (sekundy)
- Brak zarządzania infrastrukturą
- Publiczny adres IP i etykieta DNS
- Izolacja na poziomie hiperwizora
- Obsługa kontenerów Linux i Windows
- Wsparcie dla grup kontenerów (sidecar pattern)
- Montowanie wolumenów (Azure Files, emptyDir, gitRepo, secret)

---

## Tworzenie Container Instance

### Podstawowe tworzenie

```bash
# Najprostsze utworzenie kontenera
az container create \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --image <image-name> \
  --dns-name-label <dns-name> \
  --ports 80
```

### Przykład z obrazem nginx

```bash
az container create \
  --resource-group MyResourceGroup \
  --name mynginx \
  --image nginx:latest \
  --dns-name-label mynginx-demo \
  --ports 80
```

### Określenie zasobów CPU i pamięci

```bash
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --cpu 2 \
  --memory 4 \
  --dns-name-label myapp-demo \
  --ports 80
```

> **Uwaga:** Domyślne wartości to 1 CPU i 1.5 GB pamięci. Maksimum to 4 CPU i 16 GB RAM (zależnie od regionu).

### Wybór systemu operacyjnego

```bash
# Kontener Linux (domyślny)
az container create \
  --resource-group MyResourceGroup \
  --name mylinux \
  --image nginx \
  --os-type Linux \
  --ports 80

# Kontener Windows
az container create \
  --resource-group MyResourceGroup \
  --name mywindows \
  --image mcr.microsoft.com/windows/servercore/iis:latest \
  --os-type Windows \
  --cpu 2 \
  --memory 3.5 \
  --ports 80
```

### Określenie lokalizacji

```bash
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image nginx \
  --location westeurope \
  --ports 80
```

---

## Restart Policy (polityka restartu)

Określa zachowanie kontenera po zakończeniu procesu.

```bash
# Always - kontener jest zawsze restartowany (domyślne)
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image nginx \
  --restart-policy Always \
  --ports 80

# OnFailure - restart tylko w razie błędu (exit code != 0)
az container create \
  --resource-group MyResourceGroup \
  --name mybatchjob \
  --image myimage \
  --restart-policy OnFailure

# Never - kontener nigdy nie jest restartowany
az container create \
  --resource-group MyResourceGroup \
  --name myonetimejob \
  --image myimage \
  --restart-policy Never
```

> **Tip:** Użyj `OnFailure` lub `Never` dla zadań jednorazowych (batch jobs), aby uniknąć nieskończonego restartowania.

---

## Zmienne środowiskowe

### Zwykłe zmienne środowiskowe

```bash
az container create \
  --resource-group MyResourceGroup \
  --name myapp \
  --image myimage \
  --environment-variables \
    'APP_ENV=production' \
    'LOG_LEVEL=info' \
    'API_URL=https://api.example.com' \
  --ports 80
```

### Bezpieczne zmienne środowiskowe (sekrety)

Wartości secure environment variables nie są widoczne w portalu ani w `az container show`.

```bash
az container create \
  --resource-group MyResourceGroup \
  --name myapp \
  --image myimage \
  --environment-variables 'APP_ENV=production' \
  --secure-environment-variables \
    'DB_CONNECTION_STRING=Server=myserver;Password=secret' \
    'API_KEY=my-secret-api-key' \
  --ports 80
```

---

## Tworzenie z rejestrem prywatnym

### Azure Container Registry (ACR)

```bash
# Z poświadczeniami (username/password)
az container create \
  --resource-group MyResourceGroup \
  --name myapp \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-login-server myregistry.azurecr.io \
  --registry-username <acr-username> \
  --registry-password <acr-password> \
  --dns-name-label myapp-demo \
  --ports 80

# Z managed identity (zalecane)
az container create \
  --resource-group MyResourceGroup \
  --name myapp \
  --image myregistry.azurecr.io/myapp:v1 \
  --acr-identity <managed-identity-resource-id> \
  --dns-name-label myapp-demo \
  --ports 80
```

### Docker Hub (rejestr prywatny)

```bash
az container create \
  --resource-group MyResourceGroup \
  --name myapp \
  --image myuser/myapp:latest \
  --registry-login-server docker.io \
  --registry-username <dockerhub-username> \
  --registry-password <dockerhub-password> \
  --ports 80
```

---

## Montowanie wolumenów

### Azure File Share

```bash
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image nginx \
  --azure-file-volume-account-name <storage-account-name> \
  --azure-file-volume-account-key <storage-account-key> \
  --azure-file-volume-share-name <file-share-name> \
  --azure-file-volume-mount-path /mnt/data \
  --ports 80
```

### Nadpisanie komendy startowej

```bash
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azuredocs/aci-wordcount \
  --restart-policy OnFailure \
  --command-line "python wordcount.py http://shakespeare.mit.edu/romeo_juliet/full.html"
```

---

## Wdrożenie z pliku YAML

Plik YAML pozwala na bardziej zaawansowaną konfigurację, w tym grupy kontenerów (container groups).

### Przykładowy plik `deploy-aci.yaml`

```yaml
apiVersion: '2021-09-01'
location: westeurope
name: mycontainergroup
properties:
  containers:
  - name: myapp
    properties:
      image: nginx:latest
      resources:
        requests:
          cpu: 1.0
          memoryInGb: 1.5
      ports:
      - port: 80
        protocol: TCP
      environmentVariables:
      - name: APP_ENV
        value: production
      - name: DB_PASSWORD
        secureValue: supersecret123
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - port: 80
      protocol: TCP
    dnsNameLabel: myapp-demo
  restartPolicy: Always
type: Microsoft.ContainerInstance/containerGroups
```

### Wdrożenie z YAML

```bash
az container create \
  --resource-group MyResourceGroup \
  --file deploy-aci.yaml
```

### Eksport istniejącego kontenera do YAML

```bash
az container export \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --file exported-container.yaml
```

---

## Container Groups (grupy kontenerów)

Grupa kontenerów pozwala uruchamiać wiele kontenerów na tym samym hoście (sidecar pattern). Kontenery w grupie dzielą cykl życia, sieć i wolumeny.

### Przykładowy YAML z grupą kontenerów (sidecar)

```yaml
apiVersion: '2021-09-01'
location: westeurope
name: multi-container-group
properties:
  containers:
  - name: webapp
    properties:
      image: nginx:latest
      resources:
        requests:
          cpu: 1.0
          memoryInGb: 1.0
      ports:
      - port: 80
      volumeMounts:
      - name: shared-data
        mountPath: /usr/share/nginx/html
  - name: sidecar
    properties:
      image: busybox
      resources:
        requests:
          cpu: 0.5
          memoryInGb: 0.5
      command:
      - /bin/sh
      - -c
      - 'while true; do echo "$(date) - Hello from sidecar" > /mnt/shared/index.html; sleep 10; done'
      volumeMounts:
      - name: shared-data
        mountPath: /mnt/shared
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - port: 80
  volumes:
  - name: shared-data
    emptyDir: {}
type: Microsoft.ContainerInstance/containerGroups
```

```bash
az container create --resource-group MyResourceGroup --file multi-container.yaml
```

---

## Zarządzanie Container Instances

### Wyświetlanie informacji

```bash
# Lista wszystkich kontenerów w subskrypcji
az container list --output table

# Lista w konkretnej grupie zasobów
az container list --resource-group <resource-group-name> --output table

# Szczegółowe informacje o kontenerze
az container show \
  --resource-group <resource-group-name> \
  --name <container-name>

# Pobranie statusu kontenera
az container show \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --query instanceView.state

# Pobranie adresu IP kontenera
az container show \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --query ipAddress.ip --output tsv

# Pobranie FQDN kontenera
az container show \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --query ipAddress.fqdn --output tsv

# Pobranie statusu zasobów (CPU, pamięć)
az container show \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --query "containers[0].instanceView.currentState"
```

### Logi kontenera

```bash
# Wyświetlenie logów kontenera
az container logs \
  --resource-group <resource-group-name> \
  --name <container-name>

# Logi konkretnego kontenera w grupie kontenerów
az container logs \
  --resource-group <resource-group-name> \
  --name <container-group-name> \
  --container-name <container-name>

# Śledzenie logów w czasie rzeczywistym (attach)
az container attach \
  --resource-group <resource-group-name> \
  --name <container-name>
```

### Wykonywanie poleceń w kontenerze

```bash
# Interaktywna sesja bash
az container exec \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --exec-command "/bin/bash"

# Interaktywna sesja sh
az container exec \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --exec-command "/bin/sh"

# Wykonanie konkretnej komendy
az container exec \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --exec-command "ls -la /app"
```

---

## Cykl życia kontenera

### Uruchamianie, zatrzymywanie, restart

```bash
# Uruchomienie zatrzymanego kontenera
az container start \
  --resource-group <resource-group-name> \
  --name <container-name>

# Zatrzymanie kontenera (dealokacja zasobów)
az container stop \
  --resource-group <resource-group-name> \
  --name <container-name>

# Restart kontenera
az container restart \
  --resource-group <resource-group-name> \
  --name <container-name>
```

### Usuwanie kontenera

```bash
# Usunięcie kontenera (z potwierdzeniem)
az container delete \
  --resource-group <resource-group-name> \
  --name <container-name>

# Usunięcie bez potwierdzenia
az container delete \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --yes
```

---

## Sieć (Networking)

### Publiczny adres IP

```bash
# Kontener z publicznym IP i DNS
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image nginx \
  --ip-address Public \
  --dns-name-label myapp-unique-name \
  --ports 80 443
```

### Wdrożenie w sieci wirtualnej (VNet)

```bash
# Tworzenie kontenera w istniejącej sieci VNet/Subnet
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image nginx \
  --vnet <vnet-name> \
  --subnet <subnet-name> \
  --ports 80

# Z nowym VNet i subnet
az container create \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --image nginx \
  --vnet myVNet \
  --vnet-address-prefix 10.0.0.0/16 \
  --subnet mySubnet \
  --subnet-address-prefix 10.0.0.0/24 \
  --ports 80
```

> **Uwaga:** Kontenery w VNet nie mają publicznego IP — dostęp odbywa się przez inne zasoby w sieci.

---

## GPU (karty graficzne)

```bash
# Kontener z GPU (dostępne w wybranych regionach)
az container create \
  --resource-group MyResourceGroup \
  --name mygpucontainer \
  --image nvidia/cuda:11.0-base \
  --gpu-count 1 \
  --gpu-sku K80 \
  --os-type Linux
```

> Dostępne SKU GPU: `K80`, `P100`, `V100` (zależnie od regionu).

---

## Diagnostyka i rozwiązywanie problemów

```bash
# Sprawdzenie stanu kontenera
az container show \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --query "containers[0].instanceView" \
  --output table

# Sprawdzenie eventów kontenera
az container show \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --query "containers[0].instanceView.events" \
  --output table

# Pobranie exit code
az container show \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --query "containers[0].instanceView.currentState.exitCode"

# Sprawdzenie powodu zakończenia
az container show \
  --resource-group <resource-group-name> \
  --name <container-name> \
  --query "containers[0].instanceView.currentState.detailStatus"
```

---

## Przydatne sztuczki i wskazówki

### Szybkie formatowanie wyników

```bash
# Wyświetlanie w tabeli
az container list --output table

# Wyświetlanie jako JSON
az container show --resource-group MyRG --name mycontainer --output json

# Filtrowanie wyników z JMESPath
az container show --resource-group MyRG --name mycontainer \
  --query "{Name:name, Status:instanceView.state, IP:ipAddress.ip, FQDN:ipAddress.fqdn}" \
  --output table
```

### Oczekiwanie na gotowość kontenera

```bash
# Czekanie aż kontener będzie gotowy
az container show \
  --resource-group MyResourceGroup \
  --name mycontainer \
  --query instanceView.state

# Sprawdzenie czy kontener działa w pętli
while [ "$(az container show --resource-group MyRG --name mycontainer --query instanceView.state -o tsv)" != "Running" ]; do
  echo "Czekam na uruchomienie..."
  sleep 5
done
echo "Kontener działa!"
```

---

## Podsumowanie najważniejszych poleceń

| Polecenie | Opis |
|-----------|------|
| `az container create` | Tworzenie nowego kontenera |
| `az container show` | Wyświetlenie szczegółów kontenera |
| `az container list` | Lista kontenerów |
| `az container logs` | Wyświetlenie logów |
| `az container attach` | Śledzenie logów w czasie rzeczywistym |
| `az container exec` | Wykonanie polecenia w kontenerze |
| `az container start` | Uruchomienie kontenera |
| `az container stop` | Zatrzymanie kontenera |
| `az container restart` | Restart kontenera |
| `az container delete` | Usunięcie kontenera |
| `az container export` | Eksport konfiguracji do YAML |