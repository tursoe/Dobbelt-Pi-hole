# Dobbelt Pi-hole-kontrolpanel

[English](README.md) | **Dansk**

**v1.5.011**

Et let kontrolpanel til samlet administration af to, tre eller fire Pi-hole v6-installationer.
Løsningen består af én selvstændig `index.html` uden backend, database eller
buildproces. Browseren kommunikerer direkte med de aktive Pi-hole-installationers
REST API gennem en nginx reverse proxy.

Kontrolpanelet indeholder:

- Dashboard med samlede tal og grafer
- Overblik med status og sammenligning af opsætning på tværs af installationerne
- Fælles forespørgselslog med filtrering og sortering
- Administration af tilladte og blokerede domæner
- Lokale DNS-poster og CNAME med visning af forskelle mellem installationerne
- Kendte klienter, forslag til ukendte enheder og oprydning i Pi-holes netværkstabel
- Administration af grupper og gruppetilknytninger for domæner, klienter og lister
- Administration af adlists
- Gravity-opdateringer med sekventiel kø, log og tidspunktet for seneste opdatering
- Download af Teleporter-backup fra de valgte installationer
- Valgfri startfane under Indstillinger
- Handlinger, der kan udføres på én eller alle aktive installationer
- Dansk, norsk bokmål, nynorsk og engelsk brugerflade

Pi-hole-installationerne forbliver selvstændige DNS-servere. Projektet
opretter ikke failover, virtuel IP, primær eller sekundær server eller automatisk
baggrundssynkronisering. Formålet er at gøre det enkelt at administrere de
relevante indstillinger ens fra ét kontrolpanel.

![Pi-hole dashboard](images/dashboard.png)

## Letvægt som udgangspunkt

Kontrolpanelet er bevidst holdt enkelt:

- Én HTML-fil med HTML, CSS og JavaScript
- Ingen backend
- Ingen database
- Ingen buildproces
- Ingen Node.js, PHP eller Python
- Antal servere samt deres navne på tværs af klienter sættes i HTML-filen; egne navne og visningsindstillinger gemmes lokalt i browseren
- Direkte kommunikation med Pi-hole v6 API gennem nginx

Chart.js indlæses som en separat lokal fil og bruges kun til graferne.

## Krav

- To til fire Pi-hole **v6**-installationer med REST API. Løsningen fungerer ikke med
  det gamle PHP API i Pi-hole v5.
- nginx eller en tilsvarende reverse proxy foran kontrolpanelet. Forespørgsler
  til `/api1/...` og `/api2/...` samt `/api3/...` og `/api4/...`, når de bruges,
  skal videresendes til de tilsvarende Pi-hole-installationer. Dermed undgås CORS-problemer.
- Et app password til hver Pi-hole. Det oprettes under **Settings → API** i den
  enkelte Pi-holes eget webinterface.

### Installation med Docker

Hent `compose.yml`, og læg den i en tom mappe. Tilpas adresser og navne til dine Pi-hole installationer. Vælg HTTP eller HTTPS efter din opsætning, og tilpas indstillingerne for certifikatkontrol.

Åbn en terminal i mappen, og start dashboardet:

```bash
docker compose up -d
```

Docker henter automatisk det færdigbyggede image med dashboardet, nginx og Chart.js. Åbn derefter `http://DIN-SERVERS-IP:8080`.

Når en nyere version findes, viser Docker-udgaven disse opdateringskommandoer
under Indstillinger. Kør dem i mappen med din `compose.yml`:

```bash
docker compose pull
docker compose up -d
```

Se [Dockerprojektet](https://github.com/tursoe/Dobbelt-Pi-hole-Docker) for flere indstillinger og dokumentation.

## Opsætning med nginx

1. Kopiér `index.html` til den mappe, hvor nginx skal levere kontrolpanelet fra,
   eksempelvis `/srv/web/pihole-dashboard/index.html`.
   Kopiér også `icon.svg`, `favicon-*.png`, `android-chrome-*.png`,
   `apple-touch-icon.png` og `site.webmanifest` fra udgivelsen til samme mappe.
   Kopiér `language.js` til samme mappe for at aktivere tysk og andre sprogbidrag.
2. Download Chart.js til samme mappe:

   ```bash
   cd /srv/web/pihole-dashboard
   curl -o chart.umd.min.js https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js
   ```

3. Kopiér `nginx.conf.DA.example` til din nginx-konfiguration.
4. Tilpas domænet, Pi-hole-adresserne, certifikatstierne og placeringen af
   webfilerne. Sæt `DEFAULT_SERVER_COUNT` i `index.html` til `2`, `3` eller `4`.
   nginx-eksemplet har `/api1/` og `/api2/` aktive som standard. Fjern
   kommentartegnene fra `/api3/`-blokken, og tilpas den ved tre installationer.
   Ved fire installationer aktiveres og tilpasses både `/api3/` og `/api4/`.
5. Kontrollér og genindlæs nginx:

   ```bash
   nginx -t && systemctl reload nginx
   ```

6. Åbn kontrolpanelets adresse og log ind på alle aktive Pi-hole-installationer med
   deres app passwords.

## Antal servere og navne

Antallet af aktive Pi-hole sættes **kun i `index.html`**:

```js
const DEFAULT_SERVER_COUNT = 2; // 2, 3 eller 4
```

De første to, tre eller fire poster i `DEFAULT_SERVERS` bruges. API-stierne er
`/api1`, `/api2`, `/api3` og `/api4`. nginx skal have en proxy til hver aktiv
installation. De faktiske værtsnavne eller IP-adresser angives i nginx og ikke
i browserens indstillinger.

Standardnavne på tværs af klienter angives med `label` i `DEFAULT_SERVERS`:

```js
const DEFAULT_SERVERS = [
    { id: "1", label: "Pi-hole 1", base: "/api1", color: "c-1", chip: "s1" },
    { id: "2", label: "Pi-hole 2", base: "/api2", color: "c-2", chip: "s2" },
    { id: "3", label: "Pi-hole 3", base: "/api3", color: "c-3", chip: "s3" },
    { id: "4", label: "Pi-hole 4", base: "/api4", color: "c-4", chip: "s4" },
];
```

Serverfarverne er henholdsvis blå, gul, lilla og orange. Visningen tilpasser sig
antallet og skærmbredden med en maksimal sidebredde på 1900 px.

Navnene på de aktive installationer kan også ændres under **Indstillinger**.
Disse navne gemmes i browserens `localStorage` og erstatter standardnavnene i
netop den browser. Antallet og API-stierne kan ikke ændres under Indstillinger.
Ændringer af standardnavne i filen overskriver ikke lokalt gemte navne.

## Funktioner i v1.5.011

### Overblik

Overblik viser instansernes status og sammenligner domæner, lokale DNS-poster,
CNAME, klienter, grupper og lister. Visningen fremhæver manglende poster og
forskellige værdier eller gruppetilknytninger. En ufuldstændig sammenligning
markeres særskilt. Du retter selv forskellene på den relevante fane.

Et tryk på Overblik eller Grupper henter data igen, også når fanen allerede
er valgt. Visningen bygger på senest hentede data og overvåger ikke ændringer
i baggrunden.

### Grupper

Opret grupper med én fælles formular til én eller alle aktive instanser.
Manglende grupper kan tilføjes på den enkelte instans, og eksisterende grupper
kan redigeres eller slettes. Standardgruppen kan ikke omdøbes eller slettes.
Domæner, klienter og lister har egne handlinger til gruppetilknytninger.
Grupper matches efter navn på tværs af instanser; deres numeriske ID kan være
forskelligt. At kopiere en gruppe kopierer selve gruppen, ikke dens medlemmer.

Gruppetekster og knapper til gruppetilknytninger skjules automatisk på Domæner,
Klienter og Lister, når alle aktive instanser kun har standardgruppen (ID 0),
og alle poster på den aktuelle fane er tilknyttet den. Ekstra grupper, poster
uden standardtilknytning eller ufuldstændige data bevarer gruppevisningen.
Kontrollen bruger hele den hentede liste, også poster på andre sider.
Fanen Grupper er altid synlig. Visningen vurderes igen, når fanens data hentes;
gruppetilknytninger ændres ikke af, at felterne skjules.

### Lokal DNS og CNAME

Lokal DNS indeholder også oprettelse, redigering og sletning af CNAME-poster
med valgfri TTL og kontrol for ugyldige navne og CNAME-løkker. Ændringer af
CNAME genstarter Pi-holes DNS-tjeneste. Hjælpeteksten forklarer kravene til
måldomænet. Et direkte link med `#cnames` åbner CNAME-sektionen.

### Gravity og backup

Gravity kan opdateres på én eller flere valgte instanser. Køen tager én instans
ad gangen og viser fremdrift, fejl og advarsler. Følg DNS-kontrollen og eventuelle
bekræftelser i køen. Hold siden åben, indtil køen er færdig. Lukkes siden, kan
den igangværende opdatering fortsætte, men de næste starter ikke.

Overblik kan hente Teleporter-backups fra de valgte instanser. Hver backup
gemmes som en separat fil; browseren kan bede om tilladelse til flere downloads.
Gendannelse foretages i Pi-holes eget webinterface.

### Oprydning i sete klienter

Slet ved sete, ukendte enheder fjerner enhedsregistreringen fra netværkstabellen
på de valgte instanser. Bekræftelsen viser alle berørte IP-adresser, fordi
Pi-hole sletter hele enhedsregistreringen med dens tilknyttede adresser.
Klientopsætning og forespørgselshistorik bevares. Enheden kan dukke op igen,
hvis Pi-hole registrerer den på netværket igen.

### Startfane

Vælg startfane under Indstillinger. Valget gemmes i den aktuelle browser.
Dashboard er standard, og et direkte link til en fane har forrang for startfanen.

## Sprog

Brugerfladen findes på **English**, **Dansk**, **norsk bokmål** og **nynorsk**.
Sprogvælgeren øverst i kontrolpanelet ændrer straks brugerfladen, herunder
dialogtekster, dynamiske meddelelser, tabeller og grafer. Tal, procenttal,
datoer og klokkeslæt følger det valgte sprog.

Roy-Magne Mo har bidraget med norsk bokmål og nynorsk, baseret på commit
`8e90275`. Hans navn vises ved sprogvælgeren, når et af de norske sprog er valgt.

Det indledende sprog vælges i denne rækkefølge:

1. Et understøttet sprog, der tidligere er valgt og gemt i browserens
   `localStorage`.
2. Det første understøttede sprog i browserens prioriterede sprogliste.
   Regionale varianter som `de-DE`, `de-AT`, `en-GB` og `da-DK` understøttes.
3. Engelsk, hvis der ikke findes et understøttet sprog.

Dansk, engelsk, bokmål og nynorsk er indbygget i `index.html`. Den valgfrie [language.js](language.js)
tilføjer tysk og kan registrere flere sprogbidrag. Læg filen ved siden af
`index.html` for at aktivere dem. Mangler filen, eller kan den ikke indlæses,
fortsætter kontrolpanelet med de indbyggede sprog. Sprog, der ikke er tilgængelige,
udelades fra sproglisten. Et gemt sprog, som mangler, falder tilbage til et
understøttet browsersprog og derefter engelsk.

Hvert sprog i `window.PIHOLE_LANGUAGES` angiver `label` (sprogets eget navn),
`locale`, `author` og `messages`. Kopiér hele det engelske katalog fra `I18N.en`
i `index.html`, oversæt værdierne og bevar alle nøgler og navngivne parametre.
Gyldige bidrag tilføjes automatisk. Ældre sprogpakker til v1.4.010
accepteres fortsat; manglende nye tekster vises på engelsk. De oprindelige
obligatoriske nøgler skal stadig findes, og medsendte tekster skal have korrekte
navngivne parametre. Ugyldige bidrag springes over. Filen kan ikke erstatte de
indbyggede sprog. Nye bidrag bør indeholde alle aktuelle nøgler. Skabelonen markerer tekster,
der kun bevares af hensyn til kompatibilitet og kan stå uændret på engelsk.

Brug [language.template.js](language.template.js) som skabelon til et nyt sprog.
Skabelonen til v1.5.011 indeholder alle 368 tekstnøgler og en vejledning til
oversættere. Den kan også hentes under Indstillinger. Bevar nøglerne og oversæt
kun tekstværdierne. `language.template.js` indlæses ikke automatisk; det færdige
sprog skal tilføjes i `language.js`.
Katalogerne bruger `.one` til ental efter sprogets regler og `.other` til alle
øvrige antal, også når sproget har kategorierne zero, two, few eller many.

Oversættelserne bruger faste tekstnøgler og navngivne parametre. Flere sprog
kræver hverken backend, buildproces eller ekstra afhængigheder. Værdier som
domæner, navne og kommentarer fra brugeren oversættes aldrig.

**Kreditering:** Daniel Dietz har bidraget med den nøglebaserede sprogmotor og
den tyske oversættelse. Denne version bygger på hans arbejde frem til commit
`4a32733`. Hans navn vises også ved sprogvælgeren, når tysk er valgt. Andre
oversættere kan krediteres gennem sprogets `author`-felt. Daniels eksisterende
tekster i den vedlagte `language.js` er bevaret; de 115 nye tyske tekster til
v1.5.011 er suppleret med Google Translate og er ikke gennemgået af ham.

Opsætningsvejledning og nginx-eksempler findes på dansk og engelsk:

| Sprog | Dokumentation | nginx-eksempel |
| --- | --- | --- |
| English | [README.md](README.md) | [nginx.conf.example](nginx.conf.example) |
| Dansk | [README.DA.md](README.DA.md) | [nginx.conf.DA.example](nginx.conf.DA.example) |

## Sikkerhed

- API- og loginkald afviser HTTP-redirects. Lad nginx videresende direkte til den endelige Pi-hole API-adresse med korrekt HTTP eller HTTPS.
- Løsningen er beregnet til personlig brug eller personer, du har tillid til.
  Alle, der kan logge ind, får administratoradgang til de tilknyttede
  Pi-hole-installationer.
- Login-sessioner gemmes i browserens `sessionStorage` og forsvinder, når fanen
  lukkes. Adgangskoder gemmes ikke.
- Kontrolpanelet neutraliserer værdier, der ikke er tillid til, før de vises,
  eksempelvis klientnavne, domæner og kommentarer.
- Løsningen indeholder JavaScript direkte i HTML-filen. Det begrænser, hvor
  restriktiv en Content Security Policy kan være. Se kommentaren i
  `nginx.conf.DA.example`.
- Hvis kontrolpanelet afvikles på en Synology NAS, kan en profil til
  adgangskontrol i DSM begrænse adgangen til bestemte IP-adresser eller lokale
  netværk.

Kontrolpanelet bør ikke eksponeres direkte på internettet uden passende
adgangskontrol foran det.

## Versionshistorik

- **v1.1.005:** Komplet dansk og engelsk brugerflade og dokumentation
- **v1.2.006:** Understøttelse af to, tre eller fire Pi-hole-installationer
- **v1.3.007:** Sprogmotoren genskrevet med keys, ekstra sprog findes i language.js
- **v1.4.008:** Docker-image og forenklet installation
- **v1.4.010:** Tilføjet norsk bokmål og nynorsk samt forberedt versionskontrol til Docker udgaven. Tidsvalg i forespørgselsloggen er begrænset til 1, 6 og 24 timer.
- **v1.5.011:** Overblik, grupper og gruppetilknytninger, CNAME, Teleporter-backup, sekventiel Gravity-kø, oprydning i sete klienter og valgfri startfane. Udvidet sprogskabelon med engelsk fallback for ældre sprogpakker.

Kontrolpanelet understøtter to til fire aktive installationer. Ekstra
installationer kræver både det tilsvarende antal i `index.html` og en proxy i nginx.

## Bidrag

**Udviklingssprog:** Kildekodekommentarer og HTML-grundtekster vedligeholdes på dansk. Oversættelseskatalogerne bevarer deres respektive sprog. Eksisterende funktionsnavne, API-felter og tekstnøgler bevares af hensyn til kompatibilitet.

Issues og pull requests er velkomne. Bidrag kan blandt andet være fejlrettelser,
forslag og oversættelser til flere sprog. Brug de samme meddelelsesnøgler og
navngivne parametre i alle sprogkataloger, og medtag oversættelser af statisk
tekst, dynamiske tilstande og tilgængelighedstekster.

## Licens

Copyright © 2023–2026 Christian Kragh Jessen  
https://christiankragh.dk

Projektet blev oprindeligt udviklet til Pi-hole v5 i 2023 og opdateret til
Pi-hole v6 i 2026.

Projektet er udgivet under MIT-licensen og må bruges, kopieres, ændres og
videredistribueres i overensstemmelse med licensens vilkår. Se filen `LICENSE`.

Ved manuel installation skal Chart.js hentes separat som beskrevet under
opsætningen. Docker-imaget indeholder Chart.js.
Chart.js er et selvstændigt tredjepartsprojekt med sin egen licens.
