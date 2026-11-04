---
tags: [gymclash, maui, sprint-review, bugfix, mvvm]
date: 2026-10-19
---
## Sprint1
## User Story: ON-6 - RPG Profil & Level-System

## 🎯 Ziel
Entwicklung der plattformübergreifenden Charakter-Profilseite für GymClash mit funktionierendem RPG-Level-System für Windows, Android und iOS.

## 🛠️ Umsetzung (MVVM)
* **View (`CharacterPage.xaml`):** 
  * UI-Aufbau in drei Sektionen: Header (Name, Level, Rang, Workout-Button), Mitte (XP-ProgressBar), Unten (Attribute: STR, STA, AGI).
* **ViewModel (`CharacterViewModel.cs`):**
  * Implementierung von `INotifyPropertyChanged` (inkl. Lambda-Operatoren `=>` für kompakten Code) für sofortige UI-Updates.
  * **Progression:** Dynamische XP-Skalierung (`Math.Max(0.02, 0.35 - (Level * 0.01))`).
  * **Level-Up Logik:** Bei 100 % XP steigt das Level, XP resetten auf 0, und die Attribute (STR +2, STA +1, AGI +1) steigen an.
  * **Rang-System:** Automatische Einstufung von "IronBro" bis "DiamondBro" alle 10 Level.
  * **Pop-ups:** Asynchrone In-App-Benachrichtigung (`await Shell.Current.DisplayAlertAsync`) bei einem Rangaufstieg.

## 🐛 Bugfixes & Learnings
* **Visual Studio Deployment (Fehler XA0129):**
  * *Problem:* Hot Reload hat Änderungen verschluckt und alte App-Versionen geladen, der Emulator-Cache war fehlerhaft.
  * *Lösung:* Hard Reset durchgeführt (manuelles Löschen der `bin`- und `obj`-Ordner im Datei-Explorer, Projektmappe bereinigt) und das Test-Environment dauerhaft auf einen stabilen Android 14/15 Emulator (API 34/35) gewechselt.




## Sprint 2
# Workout-Seite (TrainingPage)

## Ziel (User Story)
Als Nutzer möchte ich eine übersichtliche Trainings-Ansicht aufrufen können, auf der ich mein Workout starten und beenden kann, damit das System meine Trainingszeit und das Anti-Cheat-System aktivieren kann.

## Betroffene Dateien
| Datei | Zweck |
|---|---|
| `Views/TrainingPage.xaml` | Oberfläche (UI) der Workout-Seite |
| `Views/TrainingPage.xaml.cs` | Logik (Code-Behind) |
| `AppShell.xaml` | Tab „Workout“ in der Hauptnavigation (war bereits vorhanden) |

## Umgesetzte Akzeptanzkriterien

| Kriterium | Umsetzung |
|---|---|
| Tab in der Hauptnavigation | Tab „Workout“ in der `TabBar` der `AppShell` verweist auf `TrainingPage` |
| Auffälliger Start-Button | Großer grüner Button „▶ Workout starten“ (Höhe 64). Beim Start wird er zum roten Button „■ Workout beenden“ |
| Timer / Stoppuhr | Großes Label im Format `hh:mm:ss`. Ein `IDispatcherTimer` aktualisiert es jede Sekunde |
| Übungen eingeben/auswählen | `Picker` mit 12 Standardübungen und ein `Entry` für eigene Übungen. Jede Übung lässt sich wieder entfernen |
| Workout-Templates | Aktuelle Übungsliste kann mit Namen als Vorlage gespeichert, geladen und gelöscht werden |
| Videos zu Übungen | Button 🎥 neben jeder Übung öffnet über den `Launcher` eine YouTube-Suche zur Übung |
| Anti-Cheat-Platzhalter | Drei Statusanzeigen: 📶 WLAN, 📍 GPS, ✅ Legitimation. Sie zeigen „Inaktiv“ und wechseln während des Workouts auf „Prüfung läuft“ |

## Technische Umsetzung

### Oberfläche (XAML)
- Aufbau aus `Border`-Karten in einem `ScrollView`, im gleichen Stil wie die anderen Seiten (Farben über `AppThemeBinding`, unterstützt Light/Dark-Mode).
- Benannte Elemente (`x:Name`) wie `TimerLabel`, `StartStopButton`, `ExerciseList`, `TemplateList`. Der Code-Behind greift darüber auf sie zu.

### Logik (C#)
- **Workout starten/beenden:** `OnStartStopClicked` schaltet einen Zustand (`_running`) um. Beim Start werden Startzeit gespeichert, Timer gestartet und Anti-Cheat-Anzeige aktiviert. Beim Beenden werden Timer gestoppt, Dauer berechnet und in einem Dialog angezeigt.
- **Timer:** `Dispatcher.CreateTimer()` mit 1 Sekunde Intervall. Angezeigt wird `DateTime.Now - _startedAt`, daher bleibt die Zeit auch bei Verzögerungen korrekt.
- **Übungen:** Interne `List<string>`. `RefreshExercises()` baut die Anzeige nach jeder Änderung neu auf.
- **Templates:** Interne Liste aus Name und Übungen. Beim Speichern wird eine Kopie der Übungsliste abgelegt, damit spätere Änderungen die Vorlage nicht verändern.
- **Videos:** Der Suchbegriff wird mit `Uri.EscapeDataString` kodiert und im Browser geöffnet.
- **Anti-Cheat:** `SetAntiCheat(bool)` ändert Text und Farbe der drei Status-Labels.

## Aktuelle Einschränkungen
- Übungen und Vorlagen liegen nur im Arbeitsspeicher und gehen beim App-Neustart verloren.
- Die Anti-Cheat-Anzeige ist nur ein visueller Platzhalter. Die echten Prüfungen (WLAN, GPS, Legitimation) sind nicht implementiert.
- Videos sind eine YouTube-Suche, keine eigenen eingebetteten Videos.

## Mögliche nächste Schritte
- Übungen, Vorlagen und abgeschlossene Workouts in der Datenbank speichern (`DatabaseConfig`).
- Echte Anti-Cheat-Prüfungen anbinden (WLAN-Name, GPS-Standort, Legitimation).
- Workout-Ende mit XP im RPG-System verbinden (`CharacterViewModel.Instance`).





# App Branding (Icon und Splash Screen)

## Ziel (Task)
Als Entwickler möchte ich ein Platzhalter-Logo (oder das finale GymClash-Logo) als App-Icon für den Homescreen und als Splash Screen beim Laden der App hinterlegen.

**Akzeptanzkriterien**
- SVG-Dateien für App-Icon und Splash Screen sind im `Resources`-Ordner des MAUI-Projekts konfiguriert.
- Die App startet auf dem Simulator (z. B. Pixel 7) mit dem GymClash-Logo.

## Ausgangslage
Das MAUI-Projekt enthält durch die Projektvorlage schon die Ordner und die Konfiguration für Icon und Splash, aber mit dem Standard-Logo (dotnet bot). Es musste also kein neuer Ordner angelegt werden, nur die Inhalte wurden ausgetauscht.

## Betroffene Dateien
| Datei | Änderung |
|---|---|
| `Resources/AppIcon/appicon.svg` | Enthält jetzt das GymClash-Logo |
| `Resources/Splash/splash.svg` | Enthält jetzt das GymClash-Logo |
| `Resources/AppIcon/appiconfg.svg` | Wird nicht mehr verwendet (früherer Vordergrund) |
| `gymclash.csproj` | Icon und Splash zeigen auf die neuen SVGs, Hintergrundfarbe angepasst |

## Das Logo
- Motiv: goldenes Schwert, gekreuzt mit einer silbernen Hantel (RPG und Fitness).
- Farben: Lila und Gold, passend zur App (Hintergrund `#3813BB`).
- Das Bild wurde mit einem KI-Bildgenerator erstellt (1024 × 1024 px, quadratisch, ohne Text).
- Für die Abnahme (SVG gefordert) wurde das Bild auf 512 × 512 px verkleinert und als PNG-Daten in eine SVG-Datei eingebettet.

## Konfiguration in `gymclash.csproj`
```xml
<!-- App Icon -->
<MauiIcon Include="Resources\AppIcon\appicon.svg" Color="#3813BB" />

<!-- Splash Screen -->
<MauiSplashScreen Include="Resources\Splash\splash.svg" Color="#3813BB" BaseSize="288,288" />
```
- `MauiIcon` erzeugt beim Build alle Icon-Größen für Android, iOS und Windows.
- `MauiSplashScreen` erzeugt den Splash. `Color` ist die Hintergrundfarbe, `BaseSize` die Größe des Logos.
- `Color` entspricht der Hintergrundfarbe des Bildes, damit am Rand keine Kanten sichtbar sind.

## Testen
1. Ziel „Android Emulator – Pixel 7“ wählen.
2. *Projektmappe bereinigen*, dann *Neu erstellen* (sonst bleibt oft das alte Logo).
3. Alte App vom Emulator deinstallieren (der Launcher cached das Icon).
4. App starten: Der Splash mit Logo erscheint, danach die Login-Seite.

## Einschränkungen
- Das SVG ist eine Hülle um ein Rasterbild und kein echtes Vektorlogo. Bei großen Darstellungen kann es unscharf wirken.
- Die Login-Seite (`LoginPage.xaml`) und die Startseite (`HomePage.xaml`) zeigen weiterhin eine lila Kachel mit dem Text „GC“. Das ist nicht das App-Icon und gehört nicht zu dieser Aufgabe.

## Mögliche nächste Schritte
- Echtes Vektor-Logo erstellen (z. B. in Inkscape oder Figma) und die SVG-Dateien ersetzen.
- Das „GC“ auf Login- und Startseite durch das Logo ersetzen (`<Image Source="gymclash_logo.png" />`).
- Das Schwert-Hantel-Logo als finales Branding festlegen und weitere Varianten (z. B. mit Schriftzug) vorbereiten.




