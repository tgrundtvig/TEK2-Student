# Eksamen i Teknologi 2
Version: 20. April 2026

## Disclaimer
Dette er første udkast til eksamensspørgmål og der KAN evt. forekomme mindre justeringer.

## Emner i Teknologi 2

Der er 7 emner til eksamen, man kan trække:

1. CI/CD, GitHub Actions
2. Docker
3. TCP/IP
4. DNS og HTTP(S)
5. Linux-terminalen
6. Cloud
7. Kryptering og sikkerhed

Man kan til hvert af emnerne forberede en disposition, hvor man
svarer på en række spørgsmål og demonstrerer en række
færdigheder på computeren. Mange ting er nemmere at forklare, når
man viser dem. Det er også mere overbevisende at demonstrere
færdighed inden for et emne, end viden om et emne.

Her i dokumentet finder du forslag til spørgsmål og demonstrationer.
Du må gerne gøre noget andet, men du skal være forberedt på at
eksaminator spørger ind til nedenstående. Man kan også risikere at
eksaminator spørger ind til et andet, relateret emne, end det man
trækker. For eksempel Docker, selvom man trak Cloud.

Til demonstration kan det være en god idé at have eksempler parat,
fx fra en opgave eller et projekt hvor du har brugt det. Alternativt
kan du tænke på noget, du kan søge efter på nettet, for at få et eksempel, du kan snakke ud fra.

Fordelen ved at eksemplet ligger parat på GitHub eller i en
projektmappe på computeren er, at man kan interagere med det.

---

## 1. CI/CD, GitHub Actions

Eksempler på spørgsmål:

1. Hvilke tjek laver man oftest i sin CI-pipeline?
2. Hvad er forskellen på `jobs:` og `steps:` i GitHub Actions?
3. Hvilken betydning har nøgleordene `uses:` og `run:`?
4. Hvad er konkrete eksempler på hvor `run:` er nyttig?
5. Hvordan virker caching i GitHub Actions; hvad er en cache key?
6. Hvad er formålet med hash-pinning af ens byggetrin?
7. Hvilke af Maven's build lifecycle-trin giver mening at køre i CI?
8. Hvordan aktiverer man et GitHub Actions workflow manuelt?
9. Hvordan bygges og uploades Docker images i GitHub Actions?

Eksempler på demonstration:

1. Forklar en GitHub Actions-fil til at teste Java-kode med Maven
2. Tilføj caching af 3rd-party Maven dependencies
3. Tilføj `workflow_dispatch:` til et projekt og kør CI manuelt
4. Udfør hash-pinning af en enkelt 3rd-party action

---

## 2. Docker, Docker Compose, Dockerfile

Eksempler på spørgsmål til Dockerfile:

1. Hvad er forskellen på en Dockerfile, et image, og en container?
2. Hvad er en Dockerfile, og hvordan fungerer Docker's build cache?
3. Hvad betyder `FROM maven:3.9.9-eclipse-temurin-21`?
4. Hvad er formålet med multi-stage builds?
5. Hvad er formålet med et volume mount i en Docker-container?
6. Hvornår vil man bruge Dockerfile's `COPY`, og hvornår
   `compose.yaml`'s `volumes`?

Eksempler på spørgsmål til Docker Compose:

7. Hvorfor vil man gerne bruge Docker Compose ud over Docker?
8. Hvordan deployer man med Docker eller Docker Compose?
9. Hvad er forskellen på `image:` og `dockerfile:` i `compose.yaml`?
10. Hvordan fungerer port mapping mellem host og container? For
    eksempel `8080:8080`, `127.0.0.1:3307:3306`

Eksempler på demonstration:

1. Forklar opbygningen af en Dockerfile fra undervisningen
2. Start/stop/slet container, vise tilgængelige containere
3. Byg en simpel Docker container og kør den med `-it`
4. Forklar en simpel `compose.yaml` og `up` den (fx mysql)
5. Fjern et image, ryd op i image cachen, prune systemet
6. Tilføj optimering af build cache for Dockerfile

---

## 3. TCP/IP

Eksempler på teoretiske spørgsmål om TCP/IP / OSI-modellen:

1. Hvilke netværks-lag findes der i TCP/IP- eller OSI-modellen?
2. Hvad er en IP-adresse og hvad er en MAC-adresse?
3. Hvorfor kan jeg ikke aflæse ek.dk's MAC-adresse i WireShark?
4. Hvilke funktioner og forskelle har en router og en switch?

Eksempler på mere praktiske spørgsmål:

5. Hvad er forskellen på din lokalnet-adresse og din public adresse?
6. Hvordan filtrerer jeg trafikken til en specifik hjemmeside i
   WireShark, eller for en specifik protokol som fx `tcp` eller `dhcp`?
7. Hvad er en port, og hvorfor har man porte? Hvad er nogle
   eksempler på standardporte?
8. Hvad betyder src og dst for en IP-pakke?
9. Hvad betyder src port og dst port for en TCP-pakke?
10. Hvad er SYN, ACK, og FIN? Hvordan kan jeg se det i WireShark?
11. Hvad er forskellen på TCP og UDP?
12. Hvad bruges Sequence Number og Acknowledgment
    Number til? Vis et eksempel i WireShark.

Eksempler på demonstration:

1. Vis en illustration af TCP/IP- / OSI-modellen og forklar lagene.
2. Find din lokalnet IP-adresse og din offentlige IP-adresse.
3. Forklar outputtet fra `nslookup ek.dk` og `ping ek.dk`
4. I WireShark: filtrér så det kun er trafik til/fra en hjemmeside, fx
   ek.dk eller egen VM.
5. I WireShark: Vis hvordan man får tildelt en IP-adresse med DHCP
6. Vis TCP handshake i WireShark ved at `curl`'e en hjemmeside (fx
   egen vm)
7. Vis TCP teardown i WireShark.

---

## 4. HTTP og DNS

Eksempler på DNS-spørgsmål:

1. Hvem bestemmer hvilken IP-adresse et domæne peger på?
2. Hvilke slags DNS record-typer findes der?
3. Hvad er TTL for en DNS record, og hvorfor sætter man den højt
   eller lavt?
4. Hvad er forskellen på en autoritativ navneserver og en cachende
   navneserver?
5. Hvordan fungerer en rekursiv resolver?

Eksempler på HTTP-spørgsmål:

1. Hvornår bruger man hhv. GET og POST?
2. Hvilke HTTP-statuskoder findes der? 2xx, 3xx, 4xx, 5xx
3. Hvad er forskellen på HTTP-headerne Accept og Content-Type?
4. Hvis HTTP er en tilstandsløs protokol, hvordan forbliver man
   logget ind på en hjemmeside?

Eksempler på demonstration:

1. Lav et simpelt DNS-opslag med `nslookup` eller `host` på domænet
   ek.dk og forklar
2. Lav et avanceret DNS-opslag med `dig` eller dnschecker.org på
   domænet ek.dk og forklar
3. Besøg en hjemmeside mens DevTools Netværksfanebladet er
   åbent, og vis request, response mv.
4. I WireShark: Vis en HTTP GET-request og dens response ved at
   `curl`'e en hjemmeside (fx egen vm)

---

## 5. Linux-terminalen

Eksempler på spørgsmål:

1. Hvad er forskellen på en terminal (fx `cmd.exe`), en shell (fx `bash`)
   og en kommando (fx `ls`)?
2. Hvad er SSH, og hvordan fungerer public key authentication til en
   cloud VM?
3. Hvordan opretter man forbindelse til en remote server via SSH?

Eksempler på demonstration:

1. Opret og naviger mellem mapper i Linux-terminalen
2. Opret og rediger en tekstfil ved hjælp af en terminal-baseret
   teksteditor
3. Overfør filer mellem lokal maskine og en remote server ved hjælp
   af SCP
4. Søg en tekstfil vha. `head`, `tail`, `cat`, `less` og `grep`.
5. Pipe outputtet fra en kommando ind i en tekstfil (`>`)
6. Pipe outputtet fra en kommando ind i et andet program (`|`)
7. Vis hvordan man undgår at overskrive filer, når man piper data
   ind i dem
8. Forklar filrettigheder såsom `-rwxr--r--` og skift dem med `chmod`

---

## 6. Cloud

Eksempler på spørgsmål:

1. Hvad er IaaS og PaaS, og hvad har du brugt?
2. Foruden VM'er, hvilke slags produkter tilbyder clouds?
3. Hvordan deployer man en Spring Boot-app i cloud'en?
4. Hvordan kan ens Docker port-konfiguration påvirke sikkerheden?

Eksempler på demonstration:

1. Vis CD-delen af et GitHub Actions-workflow for et projekt
2. Vis hvordan man ændrer firewall-regler for en cloud VM
3. Vis hvordan man logger ind på en cloud VM

---

## 7. Kryptering og sikkerhed

Eksempler på spørgsmål om kryptering:

1. Hvad er forskellen på symmetrisk og asymmetrisk kryptering, og
   hvornår bruger man hvad?
2. Hvad er en hash-funktion, og hvordan adskiller den sig fra
   kryptering?
3. Hvad er en digital signatur, og hvad kan man bruge den til?
4. Hvad er GPG, og hvad kan man bruge det til?

Eksempler på spørgsmål om TLS og certifikater:

5. Hvordan foregår et TLS handshake? (ClientHello, ServerHello,
   certifikat, nøgleudveksling)
6. Hvad er en certifikatkæde, og hvad er en Certificate Authority (CA)?
7. Hvorfor stoler min browser på et certifikat udstedt af Let's Encrypt?
8. Hvorfor kan jeg se en HTTP-request i klartekst i WireShark, men
   ikke en HTTPS-request?

Eksempler på spørgsmål om SSH, VPN og tunnels:

9. Hvordan fungerer SSH public key authentication under
    motorhjelmen?
10. Hvad er forskellen på `id_ed25519` og `id_ed25519.pub`, og
    hvilken af dem må man dele?
11. Hvad er en SSH tunnel (port forwarding), og hvornår er det nyttigt?
12. Hvad er en VPN, og hvordan adskiller en VPN som WireGuard sig
    fra en SSH tunnel?

Eksempler på demonstration:

1. Kryptér og dekryptér en fil med `openssl` (fx symmetrisk AES)
2. Generér et GPG-nøglepar og kryptér/dekryptér en besked
3. Generér et SSH-nøglepar og forklar forskellen på den private og
   den offentlige nøgle
4. Inspicér et website's certifikat med `openssl s_client -connect
   ek.dk:443` eller `openssl x509 -text`
5. Vis et TLS handshake i WireShark — peg på ClientHello,
   ServerHello og Certificate-pakkerne
6. Sæt en SSH tunnel op (fx `ssh -L 8080:localhost:80 user@vm`) og
   forklar hvorfor trafikken er krypteret
7. Vis en WireGuard-opsætning mellem to containere eller VM'er
