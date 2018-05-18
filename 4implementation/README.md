# Programmeringskladd
Tanker om programmeringen og modelleringen rundt masteroppgaven

## Stikkord
* Språk: Java
* Type: NoSQL
* Basert på: Voldemort
* Status: Arkitektur av testapplikasjon uferdig

**Notabene**: Man vil forsøke å implementere en modulær dataevolusjonsløsning som kjører 'oppå' databaseklienten, det vil si at programmet er "klynge-ignorant" akkurat som den jevne webapplikasjon.

## Klasser som må implementeres:
* `package voldemort.tools` (ekstra)
  * En fin idé hadde vært å implementere et konsollbasert spørregrensesnitt eller et annet Admin-verktøy der man kan operere på det implisitte dataskjemaet ved å definere endringer i et ordnet format, noe KVolve savner i sin proof-of-concept-implementasjon
* `package dbupgradinator.deatheater`
* `package voldemort.client`
  * DBUpgradinatorStoreClient - Separat databaseklient som importerer de nye klassene fra DBUpgradinator - pakken
  * DBUpgradinatorStoreClientFactory (?)

Alternativt kan man benytte RMI - Remote Method Invocation, det vil si at hver enkelt databaseklient kontakter en kjørende "oppgraderingskoordinator".

# Om det fysiske perspektiv av testarkitekturen
 * Hver enkelt av de fire applikasjonsnodene gjestes av DigitalOcean, og alle er spredd utover fire forskjellige datasentre i London, Amsterdam (to separate datasentre), og Frankfurt. Dette gjør at hver enkelt forespørsel til tjener kan gå omtrent like raskt med det gitte replikeringsskjema.
 * Replikeringsmønstre, 1 node per senter
   * AMS1 -> AMS1; LON1; FRA1
   * AMS2 -> AMS2; LON1; FRA1
   * LON1 -> LON1; AMS1; AMS2
   * FRA1 -> FRA1; AMS1; AMS2

## Testprossess
 * Total størrelse på testdata: 24 . 100 GB
 * Partisjon på hver instans: 4 . 25 GB; Replikert datavolum på hver instans: 8 . 50 GB; Volum per maskin: 12 *75 GB*
 * Gjenværende volum reservert OS og hurtigminne: 80 GB (SSD-plass per droplet) - 75 GB = 5 GB / 20 G - 12 = 8
 * 1000 KB = 1 GB
 * Antakelse: Hvert aggregat i datamodell opptar i snitt 1 KB lagringsplass

## Trivia (Harry Potter-tema):
 * Brukernavn for UNIX-brukere på hver av dropletene: avery, black, crabbe, dolohov
 * IDer for lagringstjenere: riddle-diary, slytherin-locket, rawenclaw-diadem, nagini

https://shrib.com/#HnKm7VIOBxH.Qt3POwtJ

# Notater til masteroppgaven

## Problemstilling
Oppgaven tar for seg temaet vedlikehold av distribuerte databasesystem i produksjonsmiljø, det vil si kjørende systemer som tjener forespørsler fra brukere over HTTP - grensesnitt. Oppgaven tar for seg spesifikt hvordan en webapplikasjons database (web - skala) kan oppgraderes uten at produksjonsflyten forstyrres.

## Programmering
 - Java - prosjekt med Voldemort
 - Oppgave: Lag et datamodellmigrasjonssystem som kombinerer elementer fra KVolve (rettet mot semi-strukturerte/ustrukturerte modeller) og QuantumDB (modulær modellversjonering)
 - For å få til designet må jeg lese, forstå og oppsummere:
    - EBR - paperet
    - QuantumDB - presentasjon og - artikkel
    - **KVolve**
 - Implementer skjema-kontroll i webapplikasjon med Voldemort som realiserer db
    - Applikasjonen (klienten) holder styr på hvilke skjemaer den forventer å operere i

Implementasjon av KVolve sitt design med Voldemort
 - "Intelligent routing"/"smart klient", client side, det vil si backenden til webappen, av databasesystemarkitekturen, 2-hop-system
 - Backend for testapplikasjonen må dermed implementeres i Java
 - Testapplikasjonen etterlikner en netthandel i stil med eksemplene fra Sadalage sin bok
 - Versjonering av data og skjema: avro-generic-versioned; versjonering av tupler er "allerede programmert" i Voldemort
 - Datamigrasjon kan per inneværende kildekode også utføres med Avro, hvis gitte skjema er forover - og bakoverkompatible
 - Hvis den nye versjonen av skjemaet ikke kan migreres med Avro kan dette verktøyet brukes istedet
 - Transformatoren er et Java - objekt som serialiseres, sendes over Internett til backendtjenerne
 - Lazy data migration
 - Transformasjon av tilstandsdata

*Lat transformering:* Avart av avro-generic-versioned som kan utføre transformasjonen: En funksjon mottar en gammel nøkkel pluss dets tilhørende verdi på binær form. Verdien som oppdateres kan parses med den implementerte Serializer-klassen: **KVolve**
 - "On-demand (lazy) transformation: Once the update specification is installed, applications connected to the database that are out of date must be disconnected. They will reconnect at the new code version (mirroring the situation with eager upgrades). Doing this is not onerous for most applications..."

### KVolves mål: Logisk konsistente data for _databaseklienter_

KVolve sitt hovedmål er å realisere levende datamigrasjon i et nøkkel-verdi-lager uten at oppgraderingsprosessen går ut over spørringene til applikasjoner som henter data fra dem.

## TODO: Serverside
 - ObjectInputStream must be constructed on another stream
 - The objects MUST be read from the stream in the same order in which they were written
 - The readObject method deserializes the next object in the stream and traverses its references to other objects recursively to deserialize all objects that are reachable from it

```java
// Server?
ServerSocket server = new ServerSocket(1234);
Socket s = server.accept();
ObjectInputStream in = new ObjectInputStream(s.getInputStream());
CustomObject objectReceived = (CustomObject) in.readObject();
```

Modellbasert oppgradering: transformasjons-funksjon tar inn et objekt som er en instans av Aggregate, den eksplisitte modellen som utgjør M i MVC-mønsteret til webapplikasjonen, i tillegg til en strengnøkkel k:x. I kontekst av orbjektorientert programmering ble følgende programmatiske problem oppdaget: AggregatTransformasjonsklassen må importere modell-klassen dynamisk som et parameter.

## Evaluering

Diskutere krav-/måloppnåelse beskrevet i dette kapittel, forårsaker oppdateringen stopp i spørringstjeneste (kvalitativ); tid for oppdatering (separat test - tvungen oppdatering med separat adminprogram A som bruker getAll-funksjonen); beregning av ekstra lagringsvolum (også utført i program A som oppdaterer nøkler i serie av skriveoperasjoner); behandlede spørringer per sekund (ha en asynkron tellefunksjon som inkrementerer en spørringsteller-klasse i eksempelapplikasjon)

## 4.4

```latex
\subsection{Om adminprogrammet dba.jar}

For å sende en transformasjonsklasse til hver av applikasjonstjenerne i produksjonsklyngen brukes et enkelt hjelpeprogram kalt dba.jar.

Programmet dba.jar kjøres i Java 8 - kjøretidsmiljøet fra kommandolinjen og tar inn følgende seks argumenter:
\begin{enumerate}
  \setcounter{enumi}{0}
  \item Absolutt sti til transformasjonsklassen implementert av utvikleren
  \item Streng som indikerer det forrige versjonsprefikset til skjemaet denne klassen opererer på (spesifisert for å tillate tilbakerulling av aggregatskjema)
  \item Streng som indikerer det nåværende versjonsprefikset til skjemaet denne klassen opererer på (spesifisert for at Migrator-programmet skal kunne vite når klassens transformator skal brukes)
  \item Streng som indikerer det neste versjonsprefikset til skjemaet denne klassen opererer på (spesifisert for at Migrator-programmet skal kunne vite skjemaversjonen til aggregatet som opprettes i og returneres fra transformasjonsklassens transformasjons-funksjon)
  \item Streng som deklarerer klassenavnet til transformasjonsklassen implementert av utvikleren
  \item Absolutt sti til XML-konfigurasjonsfilen som deklarerer IP-adresse og TCP-port
\end{enumerate}
```
