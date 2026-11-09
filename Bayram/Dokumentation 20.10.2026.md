User-Story ON-2 API-Endpunkt für den Login

**Umgesetzte Klassen für User Story ON-2:**

- **Backend:**
    
    - `AuthController`: Bereitstellung der Route `POST /api/auth/login`, Validierung der Anmeldedaten und Rückgabe des Statuscodes (`200 OK` vs. `401 Unauthorized`).
        
    - `LoginRequest` & `AuthResponse`: Datentransferobjekte zur Modellierung des JSON-Request- und Response-Bodys.
        
    - `TokenService`: Kapselung der JWT-Erstellung und -Signierung.
        
- **Frontend (MAUI):**
    
    - `AuthService`: Kapselung des HTTP-Aufrufs, Token-Speicherung und Verwaltung der Anmeldesitzung (`CurrentUserEmail`).
        
    - `LoginPage` / `LoginPageViewModel`: Benutzeroberfläche und Binding für die Authentifizierungseingaben.

### 1. `AuthController(IConfiguration configuration)` _(Konstruktor)_

![[Pasted image 20261104093106.png]]
- **Zweck:** Initialisiert den Controller und liest die PostgreSQL-/Supabase-Verbindungszeichenfolge (Connection String) aus der Konfiguration (`appsettings.json`) aus.
    
- **Parameter:**
    
    - `configuration` (`IConfiguration`): Zugriff auf Anwendungseinstellungen und Connection Strings.
        
- **Funktionsweise:**
    
    1. Sucht nach dem Schlüssel `"DefaultConnection"`.
        
    2. Falls dieser leer ist, wird alternativ nach `"Supabase"` gesucht.
        
    3. Speichert den ermittelten String im privaten Feld `_connectionString`.
        

### 2. `Login(LoginRequestDto request)`
![[Pasted image 20261104093921.png]]

- **Datentyp Rückgabewert:** `Task<IActionResult>`
    
- **Zweck:** Authentifiziert einen bestehenden Benutzer an der API über HTTP POST (`/api/auth/login`).
    
- **Parameter:**
    
    - `request` (`LoginRequestDto`): Das aus dem HTTP-Anfragekörper übertragene Datenobjekt, bestehend aus der E-Mail-Adresse und dem Klartextpasswort.
        
- **Rückgabewerte (HTTP-Statuscodes):**
    
    - `200 OK`: Bei erfolgreicher Authentifizierung.
        
    - `400 Bad Request`: Wenn Pflichtangaben fehlen (E-Mail oder Passwort leer bzw. ungültig).
        
    - `401 Unauthorized`: Wenn die E-Mail nicht existiert oder der errechnete Hashwert nicht mit dem Datenbank-Eintrag übereinstimmt.
        
    - `500 Internal Server Error`: Bei unerwarteten Datenbank- oder Verbindungsfehlern.
        
- **Ablauf im Detail:**
    
    1. Prüft, ob das Anfrageobjekt vorhanden ist und weder E-Mail noch Passwort leer sind.
        
    2. Stellt asynchron eine Verbindung zur Datenbank her.
        
    3. Sucht per parametrisierter SQL-Abfrage nach dem hinterlegten Passwort-Hash für die angegebene E-Mail.
        
    4. Berechnet den kryptografischen SHA-256-Hash des eingegebenen Passworts.
        
    5. Vergleicht den berechneten Hash mit dem in der Datenbank gespeicherten Wert und gibt das Ergebnis zurück.
        

### 3. `Register(RegisterRequestDto request)`
![[Pasted image 20261104094330.png]]
![[Pasted image 20261104094507.png]]
- **Datentyp Rückgabewert:** `Task<IActionResult>`
    
- **Zweck:** Registriert ein neues Benutzerkonto in der Datenbank über HTTP POST (`/api/auth/register`).
    
- **Parameter:**
    
    - `request` (`RegisterRequestDto`): Das aus dem HTTP-Anfragekörper übertragene Datenobjekt mit den Feldern E-Mail, Passwort und einem optionalen Benutzernamen.
        
- **Rückgabewerte (HTTP-Statuscodes):**
    
    - `200 OK`: Bei erfolgreicher Registrierung.
        
    - `400 Bad Request`: Wenn Pflichtfelder fehlen oder die E-Mail syntaktisch ungültig ist (fehlendes `@` oder `.`).
        
    - `409 Conflict`: Wenn bereits ein Benutzer mit dieser E-Mail-Adresse registriert ist.
        
    - `500 Internal Server Error`: Bei unerwarteten Fehlern während der Datenbanktransaktion.
        
- **Ablauf im Detail:**
    
    1. Validiert die Eingabedaten auf Vollständigkeit und grundlegende Formatregeln der E-Mail.
        
    2. Bestimmt den Benutzernamen (übernimmt die Eingabe, leitet ihn aus dem E-Mail-Präfix ab oder nutzt den Standardwert "User").
        
    3. Führt eine Datenbankabfrage durch, um sicherzustellen, dass die E-Mail noch nicht vergeben ist.
        
    4. Erzeugt mittels Einweg-Hashing (SHA-256) den Passwort-Hash.
        
    5. Fügt den neuen Datensatz mit E-Mail, Hash und Username per parametrisiertem INSERT-Befehl in die Tabelle ein.
### 4. `ComputeSha256Hash(string rawData)` _(Hilfsmethode)_
![[Pasted image 20261104094535.png]]

- **Sichtbarkeit:** `private static`
    
- **Zweck:** Einweg-Verschlüsselung (Hashing) eines Klartext-Strings zur sicheren Speicherung und Prüfung von Passwörtern.
    
- **Parameter:**
    
    - `rawData` (`string`): Das zu hashende Passwort im Klartext.
        
- **Rückgabewert:**
    
    - `string`: Der errechnete kryptografische Hashwert als 64-stellige Hexadezimal-Zeichenkette.
        
- **Ablauf im Detail:**
    
    1. Erzeugt eine `SHA256`-Instanz aus `System.Security.Cryptography`.
        
    2. Konvertiert den Eingabestring in ein UTF-8-Byte-Array.
        
    3. Berechnet das Byte-Array des Hashes.
        
    4. Konvertiert jedes Byte über das Format `"x2"` in Kleinbuchstaben-Hexwerte und fügt diese mittels `StringBuilder` zusammen.
        

### DTOs (Data Transfer Objects)

- **`LoginRequestDto`:** Kapselt die Parameter für die Login-Anfrage (`Email`, `Password`).
    
- **`RegisterRequestDto`:** Kapselt die Registrierungsdaten (`Email`, `Password`, `Username`).


Klasse: ==🔴**AuthService**==
### I. Variablen & Felder der Klasse `AuthService`

- **`AuthToken`**
    
    - **Sichtbarkeit:** `public static`
        
    - **Datentyp:** `string`
        
    - **Initialwert:** `string.Empty`
        
    - **Zweck:** Speichert das nach erfolgreichem Login empfangene Authentifizierungstoken (JWT) im flüchtigen Arbeitsspeicher des Clients, um nachfolgende autorisierte API-Anfragen zu authentifizieren.
        
- **`CurrentUserEmail`**
    
    - **Sichtbarkeit:** `public static`
        
    - **Datentyp:** `string`
        
    - **Initialwert:** `string.Empty`
        
    - **Zweck:** Hält die E-Mail-Adresse des aktuell angemeldeten Benutzers sitzungsweit vor. Sie dient als zentraler Identifikator für abhängige Dienste wie das Laden und Verarbeiten von Quests oder das Eintragen von Trainingsdaten.
        
- **`LastRegisteredEmail`**
    
    - **Sichtbarkeit:** `public static`
        
    - **Datentyp:** `string`
        
    - **Initialwert:** `string.Empty`
        
    - **Zweck:** Zwischenspeicher für die zuletzt erfolgreich registrierte oder angemeldete E-Mail-Adresse. Dient als Fallback, falls `CurrentUserEmail` kurzzeitig nicht gesetzt sein sollte.
        
- **`BaseApiUrl`**
    
    - **Sichtbarkeit:** `private static`
        
    - **Datentyp:** `string`
        
    - **Initialwert:** Ergebnis des Methodenaufrufs `GetBaseApiUrl()`
        
    - **Zweck:** Enthält die plattformabhängige Basis-URL der ASP.NET Core Web-API (z. B. Emulator-Loopback oder Localhost) als Präfix für sämtliche ausgehenden HTTP-Routen.
        
- **`httpClient`**
    
    - **Sichtbarkeit:** `private static`
        
    - **Datentyp:** `HttpClient`
        
    - **Initialwert:** `new HttpClient()`
        
    - **Zweck:** Wiederverwendbare Instanz des HTTP-Clients zur Durchführung asynchroner Netzwerkaufrufe (POST/GET) an das Backend, um Socket-Erschöpfung (Socket Exhaustion) zu verhindern.
        

### II. Methoden der Klasse `AuthService`

#### 1. `GetBaseApiUrl()`

- **Sichtbarkeit:** `private static`
    
- **Datentyp Rückgabewert:** `string`
    
- **Zweck:** Ermittelt dynamisch die netzwerktechnisch korrekte Basisadresse des Backend-Servers abhängig vom aktuellen Betriebssystem des Geräts.
    
- **Parameter:** Keine.
    
- **Rückgabewerte:**
    
    - `"[http://10.0.2.2:5052](http://10.0.2.2:5052)"`: Spezielle Alias-IP für das Android-Subsystem im Emulator, um den Host-Rechner über den konfigurierten Kestrel-Port zu erreichen.
        
    - `"http://localhost:5052"`: Für native Ausführungen (z. B. Windows Machine) oder Desktop-Builds.
        
- **Ablauf im Detail:**
    
    1. Prüft über `DeviceInfo.Platform`, ob die Anwendung unter Android läuft.
        
    2. Wählt anhand des Ergebnisses die passende Host-Adresse.
        
    3. Liefert den vollständigen Basis-URL-String inklusive Protokoll und Port zurück.
        

#### 2. `LoginAsync(string email, string password)`

- **Sichtbarkeit:** `public static async`
    
- **Datentyp Rückgabewert:** `Task<AuthResult>`
    
- **Zweck:** Sendet die Anmeldedaten des Nutzers asynchron an den Backend-Endpunkt `/api/auth/login`, verarbeitet die Antwort und initialisiert die Benutzersitzung.
    
- **Parameter:**
    
    - `email` (`string`): Die vom Benutzer eingegebene E-Mail-Adresse.
        
    - `password` (`string`): Das zugehörige Passwort im Klartext.
        
- **Rückgabewerte:**
    
    - `AuthResult(true, "Willkommen zurück, ...")`: Bei Statuscode `200 OK`.
        
    - `AuthResult(false, ...)`: Bei unvollständigen Eingaben, abgewiesener Authentifizierung (`401 Unauthorized`), Serverfehlern oder Verbindungsunterbrechungen.
        
- **Ablauf im Detail:**
    
    1. Validiert vorab, ob `email` oder `password` leer sind, und bricht ggf. vor dem Netzwerkaufruf ab.
        
    2. Bereinigt Whitespaces mit `.Trim()`.
        
    3. Baut den JSON-String manuell zusammen und kapselt ihn in ein `StringContent`-Objekt mit UTF-8-Codierung und dem Header `application/json`.
        
    4. Sendet den POST-Request asynchron an `BaseApiUrl + "/api/auth/login"`.
        
    5. Wertet den zurückgegebenen HTTP-Statuscode aus:
        
        - **`200 OK`:** Parst den Antwort-Body via `JsonDocument`. Liest die Eigenschaft `"token"` aus und weist sie `AuthToken` zu. Weist `CurrentUserEmail` und `LastRegisteredEmail` die bereinigte E-Mail zu.
            
        - **`401 Unauthorized`:** Erzeugt ein negatives `AuthResult` mit Fehlermeldung bezüglich ungültiger Zugangsdaten.
            
        - **Sonstige Statuscodes:** Gibt ein negatives `AuthResult` mit Angabe des numerischen Statuscodes zurück.
            
    6. Fängt Exceptions per `try-catch` ab, loggt die Meldung über `Debug.WriteLine` und gibt ein negatives `AuthResult` mit der Fehlermeldung zurück.
        

#### 3. `Logout()`

- **Sichtbarkeit:** `public static`
    
- **Datentyp Rückgabewert:** `void`
    
- **Zweck:** Beendet die aktive Benutzersitzung im Frontend und setzt sensible Sitzungsdaten zurück.
    
- **Parameter:** Keine.
    
- **Rückgabewerte:** Keine (`void`).
    
- **Ablauf im Detail:**
    
    1. Setzt die statische Variable `AuthToken` auf einen leeren String (`string.Empty`) zurück.
        
    2. Setzt `CurrentUserEmail` auf `string.Empty` zurück, damit nachfolgende Aktionen nicht mehr auf ein aktives Benutzerkonto zugreifen können.
        

### III. Hilfsklasse `AuthResult`

- **Zweck:** Data Transfer Object (DTO) auf Clientseite zur einheitlichen Übermittlung von Erfolgs- und Statusmeldungen zwischen `AuthService` und den aufrufenden UI-Klassen bzw. ViewModels.
    
- **Eigenschaften:**
    
    - `Success` (`bool`): Zeigt an, ob die angeforderte Operation (z. B. Login) erfolgreich durchgeführt wurde.
        
    - `Message` (`string`): Enthält den Benutzer-Hinweistext zur Anzeige in Alert-Dialogen oder UI-Labels.
        
- **Konstruktor:**
    
    - `AuthResult(bool success, string message)`: Initialisiert die Instanz direkt mit den beiden erforderlichen Statuswerten.