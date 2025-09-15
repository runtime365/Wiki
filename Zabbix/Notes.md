









## 9. Zabbix Agent Installeren op Hetzelfde Netwerk als Zabbix Server

### Overzicht
We gaan nu een Zabbix agent installeren op Ubuntu op hetzelfde netwerk als de Zabbix server. In onze huidige Zabbix setup hebben we één host op de Zabbix server zelf (127.0.0.1:10050), maar nu voegen we een aparte server toe.

### Nieuwe Server Opzetten

**DigitalOcean Droplet Aanmaken:**
1. Ga naar DigitalOcean en maak een nieuwe droplet aan
2. Gebruik dezelfde regio als je Zabbix server (bijv. Frankfurt)
3. Kies Ubuntu 24.04 LTS
4. Selecteer de goedkoopste optie (voor demonstratie doeleinden)
5. Gebruik SSH key of stel een wachtwoord in
6. Noem de server "host-a"
7. Maak de droplet aan

**Netwerk Configuratie:**
- **Nieuwe server:** host-a
- **Public IP:** [nieuw IP adres van DigitalOcean]
- **Private IP:** 10.114.0.3 (voorbeeld)
- **Zabbix server private IP:** 10.114.0.2

### SSH Verbinding Opzetten

**In PuTTY:**
1. Voer het nieuwe IP adres in
2. Noem de sessie "host-a"
3. Sla de configuratie op
4. Pas het lettertype aan voor betere leesbaarheid
5. Voeg SSH certificaat toe (indien van toepassing)
6. Sla opnieuw op en maak verbinding

### Zabbix Agent Installatie

**Stap 1: Repository Toevoegen**
```bash
# Download en installeer Zabbix repository
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-2+ubuntu24.04_all.deb
dpkg -i zabbix-release_7.0-2+ubuntu24.04_all.deb
apt update
```

**Stap 2: Alleen Agent Installeren**
```bash
# Installeer alleen de Zabbix agent (niet server/frontend)
apt install zabbix-agent
```

**Stap 3: Service Status Controleren**
```bash
# Controleer of de agent draait
service zabbix-agent status
```

De agent start automatisch maar is nog niet correct geconfigureerd.

### Agent Configuratie

**Configuratiebestand Bewerken:**
```bash
nano /etc/zabbix/zabbix_agentd.conf
```

**Belangrijke Parameters:**

1. **Server Parameter:**
   - Zoek naar `Server=127.0.0.1`
   - Wijzig naar het private IP van je Zabbix server
   - Bijvoorbeeld: `Server=10.114.0.2`
   - Dit bepaalt van welk IP passieve checks geaccepteerd worden

2. **Hostname Parameter:**
   - Zoek naar `Hostname=Zabbix server`
   - Wijzig naar `Hostname=host-a`
   - **Let op:** Dit moet exact overeenkomen met wat je later in de Zabbix UI invoert
   - Case-sensitive en spaties zijn toegestaan

3. **ServerActive Parameter (optioneel):**
   - Hoewel we nu alleen passieve checks gebruiken, kun je dit instellen voor toekomstig gebruik
   - Stel in op het IP van de Zabbix server: `ServerActive=10.114.0.2`

**Configuratie Opslaan:**
```bash
# Sla het bestand op (Ctrl+X, Y, Enter)
# Herstart de service
service zabbix-agent restart
service zabbix-agent status
```

### Host Toevoegen in Zabbix UI

**Navigatie:**
Data Collection → Hosts → Create host

**Host Configuratie:**
1. **Host name:** `host-a` (exact zoals in agent config)
2. **Templates:** Selecteer "Linux by Zabbix agent" (bevat passieve checks)
3. **Host groups:** Voeg toe aan "Linux servers"
4. **Interfaces:**
   - Type: Agent
   - IP adres: `10.114.0.3` (private IP van host-a)
   - Poort: `10050`
   - Monitored by: Server

**Verificatie:**
- Ga naar Monitoring → Latest data
- Filter op "host-a"
- Je zou binnen enkele minuten data moeten zien

### Connectiviteit Testen

**Vanaf Zabbix Server:**

1. **Ping Test:**
```bash
ping 10.114.0.3
```

2. **Port Connectiviteit Test:**
```bash
telnet 10.114.0.3 10050
```

Als beide testen slagen, zou de Zabbix monitoring moeten werken.

### Firewall Configuratie (Optioneel)

**DigitalOcean Firewall Aanmaken:**

1. **Basis Regels:**
   - SSH (poort 22): Sta je eigen IP range toe
   - Zabbix Agent (poort 10050): Sta alleen Zabbix server toe

2. **Custom Regel voor Zabbix:**
   - Protocol: TCP
   - Poort: 10050
   - Source: 10.114.0.2 (Zabbix server IP)

3. **ICMP (Optioneel):**
   - Voor ping functionaliteit
   - Source: 10.114.0.2

### Troubleshooting

**Veelvoorkomende Problemen:**

1. **Rode Availability Icon:**
   - Controleer agent configuratie
   - Herstart zabbix-agent service
   - Verwijder en maak host opnieuw aan in UI

2. **Geen Data:**
   - Verificeer IP adressen (gebruik private IPs)
   - Controleer hostname matching
   - Test connectiviteit met ping/telnet

3. **Firewall Issues:**
   - Zorg dat poort 10050 open is
   - Controleer of juiste IP ranges toegestaan zijn

### Passieve vs Actieve Checks

**Passieve Checks (huidige setup):**
- Zabbix server vraagt data op van agent
- Agent reageert op verzoeken
- Server parameter bepaalt wie mag vragen

**Actieve Checks (toekomstige setup):**
- Agent stuurt data proactief naar server
- Agent draait op eigen timer
- ServerActive parameter bepaalt waar naartoe te sturen

### Resultaat

Na succesvolle configuratie heb je:
- Twee hosts in je Zabbix environment
- Monitoring data van beide servers
- Grafieken en dashboards beschikbaar

Dit vormt de basis voor het uitbreiden naar hosts op verschillende netwerken in volgende stappen.













