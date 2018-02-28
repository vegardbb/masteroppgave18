# Programmeringskladd
Tanker om programmeringen og modelleringen rundt masteroppgaven

## Stikkord
* Språk: Java
* Type: NoSQL
* Basert på: Voldemort
* Status: Arkitektur av testapplikasjon uferdig

## Om det fysisk perspektiv av arkitekturen
 * Hver enkelt av de fire applikasjonsnodene gjestes av DigitalOcean, og alle er spredd utover fire forskjellige datasentre i London, Amsterdam (to separate datasentre), og Frankfurt. Dette gjør at hver enkelt forespørsel til tjener kan gå omtrent like raskt med det gitte replikeringsskjema. Alternativt kan de ligge på virkelig avsidesliggende datasentre, som for eksempel i New York, Toronto, og San Fransisco.
 * Replikeringsmønstre, 1 node per senter
   * AMS1 -> AMS1; LON1; FRA1
   * AMS2 -> AMS2; LON1; FRA1
   * LON1 -> LON1; AMS1; AMS2
   * FRA1 -> FRA1; AMS1; AMS2

