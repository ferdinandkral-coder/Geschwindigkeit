# SpeedCam v2 – Geschwindigkeit & Höhe per Handykamera

Web-App (eine einzige `index.html`), die mit der Rückkamera eines Smartphones (z. B. Galaxy S24) ein
fahrendes oder fliegendes Objekt erkennt, verfolgt und daraus Entfernung, Höhe und Geschwindigkeit schätzt.

## Neu in v2

- **Besseres KI-Modell:** EfficientDet-Lite2 (MediaPipe, GPU) statt COCO-SSD-Lite. Alternativ Lite0 (schneller).
- **Ausschnitt-Erkennung:** Sobald ein Ziel gewählt ist, wird nur ein Ausschnitt um das Ziel aus dem
  *vollen* Kamerabild (Full-HD/4K) herausgeschnitten und der KI gegeben. Kleine, weit entfernte Objekte
  werden so deutlich besser erkannt und verfolgt, weil das Modell nicht das ganze Bild auf 320–448 px verkleinern muss.
- **Kacheln-Suche:** Optional wird das Bild vor der Zielwahl in Kacheln gescannt (findet kleinere Objekte, langsamer).
- **Manuelles Ziel + Bild-Tracker:** Auf eine freie Stelle tippen (z. B. Drohne, die die KI nicht kennt) →
  Rahmen wird gesetzt und per Bildvergleich verfolgt. Erkennt die KI das Objekt später, übernimmt sie automatisch.
  Auch wenn die KI ein bereits gewähltes Ziel kurz verliert, springt der Bild-Tracker ein.
- **Echter Kamera-Zoom:** Zoom-Schieberegler und Zwei-Finger-Geste (Hardware-Zoom, falls Chrome ihn anbietet).
  Die Brennweite wird mit dem Zoom mitgerechnet, die Entfernungs-/Geschwindigkeitsrechnung bleibt korrekt.
- **Objektiv-Wahl:** Auswahl der Rückkameras (Ultraweit/Haupt/Tele) und der Auflösung (HD/Full-HD/4K).
  Der Bildwinkel wird pro Objektiv gespeichert.

## Neigungssensor (Update)

- **Sensorquelle:** Standard ist jetzt *Gyro + Beschleunigung*. Die frühere Kompass-Quelle lässt sich in den
  Einstellungen wieder wählen, wird aber durch Magnetfelder (Auto, Metall, Magnet-Hülle) oft verfälscht.
  Eine Himmelsrichtung braucht die Messung nicht.
- **Anzeige „Kamera-Neigung“** ist leicht geglättet; die Rechnung nutzt weiter die Rohwerte.
  Bleibt der Sensor stehen, steht dort „Sensor steht“.
- **Diagnose:** Die Infozeile oben zeigt Quelle und Rohwinkel β/γ.
- **Horizont = 0° setzen:** Kamera exakt waagerecht ausrichten (Stativ/Wasserwaage, in der späteren Messhaltung),
  tippen – ein konstanter Neigungsfehler wird danach in Anzeige und Rechnung herausgenommen („Korrektur löschen“ setzt zurück).

## Veröffentlichen mit GitHub Pages

1. Neues GitHub-Repository anlegen (z. B. `speedcam`).
2. `index.html` und diese `README.md` hochladen (Add file → Upload files). Alte Version ersetzen.
3. Repository → **Settings → Pages** → Source: *Deploy from a branch* → Branch `main`, Ordner `/ (root)` → Save.
4. Nach ca. 1 Minute: `https://<dein-name>.github.io/speedcam/` im **Chrome** auf dem Handy öffnen.
5. Start → Kamera- und Bewegungssensor-Zugriff erlauben. Beim ersten Start lädt die App das KI-Modell (~10 MB).

Nach dem Update ggf. die Seite im Chrome neu laden (Seite ohne Cache laden / Website-Daten löschen),
sonst läuft noch die alte Version. Einstellungen der v1 werden nicht übernommen.

## Bedienung

1. **Einstellungen:** Messmodus, Objektvorlage, Kamerahöhe.
2. **Objektiv wählen:** Für Zoom das 3×-Tele-Objektiv probieren. Danach unter „Bildwinkel bei 1× Zoom“ den
   passenden Richtwert (Ultraweit ≈106°, Haupt ≈70°, Tele ≈29°) setzen oder – besser – **kalibrieren**:
   Objekt bekannter Größe in bekannter Entfernung antippen, Abstand eingeben → „Bildwinkel berechnen“.
   Der aktuelle Zoom wird dabei herausgerechnet.
3. **Zoomen** vor der Messung (Regler oder Pinch), dann Ziel antippen. Während der Messung nicht mehr zoomen.
4. Ziel antippen und im Bild halten. Durchgezogener Rahmen = KI, gestrichelt = Bild-Tracker.
5. „Kamera-Neigung“ prüfen: Handy gerade nach vorn → ca. 0°, nach oben kippen → positiv.

## Wie es funktioniert

1. **KI-Erkennung** liefert Rahmen; ein Tipp wählt das Ziel, danach wird es von Bild zu Bild verfolgt.
2. **Entfernung (Lochkameramodell):** Größen-Modus `Z = f_px × reale Größe ÷ Pixelgröße` mit `f_px` aus Bildwinkel × Zoom;
   Boden-Modus: Kamerahöhe ÷ tan(Blickwinkel nach unten zur Unterkante).
3. **Richtung/Höhe:** Der Lagesensor liefert die Drehung des Handys → Position (Ost/Nord/oben), Höhe = z + Kamerahöhe.
4. **Geschwindigkeit:** Lineare Regression der Positionen über ein einstellbares Zeitfenster.

## Genauigkeit & Grenzen (ehrlich)

- Entfernung nur so gut wie die angenommene Objektgröße; realistisch ±10–30 %.
- Schräge Sicht auf das Objekt → erscheint kleiner → Entfernung zu groß.
- Beim **manuellen Ziel** bestimmt dein Rahmen die Pixelgröße: Der Rahmen muss das Objekt eng umschließen,
  sonst stimmt die Entfernung nicht. Der Bild-Tracker hält die Rahmengröße fest (kein Skalentracking), deshalb
  ist die Geschwindigkeit in Blickrichtung dann unzuverlässig; quer zur Blickrichtung funktioniert sie gut.
- Die KI kennt keine „Drohne“ (nur COCO-Klassen wie Auto, Flugzeug, Vogel, Drachen …).
- **Zoom:** Ob Chrome Hardware-Zoom und die einzelnen Objektive des S24 anbietet, hängt von Gerät/Chrome-Version ab.
  Fehlt der Zoom-Regler, bietet dieses Gerät/Objektiv ihn im Browser nicht an – dann das Tele-Objektiv wählen.
  Reiner Digital-Zoom hilft nicht viel, die Ausschnitt-Erkennung nutzt aber ohnehin das volle Bild.
- Handy fest halten oder Stativ nutzen. Kein geeichtes Messgerät – nicht für Verkehrsüberwachung oder
  Sicherheitsentscheidungen verwenden.

## Technik

- Reines HTML/JS. Per CDN: `@mediapipe/tasks-vision` (ObjectDetector), Modelle von `storage.googleapis.com/mediapipe-models`.
- Einstellungen im `localStorage` (`speedcam-settings-v2`).
