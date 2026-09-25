# Spelarutveckling – installera som egen PWA på Firebase

Den här mappen innehåller en fristående version av appen, redo att driftsättas på ditt
eget Firebase-projekt (Realtime Database + Hosting). Filerna:

```
index.html          – hela appen (samma funktioner som i Claude-versionen)
manifest.json        – gör appen installerbar (namn, ikon, färger)
service-worker.js    – cachar appen så den fungerar offline och kan installeras
icon-192.png / icon-512.png – appikoner, genererade från klubbloggan
firebase.json         – pekar Firebase Hosting mot "public"-mappen
database.rules.json   – säkerhetsregler för Realtime Database
```

## 1. Skapa Firebase-projekt

1. Gå till https://console.firebase.google.com och klicka **Lägg till projekt**.
2. Ge det ett namn, t.ex. "lilla-torg-ff-app". Google Analytics behövs inte.

## 2. Registrera en webbapp och hämta din config

1. I projektöversikten, klicka på webb-ikonen (`</>`) för att lägga till en webbapp.
2. Ge den ett smeknamn, t.ex. "Spelarutveckling". Du behöver INTE kryssa i Firebase Hosting
   här (vi gör det via CLI:n nedan istället).
3. Firebase visar en kodsnutt med `firebaseConfig = { apiKey: ..., authDomain: ..., ... }`.
   Kopiera hela objektet.
4. Öppna `index.html` i den här mappen, sök upp `var firebaseConfig = {` (nära slutet av
   filen) och klistra in dina riktiga värden istället för platshållartexten
   (`DIN_API_KEY`, `DITT_PROJEKT`, osv).

## 3. Aktivera Realtime Database

1. I Firebase-konsolen: **Build > Realtime Database > Skapa databas**.
2. Välj en region (t.ex. Europe/Belgium eller Europe-west1 för lägst latens i Sverige).
3. Starta i **låst läge** (Locked mode) – det är säkrast som standard.
4. När databasen är skapad: gå till fliken **Regler**, klistra in innehållet från
   `database.rules.json` i den här mappen, och klicka **Publicera**.
5. Kontrollera att `databaseURL` i din `firebaseConfig` (steg 2) matchar den URL som visas
   överst på databas-sidan – uppdatera i `index.html` om den skiljer sig (särskilt om du
   valde en annan region än us-central1, då slutar URL:en ofta på
   `...-default-rtdb.europe-west1.firebasedatabase.app` istället för bara `firebaseio.com`).

**Viktigt om säkerhet:** reglerna i `database.rules.json` är medvetet enkla – vem som helst
med länken till appen kan läsa och skriva data. Det motsvarar ungefär hur artefakten
fungerade inne i Claude. Om du vill låsa det hårdare senare (t.ex. bara inloggade tränare
får skriva) kan du aktivera **Authentication** i Firebase och byta reglerna mot något i stil med:

```json
{
  "rules": {
    "players": {
      ".read": "auth != null",
      ".write": "auth != null"
    }
  }
}
```

Säg till om du vill att jag hjälper till att lägga in inloggning (t.ex. e-post/lösenord
eller Google-inloggning för tränarna) – det går bra att lägga till senare utan att
resten av appen behöver byggas om.

## 4. Installera Firebase CLI och logga in

I en terminal på din dator:

```bash
npm install -g firebase-tools
firebase login
```

(Kräver Node.js – ladda ner från https://nodejs.org om du inte har det.)

## 5. Sätt ihop mappstrukturen och initiera hosting

1. Skapa en mapp för projektet, t.ex. `lilla-torg-app/`, och lägg **alla filer från den här
   leveransen** i en undermapp som heter `public/`:

```
lilla-torg-app/
├── firebase.json
├── database.rules.json
└── public/
    ├── index.html
    ├── manifest.json
    ├── service-worker.js
    ├── icon-192.png
    └── icon-512.png
```

2. Öppna en terminal i `lilla-torg-app/`-mappen och kör:

```bash
firebase use --add
```

Välj det Firebase-projekt du skapade i steg 1.

Filen `firebase.json` som redan ligger där pekar hostingen mot `public`-mappen och
databasreglerna mot `database.rules.json`, så du behöver inte köra `firebase init` på nytt
(men om du hellre vill köra guiden själv: `firebase init hosting` + `firebase init database`,
välj samma projekt, svara "public" som hosting-mapp).

## 6. Deploya

```bash
firebase deploy
```

När det är klart visar terminalen en **Hosting URL**, vanligtvis
`https://DITT-PROJEKT.web.app`. Det är länken till din egen, installerbara app.

## 7. Installera appen på mobilen/datorn

- **Android/Chrome:** öppna länken → meny (⋮) → "Installera app" eller "Lägg till på
  startskärmen".
- **iPhone/Safari:** öppna länken → dela-ikonen → "Lägg till på hemskärmen".
- **Dator (Chrome/Edge):** öppna länken → ikon i adressfältet → "Installera".

Appen får då en egen ikon (klubbloggan), öppnas i eget fönster utan webbläsarfält, och
fungerar offline för själva gränssnittet (data kräver fortfarande internet mot Firebase).

## 8. Uppdatera appen senare

Ändra i `public/index.html` (eller be mig om nya funktioner och skicka in den uppdaterade
filen), kör sedan bara:

```bash
firebase deploy
```

igen. Eftersom `service-worker.js` cachar filerna kan det ibland ta en omstart av appen
(stäng och öppna igen) innan uppdateringen syns – det är normalt för PWA:er.

## Att tänka på

- **Kostnad:** Firebase Hosting och Realtime Database har generösa gratisnivåer (Spark-planen)
  som med god marginal räcker för ett lag på 51 spelare och ett fåtal tränare. Ni behöver
  inte lägga in betalkort för att komma igång.
- **Backup:** Realtime Database kan exporteras som JSON direkt i Firebase-konsolen
  (Realtime Database > ⋮ > Exportera JSON) – bra att göra då och då som säkerhetskopia,
  utöver appens egen "Exportera"-knapp per spelare.
- **jsPDF** (för PDF-export) laddas fortfarande från cdnjs.cloudflare.com, så internet
  krävs för PDF-export och Firebase-synk, men inte för att öppna själva appen.
