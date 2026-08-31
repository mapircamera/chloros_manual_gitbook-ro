# Rețea și sincronizare temporală DAQ-E

> Configurarea fizică a rețelei pentru senzor — cablare, PoE, alocarea adreselor IP și setările de rețea proprii ale dispozitivului — este descrisă în **[manualul de utilizare DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Această pagină tratează partea legată de Chloros: conectare, sincronizare temporală și ce trebuie făcut atunci când detectarea nu dă rezultate.

DAQ-E este membrul Ethernet al familiei DAQ: alimentat prin PoE, detectat prin mDNS (serviciul `_daq-e._tcp`) și adresabil printr-un nume de gazdă derivat din ID-ul senzorului său — `daq-e-<6 hex>.local`, de exemplu `daq-e-def330.local`. Această pagină prezintă modul în care dispozitivul transferă date în rețea și modul în care participă la sincronizarea temporală PTP.

## Moduri de transport

| Mod | Punct terminal | Consumatori | Note |
| --- | --- | --- | --- |
| **Multicast** (implicit) | UDP `239.10.10.10:5002` | Orice număr de pe aceeași rețea LAN primește același flux | Fiecare datagramă este validată prin CRC-16/CCITT |
| **Raw** | Port TCP `5000` | Exact un singur client (exclusiv) | Compatibil la nivel de octeți cu DAQ-U |

Chloros utilizează multicast în mod implicit, ceea ce permite interfeței grafice, CLI și SDK să monitorizeze toate un singur senzor simultan.

## Cerințe de rețea

* **Același domeniu de difuzare.** Mașina pe care rulează Chloros trebuie să se afle pe același segment de rețea L2 ca și senzorul — descoperirea mDNS nu traversează routerele.
* **Solicitarea firewall-ului pentru Windows: acceptați-o.** La prima legare a socket-urilor multicast de către Chloros, Windows Defender vă întreabă o singură dată. Permiterea acestuia acoperă datele DAQ-E (UDP 5002), mDNS (UDP 5353) și PTP (UDP 319/320). Pe Linux acest proces se desfășoară în mod silențios.
* **Alimentare PoE, fără LED de stare.** DAQ-E nu are LED propriu — verificați alimentarea prin indicatorul link/PoE de pe comutator sau de pe portul injectorului și așteptați câteva secunde după pornire pentru ca acesta să se inițializeze și să se conecteze la rețea.

## Conectare

**GUI:** Fila Senzori de lumină → Conectare senzor → Tip dispozitiv „DAQ-E (Ethernet)”. Detectarea se efectuează numai cât timp fereastra de conectare este afișată pe ecran (căutare mDNS plus o scanare ARP pe Windows), repetându-se la fiecare 15 secunde; butonul „Refresh” (Reîmprospătare) efectuează imediat o nouă scanare. Senzorii detectați apar în meniul derulant; primul senzor detectat este selectat automat.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend în funcțiune):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Gazdele cu mai multe plăci de rețea și prima conectare după pornire

Pe gazdele cu mai mult de o interfață de rețea activă, **prima** `pool-connect --eth` după pornire poate rămâne goală chiar și atunci când senzorul este funcțional — procesul de descoperire poate omite interfața pe care se află senzorul cât timp cache-ul ARP este încă inactiv. Soluția fiabilă este să omiteți procesul de descoperire și să specificați adresa în mod explicit:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` acceptă numele de gazdă mDNS sau adresa IP, vizează întotdeauna senzorul corect și este forma recomandată pentru scripturi și instalări fără interfață grafică. În interfața grafică, utilizați butonul „Refresh” (Reîmprospătare) din fereastra de dialog de conectare și permiteți un ciclu de rescanare.

## Setările dispozitivului și firmware-ul

Senzorul însuși stochează setările de rețea — IP static vs. DHCP + adresare link-local, numele dispozitivului, transmisie automată la pornire, parolă OTA. Aceste setări ale dispozitivului nu sunt expuse ca comenzi în CLI livrat; ele sunt gestionate prin interfața grafică Chloros, acolo unde sunt afișate, sau cu ajutorul serviciului de asistență MAPIR.

**Actualizările de firmware sunt integrate în interfața grafică.**Atunci când un DAQ-E conectat rulează un firmware mai vechi decât imaginea inclusă în versiunea dvs. Chloros, rândul senzorului afișează o pastilă portocalie**Actualizare disponibilă**, iar fereastra modală de setări sub formă de roată dințată oferă un<version>

buton</version> „Actualizează la<version>

„. Actualizarea se descarcă prin rețea în aproximativ 30 de secunde; senzorul se repornește și se reconectează automat, iar o întrerupere a transferului lasă firmware-ul curent intact.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sincronizarea timpului PTP

Firmware-ul DAQ-E v1.2.0+ participă la standardul IEEE 1588 PTPv2 ca ceas obișnuit (doar slave). **Backend-ul gazdei Chloros este grandmaster-ul PTP** — fiecare DAQ-E și fiecare cameră LATTICE din rețeaua LAN îi sunt subordonate în domeniul 0, menținând toate marcajele de timp ale dispozitivelor într-o toleranță de ~1 ms. Acest ceas comun este cel care permite corelarea marcajelor temporale ale citirilor DAQ cu expunerile camerei (vezi [Înregistrarea și formatul .daq](recording.md)).

Verificați sincronizarea din CLI:

| Comandă | Afișează |
| --- | --- |
| `chloros-cli time-sync status` | Starea gazdei grandmaster, prioritățile BMCA, identitatea ceasului |
| `chloros-cli time-sync peers` | Toate dispozitivele subordonate detectate (senzori DAQ-E + camere LATTICE) |
| `chloros-cli time-sync cameras` | Starea PTP pentru fiecare cameră (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Repornirea procesului grandmaster |

În interfața grafică (GUI), fereastra modală de setări DAQ-E afișează un rând **PTP Sync** în timp real cu starea PTP curentă a senzorului.

Detalii pentru consumatorii care necesită aliniere strictă:

* Fiecare datagramă transmisă conține un câmp de indicatori; **bitul 2 este setat pe cadrele a căror marcă temporală este sincronizată PTP**. Conductele care necesită aliniere strictă între cameră și DAQ trebuie să se bazeze pe acel bit.
* Înainte de o captură sincronizată, asigurați-vă că senzorul apare în `chloros-cli time-sync peers`. (Instrumentele interne de hardware direct ale modelului MAPIR pot, de asemenea, să condiționeze înregistrarea de blocarea PTP cu un indicator `--wait-ptp` care așteaptă până la 15 s ca senzorul să atingă starea SLAVE; acel instrument nu face parte din CLI livrat.)
* În timp ce PTP funcționează activ în mod slave, senzorul refuză transmiterea manuală a ceasului („PTP furnizează ceasul”). Aceasta este o caracteristică intenționată — aveți încredere în PTP.

## Note privind Linux

* **PTP necesită `libcap2-bin` în momentul instalării.** Comanda postinst `.deb` acordă drepturi `cap_net_bind_service=+ep` pe `/usr/lib/chloros/chloros-backend`, astfel încât acesta să poată lega porturile PTP 319/320 fără drepturi de root. Dacă `libcap2-bin` lipsește, acel pas este omis și PTP nu va reuși să pornească. Soluție:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Jetson / Raspberry Pi fără interfață grafică:** la prima instalare, unitatea systemd `chloros-backend.service` este generată, dar nu este activată. Pentru ca PTP (și disponibilitatea DAQ) să fie mereu active fără interfața grafică:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Fără aceasta, PTP funcționează doar cât timp interfața grafică Chloros este deschisă.

## Depanare: „Nu s-au găsit dispozitive DAQ-E”

| Verificare | Detalii |
| --- | --- |
| Alimentare | LED-ul de pe senzor nu este aprins — verificați indicatorii PoE și de legătură ai comutatorului/portului injectorului; așteptați câteva secunde după pornire |
| Domeniu de difuzare | Gazda și senzorul se află pe același segment L2; mDNS nu rutează |
| Firewall Windows | Acceptați solicitarea Defender la prima rulare (UDP 5002, 5353, 319/320) |
| Gazdă cu mai multe plăci de rețea | Este posibil ca senzorul să nu fie detectat la prima descoperire după pornire — conectați-vă cu `--eth-host <ip-or-hostname>` |
| Rescanare GUI | Descoperirea se execută numai cât timp fereastra de conectare este deschisă; utilizați butonul „Refresh” |</version>
