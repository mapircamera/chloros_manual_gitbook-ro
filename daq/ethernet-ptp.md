# Rețea și sincronizare temporală DAQ-E

> Configurarea fizică a rețelei pentru senzor — cablarea, PoE, atribuirea adreselor IP și setările de rețea proprii ale dispozitivului — este descrisă în **[manualul de utilizare DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Această pagină tratează aspectele legate de Chloros: conectarea, sincronizarea timpului și ce trebuie făcut atunci când detectarea nu dă rezultate.

DAQ-E este membrul Ethernet al familiei DAQ: alimentat prin PoE, detectat prin mDNS (serviciul `_daq-e._tcp`) și adresabil printr-un nume de gazdă derivat din ID-ul senzorului său — `daq-e-<6 hex>.local`, de exemplu `daq-e-def330.local`. Această pagină prezintă modul în care acesta transferă date în rețea și modul în care participă la sincronizarea temporală PTP.

## Moduri de transport

| Mod | Punct final | Consumatori | Note |
| --- | --- | --- | --- |
| **Multicast** (implicit) | UDP `239.10.10.10:5002` | Orice dispozitiv din aceeași rețea LAN primește același flux | Fiecare datagramă este validată prin CRC-16/CCITT |
| **Raw** | Port TCP `5000` | Exact un singur client (exclusiv) | Compatibil la nivel de octeți cu DAQ-U |

Chloros utilizează multicast în mod implicit, ceea ce permite ca interfața grafică (GUI), interfața de linie de comandă (CLI) și kitul de dezvoltare (SDK) să monitorizeze toate un singur senzor simultan.

## Cerințe de rețea

* **Același domeniu de difuzare.** Mașina pe care rulează Chloros trebuie să se afle pe același segment de rețea L2 ca și senzorul — descoperirea mDNS nu traversează routerele.
* **Firewall.** Pe Windows, programul de instalare adaugă regula de firewall **Chloros Backend (UDP-In)**pentru backend, în fiecare profil de rețea, care acoperă datele DAQ-E (UDP 5002), mDNS (UDP 5353) și PTP (UDP 319/320), precum și o regulă**„Chloros Backend (TCP-In)”**, limitată la subrețeaua locală, pentru conexiunea pe care un DAQ-E o stabilește înapoi către computer în timpul unei actualizări de firmware. Pe Linux, `.deb` deschide porturile UDP în `ufw` sau `firewalld` atunci când acel firewall este activ în momentul instalării. Un firewall terț sau unul activat ulterior necesită ca aceleași porturi să fie permise.
* **Alimentare PoE, fără LED de stare.** DAQ-E nu are LED propriu — verificați alimentarea prin indicatorul link/PoE de pe comutator sau de pe portul injectorului și așteptați câteva secunde după pornire pentru ca acesta să se inițializeze și să se conecteze la rețea.

## Conectare

**GUI:** Fila Senzori de lumină → Conectare senzor → Tip dispozitiv „DAQ-E (Ethernet)”. Detectarea se execută numai cât timp fereastra de conectare este afișată pe ecran (răsfoire mDNS plus o scanare ARP pe Windows), repetându-se la fiecare 15 secunde; butonul Reîmprospătare efectuează o nouă scanare imediat. Senzorii descoperiți apar în meniul derulant; primul senzor detectat este selectat automat.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend în execuție):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

Un DAQ-E care se află deja în grup este recunoscut atât după numele de gazdă, cât și după adresa IP: conectarea sa din nou sub cealaltă adresă returnează intrarea existentă din grup, în loc să deschidă unitatea a doua oară.

### Gazdele cu mai multe plăci de rețea și prima conectare după pornire

Pe gazdele cu mai multe interfețe de rețea active, **prima** conexiune `pool-connect --eth` după pornire poate fi goală chiar și atunci când senzorul este funcțional — procesul de descoperire poate omite interfața pe care se află senzorul, atâta timp cât cache-ul ARP este încă inactiv. Soluția sigură este să omiteți procesul de descoperire și să specificați adresa în mod explicit:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` acceptă numele de gazdă mDNS sau adresa IP, vizează întotdeauna senzorul corect și reprezintă forma recomandată pentru scripturi și instalări fără interfață grafică. În interfața grafică, utilizați butonul „Refresh” (Reîmprospătare) din fereastra de dialog de conectare și permiteți un ciclu de rescanare.

## Setările dispozitivului și firmware-ul

Senzorul însuși păstrează setările sale de rețea: DHCP (setarea din fabrică, care revine la o adresă locală de legătură 169.254.x.x atunci când niciun server DHCP nu răspunde) sau o adresă statică. De asemenea, reține dacă începe transmisia la pornire. Modificați-le în interfața grafică (setările rotiței de angrenaj ale senzorului, **Rețea**:**Modificare…**,**Transmisie la pornire**,**Repornire senzor**; consultați [Fila DAQ](gui.md)), sau din CLI și SDK cu `pool-set-network`, `pool-set-auto-stream` și `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; consultați [Referința CLI](../reference/cli-reference.md#daq-e-device-settings)). Senzorul se repornește pentru a aplica o modificare de rețea, iar Chloros îl reconectează. O adresă statică trebuie să se afle în rețeaua computerului pe care rulează Chloros: Chloros refuză o adresă la care nu a putut accesa după repornire. ID-ul senzorului și numele său de gazdă `daq-e-<id>.local` nu se schimbă niciodată. Noutate în versiunea 1.3.0.

**Actualizările de firmware sunt integrate în interfața grafică (GUI).** Când un DAQ-E conectat rulează o versiune de firmware mai veche decât imaginea inclusă în pachetul Chloros, rândul senzorului respectiv afișează o pictogramă portocalie **Actualizare disponibilă**, iar fereastra de setări sub formă de roată dințată oferă un <version>buton</version>

„Actualizează la<version>”. Actualizarea se descarcă prin rețea în aproximativ 30 de secunde; senzorul se repornește și se reconectează automat, iar o întrerupere a transferului lasă firmware-ul curent intact. Din CLI sau SDK și pe Linux, care nu are interfață grafică, rulați `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sincronizarea timpului PTP

Firmware-ul DAQ-E v1.2.0+ participă la standardul IEEE 1588 PTPv2 ca ceas obișnuit (doar slave). **Backend-ul gazdei Chloros este grandmaster-ul PTP** — fiecare DAQ-E și fiecare cameră LATTICE din rețeaua LAN îi sunt subordonate în domeniul 0, menținând toate marcajele de timp ale dispozitivelor într-o toleranță de ~1 ms. Acest ceas partajat este cel care face ca citirile DAQ să poată fi sincronizate temporal cu expunerile camerei (vezi [Înregistrarea și formatul .daq](recording.md)).

Verificați sincronizarea din CLI:

| Comandă | Afișează |
| --- | --- |
| `chloros-cli time-sync status` | Starea „grandmaster” a gazdei, prioritățile BMCA, identitatea ceasului și `health`: `OK` sau `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` cu un `health_detail` care explică de ce pachetele sale nu ajung în rețea |
| `chloros-cli time-sync peers` | Toate dispozitivele slave detectate (senzori DAQ-E + camere LATTICE) |
| `chloros-cli time-sync cameras` | Starea PTP pentru fiecare cameră (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Repornirea procesului grandmaster |

În interfața grafică (GUI), fereastra modală de setări DAQ-E afișează un rând **PTP Sync** în timp real cu starea PTP curentă a senzorului.**Un DAQ-E cu PTP dezactivat rămâne dezactivat.** Unitatea păstrează setarea de activare/dezactivare a PTP în propria memorie, astfel încât o unitate al cărei PTP a fost dezactivat rămâne dezactivată după o reconectare sau o repornire a alimentării, iar marcajele sale de timp nu sunt sincronizate prin PTP. `chloros-cli daq pool-list` semnalează o astfel de unitate (`!! PTP is switched OFF on this unit …`) și afișează comanda care remediază problema:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros nu reactivează niciodată o unitate de la sine, deoarece dezactivarea poate fi intenționată.

Detalii pentru utilizatorii care necesită aliniere strictă:

* Fiecare datagramă transmisă în flux conține un câmp de indicatori; **bitul 2 este setat pentru cadrele ale căror marcaje temporale sunt sincronizate PTP**. Conductele care necesită o aliniere strictă a camerei/DAQ ar trebui să se bazeze pe acel bit.
* Înainte de o captură sincronizată, asigurați-vă că senzorul apare în `chloros-cli time-sync peers` și că `chloros-cli daq pool-list` nu semnalează că PTP-ul acestuia este oprit. (Instrumentele interne MAPIR de control direct al hardware-ului pot, de asemenea, să activeze înregistrarea la blocarea PTP cu un indicator `--wait-ptp` care așteaptă până la 15 s ca senzorul să ajungă în starea SLAVE; aceste instrumente nu fac parte din CLI-ul livrat.)
* În timp ce PTP funcționează activ în mod slave, senzorul refuză transmiterile manuale de ceas („PTP furnizează ceasul”). Aceasta este o caracteristică intenționată — aveți încredere în PTP.

## Note privind Linux

* **Porturi PTP fără drepturi de root.** PTP utilizează porturile UDP 319/320. `.deb` scrie `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), astfel încât backend-ul le leagă rulând sub contul dvs. de utilizator. Începând cu versiunea 1.3.0, pachetul nu setează nicio capacitate de fișier (`setcap`) pe backend și nu are nevoie de `libcap2-bin`; nu adăugați voi înșivă o capacitate, deoarece backend-ul despachetat nu va porni în acest caz. Dacă PTP nu poate ocupa portul 319, consultați [Instalarea pe Linux](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Jetson fără monitor / gazdă Linux:** pachetul instalează serviciile backend, dar nu activează niciunul. Pentru ca PTP să fie mereu activ (și DAQ disponibil) fără ca cineva să execute o comandă, activați serviciul ca utilizator:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Fără aceasta, PTP rulează doar atâta timp cât ceva a pornit backend-ul — o comandă `chloros-cli` sau un script SDK. Consultați [Instalarea pe Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) pentru contul root `chloros-backend.service` și modul de trecere de la acesta.

## Depanare: „Nu s-au găsit dispozitive DAQ-E”

| Verificare | Detalii |
| --- | --- |
| Alimentare | LED-ul de pe senzor nu este aprins — verificați indicatorii PoE și de legătură ai comutatorului/portului injectorului; așteptați câteva secunde după pornire |
| Domeniu de difuzare | Gazda și senzorul se află pe același segment L2; mDNS nu rutează |
| Firewall | Windows: regula **Chloros Backend (UDP-In)** a programului de instalare permite intrarea fluxului DAQ-E (UDP 5002), mDNS (5353) și PTP (319/320); un firewall terț trebuie să permită același lucru. Linux: `.deb` deschide aceste porturi în `ufw` / `firewalld` numai dacă firewall-ul era activ la momentul instalării — permiteți-le manual într-unul activat ulterior |
| Gazdă cu mai multe plăci de rețea | Funcția de descoperire scanează mDNS cu o opțiune de rezervă ARP; `--eth-host <ip-or-hostname>` omite descoperirea și se conectează direct |
| Rescanare din interfața grafică | Funcția de descoperire rulează numai cât timp este deschis dialogul de conectare; utilizați opțiunea „Refresh” |
