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




