# Studentene spør – bransjen svarer · live resultatskjerm

En resultatskjerm for paneldebatten til IxDA Bergen. Publikum og panel scanner en QR-kode, svarer i et Google-skjema, og skjermen viser live hvordan **studenter/jobbsøkere**, **konsulenthus**, **inhouse-miljøer** og **andre** svarte, og samler spørsmål fra salen. Utsagnene sorteres etter hvor uenige gruppene er, slik at debatten kan starte der det er størst avstand.

- Gratis: Google Skjema + Google Regneark + GitHub Pages
- Ingen innlogging for de som svarer, og svarene er anonyme
- Alt oppsett gjøres i én konfig-blokk øverst i `index.html`

Åpne `index.html?demo` for å se skjermen med oppdiktede testdata.

---

## 1. Lag Google-skjemaet (ca. 10 min)

1. Gå til [forms.google.com](https://forms.google.com) og lag et nytt skjema.
2. Under **Innstillinger**: sørg for at *Samle inn e-postadresser* er **av** og at *Begrens til 1 svar* er **av**, så slipper folk å logge inn.
3. **Første spørsmål**: «Hvem er du?» med typen *Flervalg*. Gjør det obligatorisk. Svaralternativer:
   - `Student / jobbsøker`
   - `Representerer konsulenthus`
   - `Representerer bedrift med in-house miljø`
   - `Andre`

   Skjermen kjenner igjen gruppene på ordene *student*, *konsulent*, *inhouse* (med eller uten bindestrek) og *andre*, så formuleringen kan endres så lenge de ordene er med. Ordene kan ikke stå i mer enn ett av alternativene.
4. **Ett spørsmål per utsagn** med typen *Lineær skala* fra **1 til 5**, med etikettene *Helt uenig* (1) og *Helt enig* (5). Spørsmålsteksten må være den samme som i `statements` i konfigurasjonen (små forskjeller i tegnsetting går fint).
   Tips: lag det første utsagnet ferdig, og bruk *Dupliser* for resten.
5. **Til slutt (valgfritt):** et spørsmål med typen *Avsnitt* (fritekst), f.eks. «Har du et spørsmål til panelet?». Det må ikke være obligatorisk. Teksten må inneholde ordet **spørsmål**, så finner skjermen det.
6. Gå til fanen **Svar** og klikk **Koble til Regneark** → *Opprett et nytt regneark*.

## 2. Del regnearket

1. Åpne regnearket og klikk **Del** øverst til høyre.
2. Under *Generell tilgang*: velg **Alle som har linken** med rollen **Leser**.
3. Kopier ID-en fra adresselinjen, altså den lange koden mellom `/d/` og `/edit`:
   `https://docs.google.com/spreadsheets/d/`**`1AbC...xyz`**`/edit`
4. Sjekk navnet på fanen nederst. På norsk heter den vanligvis **Skjemasvar 1**.

> Svarene inneholder bare tidspunkt, gruppe og tall på skalaen, men alle med lenken til regnearket kan se dem. Ikke legg til spørsmål om navn eller e-post.

## 3. Fyll inn konfigurasjonen

Åpne `index.html` i en teksteditor. Øverst ligger blokken `const CONFIG = { ... }`:

| Felt | Hva du legger inn |
|---|---|
| `formUrl` | Lenken til skjemaet (Send → lenke-ikonet). Denne blir QR-koden. |
| `shortUrl` | Valgfri: forkortet lenke (`forms.gle/...`) som vises under QR-koden |
| `sheetId` | ID-en fra steg 2 |
| `sheetName` | Navnet på fanen, f.eks. `Skjemasvar 1` |
| `roleQuestion` | Teksten på «Hvem er du?»-spørsmålet |
| `groups` | Gruppenavnene som vises, ordet som skal kjennes igjen i svaret, og formen (`circle`, `square`, `triangle`, `diamond`) |
| `statements` | Utsagnene, i samme rekkefølge og ordlyd som i skjemaet |
| `scale` | Bare endre den hvis dere bruker en annen skala enn 1–5 |

Utsagnene som ligger der nå er **forslag** som dere kan bytte ut.

## 4. Publiser på GitHub Pages

1. Lag et nytt repo på GitHub (f.eks. `ixda-panel`) og last opp `index.html` (og gjerne denne README-en).
2. Gå til **Settings → Pages**, velg *Deploy from a branch*, branch `main` og mappen `/ (root)`, og lagre.
3. Etter et minutt eller to ligger skjermen på `https://<brukernavn>.github.io/ixda-panel/`.

Hvis noe ikke stemmer, viser statuslinjen nederst til høyre en feilmelding. Vanlige årsaker:
- *«Fant ikke spørsmålet …»*: `roleQuestion` står ikke likt som i skjemaet.
- *«X utsagn ikke funnet»*: ordlyden i `statements` avviker fra spørsmålene.
- *«Kunne ikke nå regnearket»*: regnearket er ikke delt med «Alle som har linken».

## 5. Før arrangementet

- Send inn et par testsvar og sjekk at de dukker opp (skjermen henter nye svar hvert 3. sekund).
- **Slett testsvarene** etterpå: i regnearket markerer du radene og sletter dem, eller du bruker *Svar → Slett alle svar* i skjemaet.
- Test QR-koden fra bakerste rad i salen.

## Raskere oppdatering (anbefalt for live-bruk)

Skjermen spør etter nye svar hvert 3. sekund. Med standardoppsettet (regneark-ID) kan Google likevel sende en litt gammel kopi, så det kan ta fra noen sekunder til et par minutter før nye svar dukker opp. Hvis dere vil være sikre på at svarene kommer fram med en gang, kan dere sette opp et lite Apps Script som leser regnearket direkte. Det tar ca. 5 minutter:

1. Åpne regnearket med svarene og velg **Utvidelser → Apps Script**.
2. Slett det som står der, og lim inn dette:

   ```js
   function doGet(e) {
     const ss = SpreadsheetApp.getActiveSpreadsheet();
     const sheet = ss.getSheetByName("Skjemasvar 1") || ss.getSheets()[0];
     const body = JSON.stringify(sheet.getDataRange().getDisplayValues());
     const cb = e && e.parameter && e.parameter.callback;
     if (cb && /^[\w$]+$/.test(cb)) {
       return ContentService.createTextOutput(cb + "(" + body + ")")
         .setMimeType(ContentService.MimeType.JAVASCRIPT);
     }
     return ContentService.createTextOutput(body).setMimeType(ContentService.MimeType.JSON);
   }
   ```
   Heter fanen noe annet enn «Skjemasvar 1», endrer du navnet i den tredje linjen.
3. Klikk **Lagre**, og så **Implementer → Ny implementering**.
4. Klikk tannhjulet og velg **Nettapp**. Sett *Kjør som* til **Meg** og *Hvem har tilgang* til **Alle**. Klikk **Implementer** og godkjenn tilgangen. Google viser en advarsel fordi skriptet ikke er verifisert. Det er normalt for egne skript: velg *Avansert → Gå til …*.
5. Kopier **nettapp-lenken** (slutter på `/exec`) og lim den inn i `scriptUrl` i `index.html`.

Når `scriptUrl` er fylt inn, bruker skjermen den i stedet for regneark-ID-en. Da trenger heller ikke regnearket være delt med «Alle som har linken».

**Verdt å vite**
- Alle som har nettapp-lenken kan lese svarene, akkurat som med en delt regneark-lenke. Svarene er anonyme, så det er greit, men ikke del lenken unødvendig.
- Endrer dere skriptet senere, må dere lage en ny versjon under *Implementer → Administrer implementeringer*. Lenken forblir den samme.
- Google har grenser for hvor mye et skript kan kjøres per dag, men én skjerm som spør hvert 3. sekund i et par timer er godt innenfor dem. Det er bare storskjermen som henter svar, ikke telefonene til publikum.
- Ikke sett `refreshSeconds` lavere enn 2–3 sekunder. Det gjør ikke svarene raskere, men øker sjansen for at Google begrenser forespørslene.

**Test før arrangementet:** send inn et svar fra mobilen og tell sekundene til tallet øker nederst til høyre på skjermen.

## Styring under debatten

| Tast | Knapp | Hva skjer |
|---|---|---|
| `Q` | QR | Stor QR-kode og live antall svar per gruppe. Vis den mens folk svarer. |
| `?` | Spørsmål | Spørsmål fra salen. Merk dem som *Tatt ✓* når panelet har svart, eller *Skjul* for å fjerne et spørsmål fra skjermen. |
| `O` / `Esc` | Oversikt | Alle utsagn med gruppenes snitt, sortert etter størst uenighet |
| `→` / `←` | Neste / Forrige | Ett utsagn om gangen med fordeling per gruppe |
| `S` | Sorter | Bytt mellom «størst uenighet» og opprinnelig rekkefølge |
| `A` | Vis alle svar / Vis snitt | Bytter mellom å vise bare snittet per gruppe og å vise hvert enkelt svar som en prikk i gruppefargen, så dere ser spredningen |
| `H` | Skjul resultater | Skjuler tallene til dere vil vise dem, så panelet ikke blir påvirket |
| `T` | Tabell | Viser tallene som tabell under grafen |
| `F` | Fullskjerm | Gjør siden til fullskjerm (trykk igjen eller Esc for å avslutte) |
| `P` | Pause / Fortsett | Stopper oppdateringen, så grafene står stille mens dere diskuterer. Trykk igjen for å hente nye svar. |
| `R` | – | Hent svar på nytt med en gang |

Klikk på et utsagn i oversikten for å gå rett til det. En klikker eller presenter-fjernkontroll fungerer også, siden den sender piltastene.

**Slik leser du skjermen**
- *Oversikt*: hver prikk er en gruppes gjennomsnitt. Langt fra hverandre betyr uenighet. «Avstand» er forskjellen mellom høyeste og laveste gruppesnitt, og utsagnet med størst avstand er merket med ▲.
- *Alle svar* (`A`): hver prikk er ett svar i fargen til gruppen sin, og den hvite streken er snittet. Da ser dere om en gruppe er samstemt eller delt i to.
- *Detalj*: rødt er uenig, grått er nøytralt og blått er enig. Midtlinjen er nøytral, så stolper som drar i hver sin retning viser hvor debatten ligger.

Merk: med få svar i en gruppe (f.eks. 4 fra inhouse) kan én person flytte snittet mye. Antall svar står alltid ved siden av gruppenavnet.

**Universell utforming:** gruppene skilles både på farge og form (sirkel, firkant, trekant), så skjermen fungerer også for fargeblinde og i svart-hvitt. Fargene er valgt slik at de skiller seg fra hverandre også ved de vanligste formene for fargeblindhet.

**Spørsmål fra salen:** fritekst vises akkurat slik folk skrev den. Se gjerne over spørsmålene på din egen skjerm før dere viser dem på storskjermen, og bruk *Skjul* på det som ikke passer. Hva som er markert som tatt eller skjult, huskes i nettleseren på maskinen som viser skjermen.
