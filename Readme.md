# Brute Force Toolkit – Cheat Sheet

Ez a dokumentum összefoglalja a legfontosabb **jelszótörési módszereket** és a hozzájuk tartozó **brute-force eszközöket** (Hydra, Ncrack, CrackMapExec, Crowbar, Medusa, Crunch, CeWL, John the Ripper), kifejezetten **lab / pentest / SOC tanulás** céljára.

> ⚠️ **Figyelem:** Minden példa **csak saját lab környezetben**, saját eszközöd ellen használandó. Más rendszerek elleni brute-force támadás illegális.

---

## 1. Jelszótörési módszerek (Password Attack Types)

### 1.1 Brute-force attack
- **Mit csinál?**  
  Minden lehetséges karakterkombinációt végigpróbál (pl. `aaaaaa` → `zzzzzz`).
- **Előny:**  
  Garantált találat, ha a jelszó a keresési térben van.
- **Hátrány:**  
  Nagyon lassú, amint a jelszó hossza és a karakterkészlet nő.

### 1.2 Dictionary attack
- **Mit csinál?**  
  Előre megadott jelszólistát (wordlistet) használ (pl. `rockyou.txt`).
- **Előny:**  
  Gyorsabb, mert valós, gyakran használt jelszavakat próbál.
- **Hátrány:**  
  Ha a jelszó nincs a listában, sosem találja meg.

### 1.3 Hybrid attack
- **Mit csinál?**  
  Szótár + variációk kombója.  
  Példa: kulcsszó `Princess` → `Princess1`, `Princess123`, `Princess74!`, `pr1nc3ss`.
- **Használat:**  
  Ha tudsz valamilyen személyes infót: kutya neve, becenév, kedvenc játék, stb.

### 1.4 Rule-based attack
- **Mit csinál?**  
  Szabályok alapján alakít át jelszavakat:
  - kezdőbetű nagybetű
  - számok hozzáadása a végére
  - betűk cseréje: `a → @`, `o → 0`, `i → 1`
- **Előny:**  
  Nagyon jól modellezi, ahogy emberek „okosnak hitt” jelszavakat csinálnak.

### 1.5 Credential stuffing
- **Mit csinál?**  
  Kiszivárgott **valós** (username + password) párokat próbálgat más szolgáltatásokon.
- **Példa:**  
  Ha egy weboldal adatai szivárogtak, a támadó ugyanazt próbálja e-mailen, VPN-en, O365-ön.

### 1.6 Password spraying
- **Mit csinál?**  
  Sok felhasználóra **ugyanazt** a jelszót próbálja (pl. `Welcome2024!`).
- **Cél:**  
  Account lockout elkerülése – ritkán, de sok accounton próbál.

---

## 2. Brute-force eszközök – összefoglaló

| Eszköz              | Rövid leírás                                             | Tipikus protokollok                         | Támadástípusok                      |
|---------------------|----------------------------------------------------------|---------------------------------------------|-------------------------------------|
| **Hydra**           | Gyors, többprotokollos online brute-forcer              | SSH(22), FTP(21), SMB(445), RDP(3389), HTTP | brute-force, dictionary, hybrid     |
| **Ncrack**          | Nmap csapat enterprise brute-force eszköze              | SSH, RDP, VNC, FTP, Telnet                  | brute-force, slow brute-force       |
| **CrackMapExec**    | Windows hálózati keretrendszer, credential checking     | SMB(445), RDP(3389), WinRM(5985/5986)       | credential stuffing, password spray |
| **Crowbar**         | Egyszerű RDP/VNC brute-forcer                           | RDP(3389), VNC(5900)                        | brute-force                         |
| **Medusa**          | Moduláris, párhuzamos brute-forcer                      | SSH, SMB, FTP, HTTP, SQL, POP3, IMAP        | brute-force                         |
| **Crunch**          | Mask-based jelszógenerátor                              | –                                           | hybrid / brute-force előkészítés    |
| **CeWL**            | Weboldal tartalmából wordlist generátor (OSINT)         | HTTP/HTTPS                                  | dictionary, hybrid                   |
| **John the Ripper** | Offline hash-törő (password cracker)                    | – (hash input)                              | offline dictionary, rule-based      |

---

## 3. Hydra – Fast Multi-Protocol Brute Forcer

### 3.1 Mit csinál?

- Többféle protokollhoz képes brute-force vagy dictionary támadást indítani.
- Több szálon (`-t`) párhuzamosan fut.
- Moduláris felépítés: `ssh://`, `rdp://`, `smb://`, stb.

### 3.2 Támadható protokollok & portok (példák)

- SSH – 22  
- FTP – 21  
- Telnet – 23  
- HTTP/HTTPS form login – 80 / 443  
- SMB – 445  
- RDP – 3389  
- VNC – 5900  
- POP3 – 110  
- IMAP – 143  

### 3.3 Alap parancsok

**Egy jelszó tesztelése SSH ellen:**

```bash
hydra -l admin -p Password1! ssh://10.10.10.10
```

**Wordlist támadás SSH ellen:**

```bash
hydra -l admin -P rockyou.txt -t 4 -W 3 ssh://10.10.10.10
```

**RDP brute-force (lassú, óvatos):**

```bash
hydra -l admin -P passwords.txt -t 1 -W 2 rdp://34.34.20.16
```

### 3.4 Paraméterek magyarázata

| Paraméter         | Jelentés                                                |
|-------------------|---------------------------------------------------------|
| `-l admin`        | Fix felhasználónév                                     |
| `-L users.txt`    | Felhasználónév lista                                   |
| `-p Password1!`   | Egyetlen jelszó                                        |
| `-P passlist.txt` | Jelszólista (wordlist)                                 |
| `-t 4`            | Thread-ek száma – több szál → gyorsabb                 |
| `-W 3`            | 3 másodperc várakozás két sikertelen próbálkozás közt  |
| `ssh://`          | SSH modul                                              |
| `rdp://`          | RDP modul                                              |
| `-s 2222`         | Egyedi port megadása (pl. nem default SSH port)        |

### 3.5 Recommended usage (miért jó?)

- Gyors multi-protocol brute-forcer.
- Kisebb hálózatokban, labban kiváló.
- SOC szempontból sok 4625 (Failed Logon) eventet generál → jó logelemzési alap.

---

## 4. Ncrack – Reliable Enterprise Brute Forcer

### 4.1 Mit csinál?

- Az Nmap készítői fejlesztették.
- Nagy hálózatokra és stabil, kontrollált brute-force-ra optimalizált.
- Jobban kezeli a késleltetést, time-outokat, mint a Hydra.

### 4.2 Támadott protokollok (példák)

- SSH – 22  
- RDP – 3389  
- VNC – 5900  
- FTP – 21  
- Telnet – 23  

### 4.3 Alap parancsok

**SSH brute-force:**

```bash
ncrack -p 22 --user admin -P rockyou.txt 10.10.10.10
```

**RDP brute-force:**

```bash
ncrack -p 3389 --user admin -P passwords.txt 34.34.20.16
```

**Lassabb, óvatosabb módban (delay):**

```bash
ncrack -p 3389 --user admin -P passwords.txt --connection-limit 1 --delay 5s 34.34.20.16
```

### 4.4 Recommended usage

- Nagyobb wordlistek lassú, „no-lockout” bruteforce-jához.
- Stabilabb RDP támadáshoz, ha a Hydra modul szarakodik.
- Jó enterprise szimulációhoz.

---

## 5. CrackMapExec (CME) – Credential Validation & Lateral Movement

### 5.1 Mit csinál?

- Windows-os hálózati eszköz: **nem csak brute-force**, hanem:
  - credential validation (megnézi, jó-e a jelszó)
  - SMB/WinRM/RDP hozzáférés ellenőrzés
  - domain / AD enumeration
  - később későbbi lateral movement lépések

### 5.2 Portok

- SMB – 445  
- RDP – 3389  
- WinRM – 5985 (HTTP) / 5986 (HTTPS)  

### 5.3 Alap parancsok

**SMB credential check egy jelszóval:**

```bash
crackmapexec smb 10.10.10.10 -u admin -p Password1!
```

**SMB brute-force egy userre:**

```bash
crackmapexec smb 10.10.10.10 -u admin -P passwords.txt
```

**RDP login check:**

```bash
crackmapexec rdp 10.10.10.10 -u admin -p Password1!
```

**WinRM ellenőrzés (PowerShell remote):**

```bash
crackmapexec winrm 10.10.10.10 -u admin -p Password1!
```

### 5.4 Recommended usage

- Windows környezetben **sokkal hasznosabb**, mint sima Hydra.
- Credential stuffing / password spraying szimulációra tökéletes.
- SOC oldalon:
  - 4625 / 4624 eventek
  - WinRM esetén PowerShell logok
  - SMB connection logok (Sysmon 3)

---

## 6. Crowbar – Simple RDP/VNC Brute Forcer

### 6.1 Mit csinál?

- Kifejezetten **RDP és VNC brute-force** támogatására készült.
- Kevés feature, cserébe egyszerű és stabil ezeknél a protokolloknál.

### 6.2 Portok

- RDP – 3389  
- VNC – 5900  

### 6.3 Alap parancsok

**RDP brute-force:**

```bash
crowbar -b rdp -s 34.34.20.16/32 -u admin -C passwords.txt
```

**VNC brute-force:**

```bash
crowbar -b vnc -s 10.10.10.10/32 -u admin -C vnc_pass.txt
```

### 6.4 Recommended usage

- Ha kifejezetten RDP-t akarsz támadni, és a Hydra modul bizonytalan.
- Egyszerű lab szituációkhoz.

---

## 7. Medusa – Modular Fast Brute Forcer

### 7.1 Mit csinál?

- Moduláris, nagy teljesítményű brute-force tool.
- Sok különböző szolgáltatás ellen használható.

### 7.2 Támadható protokollok (példák)

- SSH  
- FTP  
- HTTP  
- SMB  
- POP3 / IMAP  
- MSSQL / MySQL  

### 7.3 Alap parancs

```bash
medusa -h 10.10.10.10 -u admin -P passlist.txt -M ssh
```

| Paraméter           | Jelentés                      |
|---------------------|-------------------------------|
| `-h 10.10.10.10`    | target host                   |
| `-u admin`          | felhasználónév                |
| `-P passlist.txt`   | jelszólista                   |
| `-M ssh`            | modul / protokoll (ssh/http…) |

### 7.4 Recommended usage

- Ha más toolok valamiért instabilak, Medusa jó alternatíva.
- Több protokoll támogatás → hasznos lab környezetben.

---

## 8. Crunch – Advanced Password Generator (Mask-Based)

### 8.1 Mit csinál?

- **Nem brute-force-ol**, hanem **jelszólistát generál**:
  - adott hosszra
  - adott karakterkészlettel
  - vagy maszk alapján

### 8.2 Karakterkészlet / maszk szimbólumok

Crunch `-t` (pattern) maszk:

| Szimbólum | Jelentés              |
|----------|------------------------|
| `@`      | kisbetűk (a–z)         |
| `,`      | nagybetűk (A–Z)        |
| `%`      | számok (0–9)           |
| `^`      | speciális karakterek   |

### 8.3 Példák

**1) Princess + két szám (`Princess00`–`Princess99`)**

```bash
crunch 10 10 -t Princess%% -o princess.txt
```

- 10 = minimális hossz
- 10 = maximális hossz
- `Princess%%` = 8 karakter fix, + 2 számjegy

**2) 8 karakter hosszú kisbetű brute-force wordlist**

```bash
crunch 8 8 abcdefghijklmnopqrstuvwxyz -o 8lower.txt
```

**3) Név + 4 speciális karakter (maszk)**

```bash
crunch 9 9 -t Adam^^^^ -o adam_symbols.txt
```

### 8.4 Recommended usage

- Hybrid támadásokhoz **kulcsszó + variációk** generálására.
- Sokkal célzottabb jelszólista, mint a sima `rockyou.txt`.
- Utána a generált `.txt` mehet Hydra/Ncrack/CME inputnak.

---

## 9. CeWL – OSINT-Based Wordlist Generator

### 9.1 Mit csinál?

- Egy weboldalt bejár (crawl), és:
  - kigyűjti a hosszabb szavakat (pl. 5+ karakter)
  - domainhez kapcsolódó neveket / projekteket
  - ezekből wordlistet készít

### 9.2 Miért hasznos?

- Valós támadók gyakran használnak cégnévhez, projektnévhez kötődő jelszavakat.
- Pl.: `Acme2024!`, `ProjectX!`, `AlphaTeam01`, stb.

### 9.3 Példák

**Alap wordlist generálás:**

```bash
cewl https://target-site.com -w wordlist.txt
```

**Mélyebbre menő crawl (több link követése):**

```bash
cewl https://target-site.com -d 3 -w deep.txt
```

**Számokkal együtt (dátumszerű minták):**

```bash
cewl https://target-site.com --with-numbers -w words_numbers.txt
```

### 9.4 Recommended usage

- OSINT alapú támadási előkészítéshez.
- Ha tudod, milyen céges / projekt szavak jellemzőek az adott célpontra.
- Hybrid-ben Crunch-csal kombinálva: CeWL → `wordlist.txt` → Crunch maszkok.

---

## 10. John the Ripper – Offline Password Cracker

### 10.1 Mit csinál?

- Nem hálózaton támad → **hashből próbál jelszót kitalálni**, offline.
- Típusok:
  - /etc/shadow (Linux) hash-ek
  - Windows NTLM/SAM hash
  - ZIP/RAR jelszavak
  - stb.

### 10.2 Fő támadási módok

- Dictionary attack → `--wordlist=...`
- Rule-based attack → dictionary + szabály
- Incremental (teljes brute-force)  

### 10.3 Példák

**Hash törése wordlisttel:**

```bash
john hashes.txt --wordlist=rockyou.txt
```

**Törési session folytatása:**

```bash
john --restore
```

**Talált jelszavak megjelenítése:**

```bash
john --show hashes.txt
```

**Incremental mód (lassú, de teljes brute-force):**

```bash
john --incremental=All hashes.txt
```

### 10.4 Recommended usage

- Ha már megszereztél hash-t (pl. labban Windows SAM dump).
- Sokkal gyorsabb, mint online brute-force.
- SOC oldalon:  
  - gyanús `reg save`, `ntdsutil`, `vssadmin` parancsok → hash dumping indikátora.

---

## 11. SOC nézőpont – Milyen logot generálnak ezek?

### 11.1 Windows Security Log – Event ID-k

- **4625** – Failed logon  
- **4624** – Successful logon  
- **4648** – Explicit credential use  
- **4776, 4768, 4769, 4771** – Kerberos/NTLM auth események (AD környezet)

### 11.2 Sysmon

- **Event ID 1** – Process creation (pl. `hydra.exe`, `ncrack`, stb.)  
- **Event ID 3** – Network connection (tömeges login próbák egy IP-re)  
- **Event ID 10** – Process access (ha memory dumping, stb.)

### 11.3 Wazuh / SIEM

- Authentication failure corralation rule-ok  
- RDP brute-force detection (sok 4625 kis időn belül)  
- WinRM anomalous login detection  
- PowerShell suspicious activity (reverse shell, encoded command)

---

## 12. Gyakorlati tanácsok labhoz

- **Always snapshot**: brute-force előtt csinálj snapshotot a VM-ekről.  
- **Lockout policy ismerete**:  
  - `net accounts` → Lockout threshold / duration  
- **Lassú brute-force production-szimulációhoz:**  
  - Hydra: `-W 3` vagy több  
  - Ncrack: `--delay 5s`  
- **Wordlist optimalizálás:**  
  - nagy általános lista: `rockyou.txt`  
  - célzott lista: CeWL + Crunch  
- **Dokumentáld a támadást:**  
  - parancsok  
  - log screenshotok  
  - Wazuh / SIEM alert screenshotok  
  - ebből lesz a legjobb portfólió anyagod.

---
