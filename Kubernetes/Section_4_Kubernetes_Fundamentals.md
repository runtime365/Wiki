# Kubernetes Fundamentals - Section 4

## Inhoudsopgave

- [57. Kubernetes Pods - Part 1](#57-kubernetes-pods---part-1) - Basis pod operaties en management
- [58. Kubernetes Pods - Part 2](#58-kubernetes-pods---part-2) - YAML generatie, declarative management en multi-container pods
- [59. Kubernetes Pods - Part 3](#59-kubernetes-pods---part-3) - Multi-container testing en troubleshooting
- [63. Kubernetes Namespaces](#63-kubernetes-namespaces) - Resource isolatie en context management
- [66. Kubernetes Deployments and ReplicaSets](#66-kubernetes-deployments-and-replicasets) - Productie-ready applicatie deployment
- [70. Kubernetes Services](#70-kubernetes-services) - Network abstraction en service discovery

---

#### 57. Kubernetes Pods - Part 1

**Doel:** Basis pod operaties en management

**Context:** Pods zijn de kleinste deployable units in Kubernetes. Een pod is simpelweg een groep van één of meer containers die netwerk en storage delen. Containers binnen een pod communiceren via localhost en krijgen een uniek IP adres binnen het cluster.

**Pod Fundamentals:**

- **Smallest deployable unit** - Pods zijn de basis compute eenheid in Kubernetes
- **Container grouping** - Groep van één of meer containers die resources delen
- **Shared networking** - Containers delen netwerk en communiceren via localhost
- **Unique IP address** - Elke pod krijgt zijn eigen IP adres binnen het cluster
- **Inter Process Communication (IPC)** - Containers kunnen communiceren via IPC
- **Application encapsulation** - Pod kapselt applicatie, dependencies, storage en networking in

```bash
# 1. Pod aanmaken (single container)
kubectl run nginx --image=nginx                             # Basis nginx pod aanmaken

# 2. Pod status bekijken
kubectl get pods                                             # Basis pod informatie
kubectl get pods -o wide                                     # Uitgebreide info: IP, node, ready status

# 3. Pod logs bekijken
kubectl logs pod/nginx                                       # Pod logs (volledige syntax)
kubectl logs nginx                                           # Verkorte versie (pods zijn default entity)

# 4. Pod IP connectiviteit testen (indien toegankelijk)
ping <POD_IP>                                               # Direct ping naar pod IP (van cluster node)
ssh <NODE_IP>                                               # SSH naar andere node voor testing
ping <POD_IP>                                               # Pod IP bereikbaar van alle cluster nodes

# 5. Lokale toegang via port forwarding
kubectl port-forward --help                                 # Help voor port forwarding opties
kubectl port-forward pod/nginx 8080:80                     # Localhost:8080 → pod:80
# Browse naar http://localhost:8080 of http://127.0.0.1:8080

# 6. Pod-to-pod communicatie testen
kubectl run -it --rm curl --image=curlimages/curl:8.4.0 --restart=Never -- <POD_IP>  # Curl naar pod IP
```

### Interactive Pod Management

```bash
# 7. Background pod aanmaken
kubectl run ubuntu --image=ubuntu --env="NGINX_IP=<POD_IP>" -- sleep infinity        # Ubuntu pod met environment variable

# 8. In running pod uitvoeren (exec)
kubectl exec -it ubuntu -- bash                             # Interactive bash shell in pod
ps -ef                                                       # Processen binnen container bekijken
# PID 1 = hoofdprocess (sleep infinity)
# Bash shell = child process via kubectl exec

# 9. Packages installeren in running pod
apt update                                                   # Ubuntu package cache updaten
apt install curl -y                                         # Curl installeren
curl <NGINX_IP>                                             # Pod-to-pod communicatie testen

# 10. Pod cleanup
exit                                                         # Exit uit exec session
kubectl delete pod/nginx pod/ubuntu --now                   # Geforceerd verwijderen (nieuwere Kubernetes)
kubectl delete pod/nginx pod/ubuntu --grace-period=0        # Alternatief voor oudere versies
```

### Pod Networking Concepts

**IP Address Accessibility:**
- **Within cluster**: Pod IPs zijn toegankelijk vanaf alle cluster nodes
- **External access**: Afhankelijk van Kubernetes setup (desktop vs cloud vs on-premise)
- **Port forwarding**: Universele oplossing voor lokale toegang
- **No NAT required**: Pods communiceren direct zonder Network Address Translation

**Communication Patterns:**
- **Localhost binnen pod**: Containers in zelfde pod via 127.0.0.1
- **Pod-to-pod**: Via unieke pod IP adressen
- **Cross-node**: Pods op verschillende nodes kunnen direct communiceren
- **Service discovery**: DNS-based via Kubernetes services

**Command uitleg:**

- `-it`: Interactive terminal (stdin + tty) voor shell access
- `--rm`: Automatisch pod verwijderen na exit (cleanup)
- `--restart=Never`: Pod niet herstarten bij failure
- `--now`: Moderne manier voor geforceerd verwijderen (--grace-period=0)
- `-o wide`: Uitgebreide output met IP adressen en node informatie
- `--env`: Environment variabelen doorgeven aan container

**Networking Tips:**

- Pod IP adressen zijn **cluster-internal** - niet extern toegankelijk
- **Port forwarding** (`kubectl port-forward`) maakt lokale development mogelijk
- **kubectl exec** biedt debugging toegang tot running containers
- **--rm flag** voorkomt pod accumulation bij testing
- **ps -ef** toont proces hiërarchie binnen containers

**Development Workflow:**
1. **Deploy** - `kubectl run` voor snelle pod deployment
2. **Monitor** - `kubectl get pods -o wide` voor status en IP
3. **Debug** - `kubectl logs` en `kubectl exec` voor troubleshooting
4. **Test** - Pod-to-pod communicatie via IP adressen
5. **Access** - Port forwarding voor lokale applicatie toegang
6. **Cleanup** - `kubectl delete --now` voor resource management

**Best Practices:**
- Gebruik **port forwarding** voor development access
- **Monitor logs** voor applicatie debugging
- **Test connectiviteit** tussen pods voor netwerk validatie
- **Clean up** test pods om resource waste te voorkomen
- **Use meaningful names** voor pod identificatie

---

#### 58. Kubernetes Pods - Part 2

**Doel:** YAML generatie, declarative management en multi-container pods

**Context:** YAML is een eenvoudige data serialization language die vaak gebruikt wordt voor configuratiebestanden. Kubectl kan helpen bij het snel genereren van YAML content die we nodig hebben.

### YAML Generatie en Dry-Run

```bash
# 1. YAML genereren zonder uitvoeren
kubectl run nginx --image=nginx --dry-run=client -o yaml | tee nginx.yaml

# 2. Pod specificaties uitleggen
kubectl explain pod.spec.restartPolicy    # Documentatie over restartPolicy

# 3. Ubuntu pod YAML genereren
kubectl run ubuntu --image=ubuntu --dry-run=client -o yaml | tee ubuntu.yaml

# 4. Meerdere YAML bestanden combineren
{ cat nginx.yaml; echo "---"; cat ubuntu.yaml; } | tee combined.yaml
```

### Imperative vs Declarative Approaches

**Imperative Commands (stap voor stap):**
- `kubectl create` - Kan alleen nieuwe resources aanmaken
- `kubectl replace` - Vervangt bestaande resource
- `kubectl delete` - Verwijdert resource
- **Probleem**: Als resource al bestaat, krijg je error bij tweede create

**Declarative Approach:**
- `kubectl apply` - Declareert gewenste state
- **Voordeel**: Werkt of resource nu wel of niet bestaat
- **Best practice**: Gebruik apply voor toekomstige wijzigingen

```bash
# 5. Imperative approach testen
kubectl create -f nginx.yaml                    # Eerste keer succesvol
kubectl create -f nginx.yaml                    # Tweede keer: error (already exists)

# 6. Declarative approach
kubectl apply -f nginx.yaml                     # Werkt altijd (warning bij eerste keer na create)
kubectl apply -f nginx.yaml                     # Geen warning bij herhaling

# 7. Multi-resource deployment
kubectl apply -f combined.yaml                  # Beide pods tegelijk aanmaken
kubectl delete -f combined.yaml --now           # Beide pods tegelijk verwijderen
```

### YAML Structure Fundamentals

**RestartPolicy Options:**
- **Always** (default): Altijd pod herstarten bij failure
- **Never**: Nooit pod herstarten
- **OnFailure**: Alleen herstarten bij failure

**YAML File Benefits:**
- **Version control**: Configuratie in Git
- **Reproducibility**: Consistente deployments
- **Documentation**: Self-documenting infrastructure
- **Multi-resource**: Meerdere resources in één bestand

```yaml
# Voorbeeld nginx.yaml structuur
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    run: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  restartPolicy: Always  # Always, Never, OnFailure
```

### Multi-Resource YAML Files

```bash
# YAML separator (---) scheidt resources
cat nginx.yaml
echo "---"                                      # YAML resource separator
cat ubuntu.yaml
```

**File Management:**
- `tee`: Output naar scherm EN bestand (T-junction effect)
- `---`: YAML separator tussen multiple resources
- `-f`: File flag voor kubectl operations
- `--dry-run=client`: Genereer YAML zonder execution

**Command uitleg:**

- `--dry-run=client`: Genereer YAML zonder uit te voeren (safe testing)
- `tee`: Schrijft naar bestand én toont op scherm (T-junction)
- `kubectl explain`: Geeft documentatie over Kubernetes resources
- `---`: YAML separator voor multiple resources in één bestand
- `-f`: File flag voor kubectl file operations

**Best Practices:**
- **Start met apply**: Voorkomt toekomstige problemen
- **Use dry-run**: Test YAML generatie zonder side effects
- **Combine resources**: Gebruik --- separator voor related resources
- **Document policies**: RestartPolicy moet match use case
- **Version control**: YAML files in Git voor change tracking

**YAML Workflow:**
1. **Generate** - `--dry-run=client -o yaml` voor template
2. **Customize** - Pas YAML aan voor specifieke requirements
3. **Validate** - `kubectl explain` voor field documentation
4. **Apply** - `kubectl apply -f` voor declarative management
5. **Update** - Wijzig YAML en re-apply voor changes
6. **Cleanup** - `kubectl delete -f` voor resource removal

**Troubleshooting Tips:**
- **kubectl explain**: Voor field documentation en valid values
- **Dry-run eerst**: Test YAML voor daadwerkelijke deployment
- **Warning messages**: Eerste apply na create geeft warning (normaal)
- **File validation**: Check YAML syntax voor deployment

---

#### 59. Kubernetes Pods - Part 3

**Doel:** Multi-container testing en troubleshooting

**Context:** Pods kunnen meerdere containers bevatten die samenwerken. Sidecar containers zijn een veelgebruikt pattern voor ondersteunende taken zoals logging, monitoring, of proxy functionaliteit.

### Multi-Container Pod Concepts

**Sidecar Pattern:**
- **Sidecar container**: Container "aan de zijkant" voor specifieke taken
- **Shared resources**: Containers delen IP adres, netwerk en volumes
- **Use cases**: Logging, monitoring, proxy, data synchronization
- **Readiness indicator**: Pod toont 2/2 ready status voor twee containers

### Multi-Container YAML Structure

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    run: mypod
spec:
  containers:
  - name: webserver
    image: nginx
  - name: sidecar
    image: ubuntu
    args:
      - /bin/sh
      - -c
      - |
        while true; do 
          echo "$(date +'%T') - Hello from the sidecar"; 
          sleep 5; 
          if [ -f /tmp/crash ]; then 
            exit 1; 
          fi; 
        done
  restartPolicy: Always
```

### Multi-Container Operations

```bash
# 1. Multi-container pod deployen
kubectl apply -f mypod.yaml

# 2. Pod status controleren
kubectl get pods                                             # Ready: 2/2 (beide containers running)
kubectl get pods -o wide                                     # Shared IP address tussen containers
kubectl describe pod/mypod                                   # Gedetailleerde container informatie

# 3. Pod connectiviteit testen
kubectl run -it --rm curl --image=curlimages/curl:8.4.0 --restart=Never -- http://$MYPOD_IP

# 4. Container-specifieke logs bekijken
kubectl logs pod/mypod -c webserver                         # Nginx logs
kubectl logs pod/mypod -c sidecar                           # Sidecar logs (timestamps zichtbaar)

# 5. Crash simulatie en troubleshooting
kubectl exec -it mypod -c sidecar -- touch /tmp/crash       # Trigger crash conditie
kubectl get pods                                             # Status: Error → Running (restart)
kubectl logs pod/mypod -c sidecar                           # Live logs (na restart)
kubectl logs pod/mypod -p -c sidecar                        # Previous logs (voor crash)

# 6. Cleanup
kubectl delete pod/mypod --now
rm *.yaml                                                    # YAML files opruimen
```

### YAML Best Practices voor Multi-Container

**Container List Structure:**
```yaml
spec:
  containers:
  - name: webserver          # Name eerst voor readability
    image: nginx             # Image tweede
  - name: sidecar           # Consistente volgorde
    image: ubuntu           # Verbetert collaboration
    args: [...]             # Args/command als laatste
```

**Readability Tips:**
- **Name first**: Container naam als eerste field voor clarity
- **Consistent order**: Zelfde field volgorde voor alle containers
- **Meaningful names**: webserver, sidecar, proxy etc.
- **Collaboration**: Readable YAML voor team development

### Container Crash Handling

**Restart Behavior:**
- **Automatic restart**: RestartPolicy: Always herstart crashed containers
- **Restart count**: Zichtbaar in `kubectl get pods`
- **Shared pod state**: Eén container crash beïnvloedt hele pod status
- **Log continuity**: Previous logs blijven beschikbaar

**Debugging Commands:**
```bash
# Container-specifieke operaties
kubectl logs pod/mypod -c <container-name>                  # Live container logs
kubectl logs pod/mypod -p -c <container-name>               # Previous container logs (crashed)
kubectl exec -it mypod -c <container-name> -- <command>     # Execute in specific container
kubectl describe pod/mypod                                  # Pod-level events en status
```

### Sidecar Implementation Patterns

**Common Sidecar Tasks:**
1. **Logging**: Log aggregation en forwarding
2. **Monitoring**: Metrics collection en health checks
3. **Proxy**: Traffic routing en load balancing
4. **Security**: Certificate management en encryption
5. **Data sync**: File synchronization en backup

**Shell Script Pattern:**
```bash
# Continuous loop met crash simulation
while true; do 
  echo "$(date +'%T') - Hello from the sidecar"
  sleep 5
  if [ -f /tmp/crash ]; then 
    exit 1                    # Simulate crash condition
  fi
done
```

### Multi-Container Troubleshooting

**Status Indicators:**
- **Ready 2/2**: Beide containers healthy
- **Ready 1/2**: Eén container failed/starting
- **Error status**: Container crash detected
- **Restart count**: Aantal container restarts

**Log Analysis:**
- **Timestamp comparison**: Voor crash timing analysis
- **Previous logs (-p)**: Voor failure root cause
- **Container isolation**: Logs per container bekijken
- **Event correlation**: Pod events vs container logs

**Command flags:**
- `-c <container>`: Specifieke container in pod targeten
- `-p`: Previous logs (van crashed/restarted container)
- `--rm`: Test pods automatisch cleanup
- `ready 2/2`: Indicator voor multi-container pod health

**Multi-Container Benefits:**
- **Resource sharing**: Netwerk, volumes, en lifecycle
- **Tight coupling**: Containers die samen moeten werken
- **Simplified networking**: Localhost communicatie tussen containers
- **Coordinated scaling**: Containers scale samen als unit
- **Shared fate**: Containers hebben same lifecycle

**Development Tips:**
- **Test crash scenarios**: Validate restart behavior
- **Monitor all containers**: Check logs voor alle containers
- **Use meaningful names**: webserver, sidecar, proxy etc.
- **Previous logs**: Essential voor crash analysis
- **Shared IP testing**: Verify network connectivity tussen containers

**Pod Lifecycle Management:**
1. **Deploy** - Multi-container pod via YAML
2. **Monitor** - Ready status en restart counts
3. **Test** - Individual container functionality
4. **Debug** - Container-specific logs en exec
5. **Simulate** - Failure scenarios voor resilience testing
6. **Cleanup** - Pod en YAML file management

---

#### 63. Kubernetes Namespaces

**Doel:** Resource isolatie en context management

**Context:** Namespaces zijn fundamentele elementen van de Kubernetes architectuur en zijn instrumentaal bij het verdelen van cluster resources tussen meerdere gebruikers en applicaties. Ze bieden isolatie, organisatie en resource management in multi-tenant omgevingen.

### Namespace Conceptualization

**Town Analogy:**
- **Kubernetes cluster** = Stad
- **Namespaces** = Huizen/gebouwen in de stad
- **Pods** = Kamers binnen de gebouwen
- **Resources** = Meubilair in elke kamer (kan beperkt worden)
- **Independent function** = Elk gebouw functioneert onafhankelijk binnen het ecosysteem

### Namespace Benefits

**Multi-Tenant Environment:**
- **Isolation**: Scheiding tussen teams en gebruikers
- **Management**: Betere organisatie van resources
- **Security**: Toegangscontrole per namespace
- **Resource allocation**: Budget toewijzing per namespace

**Resource Management:**
- **Resource Quotas**: Budgets voor memory/CPU usage per namespace
- **LimitRanges**: Min/max resource regels per pod/container
- **Budget allocation**: Zoals huishoudelijke uitgaven beheren

**Security & Access Control:**
- **Namespace-specific policies**: Toegangsbeleid per namespace
- **Authorized access**: Alleen geautoriseerde gebruikers en processen
- **RBAC integration**: Role Based Access Control per namespace
- **Enhanced security**: Beperkte toegang tot namespace resources

**Organization & Simplification:**
- **Resource management**: Beheer in grote clusters
- **Simple naming**: Namen hoeven alleen uniek binnen namespace
- **Name collision prevention**: Zelfde pod naam in verschillende namespaces
- **Large organization support**: Onafhankelijkheid tussen teams

### Default Namespaces

```bash
# 1. Overzicht van alle resources
kubectl get all -A                          # Alle resources in alle namespaces
kubectl get namespaces                      # Lijst van namespaces
kubectl get ns                              # Verkorte versie (short name)

# 2. Resource types bekijken
kubectl api-resources | more                # Alle beschikbare resource types
kubectl api-resources                       # Toont NAMESPACED kolom voor resource scope
```

**Default Kubernetes Namespaces:**

- **default**: Standaard namespace voor gebruiker resources
- **kube-system**: Namespace voor Kubernetes systeem objecten
- **kube-node-lease**: Lease objects voor node heartbeats (failure detection)
- **kube-public**: Publiek leesbaar voor alle gebruikers (ook niet-authenticated)

### Namespace vs Cluster-Level Resources

**Namespaced Resources:**
- **Pods**: Bestaan binnen specifieke namespace context
- **Services**: Tied aan particular namespace
- **ConfigMaps/Secrets**: Namespace-specific configuration
- **Deployments**: Application deployments per namespace

**Cluster-Level Resources:**
- **Nodes**: Functionaliteit across entire cluster
- **Persistent Volumes**: Cluster-wide storage
- **ClusterRoles**: Cluster-wide RBAC permissions
- **Custom Resource Definitions**: Cluster-wide API extensions

### Namespace Operations

```bash
# 3. Namespace beheer
kubectl create namespace mynamespace        # Nieuwe namespace aanmaken
kubectl delete namespace/mynamespace --now  # Namespace verwijderen (met alle content)

# 4. Resources in specifieke namespace
kubectl run nginx --image=nginx                    # Standaard namespace (default)
kubectl -n mynamespace run nginx --image=nginx     # Specifieke namespace
kubectl get pods                                   # Huidige namespace context
kubectl -n mynamespace get pods                    # Specifieke namespace query

# 5. Context switching (namespace wijzigen)
kubectl config view                                                # Huidige configuratie bekijken
kubectl config set-context --current --namespace=mynamespace       # Switch naar mynamespace
kubectl config view                                                # Configuratie verificatie
kubectl get pods                                                   # Pods in nieuwe context (geen -n nodig)
kubectl config set-context --current --namespace=default          # Terug naar default namespace
```

### Command Shortcuts & Resource Discovery

**kubectl Short Names:**
```bash
# Veel Kubernetes commando's hebben kortere versies
kubectl get namespaces     # Volledig
kubectl get ns            # Verkort

kubectl get pods          # Volledig  
kubectl get po           # Verkort

# Alle short names ontdekken
kubectl api-resources     # Toont NAME, SHORTNAMES, APIVERSION, NAMESPACED, KIND
```

### Namespace Context Management

**Current Context Workflow:**
1. **Check context**: `kubectl config view` - zie huidige namespace
2. **Switch namespace**: `kubectl config set-context --current --namespace=<name>`
3. **Verify switch**: `kubectl config view` - bevestig wijziging
4. **Use resources**: Commands werken automatisch in nieuwe namespace
5. **Revert context**: Switch terug naar default als nodig

### Resource Isolation Patterns

**Development Workflow:**
```bash
# Namespace per omgeving
kubectl create namespace development
kubectl create namespace staging  
kubectl create namespace production

# Team-based namespaces
kubectl create namespace team-frontend
kubectl create namespace team-backend
kubectl create namespace team-data

# Project-based isolation
kubectl create namespace project-alpha
kubectl create namespace project-beta
```

### Namespace Cleanup & Deletion

**Deletion Behavior:**
- **Cascade deletion**: Namespace verwijdering verwijdert ALLE content
- **Pod cleanup**: Alle pods in namespace worden verwijderd
- **Resource cleanup**: Services, ConfigMaps, Secrets etc. verdwijnen
- **Irreversible**: Namespace deletion kan niet ongedaan gemaakt worden

```bash
# Voorzichtig met namespace deletion!
kubectl delete namespace/mynamespace --now    # Verwijdert namespace + alle inhoud
kubectl get all -A                           # Verificatie dat alles weg is
```

**Namespace tips:**

- `-A`: Alle namespaces bekijken (--all-namespaces)
- `-n <namespace>`: Specifieke namespace gebruiken
- `--current`: Huidige context wijzigen
- **Namespaces isoleren resources** van elkaar
- **Default namespace**: Wordt gebruikt als geen namespace gespecificeerd
- **Cluster-level resources**: Bestaan onafhankelijk van namespaces

**Best Practices:**

- **Use meaningful names**: development, staging, production
- **Team isolation**: Separate namespaces per team/project
- **Resource quotas**: Implementeer budgets per namespace
- **Context switching**: Gebruik voor workflow efficiency
- **Cleanup awareness**: Namespace deletion is destructive
- **RBAC integration**: Combine met role-based access control

**Multi-Tenant Architecture:**
- **Team separation**: Elke team krijgt eigen namespace
- **Resource budgets**: Quotas voorkomen resource conflicts
- **Access control**: RBAC policies per namespace
- **Name collision**: Zelfde resource names in verschillende namespaces
- **Independent scaling**: Teams kunnen onafhankelijk resources beheren

**Development Workflow Benefits:**
1. **Environment separation** - dev/staging/prod isolation
2. **Team collaboration** - Multiple teams, single cluster
3. **Resource budgeting** - CPU/memory limits per namespace
4. **Security boundaries** - Access control per namespace
5. **Simplified naming** - Unique within namespace, not cluster-wide
6. **Easy cleanup** - Delete entire environment met één command

---

#### 66. Kubernetes Deployments and ReplicaSets

**Doel:** Productie-ready applicatie deployment

**Context:** Kubernetes Deployments zijn resource objecten die declarative updates voor applicaties bieden. Ze beheren ReplicaSets die zorgen voor gewenst aantal pod replicas. Rolling updates zorgen voor zero-downtime deployments met predictable application lifecycle management.

### Deployment Key Features

**Replication:**
- **Instance management**: Bepaal hoeveel replicas van applicatie moeten draaien
- **Automatic recovery**: Bij pod crash/deletion worden nieuwe pods gestart
- **Desired state**: Deployment zorgt dat aantal pods matches expectation

**Updates:**
- **Configuration updates**: Applicatie en configuratie wijzigingen
- **Health maintenance**: Overall application health tijdens updates
- **Gradual rollout**: Phased updates voorkomen simultaneous instance updates
- **Availability stability**: Geen downtime tijdens update process

**Rollbacks:**
- **Version revert**: Terugkeren naar previous version bij problemen
- **Safe updates**: Minimaliseer downtime risico's
- **Automatic recovery**: Quick revert bij failure scenarios
- **Deployment history**: Track van alle deployment revisions

### Deployment Creation & Management

```bash
# 1. Deployment aanmaken
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml | tee nginx-deployment.yaml | kubectl apply -f -

# 2. Status bekijken
kubectl get deployment                       # Deployment overzicht
kubectl get replicaset                      # ReplicaSet status (automatisch gecreëerd)
kubectl get deployments -o wide             # Uitgebreide informatie met image details

# 3. Pod relaties bekijken
kubectl get pods -o wide                    # Pods carry ReplicaSet name + unique ID
```

### Rollout History & Tracking

```bash
# 4. Rollout geschiedenis
kubectl rollout history deployment/nginx                                       # Rollout geschiedenis bekijken
kubectl annotate deployment/nginx kubernetes.io/change-cause="initial deployment"  # Annotatie toevoegen voor tracking

# 5. Rollout history na annotatie
kubectl rollout history deployment/nginx                                       # Nu met descriptive change-cause
```

### Scaling Operations

```bash
# 6. Scaling
kubectl scale deployment/nginx --replicas=10; watch kubectl get pods -o wide   # Scale up naar 10 + live monitoring

# 7. YAML-based scaling
# Edit nginx-deployment.yaml (change replicas: 12)
kubectl apply -f nginx-deployment.yaml && kubectl rollout status deployment/nginx  # YAML apply + status monitoring
kubectl get deployment -o wide                                                      # Verify 12/12 ready
```

### Rolling Update Strategy

**Default Strategy Settings:**
- **maxSurge: 25%**: Kubernetes kan pods verhogen met 25% boven desired amount tijdens update
- **maxUnavailable: 25%**: Tot 25% van pods kan unavailable zijn tijdens update
- **Rolling strategy**: Maintains availability terwijl resource usage beperkt blijft

```yaml
# Deployment strategy (default)
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```

### Image Updates & Monitoring

```bash
# 8. Image updates met monitoring
kubectl set image deployment/nginx nginx=nginx:stable && kubectl rollout status deployment/nginx  # Image update + real-time monitoring
kubectl annotate deployment/nginx kubernetes.io/change-cause="change image to nginx:stable"       # Document change

# 9. Alternatieve image updates
kubectl set image deployment/nginx nginx=nginx:alpine && kubectl rollout status deployment/nginx  # Alpine update
kubectl annotate deployment/nginx kubernetes.io/change-cause="change image to nginx:alpine"       # Track change

# 10. ReplicaSet management bekijken
kubectl get replicaset -o wide                          # Multiple ReplicaSets (old + new)
kubectl describe deployment/nginx                       # Shows old/new ReplicaSets
```

### Advanced Image Testing

```bash
# 11. Multiple image rollouts
kubectl set image deployment/nginx nginx=nginx:perl && kubectl rollout status deployment/nginx    # Perl variant
kubectl annotate deployment/nginx kubernetes.io/change-cause="change image to nginx:perl"         # Document

# 12. ReplicaSet behavior
kubectl get replicaset -o wide                          # Old ReplicaSets remain (sidelined)
kubectl rollout history deployment/nginx               # Timing corresponds to revisions
```

### Failure Handling & Rollbacks

```bash
# 13. Failure simulation
kubectl set image deployment/nginx nginx=nginx:bananas && watch kubectl get pods -o wide         # Non-existent tag
kubectl annotate deployment/nginx kubernetes.io/change-cause="change image to nginx:bananas (failed)" # Document failure

# 14. ReplicaSet during failure
kubectl describe deployment/nginx                       # Shows failing new ReplicaSet vs working old

# 15. Emergency rollback
kubectl rollout undo deployment/nginx && kubectl rollout status deployment/nginx                 # Undo + status check
kubectl rollout history deployment/nginx               # Revision rearrangement (4th becomes 6th)

# 16. Specific revision rollback
kubectl rollout undo deployment/nginx --to-revision=1 && kubectl rollout status deployment/nginx # Rollback to specific
kubectl get replicaset -o wide                         # Original ReplicaSet reused
kubectl rollout history deployment/nginx               # Revision 1 becomes new revision (7)
```

### ReplicaSet Lifecycle Management

**ReplicaSet Behavior:**
- **Automatic creation**: Deployments automatisch creëren ReplicaSets
- **Name pattern**: ReplicaSet name = deployment name + identifier
- **Pod naming**: Pods carry ReplicaSet name + unique ID
- **Persistent storage**: Old ReplicaSets remain voor rollback purposes
- **Strategy compliance**: Rolling updates follow maxSurge/maxUnavailable rules

**Revision Management:**
- **History tracking**: Rollout history maintains revision sequence
- **Revision reuse**: Rollback to previous revision becomes new revision
- **ReplicaSet reuse**: Rolling back reuses existing ReplicaSets
- **Annotation importance**: change-cause annotations voor descriptive history

### Troubleshooting & Monitoring

```bash
# 17. Comprehensive status checking
kubectl describe deployment/nginx                       # Detailed deployment information including strategy
kubectl get deployment/nginx -o yaml                   # Complete YAML configuration
kubectl get replicaset -o wide                         # Image details per ReplicaSet
kubectl rollout history deployment/nginx               # Complete revision history
```

### Cleanup Operations

```bash
# 18. Complete cleanup
kubectl delete deployment/nginx --now                  # Removes deployment + all linked ReplicaSets
kubectl get all                                        # Verify complete cleanup
```

### Deployment Strategy Benefits

**Predictable Updates:**
- **Zero-downtime**: Rolling updates maintain service availability
- **Gradual rollout**: Phased approach prevents simultaneous failures
- **Resource efficiency**: maxSurge/maxUnavailable limits resource usage
- **Health checks**: Updates respect pod readiness voordat proceeding

**Production Stability:**
- **Rollback safety**: Quick revert to known-good state
- **Change tracking**: Annotation-based deployment history
- **Multiple strategies**: Rolling updates, recreate strategies available
- **ReplicaSet persistence**: Old versions available voor emergency rollback

**Deployment basics:**

- `scale`: Aantal pods aanpassen (horizontal scaling)
- `rollout`: Updates beheren (rolling updates voor zero-downtime)
- `undo`: Terugdraaien naar vorige versie (rollback)
- `watch`: Live updates bekijken (real-time monitoring)
- `annotate`: Metadata toevoegen voor betere tracking
- `describe`: Gedetailleerde troubleshooting informatie
- **ReplicaSet management**: Automatic creation en lifecycle management
- **Revision history**: Complete audit trail van deployment changes

**Production Readiness Features:**
- **Declarative updates**: YAML-based configuration management
- **Automatic scaling**: Desired state maintenance
- **Rolling strategy**: Configurable update behavior
- **Failure recovery**: Automatic rollback capabilities
- **Multi-version support**: Parallel ReplicaSet management
- **Change documentation**: Annotation-based change tracking

**Best Practices:**
- **Always annotate**: Document changes voor future reference
- **Monitor rollouts**: Use `kubectl rollout status` voor real-time feedback
- **Test updates**: Verify image availability before deployment
- **Strategic scaling**: Use appropriate maxSurge/maxUnavailable values
- **History management**: Maintain clear revision documentation
- **Rollback readiness**: Keep old ReplicaSets voor emergency recovery

---

#### 70. Kubernetes Services

**Doel:** Network abstraction en service discovery

**Context:** Kubernetes Services zijn het mechanisme voor het exposeren van applicaties die draaien op een set pods als network service. Services bieden service discovery, load balancing en externe toegang tot applicaties binnen het cluster.

### Service Types Overview

**Four Primary Service Types:**
1. **ClusterIP** - Default service, internal cluster access only
2. **NodePort** - External access via node IP addresses
3. **LoadBalancer** - Cloud provider load balancer integration
4. **ExternalName** - DNS CNAME alias for external services

**Additional Service Type:**
5. **Headless Service** - Direct pod access via DNS without proxy

### ClusterIP Service (Default)

**Characteristics:**
- **Internal IP**: Service met internal IP adres, alleen bereikbaar binnen cluster
- **Default type**: Automatisch gebruikt als geen type gespecificeerd
- **Service discovery**: DNS-based access via service names
- **Load balancing**: Automatic load balancing across pods

```bash
# 1. Deployment aanmaken met debug capabilities
kubectl create deployment nginx --image=spurin/nginx-debug --port=80 --replicas=3 --dry-run=client -o yaml | kubectl apply -f -

# 2. ClusterIP service exposure (default type)
kubectl expose deployment/nginx --dry-run=client -o yaml    # YAML preview
kubectl expose deployment/nginx                             # Create ClusterIP service

# 3. Service en endpoints verificatie
kubectl get service                                         # ClusterIP met internal IP
kubectl get endpoints                                       # IP addresses van pods
kubectl get pods -o wide                                    # Pod IPs matchen endpoints

# 4. Service details bekijken
kubectl describe service/nginx                              # Service configuration + endpoints
```

### ClusterIP Connectivity Testing

```bash
# 5. Direct ClusterIP access (from cluster node)
curl <CLUSTER_IP>                                          # Direct service access
# Multiple requests hit different pod instances

# 6. Port forwarding voor lokale toegang
kubectl port-forward service/nginx 8080:80                 # Localhost:8080 → service:80
# Browse naar http://localhost:8080

# 7. DNS-based service discovery
kubectl run -it --rm curl --image=curlimages/curl --restart=Never -- sh
curl nginx.default.svc.cluster.local                      # Full DNS naam
curl nginx                                                 # Short naam (DNS search path)
cat /etc/resolv.conf                                       # Shows search: default.svc.cluster.local
exit
```

### NodePort Service

**Characteristics:**
- **External access**: Accessible outside cluster via node IPs
- **Port allocation**: Kubernetes allocates high port (30000-32767 range)
- **Node availability**: Requires externally accessible node IP addresses
- **Dual ports**: Service port + NodePort port mapping

```bash
# 8. ClusterIP service verwijderen
kubectl delete service/nginx

# 9. NodePort service aanmaken
kubectl expose deployment/nginx --type=NodePort
kubectl get service                                         # Shows ClusterIP + NodePort ports
kubectl get nodes -o wide                                   # Node IP addresses

# 10. NodePort access testing
curl <NODE_IP>:<NODEPORT>                                  # External access via any node
# Load balancing across pods via NodePort
```

### LoadBalancer Service

**Characteristics:**
- **Cloud integration**: Requires cloud provider load balancer support
- **External IP**: Cloud provider provisions external IP address
- **Automatic scaling**: Load balancer adjusts to pod changes
- **Port specification**: Custom listening port en target port configuration

```bash
# 11. LoadBalancer service setup
kubectl delete service/nginx
kubectl expose deployment/nginx --type=LoadBalancer --port=8080 --target-port=80

# 12. LoadBalancer monitoring
kubectl get service                                         # Shows EXTERNAL-IP (pending → assigned)
watch curl <EXTERNAL_IP>:8080 2>/dev/null                 # Load balancing in action

# 13. Dynamic scaling demonstration
kubectl scale deployment/nginx --replicas=1               # Scale down
# Watch curl output - hits same pod consistently
kubectl scale deployment/nginx --replicas=5               # Scale up  
# Watch curl output - load balancing across multiple pods

# 14. Cleanup
kubectl delete deployment/nginx service/nginx
```

### ExternalName Service

**Characteristics:**
- **DNS CNAME**: Creates DNS alias for external services
- **No proxy**: Pure DNS resolution zonder traffic proxying
- **Flexible mapping**: Can point to cluster services of external domains
- **Dynamic switching**: Change backend via manifest edit

```bash
# 15. Demo setup met multiple deployments
kubectl create deployment nginx-red --image=spurin/nginx-red --replicas=2
kubectl create deployment nginx-blue --image=spurin/nginx-blue --replicas=2

# 16. ClusterIP services voor beide deployments
kubectl expose deployment/nginx-red
kubectl expose deployment/nginx-blue

# 17. ExternalName service aanmaken
kubectl create service externalname my-service --external-name=nginx-red.default.svc.cluster.local
kubectl get service                                         # Shows CNAME mapping

# 18. DNS CNAME testing
kubectl run -it --rm curl --image=curlimages/curl --restart=Never -- sh
curl nginx-red                                             # Direct access (red background)
curl nginx-blue                                            # Direct access (blue background)  
curl my-service                                            # CNAME access (red background)
nslookup my-service                                        # Shows CNAME record

# 19. Dynamic backend switching
# In new terminal:
kubectl edit service/my-service                            # Change nginx-red → nginx-blue
# Back in curl container:
nslookup my-service                                        # Now points to nginx-blue
curl my-service                                            # Blue background (backend switched)
exit

# 20. ExternalName cleanup
kubectl delete deployment/nginx-red deployment/nginx-blue service/nginx-red service/nginx-blue service/my-service
```

### Headless Service

**Characteristics:**
- **No ClusterIP**: ClusterIP set to "None" (hence "headless")
- **Direct pod access**: DNS resolves directly to pod IPs
- **Round-robin DNS**: Multiple pod IPs returned for service name
- **No proxy**: Direct connection to pods zonder service proxy

```bash
# 21. Headless service setup
kubectl create deployment nginx --image=spurin/nginx-debug --replicas=3
kubectl expose deployment/nginx --dry-run=client -o yaml | tee headless.yaml

# 22. YAML modification voor headless
# Edit headless.yaml: Add "clusterIP: None" under spec
```

**Headless YAML Structure:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  clusterIP: None          # This makes it headless
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
```

```bash
# 23. Headless service deployment
kubectl apply -f headless.yaml
kubectl get service                                         # ClusterIP shows <none>

# 24. Headless DNS behavior
kubectl run -it --rm curl --image=curlimages/curl --restart=Never -- sh
watch nslookup nginx                                        # Multiple IPs rotate (round-robin DNS)
curl nginx                                                  # Direct pod access (no proxy)
exit

# 25. Headless cleanup
kubectl delete deployment/nginx service/nginx
rm headless.yaml
```

### Service Types Comparison

**Traffic Flow Patterns:**

**Proxied Services (ClusterIP, NodePort, LoadBalancer):**
- **Request flow**: Client → Service → Pod (proxy layer)
- **Load balancing**: Service handles distribution
- **Single endpoint**: Service IP acts as single entry point
- **Session persistence**: Service manages connection handling

**Direct Services (Headless, ExternalName):**
- **Request flow**: Client → Pod (direct connection)
- **DNS resolution**: Multiple pod IPs returned
- **No proxy overhead**: Direct network connection
- **Client-side balancing**: Client chooses pod IP

### DNS Service Discovery

**Full DNS Namen Structure:**
```
<service-name>.<namespace>.svc.cluster.local
```

**DNS Search Path Benefits:**
- **Automatic resolution**: `nginx` resolves to `nginx.default.svc.cluster.local`
- **Cross-namespace**: Access services in other namespaces
- **Standard naming**: Consistent DNS naming convention
- **Service abstraction**: Application code uses service names, not IPs

### Service Selection & Endpoints

**Label Selector Mechanism:**
```yaml
# Deployment labels
metadata:
  labels:
    app: nginx

# Service selector  
spec:
  selector:
    app: nginx    # Matches pods with this label
```

**Endpoints Management:**
- **Automatic creation**: Kubernetes creëert endpoints object per service
- **Pod IP tracking**: Endpoints contain current pod IPs
- **Health monitoring**: Only healthy pods included in endpoints
- **Dynamic updates**: Endpoints update when pods change

### Production Service Strategies

**Development Environment:**
- **Port forwarding**: Local development access
- **ClusterIP**: Internal service testing
- **DNS testing**: Service discovery validation

**Staging Environment:**
- **NodePort**: External access voor testing
- **LoadBalancer**: Cloud environment testing
- **ExternalName**: External service integration

**Production Environment:**
- **LoadBalancer**: Production external access
- **Ingress**: HTTP/HTTPS routing (advanced topic)
- **Service mesh**: Advanced traffic management
- **Monitoring**: Service health en performance tracking

### Service Best Practices

**Service Design:**
- **Use descriptive names**: Service names should reflect functionality
- **Label consistency**: Consistente labeling strategy across deployments
- **Port naming**: Named ports voor better documentation
- **Health checks**: Implement readiness probes voor endpoint management

**Security Considerations:**
- **Network policies**: Restrict service access
- **TLS termination**: Secure communication channels
- **Service accounts**: Authentication for service access
- **RBAC integration**: Role-based access control

**Monitoring & Troubleshooting:**
- **Endpoint verificatie**: `kubectl get endpoints` voor connectivity
- **Service description**: `kubectl describe service` voor configuration
- **DNS testing**: `nslookup` binnen pods voor resolution
- **Port forwarding**: Local access voor debugging

**Service command reference:**

- `expose`: Create service from deployment/pod
- `port-forward`: Local access to services
- `get endpoints`: Show pod IPs behind service
- `describe service`: Detailed service configuration
- `edit service`: Dynamic service modification
- **Type specification**: `--type` voor service type selection
- **Port mapping**: `--port` en `--target-port` configuration
- **External name**: `--external-name` voor CNAME services

**Service Types Use Cases:**
- **ClusterIP**: Internal microservice communication
- **NodePort**: Development en testing access
- **LoadBalancer**: Production external access
- **ExternalName**: Database en external API integration
- **Headless**: StatefulSet en direct pod access scenarios

**DNS Benefits:**
- **Service discovery**: Applications find services by name
- **Location transparency**: Services can move without code changes
- **Load balancing**: Automatic traffic distribution
- **Health awareness**: Only healthy pods receive traffic
- **Namespace isolation**: Service names scoped to namespace

