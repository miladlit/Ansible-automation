# Lab 11 – Jämförelse mellan Ansible och Puppet


Målet med labben var att jämföra Ansible med Puppet och förstå skillnaden mellan olika Configuration Management verktyg.

Fokus ligger på:

- Push vs Pull
- Agentless vs Agent-based
- Exekveringsordning
- Syntax
- Fördelar och nackdelar

---

## 1. Push vs Pull

Ansible använder främst en Push-modell.

Det betyder att kontrollnoden startar anslutningen till målnoderna och skickar ut konfigurationen när en Playbook körs.

Ansible använder vanligtvis SSH och behöver ingen separat agent installerad på målnoderna.

Puppet använder istället normalt en Pull-modell.

Det betyder att Puppet-agenten körs på målnoden och regelbundet kontaktar Puppet-servern för att hämta den konfiguration som ska gälla.

Skillnaden kan sammanfattas så här:

    Ansible:
    Control node -> SSH -> Managed node

    Puppet:
    Puppet Agent -> Puppet Server -> Hämtar önskad konfiguration

Fördel med Ansible är att det är enkelt att komma igång eftersom ingen agent behöver installeras.

Fördel med Puppet är att agenten regelbundet kan kontrollera serverns tillstånd och automatiskt rätta ändringar som inte stämmer med den önskade konfigurationen.

---

## 2. Exekveringsordning

Ansible kör normalt sina tasks uppifrån och ner i den ordning de är skrivna i Playbooken.

Exempel:

    - Installera Nginx
    - Kopiera konfiguration
    - Starta Nginx

Ansible kör alltså först installationen, sedan konfigurationen och sist tjänsten.

Om man ändrar ordningen på tasks kan resultatet påverkas.

Till exempel kan det bli problem om man försöker starta Nginx innan paketet är installerat.

Puppet fungerar annorlunda.

Puppet använder ett deklarativt språk och bygger en så kallad Resource Graph.

Det betyder att Puppet tittar på resurser och deras beroenden istället för att bara följa ordningen i filen.

Exempel:

En tjänst kan vara beroende av att ett paket är installerat.

Puppet kan då förstå att paketet måste installeras före tjänsten även om resurserna står i en annan ordning i filen.

Skillnaden är därför:

    Ansible:
    Task-ordningen är viktig.

    Puppet:
    Beroenden mellan resurser är viktigare än ordningen i filen.

---

## 3. Configuration Drift

Configuration Drift betyder att en server ändras från det tillstånd som konfigurationen säger att den ska ha.

Exempel:

En server ska enligt koden ha Nginx igång.

Någon loggar in manuellt och stoppar Nginx.

Med Ansible kommer detta normalt inte rättas förrän en Playbook körs igen.

Med Puppet körs agenten regelbundet och kan upptäcka att servern inte längre har rätt tillstånd.

Puppet kan då automatiskt starta Nginx igen.

Det gör Puppet bra för miljöer där man vill ha kontinuerlig kontroll av serverns tillstånd.

---

## 4. Jämförelse mellan Ansible och Puppet

| Egenskap | Ansible | Puppet |
|---|---|---|
| Arkitektur | Agentless | Agent-based |
| Modell | Push | Pull |
| Kommunikation | SSH / WinRM | Puppet Agent och Puppet Server |
| Syntax | YAML | Puppet DSL |
| Exekvering | Tasks körs normalt uppifrån och ner | Resource Graph och beroenden |
| Agent på målnod | Nej | Ja |
| Configuration Drift | Rättas när Playbook körs | Kan rättas automatiskt regelbundet |
| Installation | Relativt enkel | Kräver mer infrastruktur |
| Inlärning | Relativt enkel YAML | Eget Puppet DSL |
| Passar bra för | Automation, deployment och enkel serverkonfiguration | Kontinuerlig state enforcement och stora miljöer |

---

## 5. Fördelar med Ansible

Fördelar:

- Ingen agent behövs på managed nodes.
- Använder SSH som redan finns på Linux-servrar.
- YAML är relativt enkelt att läsa.
- Enkelt att komma igång.
- Bra för automation och deployment.
- Playbooks är lätta att följa.
- Passar bra i DevOps och CI/CD-flöden.

Nackdelar:

- Configuration Drift rättas normalt inte automatiskt hela tiden.
- Kontrollnoden måste starta körningen.
- Vid väldigt många servrar kan många SSH-anslutningar skapa mer belastning.

---

## 6. Fördelar med Puppet

Fördelar:

- Kontrollerar serverns tillstånd regelbundet.
- Bra på att upptäcka och rätta Configuration Drift.
- Passar bra i stora miljöer.
- Har tydlig state management.
- Beroenden mellan resurser kan definieras tydligt.

Nackdelar:

- Kräver en Puppet-agent på servrarna.
- Kräver mer installation och administration.
- Puppet DSL är ett separat språk som måste läras.
- Mer komplex miljö än en enkel Ansible-installation.

---

## 7. Ansible Push jämfört med Puppet Pull

Den största skillnaden är vem som startar konfigurationen.

Med Ansible startar kontrollnoden arbetet:

    Control node
        |
        | Push via SSH
        v
    Managed node

Med Puppet startar agenten på servern kontrollen:

    Managed node
        |
        | Pull
        v
    Puppet Server

Ansible passar därför bra när man vill bestämma exakt när en deployment eller konfiguration ska köras.

Puppet passar bra när man vill att servrar kontinuerligt ska kontrollera och korrigera sitt eget tillstånd.

---

## 8. Skillnad i kodens ordning

I Ansible är ordningen på tasks viktig.

Exempel:

    Task 1: Installera Nginx
    Task 2: Lägg in konfiguration
    Task 3: Starta Nginx

Om man flyttar Task 3 före Task 1 kan det bli problem eftersom Nginx ännu inte är installerat.

I Puppet spelar ordningen i själva filen mindre roll.

Puppet bygger istället en beroendegraf.

Om en tjänst är beroende av ett paket kan Puppet förstå vilken resurs som behöver hanteras först.

---

## 9. Slutsats

Ansible och Puppet löser liknande problem men arbetar på olika sätt.

Ansible är agentless och använder en Push-modell över SSH.

Det gör Ansible enkelt att installera och använda, särskilt i mindre och medelstora miljöer eller i DevOps-flöden där automation körs vid behov.

Puppet använder en agentbaserad Pull-modell.

Puppet-agenten kontrollerar regelbundet serverns tillstånd och kan automatiskt rätta Configuration Drift.

Puppet passar därför bra i stora miljöer där man vill ha kontinuerlig kontroll av konfigurationen.

För min labbmiljö passar Ansible bra eftersom jag endast har några få servrar och redan använder SSH mellan control node och managed nodes.

Ansible har också varit enkelt att kombinera med:

- Playbooks
- Roles
- Jinja2
- Ansible Vault
- Dynamic Inventory
- Deployment

Labben gav mig en bättre förståelse för att olika Configuration Management-verktyg kan lösa samma problem men använda olika arkitektur och arbetssätt.