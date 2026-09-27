# Dobbelt Pi-hole-kontrolpanel

En enkelt selvstændig `index.html`, der samler to Pi-hole v6-installationer
i én administrationsvisning: dashboard med grafer, forespørgselslog,
lister over tilladte og blokerede domæner, lokale DNS-poster, klienter
(herunder "ukendte enheder på netværket" fra Pi-holes egne forslag),
adlists og Gravity-opdateringer. Hvor det giver mening, findes der desuden
funktioner til at tilføje eller slette på begge servere samtidig.

Kontrolpanelet kommunikerer direkte med hver Pi-holes egne `/api/...`-endpoints
fra din browser. Der er ingen backend, database eller buildproces. Det hele
ligger i én HTML-fil.


## Krav

- To Pi-hole **v6**-installationer med det nye REST API. Løsningen fungerer
  ikke med det gamle PHP API i Pi-hole v5.
- En nginx eller tilsvarende reverse proxy foran kontrolpanelet, så browserens
  forespørgsler til `/api1/...` og `/api2/...` videresendes til dine to faktiske
  Pi-hole-installationer. Dermed undgås CORS-problemer helt. Se
  `nginx.conf.example`.
- En "app password" til hver Pi-hole. Den oprettes under **Settings → API** i
  den enkelte Pi-holes eget webinterface. Brug denne i stedet for din rigtige
  administratoradgangskode, når du logger ind på kontrolpanelet.


## Opsætning

1. Kopiér `index.html` til den placering, hvor nginx skal levere den fra,
   eksempelvis `/srv/web/pihole-dashboard/index.html`.
2. Download Chart.js til samme mappe. Det bruges til graferne på dashboardet:
   ```bash
   cd /srv/web/pihole-dashboard
   curl -o chart.umd.min.js https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js
   ```
3. Kopiér `nginx.conf.example` til din nginx-konfiguration. Tilpas domænet,
   værtsnavnene eller IP-adresserne på de to Pi-hole-installationer,
   certifikatstierne og placeringen af webfilerne.
4. Kør `nginx -t && systemctl reload nginx`. Åbn derefter kontrolpanelets
   domæne i en browser, og log ind på hver Pi-hole med dens app password.

Det er det hele. Det er ikke nødvendigt at redigere selve `index.html` yderligere.
Kontrolpanelet forventer allerede at finde de to Pi-hole-installationer på de
relative stier `/api1` og `/api2`, hvilket er præcis det, nginx-skabelonen
videresender for dig.

Hvis du hellere vil pege direkte på Pi-hole-installationernes faktiske adresser
i stedet for at bruge en proxy, kan du ændre de to serveradresser under fanen
**Indstillinger** i selve kontrolpanelet. Du må dog forvente, at nogle faner kan
fejle med CORS-fejl, da Pi-holes egen CORS-håndtering ikke er ensartet på tværs
af alle endpoints.


## Sikkerhed

- Løsningen er udviklet til **personlig brug eller brug blandt personer, du
  har tillid til**, som administrerer dine egne Pi-hole-installationer. Den er
  ikke beregnet som et offentligt tilgængeligt værktøj. Alle, der kan logge ind,
  har fulde administratorrettigheder på begge Pi-hole-installationer. De kan
  deaktivere blokering, slette domæner og lister samt starte Gravity-opdateringer.
- Login-sessioner gemmes i browserens `sessionStorage`, som ryddes, når fanen
  lukkes, frem for i `localStorage`. Kontrolpanelet gemmer ikke selv
  adgangskoderne nogen steder.
- Kontrolpanelet neutraliserer værdier, der ikke er tillid til, før de vises,
  eksempelvis klientnavne fra DHCP, domæner og kommentarer. Løsningen leveres
  dog som én samlet fil med JavaScript direkte i HTML-filen. Det begrænser, hvor
  restriktiv en Content Security Policy kan være. Se kommentaren i
  `nginx.conf.example`. Hvis du vil dele adgangen med andre frem for kun at
  bruge løsningen selv, bør du overveje at flytte `<script>`-blokken til en
  ekstern `app.js`-fil. Derefter kan `'unsafe-inline'` fjernes fra `script-src`
  i CSP-headeren.
- Pull requests og issues er velkomne, både ved fejl og ved forslag, der kan
  give reel understøttelse af mere end to Pi-hole-installationer.
- Hvis kontrolpanelet afvikles på en Synology NAS, kan du oprette og tilknytte
  en **Profil til adgangskontrol** i DSM. Dermed kan adgangen begrænses til
  bestemte IP-adresser eller lokale netværk.


## Licens

Copyright © 2023–2026 Christian Kragh Jessen  
https://christiankragh.dk

Projektet blev oprindeligt udviklet til Pi-hole v5 i 2023 og opdateret til
Pi-hole v6 i 2026.

Projektet er udgivet under MIT-licensen og må bruges, kopieres, ændres og
videredistribueres i overensstemmelse med licensens vilkår. Se filen `LICENSE`.

Chart.js medfølger ikke og skal hentes separat som beskrevet under opsætningen.
Chart.js er et selvstændigt tredjepartsprojekt med sin egen licens.