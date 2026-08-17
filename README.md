# Plump – digital resultattavla

En resultattavla för kortspelet **Plump**, byggd som en enda statisk HTML-fil.
Ingen server, ingen databas, inga beroenden – öppnas direkt i webbläsaren på
mobil, platta eller dator.

## Så funkar den

1. **Lägg in spelarna** (2–10 st), välj vem som ger först och hur trappan ser
   ut. Standard är 8 kort ner till 2 och upp igen – samma trappa som på pappret.
2. **Tabellen** visar en rad per runda. Vänsterkolumnen (som ligger kvar när du
   scrollar i sidled) visar antal kort och vems giv rundan är.
3. **Tryck på en ruta** och välj resultat:
   * **plump** (missat bud) = 0 poäng, visas som ●
   * **klarad nolla** = 5 poäng
   * 1 stick = 11 poäng, 2 = 12, 3 = 13 … 8 = 18
4. Efter varje val hoppar väljaren vidare till nästa spelare i rundan, så en
   hel runda matas in med ett tryck per spelare. När rundan är full stängs
   väljaren så du hinner se ställningen.
5. Varje ruta visar rundans poäng stort och **den löpande totalen** litet under.
   Totalsumman per spelare står alltid överst i den fastnålade rubriken.
6. Ledaren markeras i grönt, och när sista rundan är ifylld visas vinnaren.

Övrigt: **Ångra** ångrar senaste inmatningen och **Ny match** nollställer
poängen men behåller spelarna. **Spelare** går tillbaka till startvyn – där
tar *Tillbaka till matchen* dig in i den pågående matchen igen med poängen
kvar (bra om ett namn är felstavat), medan *Starta om matchen* nollställer.
Allt sparas automatiskt i webbläsaren, så matchen ligger kvar om telefonen
låser sig eller fliken stängs. Är lagring blockerad (privat läge) hålls
matchen i minnet istället för att appen ska sluta fungera.

En liten kontroll finns också: om de inmatade sticken inte får plats i rundan
ramas rutorna in i rött och en varning visas. Plump och klarad nolla räknas
inte som stick i den kontrollen, eftersom de sticken inte skrivs upp.

## Kör den lokalt

Öppna `index.html` i valfri webbläsare – det räcker.

## Gratis hosting via GitHub Pages

Repot innehåller bara statiska filer, så GitHub Pages räcker (och kostar inget):

1. Se till att `index.html` ligger i roten på `main`-branchen.
2. Gå till **Settings → Pages** i repot.
3. Under *Build and deployment* väljer du
   **Source: Deploy from a branch**, **Branch: `main`**, mapp **`/ (root)`**.
4. Spara. Efter ~1 minut ligger sidan på
   `https://<ditt-användarnamn>.github.io/plump/`.

Är repot privat krävs betald plan för Pages – gör det publikt, eller använd
[Netlify Drop](https://app.netlify.com/drop) (dra in mappen i webbläsaren) eller
Cloudflare Pages, som båda har gratisnivåer och funkar likadant för statiska
filer.

## Lägg till på hemskärmen

Öppna adressen i mobilen och välj *Dela → Lägg till på hemskärmen* (iOS) eller
*Installera appen* (Android/Chrome). Tack vare `manifest.webmanifest` startar
den då i helskärm och ser ut som en vanlig app.

`sw.js` gör att den också fungerar **utan täckning** när den väl har öppnats en
gång från en https-adress. Den hämtar alltid från nätet först och faller
tillbaka på cachen, så en ny version syns direkt när man är uppkopplad – ingen
risk att en gammal kopia fastnar. Vid lokal körning via `file://` är service
workern avstängd.

## Filer

| Fil | Innehåll |
| --- | --- |
| `index.html` | Hela appen – markup, css och js i en fil |
| `sw.js` | Service worker för offline-läge |
| `manifest.webmanifest` | Gör att den kan installeras på hemskärmen |
| `icon.svg` | Appikon |
| `apple-touch-icon.png` | Hemskärmsikon för iOS (180×180) |
