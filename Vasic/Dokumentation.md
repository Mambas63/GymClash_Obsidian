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



