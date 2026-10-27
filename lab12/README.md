# Lab 12: Sammanfattning och arkitektur


Målet med Lab 12 är att sammanfatta lösningen som jag har byggt under kursen och beskriva hur min Ansible-miljö är uppbyggd.

Jag beskriver också varför jag valde denna arkitektur och vilka delar som har varit viktiga under arbetet.

---

## 1. Min miljö

Jag har arbetat med tre Ubuntu maskiner.

### Control node

    milad-controlnode.local

Control node används för att köra Ansible.

Här finns bland annat:

- Ansible
- Inventory
- Playbooks
- Roles
- Templates
- Ansible Vault
- Dynamic Inventory

Control node ansluter till managed nodes via SSH.

---

### Webserver

    milad-node1.local
    IP: 10.6.69.114

Node1 används främst som webb och applikationsserver.

Under laborationerna har jag bland annat använt:

- Nginx
- Flask
- Gunicorn
- Python
- systemd

---

### Databasserver

    milad-node2.local
    IP: 10.6.69.107

Node2 används främst som databasserver.

Jag har bland annat installerat och konfigurerat:

- MariaDB
- Databaser
- Databasanvändare
- Tabeller
- Data

---


Min grundläggande Ansible miljö ser ut så här:

    Windows-dator
         |
         | SSH
         v
    milad-controlnode
         |
         | Ansible + SSH
         |
         +-----------------------+
         |                       |
         v                       v
    milad-node1             milad-node2
    Web / App               Database
    10.6.69.114             10.6.69.107

Control node styr båda servrarna med Ansible.

---

## 2. Inventory

Jag använder ett inventory för att dela upp servrarna i olika grupper.

Mitt inventory ser ut så här:

    [dbservers]
    milad-node2.local

    [webservers]
    milad-node1.local

    [servers]
    milad-node1.local
    milad-node2.local

Det gör att jag kan köra olika konfigurationer mot olika typer av servrar.

Exempel:

    ansible webservers -i ~/inventory.ini -m ping

eller:

    ansible dbservers -i ~/inventory.ini -m ping

Det är bättre än att skriva servernamnen direkt i varje Playbook.

---

## 3. Varför jag valde Ansible

Ansible passar bra för min miljö eftersom det är enkelt och agentless.

Jag behöver inte installera en separat Ansible-agent på node1 och node2.

Ansible använder SSH för att ansluta till servrarna.

Fördelarna i min miljö är:

- Enkel installation
- Ingen agent på managed nodes
- YAML är lätt att läsa
- Playbooks är tydliga
- Bra stöd för Linux
- Bra stöd för templates
- Bra stöd för secrets med Vault
- Enkelt att automatisera flera servrar

Eftersom min miljö bara består av några få servrar är Ansible ett bra val.

---

## 4. SSH

Kommunikationen mellan control node och managed nodes sker via SSH.

Jag konfigurerade SSH-nycklar så att Ansible kan ansluta till:

    milad-node1.local

och:

    milad-node2.local

Det gör automatiseringen enklare eftersom jag inte behöver skriva SSH-lösenord för varje anslutning.

---

## 5. Playbooks

Playbooks är en av de viktigaste delarna i Ansible.

Jag använder YAML-filer för att beskriva vilket tillstånd servrarna ska ha.

Exempel på saker jag har automatiserat:

- Installation av paket
- Installation av Nginx
- Installation av MariaDB
- Skapande av användare
- Skapande av filer
- Konfiguration av tjänster
- Start och restart av tjänster
- Deployment av applikationer

Istället för att göra samma arbete manuellt på varje server kan Ansible göra det automatiskt.

---

## 6. Roles

Jag har också arbetat med Ansible Roles.

Exempel:

    roles/
    ├── common/
    ├── webserver/
    └── database/

Roles gör att konfigurationen kan delas upp i mindre delar.

Till exempel kan en `webserver`-role innehålla allt som behövs för en webbserver.

Det gör koden:

- enklare att läsa
- enklare att återanvända
- enklare att underhålla

---

## 7. Templates och Jinja2

Jag har använt Jinja2 templates för att skapa dynamiska konfigurationsfiler.

Ett exempel var en webbsida där information från Ansible Facts visades.

Jag kunde automatiskt visa:

- hostname
- IP-adress
- operativsystem

Jag använde även templates i Lab 10 för Flask-applikationen, Nginx och systemd.

Det betyder att samma template kan användas med olika värden beroende på vilken server den körs på.

---

## 8. Handlers

Jag använde Ansible Handlers för att starta om tjänster när en konfigurationsfil ändrades.

Exempel:

    notify: Restart Nginx

Handler:

    - name: Restart Nginx
      ansible.builtin.service:
        name: nginx
        state: restarted

Fördelen är att tjänsten bara startas om när det verkligen behövs.

---

## 9. Ansible Facts

Ansible Facts används för att automatiskt samla information om servrarna.

Jag har bland annat använt Facts för att få information om:

- hostname
- IP-adress
- operativsystem
- kernel
- CPU
- RAM
- nätverksinterface

I Lab 8 använde jag denna information för att skapa automatiska rapporter för mina servrar.

---

## 10. CMDB och rapportering

I Lab 8 skapade jag en enkel CMDB-lösning.

Ansible samlade automatiskt information från:

    milad-node1.local

och:

    milad-node2.local

Sedan skapades Markdown-rapporter.

Rapporterna innehöll bland annat:

- Hostname
- OS
- Kernel
- CPU
- RAM
- IP-adress
- MAC-adress
- Interface
- Inventory groups
- Installerade paket
- Relationer mellan servrar

Det gjorde det möjligt att automatiskt dokumentera miljön.

---

## 11. Dynamic Inventory

I Lab 9 skapade jag ett Dynamic Inventory med Python.

Istället för ett vanligt statiskt inventory kunde Python-scriptet returnera information om mina servrar i JSON-format.

Jag kunde verifiera det med:

    ansible-inventory -i ~/lab9/dynamic_inventory.py --list

och:

    ansible-inventory -i ~/lab9/dynamic_inventory.py --graph

Jag testade också:

    ansible all -i ~/lab9/dynamic_inventory.py -m ping

Båda servrarna svarade.

Dynamic Inventory är användbart i större eller mer dynamiska miljöer där servrar kan skapas och tas bort ofta.

---

## 12. Ansible Vault

Jag använde Ansible Vault för att skydda känslig information.

Exempel på information som inte bör ligga i klartext:

- Databaslösenord
- API-nycklar
- Secrets

Vault-filen skapades med:

    ansible-vault create group_vars/all/vault.yml

Den krypterade filen kunde sedan användas från en Playbook.

Playbooken kördes med:

    ansible-playbook -i ~/inventory.ini site.yml -K --ask-vault-pass

På detta sätt behöver lösenordet inte ligga öppet i Playbooken.

---

## 13. Flerskiktsapplikation

I Lab 10 byggde jag den mest kompletta miljön under kursen.

Arkitekturen blev:

    Webbläsare
        |
        v
    Nginx
    milad-node1
        |
        v
    Gunicorn
        |
        v
    Flask
        |
        v
    MariaDB
    milad-node2

Node1 kör:

- Nginx
- Gunicorn
- Flask

Node2 kör:

- MariaDB

---

## 14. Dynamisk databasadress

Jag skrev inte databasens IP-adress manuellt i applikationen.

Ansible hämtade automatiskt IP-adressen från databasservern.

Jag använde:

    hostvars[groups['dbservers'][0]]['ansible_default_ipv4']['address']

Det gjorde att webbservern automatiskt fick rätt IP-adress till databasservern.

I min miljö blev adressen:

    10.6.69.107

Det gör konfigurationen mer flexibel eftersom IP-adressen inte behöver hårdkodas.

---

## 15. Nginx, Gunicorn och Flask

I Lab 10 användes flera lager.

### Nginx

Nginx tar emot HTTP-trafik på:

    port 80

### Gunicorn

Nginx skickar trafiken vidare till Gunicorn på:

    127.0.0.1:8000

### Flask

Flask-applikationen tar emot requesten och ansluter sedan till MariaDB.

Flödet blir:

    Browser
       |
       v
    Nginx
       |
       v
    Gunicorn
       |
       v
    Flask
       |
       v
    MariaDB

Detta är bättre än att exponera Flask-applikationen direkt.

---

## 16. systemd

Flask-applikationen körs som en systemd service.

Tjänsten heter:

    lab10.service

Jag verifierade tjänsten med:

    systemctl is-active lab10

Resultatet blev:

    active

Fördelen med systemd är att applikationen kan:

- starta automatiskt
- köras i bakgrunden
- startas om
- övervakas som en vanlig Linux-tjänst

---

## 17. Databasen

MariaDB körs på:

    milad-node2.local

Applikationen använder databasen:

    lab10db

och tabellen:

    messages

Jag lade in testdata:

    Hej från MariaDB på milad-node2!

Flask-applikationen hämtar sedan denna information från databasen.

---

## 18. Verifiering av hela lösningen

Jag verifierade först Flask health endpoint:

    curl http://milad-node1.local/health

Resultat:

    OK

Jag testade sedan hela applikationen:

    curl http://milad-node1.local

Applikationen visade:

    Webbserver: milad-node1

    Databasserver: 10.6.69.107

    Data från MariaDB: Hej från MariaDB på milad-node2!

    Status: Databasanslutningen fungerar!

Jag testade också lösningen i webbläsaren.

Det verifierade hela kedjan:

    Browser
       ↓
    Nginx
       ↓
    Gunicorn
       ↓
    Flask
       ↓
    MariaDB

---

## 19. Säkerhet

Jag har använt flera enkla säkerhetslösningar i miljön.

### SSH-nycklar

Ansible använder SSH-nycklar istället för att hela tiden kräva SSH-lösenord.

### Ansible Vault

Databaslösenord lagras krypterat.

### Environment-fil

Databaslösenordet skickas till Flask genom en environment-fil istället för att skrivas direkt i Python-koden.

Filen på servern har begränsade rättigheter.

### Separata servrar

Webbserver och databas ligger på olika servrar.

Det ger en tydligare separation mellan applikation och data.

---

## 20. Ansible jämfört med Puppet

I Lab 11 jämförde jag Ansible med Puppet.

Den största skillnaden är deras arbetssätt.

### Ansible

- Agentless
- Push
- SSH
- YAML
- Kör Playbooks när de startas

### Puppet

- Agent-based
- Vanligtvis Pull
- Puppet DSL
- Kontrollerar tillstånd regelbundet
- Bra på automatisk korrigering av Configuration Drift

För min miljö föredrar jag Ansible eftersom den är liten och Ansible är enkelt att använda utan att installera agenter på alla servrar.

---

## 21. Varför jag valde denna arkitektur

Jag valde att dela upp miljön i olika servrar eftersom det ger en tydligare struktur.

Node1 ansvarar för:

    Webb + Applikation

Node2 ansvarar för:

    Databas

Control node ansvarar för:

    Automation

Det betyder att varje server har ett tydligt ansvar.

Arkitekturen blir:

    Control Node
         |
         | Automation
         |
         +----------------+
         |                |
         v                v
    Web / App          Database
    Node1              Node2

Det gör miljön enklare att:

- förstå
- felsöka
- automatisera
- dokumentera
- utveckla vidare

---

## 22. Fördelar med lösningen

Några av de största fördelarna med min lösning är:

- Konfigurationen är automatiserad.
- Samma Playbook kan köras flera gånger.
- Servrarna är uppdelade efter funktion.
- Secrets kan skyddas med Vault.
- Templates gör konfigurationen dynamisk.
- Facts gör att serverinformation kan hämtas automatiskt.
- Roles gör koden mer strukturerad.
- Dynamic Inventory kan användas när miljön växer.
- Applikationen körs som en riktig Linux-tjänst.
- Webbserver och databas kommunicerar automatiskt.

---

## 23. Möjliga förbättringar

Om miljön skulle utvecklas vidare skulle jag kunna förbättra den med exempelvis:

- HTTPS
- Brandväggsregler
- Fler webbnoder
- Load balancer
- Monitoring
- Central loggning
- Backup av databasen
- CI/CD
- Containers
- Kubernetes
- Cloud-infrastruktur

I en större produktionsmiljö skulle det också vara viktigt med bättre hantering av secrets, övervakning och redundans.

---

## 24. Vad jag har lärt mig

Under laborationerna har jag fått arbeta praktiskt med flera delar av Configuration Management.

Jag har lärt mig att använda:

- Ansible Inventory
- Ad-hoc commands
- Playbooks
- Variables
- Facts
- Roles
- Jinja2 Templates
- Handlers
- Ansible Vault
- MariaDB
- Dynamic Inventory
- CMDB-rapportering
- Flask
- Gunicorn
- Nginx
- systemd

Jag har också fått en bättre förståelse för hur flera servrar kan automatiseras tillsammans.

En viktig sak jag har lärt mig är att automation inte bara handlar om att installera paket.

Det handlar också om att skapa en lösning som är:

- repeterbar
- tydlig
- säker
- dokumenterad
- enkel att verifiera

---

## 25. Slutsats

Under kursen har jag gått från enkla Ansible-kommandon till att automatisera en flerskiktsapplikation med flera servrar.

Min slutliga miljö består av:

    milad-controlnode.local
          |
          | Ansible
          |
          +----------------------+
          |                      |
          v                      v
    milad-node1.local      milad-node2.local
    Web / Application     Database
          |                      |
    Nginx + Gunicorn             |
          |                      |
        Flask ----------------> MariaDB

Control node hanterar konfigurationen.

Node1 kör webb- och applikationslagret.

Node2 kör databasen.

Ansible automatiserar installation, konfiguration och deployment.

Jag har även använt Vault för secrets, Dynamic Inventory, Facts, Roles, Templates och Handlers.

Den viktigaste delen för mig har varit att kunna köra automationen och sedan verifiera att resultatet faktiskt fungerar.

Den slutliga verifieringen i Lab 10 visade att webbapplikationen på node1 kunde ansluta till MariaDB på node2 och hämta riktig data.

Det visar hur Configuration Management kan användas för att automatisera en komplett miljö istället för att konfigurera varje server manuellt.