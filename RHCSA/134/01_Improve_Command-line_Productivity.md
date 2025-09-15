# Hoofdstuk 1: Verbeter Commandoregel Productiviteit

## Doel
Voer commando's efficiënter uit door geavanceerde functies van de Bash shell, shell scripts en verschillende Red Hat Enterprise Linux utilities te gebruiken.

## Doelstellingen
• Voer commando's efficiënter uit door geavanceerde functies van de Bash shell, shell scripts en verschillende Red Hat Enterprise Linux utilities te gebruiken.
• Voer repetitieve taken uit met for loops, evalueer exit codes van commando's en scripts, voer tests uit met operators en creëer conditionele structuren met if statements.
• Creëer reguliere expressies om data te matchen, pas reguliere expressies toe op tekstbestanden met het grep commando en gebruik grep om bestanden en data van piped commando's te doorzoeken.

## Secties
• Schrijf Eenvoudige Bash Scripts (en Begeleide Oefening)
• Loops en Conditionele Constructies in Scripts (en Begeleide Oefening)
• Match Tekst in Commando Output met Reguliere Expressies (en Begeleide Oefening)

---

## Schrijf Eenvoudige Bash Scripts

### Doelstellingen
Voer commando's efficiënter uit door geavanceerde functies van de Bash shell, shell scripts en verschillende Red Hat Enterprise Linux utilities te gebruiken.

### Creëer en Voer Bash Shell Scripts Uit

Je kunt veel eenvoudige, algemene systeembeheer taken uitvoeren door commandoregel tools te gebruiken. Taken met meer complexiteit vereisen vaak het aan elkaar koppelen van meerdere commando's die resultaten tussen elkaar doorgeven. Door de Bash shell omgeving en scripting functies te gebruiken, kun je Linux commando's combineren tot shell scripts om repetitieve real-world problemen op te lossen.

Een Bash shell script is een uitvoerbaar bestand dat een lijst van commando's bevat, en mogelijk met programmeerlogica om besluitvorming in de algehele taak te controleren. Wanneer goed geschreven, is een shell script een krachtige commandoregel tool op zichzelf, en je kunt het gebruiken met andere scripts.

Shell scripting vaardigheid is essentieel voor systeembeheerders in elke operationele omgeving. Je kunt shell scripts gebruiken om de efficiëntie en nauwkeurigheid van routine taak voltooiing te verbeteren.

Hoewel je elke teksteditor kunt gebruiken, begrijpen geavanceerde editors zoals vim of emacs Bash shell syntax en kunnen zij kleurgecodeerde highlighting bieden. Deze highlighting helpt bij het identificeren van veelvoorkomende scripting fouten zoals onjuiste syntax, ongepaarde aanhalingstekens, haakjes, vierkante haken en accolades, en andere structurele fouten.

### Specificeer de Commando Interpreter

De eerste regel van een script begint met de `#!` notatie, die vaak she-bang of hash-bang wordt genoemd, naar de namen van die twee karakters, sharp of hash en bang. Deze notatie is een interpreter directive die de commando interpreter en optionele commando opties aangeeft die nodig zijn om de resterende regels van het bestand te verwerken. Voor normale Bash syntax script bestanden is de eerste regel deze directive:

```bash
#!/usr/bin/bash
```

### Voer een Bash Shell Script Uit

Een shell script bestand moet execute permissions hebben om het als een gewoon commando uit te voeren. Gebruik het `chmod` commando om de bestand permissions te wijzigen. Gebruik het `chown` commando, indien nodig, om execute permission alleen voor specifieke gebruikers of groepen te verlenen.

Als het script is opgeslagen in een directory die vermeld staat in de shell's PATH environmental variable, dan kun je het shell script uitvoeren door alleen zijn bestandsnaam te gebruiken, vergelijkbaar met het uitvoeren van gecompileerde commando's. Omdat PATH parsing het eerste matchende bestand uitvoert dat gevonden wordt, vermijd altijd het gebruiken van bestaande commando namen om je script bestanden te benoemen.

Als een script niet in een PATH directory staat, voer het script uit met zijn absolute pad naam, die bepaald kan worden door het bestand te bevragen met het `which` commando. Alternatief, voer een script uit in je huidige werkdirectory door de `.` directory prefix te gebruiken, zoals `./scriptnaam`.

```bash
[user@host ~]$ which hello
~/bin/hello
[user@host ~]$ echo $PATH
/home/user/.local/bin:/home/user/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin
```

### Quote Speciale Karakters

Een aantal karakters en woorden hebben speciale betekenis voor de Bash shell. Wanneer je deze karakters nodig hebt voor hun letterlijke waarden, in plaats van voor hun speciale betekenissen, escape je ze in het script. Gebruik het backslash karakter (`\`), enkele aanhalingstekens (`''`), of dubbele aanhalingstekens (`""`) om de speciale betekenis van deze karakters te verwijderen (of escapen).

Het backslash karakter verwijdert de speciale betekenis van het enkele karakter dat onmiddellijk volgt op de backslash. Bijvoorbeeld, om het `echo` commando te gebruiken om de `# niet een comment` letterlijke string weer te geven, mag het `#` hash karakter niet geïnterpreteerd worden als een comment.

Het volgende voorbeeld toont het backslash karakter (`\`) dat het hash karakter wijzigt zodat het niet geïnterpreteerd wordt als een comment.

```bash
[user@host ~]$ echo # niet een comment
[user@host ~]$ echo \# niet een comment
# niet een comment
```

Om meer dan één karakter in een tekst string te escapen, gebruik ofwel het backslash karakter meerdere keren of omsluit de hele string in enkele aanhalingstekens (`''`) om letterlijk te interpreteren. Enkele aanhalingstekens behouden de letterlijke betekenis van alle karakters die zij omsluiten.

```bash
[user@host ~]$ echo # niet een comment #
[user@host ~]$ echo \# niet een comment #
# niet een comment
[user@host ~]$ echo \# niet een comment \#
# niet een comment #
[user@host ~]$ echo '# niet een comment #'
# niet een comment #
```

Gebruik dubbele aanhalingstekens om globbing (bestandsnaam patroon matching) en shell expansion te onderdrukken, maar sta nog steeds commando en variabele substitutie toe. Variabele substitutie is conceptueel identiek aan commando substitutie, maar kan optionele brace syntax gebruiken.

Gebruik enkele aanhalingstekens om alle omsloten tekst letterlijk te interpreteren. Naast het onderdrukken van globbing en shell expansion, zorgen enkele aanhalingstekens er ook voor dat de shell commando en variabele substitutie onderdrukt.

```bash
[user@host ~]$ var=$(hostname -s); echo $var
host
[user@host ~]$ echo "***** hostname is ${var} *****"
***** hostname is host *****
[user@host ~]$ echo Je gebruikersnaam variabele is \$USER.
Je gebruikersnaam variabele is $USER.
[user@host ~]$ echo "Zal variabele $var evalueren naar $(hostname -s)?"
Zal variabele host evalueren naar host?
[user@host ~]$ echo 'Zal variabele $var evalueren naar $(hostname -s)?'
Zal variabele $var evalueren naar $(hostname -s)?
[user@host ~]$ echo "\"Hallo, wereld\""
"Hallo, wereld"
[user@host ~]$ echo '"Hallo, wereld"'
"Hallo, wereld"
```

### Bied Output van een Shell Script

Het `echo` commando toont willekeurige tekst door de tekst als argument aan het commando door te geven. Standaard wordt de tekst naar standard output (STDOUT) gestuurd, maar je kunt tekst elders naartoe sturen door output redirection te gebruiken.

```bash
[user@host ~]$ cat ~/bin/hello
#!/usr/bin/bash
echo "Hallo, wereld"
[user@host ~]$ hello
Hallo, wereld
```

**Note:** Deze gebruiker kan hello uitvoeren bij de prompt omdat de `~/bin` (`/home/user/bin`) directory in de gebruiker's PATH variabele staat en het hello script executable permission heeft. De PATH parser zal het script eerst vinden, als geen ander uitvoerbaar bestand genaamd hello wordt gevonden in eerdere PATH directories.

Het `echo` commando wordt veel gebruikt in shell scripts om informatieve of foutmeldingen weer te geven. Berichten zijn een behulpzame indicator van een script's voortgang, en kunnen naar standard output, standard error gericht worden, of omgeleid naar een log bestand voor archivering.

```bash
[user@host ~]$ cat ~/bin/hello
#!/usr/bin/bash
echo "Hallo, wereld"
echo "FOUT: Houston, we hebben een probleem." >&2
[user@host ~]$ hello 2> hello.log
Hallo, wereld
[user@host ~]$ cat hello.log
FOUT: Houston, we hebben een probleem.
```

Het `echo` commando is ook behulpzaam voor het debuggen van een problematisch shell script. Het toevoegen van echo statements in een script, om variabele waarden en andere runtime informatie weer te geven, kan helpen verduidelijken hoe een script zich gedraagt.

---

## Begeleide Oefening: Schrijf Eenvoudige Bash Scripts

### Doelstellingen
• Schrijf en voer een eenvoudig Bash script uit.
• Redirect de output van een eenvoudig Bash script naar een bestand.

### Voorbereiding
Als de student gebruiker op de workstation machine, gebruik het lab commando om je systeem voor te bereiden voor deze oefening.

```bash
[student@workstation ~]$ lab start console-write
```

### Instructies

**1. Log in op de servera machine als de student gebruiker.**

```bash
[student@workstation ~]$ ssh student@servera
[student@servera ~]$
```

**2. Creëer en voer een eenvoudig Bash script uit.**

2.1. Gebruik het `vim` commando om het `firstscript.sh` bestand onder je home directory te creëren.

```bash
[student@servera ~]$ vim firstscript.sh
```

2.2. Voeg de volgende tekst in en sla het bestand op:

```bash
#!/usr/bin/bash
echo "Dit is mijn eerste bash script" > ~/output.txt
echo "" >> ~/output.txt
echo "#####################################################" >> ~/output.txt
```

2.3. Gebruik het `sh` commando om het script uit te voeren.

```bash
[student@servera ~]$ sh firstscript.sh
```

2.4. Bekijk het output bestand dat het script genereerde.

```bash
[student@servera ~]$ cat output.txt
Dit is mijn eerste bash script

#####################################################
```

**3. Voeg meer commando's toe aan het firstscript.sh script, voer het uit en bekijk de output.**

3.1. Gebruik de Vim teksteditor om het firstscript.sh script te bewerken:

```bash
#!/usr/bin/bash
#
echo "Dit is mijn eerste bash script" > ~/output.txt
echo "" >> ~/output.txt
echo "#####################################################" >> ~/output.txt
echo "LIJST BLOK APPARATEN" >> ~/output.txt
echo "" >> ~/output.txt
lsblk >> ~/output.txt
echo "" >> ~/output.txt
echo "#####################################################" >> ~/output.txt
echo "BESTANDSSYSTEEM VRIJE RUIMTE STATUS" >> ~/output.txt
echo "" >> ~/output.txt
df -h >> ~/output.txt
echo "#####################################################" >> ~/output.txt
```

3.2. Maak het firstscript.sh bestand uitvoerbaar met het `chmod` commando.

```bash
[student@servera ~]$ chmod a+x firstscript.sh
```

3.3. Voer het firstscript.sh script uit.

```bash
[student@servera ~]$ ./firstscript.sh
```

3.4. Bekijk het output bestand dat het script genereerde.

```bash
[student@servera ~]$ cat output.txt
```

**4. Verwijder de oefening bestanden en keer terug naar de workstation machine.**

```bash
[student@servera ~]$ rm firstscript.sh output.txt
[student@servera ~]$ exit
[student@workstation ~]$ lab finish console-write
```

---

## Loops en Conditionele Constructies in Scripts

### Doelstellingen
Voer repetitieve taken uit met for loops, evalueer exit codes van commando's en scripts, voer tests uit met operators en creëer conditionele structuren met if statements.

### Gebruik Loops om Commando's te Itereren

Systeembeheerders komen vaak repetitieve taken tegen in hun dagelijkse activiteiten. Een voorbeeld van een repetitieve taak is het uitvoeren van een commando meerdere keren op een target, zoals het controleren van een proces elke minuut gedurende 10 minuten om te weten of het voltooid is. Een ander voorbeeld is het uitvoeren van een commando één keer voor meerdere targets, zoals het back-uppen van talloze databases op een systeem. De for loop is een Bash looping construct om te gebruiken voor taak iteraties.

### Verwerk Items van de Commandoregel

In Bash gebruikt de for loop construct de volgende syntax:

```bash
for VARIABELE in LIJST; do
    COMMANDO VARIABELE
done
```

De loop verwerkt de strings die je in LIJST verstrekt en verlaat na het verwerken van de laatste string in de lijst. De for loop slaat tijdelijk elke lijst string op als de waarde van VARIABELE, voert vervolgens het blok commando's uit die de variabele gebruiken.

Voorbeelden van verschillende manieren om strings aan for loops te verstrekken:

```bash
[user@host ~]$ for HOST in host1 host2 host3; do echo $HOST; done
host1
host2
host3

[user@host ~]$ for HOST in host{1,2,3}; do echo $HOST; done
host1
host2
host3

[user@host ~]$ for HOST in host{1..3}; do echo $HOST; done
host1
host2
host3

[user@host ~]$ for PACKAGE in $(rpm -qa | grep kernel); do
    echo "$PACKAGE was geïnstalleerd op $(date -d @$(rpm -q --qf "%{INSTALLTIME}\n" $PACKAGE))"
done

[user@host ~]$ for EVEN in $(seq 2 2 10); do echo "$EVEN"; done
2
4
6
8
10
```

### Bash Script Exit Codes

Na een script interpreteert en verwerkt al zijn content, verlaat het script proces en geeft controle terug aan het parent proces dat het aanriep. Een script kan echter verlaten worden voordat het klaar is, zoals wanneer het script een fout conditie tegenkomt. Gebruik het `exit` commando om onmiddellijk het script te verlaten en verwerking van de rest van het script over te slaan.

Gebruik het `exit` commando met een optioneel integer argument tussen 0 en 255, dat een exit code vertegenwoordigt. Een exit code wordt teruggegeven aan een parent proces om de status bij exit aan te geven. Een exit code waarde van 0 vertegenwoordigt een succesvolle script voltooiing zonder fouten. Alle andere non-zero waarden geven een fout exit code aan.

```bash
[user@host bin]$ cat hello
#!/usr/bin/bash
echo "Hallo, wereld"
exit 0
[user@host bin]$ ./hello
Hallo, wereld
[user@host bin]$ echo $?
0
```

### Test Logica voor Strings en Directories, en om Waarden te Vergelijken

Om ervoor te zorgen dat onverwachte condities scripts niet gemakkelijk verstoren, wordt aanbevolen om commando input te verifiëren zoals commandoregel argumenten, gebruiker input, commando substituties, variabele expansies en bestandsnaam expansies. Je kunt integriteit in je scripts controleren door het Bash test commando te gebruiken.

Alle commando's produceren een exit code bij voltooiing. Om de exit status te zien, bekijk de `$?` variabele onmiddellijk na de uitvoering van het test commando.

### Numerieke Vergelijking Operators

```bash
[user@host ~]$ test 1 -gt 0 ; echo $?
0
[user@host ~]$ test 0 -gt 1 ; echo $?
1

[user@host ~]$ [[ 1 -eq 1 ]]; echo $?
0
[user@host ~]$ [[ 1 -ne 1 ]]; echo $?
1
[user@host ~]$ [[ 8 -gt 2 ]]; echo $?
0
```

### String Vergelijking Operators

```bash
[user@host ~]$ [[ abc = abc ]]; echo $?
0
[user@host ~]$ [[ abc == def ]]; echo $?
1
[user@host ~]$ [[ abc != def ]]; echo $?
0
```

### String Unary Operators

```bash
[user@host ~]$ STRING=''; [[ -z "$STRING" ]]; echo $?
0
[user@host ~]$ STRING='abc'; [[ -n "$STRING" ]]; echo $?
0
```

### Conditionele Structuren

#### If/Then Construct

```bash
if <CONDITIE>; then
    <STATEMENT>
    ...
    <STATEMENT>
fi
```

Voorbeeld:
```bash
[user@host ~]$ systemctl is-active psacct > /dev/null 2>&1
[user@host ~]$ if [[ $? -ne 0 ]]; then sudo systemctl start psacct; fi
```

#### If/Then/Else Construct

```bash
if <CONDITIE>; then
    <STATEMENT>
    ...
else
    <STATEMENT>
    ...
fi
```

#### If/Then/Elif/Then/Else Construct

```bash
if <CONDITIE>; then
    <STATEMENT>
elif <CONDITIE>; then
    <STATEMENT>
else
    <STATEMENT>
fi
```

---

## Begeleide Oefening: Loops en Conditionele Constructies

### Voorbereiding
```bash
[student@workstation ~]$ lab start console-commands
```

### Instructies

**1. Gebruik ssh en hostname commando's om de hostname van servera en serverb machines af te drukken.**

```bash
[student@workstation ~]$ ssh student@servera hostname
servera.lab.example.com
[student@workstation ~]$ ssh student@serverb hostname
serverb.lab.example.com
```

**2. Creëer een for loop om het hostname commando uit te voeren op servera en serverb machines.**

```bash
[student@workstation ~]$ for HOST in servera serverb
do
    ssh student@${HOST} hostname
done
servera.lab.example.com
serverb.lab.example.com
```

**3. Creëer een shell script in /home/student/bin directory.**

3.1. Creëer de /home/student/bin directory:
```bash
[student@workstation ~]$ mkdir ~/bin
```

3.2. Verifieer dat bin subdirectory in je PATH environment variabele staat:
```bash
[student@workstation ~]$ echo $PATH
```

3.3. Creëer een shell script genaamd printhostname.sh:
```bash
[student@workstation ~]$ vim ~/bin/printhostname.sh
```

Inhoud:
```bash
#!/usr/bin/bash
#Voer for loop uit om server hostname af te drukken.
for HOST in servera serverb
do
    ssh student@${HOST} hostname
done
exit 0
```

3.4. Geef het script executable permission:
```bash
[student@workstation ~]$ chmod +x ~/bin/printhostname.sh
```

3.5. Voer het script uit:
```bash
[student@workstation ~]$ printhostname.sh
servera.lab.example.com
serverb.lab.example.com
```

3.6. Verifieer de exit code:
```bash
[student@workstation ~]$ echo $?
0
```

```bash
[student@workstation ~]$ lab finish console-commands
```

---

## Match Tekst in Commando Output met Reguliere Expressies

### Doelstellingen
Creëer reguliere expressies om data te matchen, pas reguliere expressies toe op tekstbestanden met het grep commando en gebruik grep om bestanden en data van piped commando's te doorzoeken.

### Schrijf Reguliere Expressies

Reguliere expressies bieden een patroon matching mechanisme dat het vinden van specifieke content faciliteert. De `vim`, `grep` en `less` commando's kunnen reguliere expressies gebruiken. Programmeertalen zoals Perl, Python en C ondersteunen ook reguliere expressies, maar kunnen kleine verschillen in syntax hebben.

### Beschrijf een Eenvoudige Reguliere Expressie

De eenvoudigste reguliere expressie is een exacte match van de string om te zoeken. Een exacte match is wanneer de karakters in de reguliere expressie overeenkomen met het type en de volgorde van de string.

Voorbeeld bestand:
```
kat
hond
concateneren
dogma
categorie
geëduceerd
boondoggle
rechtvaardiging
chilidog
```

Zoeken naar `kat` vindt:
```
kat
concateneren
categorie
geëduceerd
rechtvaardiging
```

### Match het Begin en Einde van een Regel

Gebruik regel anker metacharacters om te controleren waar op een regel te zoeken naar een match.

- **Begin van regel:** Gebruik het caret karakter (`^`)
- **Einde van regel:** Gebruik het dollar teken (`$`)

Voorbeelden:
- `^kat` matcht regels die beginnen met "kat"
- `kat$` matcht regels die eindigen met "kat"
- `^kat$` matcht regels die alleen "kat" bevatten

### Wildcard en Multiplier Gebruik

- **Punt (.):** Matcht elk enkel karakter
- **Vierkante haken ([]):** Matcht specifieke karakters, bijv. `k[aou]t` matcht "kat", "kot", "kut"
- **Asterisk (*):** Matcht nul of meer van het vorige karakter
- **Expliciete multiplier:** `k.\{2\}t` matcht woorden met k, gevolgd door exact twee karakters, gevolgd door t

### Reguliere Expressies in Bash - Overzicht

| Optie | Beschrijving |
|-------|--------------|
| . | Matcht elk enkel karakter |
| ? | Het voorgaande item is optioneel |
| * | Het voorgaande item wordt nul of meer keer gematcht |
| + | Het voorgaande item wordt één of meer keer gematcht |
| {n} | Het voorgaande item wordt exact n keer gematcht |
| {n,} | Het voorgaande item wordt n of meer keer gematcht |
| {,m} | Het voorgaande item wordt maximaal m keer gematcht |
| {n,m} | Het voorgaande item wordt minimaal n, maximaal m keer gematcht |
| [:alnum:] | Alfanumerieke karakters |
| [:alpha:] | Alfabetische karakters |
| [:blank:] | Spatie en tab karakters |
| [:digit:] | Cijfers: 0-9 |
| [:lower:] | Kleine letters |
| [:upper:] | Hoofdletters |
| [:space:] | Whitespace karakters |

### Match Reguliere Expressies in de Commandoregel

Het `grep` commando gebruikt reguliere expressies om matchende data te isoleren.

```bash
[user@host ~]$ grep '^computer' /usr/share/dict/words
computer
computerese
computerise
```

### Grep Commando Opties

| Optie | Functie |
|-------|---------|
| -i | Case-insensitive zoeken |
| -v | Toon alleen regels die NIET matchen |
| -r | Recursief zoeken in directories |
| -A NUMBER | Toon NUMBER regels na de match |
| -B NUMBER | Toon NUMBER regels voor de match |
| -e | Meerdere reguliere expressies |

### Voorbeelden van het grep Commando

```bash
# Case-insensitive zoeken
[user@host ~]$ grep -i serverroot /etc/httpd/conf/httpd.conf

# Omgekeerd zoeken (regels die NIET matchen)
[user@host ~]$ grep -v -i server /etc/hosts

# Comments uitsluiten
[user@host ~]$ grep -v '^[#;]' /etc/ethertypes

# Meerdere patronen zoeken
[root@host ~]# cat /var/log/secure | grep -e 'pam_unix' -e 'user root' -e 'Accepted publickey' | less
```

---

## Begeleide Oefening: Match Tekst met Reguliere Expressies

### Voorbereiding
```bash
[student@workstation ~]$ lab start console-regex
```

### Instructies

**1. Log in op servera als student en switch naar root.**

```bash
[student@workstation ~]$ ssh student@servera
[student@servera ~]$ sudo -i
[root@servera ~]#
```

**2. Gebruik grep om GID en UID voor postfix en postdrop te vinden.**

```bash
[student@servera ~]$ rpm -q --scripts postfix | grep -e 'user' -e 'group'
```

**3. Zoek de eerste twee berichten in /var/log/maillog.**

```bash
[root@servera ~]# grep 'postfix' /var/log/maillog | head -n 2
```

**4. Vind de queue directory voor de Postfix server.**

```bash
[root@servera ~]# grep -i 'queue' /etc/postfix/main.cf
```

**5. Bevestig dat postfix berichten schrijft naar /var/log/messages.**

```bash
[root@servera ~]# less /var/log/messages
/Postfix
```

**6. Bevestig dat postfix server actief is.**

```bash
[root@servera ~]# ps aux | grep postfix
```

**7. Bevestig queue configuratie.**

```bash
[root@servera ~]# grep -e qmgr -e pickup -e cleanup /etc/postfix/master.cf
```

**8. Keer terug naar workstation.**

```bash
[root@servera ~]# exit
[student@servera ~]$ exit
[student@workstation ~]$ lab finish console-regex
```

---

## Lab: Verbeter Commandoregel Productiviteit

### Doel
Creëer een Bash script dat relevante informatie van verschillende hosts kan filteren en verkrijgen.

### Voorbereiding
```bash
[student@workstation ~]$ lab start console-review
```

### Instructies

**1. Creëer het /home/student/bin/bash-lab script bestand.**

**2. Bewerk je script om de volgende informatie van servera en serverb hosts te verkrijgen:**

| Commando/bestand | Gewenste inhoud |
|------------------|-----------------|
| hostname -f | Alle output |
| echo "#####" | Alle output |
| lscpu | Alleen regels die beginnen met "CPU" |
| echo "#####" | Alle output |
| /etc/selinux/config | Negeer lege regels en regels die beginnen met # |
| echo "#####" | Alle output |
| /var/log/secure | Alle "Failed password" entries |
| echo "#####" | Alle output |

Sla de informatie op in output-servera en output-serverb bestanden in /home/student directory.

**3. Voer het script uit en bekijk de output.**

### Oplossing

**1. Creëer het script bestand:**

```bash
[student@workstation ~]$ mkdir -p /home/student/bin
[student@workstation ~]$ vim ~/bin/bash-lab
```

Voeg toe:
```bash
#!/usr/bin/bash
```

Maak uitvoerbaar:
```bash
[student@workstation ~]$ chmod a+x ~/bin/bash-lab
```

**2. Bewerk het script:**

```bash
#!/usr/bin/bash
#
USR='student'
OUT='/home/student/output'
#
for SRV in servera serverb
do
    ssh ${USR}@${SRV} "hostname -f" > ${OUT}-${SRV}
    echo "#####" >> ${OUT}-${SRV}
    ssh ${USR}@${SRV} "lscpu | grep '^CPU'" >> ${OUT}-${SRV}
    echo "#####" >> ${OUT}-${SRV}
    ssh ${USR}@${SRV} "grep -v '^$' /etc/selinux/config|grep -v '^#'" >> ${OUT}-${SRV}
    echo "#####" >> ${OUT}-${SRV}
    ssh ${USR}@${SRV} "sudo grep 'Failed password' /var/log/secure" >> ${OUT}-${SRV}
    echo "#####" >> ${OUT}-${SRV}
done
```

**3. Voer het script uit:**

```bash
[student@workstation ~]$ bash-lab
[student@workstation ~]$ cat /home/student/output-servera
[student@workstation ~]$ cat /home/student/output-serverb
```

**Evaluatie:**
```bash
[student@workstation ~]$ lab grade console-review
[student@workstation ~]$ lab finish console-review
```

---

## Samenvatting

• Creëer en voer eenvoudige Bash scripts uit om simpele beheer taken te voltooien.
• Gebruik loops om te itereren door een lijst van items van de commandoregel en in een shell script.
• Gebruik conditionele structuren om besluitvorming in shell scripts op te nemen.
• Zoek naar tekst in log en configuratie bestanden door reguliere expressies en het grep commando te gebruiken.
