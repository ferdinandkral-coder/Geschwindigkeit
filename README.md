# SpeedCam – Geschwindigkeit & Höhe per Handykamera

Web-App (eine einzige `index.html`), die mit der Rückkamera eines Smartphones (z. B. Galaxy S24) ein
fahrendes oder fliegendes Objekt erkennt, verfolgt und daraus Entfernung, Höhe und Geschwindigkeit schätzt.

## Veröffentlichen mit GitHub Pages

1. Neues GitHub-Repository anlegen (z. B. `speedcam`).
2. `index.html` und diese `README.md` ins Repository hochladen (Add file → Upload files).
3. Repository → **Settings → Pages** → Source: *Deploy from a branch* → Branch `main`, Ordner `/ (root)` → Save.
4. Nach ca. 1 Minute ist die App unter `https://<dein-name>.github.io/speedcam/` erreichbar.
5. Auf dem S24 im **Chrome** öffnen → Start → Kamera- und Bewegungssensor-Zugriff erlauben.
   (Optional: Chrome-Menü → „Zum Startbildschirm hinzufügen“.)

GitHub Pages liefert HTTPS – das ist für Kamera und Sensoren zwingend nötig.

## So funktioniert es

1. **Objekterkennung (KI):** TensorFlow.js + COCO-SSD erkennt Objekte (Auto, LKW, Bus, Fahrrad, Person,
   Flugzeug, Vogel, Boot, Zug, Drachen …) und liefert Rahmen (Bounding Boxes). Ein Tipp wählt das Ziel,
   danach wird es von Bild zu Bild verfolgt.
2. **Entfernung (Trigonometrie / Lochkameramodell):**
   - *Größen-Modus:* `Z = f_px × reale Größe ÷ Pixelgröße` (f_px aus dem Bildwinkel).
   - *Boden-Modus:* Strahl zur Unterkante des Rahmens schneidet die Bodenebene; `Entfernung = Kamerahöhe ÷ tan(Winkel nach unten)`.
3. **Richtung/Höhe:** Der Gyro-/Lagesensor liefert die Drehung des Handys. Der Strahl zum Objekt wird damit
   in Welt-Koordinaten (Ost/Nord/oben) umgerechnet → Position `P = (x, y, z)`; **Höhe = z + Kamerahöhe**.
4. **Geschwindigkeit:** Lineare Regression der Positionen über ein einstellbares Zeitfenster → `v = (vx, vy, vz)`.
   Da die Handy-Drehung in die Position eingeht, funktioniert es auch, wenn du dem Objekt mit dem Handy folgst.

## Bedienung

1. **Einstellungen:** Modus wählen, passende Objektvorlage (oder eigene Größe), Kamerahöhe eintragen.
2. **Kalibrieren (empfohlen):** Objekt bekannter Größe in bekannter Entfernung (z. B. Auto in 20 m) antippen,
   Abstand eingeben → „Bildwinkel berechnen“. Der Wert wird gespeichert. Voreinstellung 70° ist nur ein Schätzwert.
3. Ziel antippen und im Bild halten. Oben stehen Geschwindigkeit, Entfernung und Höhe.
4. Prüfe die „Kamera-Neigung“: Handy gerade nach vorn halten → ca. 0°, nach oben kippen → positiv.

## Genauigkeit & Grenzen (ehrlich)

- Mit einer Kamera allein ist die Entfernung nur so gut wie die angenommene Objektgröße. Realistisch: ±10–30 %.
- Objekt schräg zur Blickrichtung → erscheint kleiner → Entfernung zu groß geschätzt. Am besten Flugzeug von
  unten/seitlich mit Spannweite, Auto von hinten/vorn mit Breite.
- Geschwindigkeit quer zur Blickrichtung ist deutlich genauer als Geschwindigkeit in Blickrichtung.
- COCO-SSD kennt keine „Drohne“; sie wird manchmal als Vogel/Drachen/Flugzeug erkannt, kleine ferne Objekte oft gar nicht.
- Handy fest halten oder Stativ nutzen; Magnetfeld-Störungen (Metall, Auto) verfälschen den Kompass, die
  Geschwindigkeit hängt aber nur von der *relativen* Drehung ab.
- Der Boden-Modus braucht ebenen Boden; ab ca. 100 m Entfernung wird er sehr empfindlich gegen Neigungsfehler.
- Kein geeichtes Messgerät – nicht für Verkehrsüberwachung, Rechtsfragen oder Sicherheitsentscheidungen verwenden.

## Technik

- Reines HTML/JS, keine Build-Schritte. Bibliotheken per CDN: `@tensorflow/tfjs`, `@tensorflow-models/coco-ssd`.
- Einstellungen werden im `localStorage` gespeichert.
