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
- **Spam-Bremse:** Niemand kann die Datenbank mit tausenden Einträgen fluten.
  Jeder Spieler darf nur so oft schreiben, wie das Spiel es wirklich tut (Details in Schritt 7).
- **Kein Fern-Abschuss mehr:** Treffen kann nur, wer selbst im Spiel ist, lebt und nah genug am Opfer steht.
  Der Schaden pro Schuss darf nicht höher sein als bei der echten Waffe.
- **Geister verschwinden:** Spieler, die seit 2 Minuten kein Lebenszeichen mehr senden, werden automatisch entfernt.
- Bugs lesen/löschen, Telemetrie sehen, Wetter/Durchsagen für alle ändern: **nur du** (Admin).
- Wer Admin ist, steht **nur auf dem Server** (in der Konsole). Im Spielcode steht keine E-Mail und keine UID.
- Alles, was nicht ausdrücklich erlaubt ist, ist verboten.

---

## 0. Wichtig: Spiel und Regeln gehören zusammen

Die Regeln in diesem Ordner passen **nur zur neuen Spielversion** (mit Spam-Bremse).
Mit einer alten Spielversion funktionieren Online-Modus, „Bug melden“ und Telemetrie nicht mehr – und umgekehrt.

Darum immer **beides gleichzeitig** erneuern:

1. Neue `index.html` hochladen.
2. Die Ultra-Version neu bauen (`python3 tools/build_ultra.py`) und `Vortexia_ULTRA.html` auch hochladen.
3. Direkt danach die Regeln veröffentlichen (Schritte 2 und 3).

Spieler, die das Spiel noch offen haben, laden einmal neu – dann klappt alles wieder.

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

> **Für dich (UID `7iZfMVbN87RJpgVqjXzuC5hGku82`) nicht nötig:** deine UID steht schon fest in `database.rules.json`. Es reicht, die Regeln einzufügen und zu veröffentlichen. Diesen Schritt brauchst du nur für **weitere** Admins.

1. **Realtime Database** → **Daten**.
2. Fahre mit der Maus über die oberste Zeile (deine Datenbank-Adresse) und klicke auf **+**.
3. Name: `admins`. Den Wert leer lassen und im gleichen Kästchen nochmal auf das kleine **+** klicken.
4. In der neuen Zeile: Name = **deine UID**, Wert = `true` (genau so, **ohne** Anführungszeichen).
5. **Hinzufügen** klicken.

Danach sieht es so aus:

```
admins
  └─ aB3dE…(deine UID): true
```

> Nur der Wert `true` zählt. Steht dort `false` oder `"true"` (mit Anführungszeichen, also Text), ist man **kein** Admin.

### d) Testen

Spiel neu laden → mit Google angemeldet bleiben → im Menü erscheint der **Admin**-Knopf.
Die Admin-Zentrale zeigt Bug-Tracker und Telemetrie. Wenn du im Online-Modus das Wetter änderst, sehen es alle.

Weitere Admins: einfach Schritt a–c mit der UID der anderen Person wiederholen.

**Admin wegnehmen:**
- Firestore: das Dokument in `admins` **löschen** (nur ein Feld ändern reicht nicht).
- Realtime Database: den Eintrag unter `admins` löschen oder auf `false` setzen.

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
- Aufeinander schießen: Treffer ziehen Leben ab, „… wurde ausgeschaltet“ erscheint beim anderen.
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
2. Einen neuen Ordner anlegen, z. B. `vortexia-online`, und so befüllen:
   ```
   vortexia-online/
   ├─ firebase.json          (aus diesem Ordner)
   ├─ firestore.rules        (aus diesem Ordner)
   ├─ database.rules.json    (aus diesem Ordner)
   └─ public/
      ├─ index.html          (die komplette HTML, z. B. über die Kopier-Seite → Herunterladen)
      └─ Vortexia_ULTRA.html (wenn du die Ultra-Version auch willst)
   ```
   Hast du dein Projekt schon mit `firebase init` angelegt, gibt es den Ordner `public` schon –
   dann einfach `public/index.html` ersetzen und `firebase.json` + die zwei Regel-Dateien daneben legen.
3. Im Terminal in diesen Ordner wechseln und hochladen:
   ```
   cd vortexia-online
   firebase deploy --project DEINE-PROJEKT-ID
   ```
   Das lädt die Webseite **und** beide Regeln gleichzeitig hoch (genau richtig, siehe Schritt 0). Nur die Regeln hochladen:
   ```
   firebase deploy --only firestore:rules,database --project DEINE-PROJEKT-ID
   ```
4. Dein Spiel ist dann unter `https://DEINE-PROJEKT-ID.web.app` erreichbar
   (die Ultra-Version unter `…/Vortexia_ULTRA.html`).

`firebase.json` sorgt dafür, dass die HTML-Dateien nicht lange im Browser zwischengespeichert werden.
Neue Versionen kommen also sofort bei den Spielern an. Hochgeladen wird nur, was im Ordner `public` liegt.

---

## 7. Spam-Bremse – was da passiert

Jeder Spieler darf nur so oft schreiben, wie das Spiel es wirklich tut. Das Spiel schreibt dafür bei jedem
Senden einen Zeitstempel mit (unter `rl` = „rate limit“). Ist der letzte zu frisch, lehnt der Server ab.

| Was | Erlaubt | Das Spiel macht |
|---|---|---|
| Position senden | höchstens alle 40 ms | alle 100 ms |
| Schuss | höchstens alle 30 ms | Maschinenpistole alle 75 ms |
| Treffer | Pistole 80 ms · MP 30 ms · Schrotflinte 200 ms · Scharfschützengewehr 200 ms | 170 · 75 · 750 · 1050 ms |
| Schaden pro Treffer | Pistole 12 · MP 8 · Schrotflinte 58 · Scharfschützengewehr 41 | genau so viel |
| Reichweite Treffer | Pistole 105 m · MP 85 m · Schrotflinte 50 m · Scharfschützengewehr 185 m | 90 · 70 · 34 · 170 m |
| „ausgeschaltet“ melden | höchstens alle 2 s | frühestens nach 3,6 s |
| Bug melden | 1 pro Minute | – |
| Telemetrie | 1 Dokument pro Spieler, höchstens alle 10 s | alle 60 s |

Gut zu wissen:

- Die Schrotflinte schickt jetzt **einen** Treffer mit dem Schaden aller Kugeln (statt 8 einzelne).
- Wer innerhalb einer Minute einen zweiten Bug meldet, bekommt „Gespeichert (lokal …)“ – die Meldung bleibt dann nur auf seinem Gerät.
- Die Zeitstempel liegen in der Realtime Database unter `rooms/main/rl` und in Firestore in der Sammlung `rl`.
  Beides ist winzig. Du kannst es in der Konsole **jederzeit löschen** – es geht nichts kaputt.
- **Geister** (Spieler ohne Lebenszeichen seit 2 Minuten) und alte Schüsse/Treffer (älter als 1 Minute)
  räumen die Spiele der anderen Spieler automatisch weg. Als Admin kannst du in der Konsole unter
  `rooms/main/p` jeden Spieler-Eintrag sofort löschen.
- Änderst du im Spiel den Schaden oder die Reichweite der Waffen (`WEAPONS`), musst du die Zahlen in
  `database.rules.json` (Abschnitte `to`, `d` und `rl`) anpassen. Schaden im Netz = `dmg` × 12.

---

## 8. Empfohlen: App Check (Schutz gegen Skripte)

Alle Bremsen gelten **pro Spieler-Konto**. Anonyme Konten kann aber jeder mit einem kleinen Skript neu anlegen
(Firebase begrenzt das nur pro Internetanschluss). Mit **App Check** dürfen nur noch echte Aufrufe von deiner
Spiel-Webseite die Datenbank benutzen. Das ist kostenlos, braucht aber eine kleine Änderung im Spiel:

1. <https://www.google.com/recaptcha/admin> → neue Website → **reCAPTCHA v3** → als Domains deine Spiel-Adressen eintragen
   (z. B. `DEINE-PROJEKT-ID.web.app`, `valoqisstudio.github.io`, deine eigene Domain).
   Du bekommst einen **Websiteschlüssel** und einen **geheimen Schlüssel**.
2. Firebase-Konsole → **App Check** → **Apps** → deine Web-App → **reCAPTCHA** → den **geheimen** Schlüssel eintragen → Speichern.
3. Im Spiel (`index.html`, danach Ultra neu bauen) **genau diese Stelle** suchen:
   ```js
   firebase.initializeApp(FIREBASE_CONFIG);
   ```
   und ersetzen durch (mit **deinem Websiteschlüssel**):
   ```js
   firebase.initializeApp(FIREBASE_CONFIG);await this.load(b+'firebase-app-check-compat.js');firebase.appCheck().activate('DEIN-WEBSITESCHLÜSSEL',true);
   ```
4. Firebase-Konsole → **App Check** → **APIs**: Ein paar Tage nur zuschauen (Messwerte).
   Wenn fast alle Anfragen als „überprüft“ angezeigt werden: bei **Realtime Database** und **Cloud Firestore** auf **Erzwingen** stellen.

> Nach dem Erzwingen klappen Online-Modus, „Bug melden“ und Telemetrie nur noch von den eingetragenen Webseiten aus.
> Wer das Spiel als Datei vom Handy öffnet, kann weiter alleine spielen – nur ohne Online-Funktionen.

---

## 9. Gut zu wissen

- **Telemetrie:** Jeder Spieler hat jetzt genau **ein** Dokument in `sessions` (Dokument-ID = seine UID).
  Alte Einträge mit kurzen Zufalls-IDs (aus der alten Spielversion) kannst du in der Konsole löschen.
  Tipp: Firestore → **TTL-Richtlinien** (Time-to-live) → Sammlungsgruppe `sessions`, Feld `ts`.
  Dann löscht Firebase alte Einträge automatisch (meist innerhalb von 24 Stunden).
- **Online-Modus kostet Datenvolumen.** Jeder Spieler sendet 10-mal pro Sekunde seine Position an alle anderen.
  Im kostenlosen Spark-Tarif hat die Realtime Database 10 GB Download pro Monat und höchstens 100 gleichzeitige Verbindungen.
  Bei 10 Spielern gleichzeitig ist das Volumen nach grob 12–15 Stunden gemeinsamer Spielzeit aufgebraucht.
  Schau ab und zu unter **Realtime Database → Nutzung** nach.
- **Was die Regeln NICHT verhindern können:** Jeder Spieler meldet seine Position selbst. Ein Betrüger kann sich also
  per Skript direkt neben dich „beamen“ und dann so schnell treffen, wie die Bremse erlaubt (höchstens ca. 290 Schaden
  pro Sekunde, ein normaler Spieler schafft ca. 90). Er muss dafür aber sichtbar im Spiel und in deiner Nähe sein.
  Ganz verhindern könnte das nur ein eigener Spiel-Server.
- **Kick und Bann gibt es (noch) nicht.** Du kannst als Admin aber den Eintrag eines Spielers unter `rooms/main/p`
  löschen (er taucht wieder auf, solange sein Spiel läuft).
- **KI-Steuerung:** Die Cloud Function `vxAI` ist nicht in diesem Projekt. Ohne sie nutzt die Admin-KI den eingebauten
  deutschen Befehls-Parser. Falls du sie später baust: Die Function muss **selbst** prüfen, ob der Aufrufer Admin ist
  (am besten auch App Check verlangen).
- **Niemals** die Regeln auf `allow read, write: if true;` oder `if request.auth != null;` stellen.
  Dann wäre jeder Google-Nutzer Admin und könnte deine Bug-Meldungen löschen.
