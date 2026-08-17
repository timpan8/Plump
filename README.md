# Plump – digital resultattavla

En resultattavla för kortspelet **Plump**, byggd som en enda statisk HTML-fil.
Ingen server, ingen databas, inga beroenden – öppnas direkt i webbläsaren på
mobil, platta eller dator.

## Så funkar den

1. **Lägg in spelarna** (2–10 st), välj vem som ger först och hur trappan ser
   ut. Standard är 8 kort ner till 2 och upp igen – samma trappa som på pappret.
2. **Tabellen** visar en rad per runda. Vänsterkolumnen (som ligger kvar när du
   scrollar i sidled) visar antal kort och **vem som ger** – och samma sak
   märks ut med ett litet *ger* uppe i den spelarens ruta, så man ser det utan
   att räkna sig fram från vem som bjuder först.
3. **Den röda knappen under tabellen visar alltid nästa steg** – vems bud eller
   vems resultat som står i tur. Tryck på den och fyll i.

### Först bud, sedan resultat

Varje runda matas in i två svep:

* **Buden.** Alla säger hur många stick de tror de tar, 0 upp till antalet kort.
  Turordningen följer bordet: den som sitter efter given bjuder först och
  **given bjuder sist**. Väljaren går automatiskt vidare till nästa spelare och
  stannar när alla bud är lagda, så handen kan spelas.

  Ordningen är svår att göra fel på: rutan för den som är i tur är inramad,
  övriga tomma rutor är nedtonade, och trycker du på fel spelare öppnas inte
  den rutan – i stället kommer en fråga som visar turordningen
  (*Anna → Kalle → Tim (ger)*) med en knapp till rätt spelare. Vill du ändå
  fylla i någon annan finns en andra knapp för det, men det kräver ett
  medvetet tryck.
* **Resultatet.** När handen är spelad fyller du i, spelare för spelare, om
  budet **klarades** (10 + budet, eller 5 poäng för en klarad nolla) eller blev
  **plump** (0 poäng, visas som ●). Samma ordning och samma spärr gäller här.

Innan resultatet är ifyllt står budet kvar som en grå siffra med texten *bud*
i rutan, så man ser vad var och en satsade.

### Budspärren för given

Budsumman får inte bli exakt lika med antalet kort, och det är given som får ta
smällen eftersom hen bjuder sist. Spelar ni 5 kort och de två första bjuder 2
och 2, så är **1 spärrat** för given – appen gråar ut den knappen och skriver
varför. Alla andra bud går att välja.

### Låst till aktuell runda

Bara den runda som står i tur går att fylla i, så man inte råkar skriva på fel
rad. Trycker du på en annan rad säger appen till istället för att öppna
väljaren. Behöver du rätta något i efterhand trycker du på **Rätta äldre**, då
låses hela tavlan upp tills du trycker igen (låset slås alltid på vid omstart).

Varje ruta visar rundans poäng stort och **den löpande totalen** litet under.
Totalsumman per spelare står alltid överst i den fastnålade rubriken. Ledaren
markeras i grönt, och när sista rundan är ifylld visas vinnaren.

### Tidigare matcher

Färdigspelade matcher sparas automatiskt och listas under **Tidigare matcher**
längst ner i startvyn – med datum, vinnare, slutpoäng och vilka som var med.
Tryck på en rad för att se hela den gamla tavlan, precis som den såg ut när
matchen tog slut. Där finns också *Ta bort matchen*, och *Rensa historiken*
tömmer hela listan.

Listan finns bara i startvyn, alltså **inte** åtkomlig mitt i en pågående
match – du kommer dit via *Spelare* och tillbaka med *Tillbaka till matchen*.
En match som avbryts (ny match startas innan sista rundan) sparas också, men
märks som avbruten. De 25 senaste matcherna behålls.

Övrigt: **Ångra** ångrar senaste inmatningen och **Ny match** nollställer
poängen men behåller spelarna. **Spelare** går tillbaka till startvyn – där
tar *Tillbaka till matchen* dig in i den pågående matchen igen med poängen
kvar (bra om ett namn är felstavat), medan *Starta om matchen* nollställer.
Allt sparas automatiskt i webbläsaren, så matchen ligger kvar om telefonen
låser sig eller fliken stängs. Är lagring blockerad (privat läge) hålls
matchen i minnet istället för att appen ska sluta fungera.

En liten kontroll finns också: om de klarade buden i en runda tillsammans är
fler än antalet kort visas en varning – då har något matats in fel. Plumpade
bud räknas inte, eftersom de sticken inte skrivs upp.

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
