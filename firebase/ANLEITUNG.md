# VORTEXIA mit Firebase – Anleitung

Hier steht Schritt für Schritt, wie du die Sicherheitsregeln einfügst, dich zum Admin machst
und das Spiel online stellst. Du brauchst dafür nur den Browser (Firebase-Konsole).
Nur Schritt 6 (Hochladen per Terminal) braucht deinen PC.

**Die Dateien in diesem Ordner:**

| Datei | Wofür? |
|---|---|
| `firestore.rules` | Regeln für **Firestore** (Bug-Meldungen, Telemetrie, Admin-Ausweis) |
| `database.rules.json` | Regeln für die **Realtime Database** (Online-Modus: Spieler, Schüsse, Wetter) |
| `firebase.json` | Nur für das Hochladen mit dem Terminal (`firebase deploy`) |

**Was die Regeln machen, kurz erklärt:**

- Spieler (automatisch anonym angemeldet) dürfen **nur** das, was das Spiel wirklich braucht:
  Bugs melden, ihr eigenes Lebenszeichen schreiben, ihre eigene Position senden, schießen.
- Jedes Feld wird geprüft (Typ, Länge). So kann niemand HTML-Code oder Müll in deine Admin-Zentrale schmuggeln.
- Bugs lesen/löschen, Telemetrie sehen, Wetter/Durchsagen für alle ändern: **nur du** (Admin).
- Wer Admin ist, steht **nur auf dem Server** (in der Konsole). Im Spielcode steht keine E-Mail und keine UID.
- Alles, was nicht ausdrücklich erlaubt ist, ist verboten.

---

## 1. Projekt vorbereiten (einmalig)

1. Öffne <https://console.firebase.google.com> und wähle dein Projekt (oder lege eins an).
2. **Authentication** → **Anmeldemethode** (Sign-in method):
   - **Anonym** aktivieren (sonst funktioniert für normale Spieler gar nichts).
   - **Google** aktivieren (das brauchst du als Admin).
3. **Authentication** → **Einstellungen** → **Autorisierte Domains**:
   Wenn das Spiel auf deinem eigenen Server läuft, trage hier deine Domain ein (z. B. `meinspiel.de`).
   `…web.app` und `…firebaseapp.com` stehen schon drin.
4. **Firestore Database** → **Datenbank erstellen** → **Produktionsmodus** → Standort in Europa (z. B. `eur3` oder `europe-west3`).
5. **Realtime Database** → **Datenbank erstellen** → Standort z. B. Belgien (`europe-west1`) → **Im gesperrten Modus starten**.
6. **Projekteinstellungen** (Zahnrad oben links) → **Allgemein** → unten **Deine Apps** → Web-App (`</>`) anlegen
   und die Konfiguration kopieren.

### Konfiguration ins Spiel eintragen

In `index.html` (und `Vortexia_ULTRA.html`) gibt es diese Zeile:

```js
const FIREBASE_CONFIG={apiKey:"",authDomain:"",databaseURL:"",projectId:"",storageBucket:"",messagingSenderId:"",appId:""};
```

Fülle sie mit deinen Werten aus Schritt 6. **Wichtig:** `databaseURL` muss mit drin sein, sonst gibt es keinen Online-Modus.
Du findest sie oben auf der Seite **Realtime Database → Daten**, z. B.
`https://dein-projekt-default-rtdb.europe-west1.firebasedatabase.app`.

> Der `apiKey` ist kein Passwort. Er darf im Spiel stehen. Geschützt wird alles durch die Regeln unten.

---

## 2. Firestore-Regeln einfügen

1. Firebase-Konsole → **Firestore Database** → Reiter **Regeln** (Rules).
2. Alles, was im Editor steht, markieren und **löschen**.
3. Den kompletten Inhalt von `firestore.rules` hineinkopieren.
4. Oben auf **Veröffentlichen** (Publish) klicken.

## 3. Realtime-Database-Regeln einfügen

1. Firebase-Konsole → **Realtime Database** → Reiter **Regeln** (Rules).
2. Alles im Editor **löschen**.
3. Den kompletten Inhalt von `database.rules.json` hineinkopieren (die Zeilen mit `//` sind Kommentare, die dürfen drin bleiben).
4. **Veröffentlichen** klicken.

Zeigt die Konsole einen roten Fehler, fehlt meistens ein Stück beim Kopieren. Dann einfach nochmal alles löschen und neu einfügen.

---

## 4. Dich selbst zum Admin machen

Du musst dich an **zwei** Stellen eintragen: einmal in Firestore (damit das Spiel dir die Admin-Zentrale zeigt)
und einmal in der Realtime Database (damit du Wetter, Durchsagen usw. für **alle** Spieler ändern darfst).

### a) Deine UID herausfinden

1. Öffne das Spiel und tippe im Menü auf **Mit Google anmelden**. Melde dich mit deinem Google-Konto an.
2. Firebase-Konsole → **Authentication** → **Nutzer** (Users).
3. Suche die Zeile mit **deiner Gmail-Adresse**. Nimm **nicht** die Zeilen, bei denen „Anonym“ steht!
4. Ganz rechts steht die **Nutzer-UID** (eine lange Zeichenkette, z. B. `aB3dE…`). Kopieren.

### b) In Firestore eintragen

1. **Firestore Database** → **Daten** → **+ Sammlung starten** (Start collection).
2. Sammlungs-ID: `admins` → **Weiter**.
3. Dokument-ID: **deine UID** einfügen (genau so, ohne Leerzeichen).
4. Ein Feld anlegen, z. B. Feldname `name`, Typ `string`, Wert `ich`. (Der Inhalt ist egal, das Dokument muss nur existieren.)
5. **Speichern**.

### c) In der Realtime Database eintragen

1. **Realtime Database** → **Daten**.
2. Fahre mit der Maus über die oberste Zeile (deine Datenbank-Adresse) und klicke auf **+**.
3. Name: `admins`. Den Wert leer lassen und im gleichen Kästchen nochmal auf das kleine **+** klicken.
4. In der neuen Zeile: Name = **deine UID**, Wert = `true`.
5. **Hinzufügen** klicken.

Danach sieht es so aus:

```
admins
  └─ aB3dE…(deine UID): true
```

### d) Testen

Spiel neu laden → mit Google angemeldet bleiben → im Menü erscheint der **Admin**-Knopf.
Die Admin-Zentrale zeigt Bug-Tracker und Telemetrie. Wenn du im Online-Modus das Wetter änderst, sehen es alle.

Weitere Admins: einfach Schritt a–c mit der UID der anderen Person wiederholen.
Admin wegnehmen: das Dokument in `admins` (Firestore) **und** den Eintrag unter `admins` (Realtime Database) löschen.

### Für Fortgeschrittene: Custom Claim statt Einträgen

Die Regeln akzeptieren auch einen „Custom Claim“ `admin: true`. Der gilt für Firestore **und** Realtime Database gleichzeitig.
Den kann man aber nur mit einem kleinen Skript setzen:

1. Projekteinstellungen → **Dienstkonten** → **Neuen privaten Schlüssel generieren** → Datei als `key.json` speichern.
   **Diese Datei ist geheim!** Niemals hochladen, niemandem schicken, nach dem Benutzen löschen.
2. Auf dem PC (Node.js muss installiert sein) im gleichen Ordner:

   ```
   npm install firebase-admin
   node -e "const a=require('firebase-admin');a.initializeApp({credential:a.credential.cert(require('./key.json'))});a.auth().setCustomUserClaims('DEINE-UID',{admin:true}).then(()=>console.log('fertig'))"
   ```

3. Im Spiel **abmelden und neu mit Google anmelden** (erst dann kennt dein Login den Claim).
4. `key.json` löschen.

---

## 5. Prüfen, ob alles klappt

- Spiel öffnen → **Bug melden** → abschicken. In der Konsole unter **Firestore → Daten → bugs** taucht die Meldung auf.
- Spiel in zwei Browsern (oder Handy + PC) öffnen → bei beiden **Online** drücken → ihr seht euch gegenseitig.
- Mit einem normalen (nicht Admin-)Konto darf der Admin-Knopf **nicht** funktionieren.
- In der Konsole gibt es bei beiden Regeln einen **Regel-Playground** (Rules Playground), um Regeln auszuprobieren.

---

## 6. Hochladen auf Firebase Hosting (optional)

Brauchst du nur, wenn das Spiel **auf Firebase** laufen soll. Läuft es auf deinem eigenen Server,
reichen die Schritte 1–5 (Regeln in der Konsole einfügen) und du kannst diesen Teil überspringen.

1. Node.js installieren (<https://nodejs.org>), dann im Terminal:
   ```
   npm install -g firebase-tools
   firebase login
   ```
2. Einen neuen Ordner anlegen, z. B. `vortexia-online`, und hineinlegen:
   - `index.html` (die komplette HTML, z. B. über die Kopier-Seite → **Herunterladen**)
   - `Vortexia_ULTRA.html` (wenn du die Ultra-Version auch willst)
   - `firebase.json`, `firestore.rules`, `database.rules.json` aus diesem Ordner
3. Im Terminal in diesen Ordner wechseln und hochladen:
   ```
   cd vortexia-online
   firebase deploy --project DEINE-PROJEKT-ID
   ```
   Das lädt die Webseite **und** beide Regeln hoch. Nur die Regeln hochladen:
   ```
   firebase deploy --only firestore:rules,database --project DEINE-PROJEKT-ID
   ```
4. Dein Spiel ist dann unter `https://DEINE-PROJEKT-ID.web.app` erreichbar
   (die Ultra-Version unter `…/Vortexia_ULTRA.html`).

`firebase.json` sorgt dafür, dass die HTML-Dateien nicht lange im Browser zwischengespeichert werden.
Neue Versionen kommen also sofort bei den Spielern an. Die Regel-Dateien selbst werden nicht als Webseite veröffentlicht.

---

## 7. Gut zu wissen

- **Telemetrie räumt sich nicht selbst auf.** Jeder Spielstart legt ein kleines Dokument in `sessions` an.
  Tipp: Firestore → **TTL-Richtlinien** (Time-to-live) → Sammlungsgruppe `sessions`, Feld `ts`.
  Dann löscht Firebase alte Einträge automatisch (meist innerhalb von 24 Stunden).
- **Online-Modus kostet Datenvolumen.** Jeder Spieler sendet 10-mal pro Sekunde seine Position an alle anderen.
  Im kostenlosen Spark-Tarif hat die Realtime Database 10 GB Download pro Monat und höchstens 100 gleichzeitige Verbindungen.
  Bei 10 Spielern gleichzeitig ist das Volumen nach grob 12–15 Stunden gemeinsamer Spielzeit aufgebraucht.
  Schau ab und zu unter **Realtime Database → Nutzung** nach.
- **Kick und Bann gibt es (noch) nicht.** Die Regeln verhindern Betrug wie falsche Durchsagen,
  fremde Spieler bewegen oder riesigen Schaden. Rauswerfen kann man aber niemanden.
- **KI-Steuerung:** Die Cloud Function `vxAI` ist nicht in diesem Projekt. Ohne sie nutzt die Admin-KI den eingebauten
  deutschen Befehls-Parser. Falls du sie später baust: Die Function muss **selbst** prüfen, ob der Aufrufer Admin ist.
- **Niemals** die Regeln auf `allow read, write: if true;` oder `if request.auth != null;` stellen.
  Dann wäre jeder Google-Nutzer Admin und könnte deine Bug-Meldungen löschen.
