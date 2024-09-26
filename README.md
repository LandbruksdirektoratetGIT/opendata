# Test av GitHub for distribusjon av datasett

Dette er eit repo for å teste å distribuere datasett via GitHub.
Både for å undersøke generelt kva fordeler og ulemper det er ved å bruke GitHub, og spesifikt for korleis overføre datasett som i dag er distribuert via Datahotellet.

Testar med Landbruksdirektoratet sine datasett på [Datahotellet](https://hotell.difi.no/).

## Vurdering av GitHub som datalager

[+] Uavhengig. Dersom ein legg ut filer på eigen nettstad, må ein sørge for å flytte desse når ein byter nettside-plattform

[+] Får historikk på datasett

[+] Kan bruke GitHub sin funksjonalitet
- Opprette Issue for spørsmål og tilbakemeldingar
- Konsumentar kan setje opp varsling
- Konsumentar kan setje opp synkronisering

[+] Gratis. For dette scenariet kan ein bruke GitHub gratis. Datautgjevar kan opprette organisasjon på GitHub om dei ikkje allereie har det, og legge ut data i eit opent repository. I eit scenario med større filer (over 100 MB) vil ein måtte betale for Large File Storage (LFS).

[+] Komprimering. Ser ut til å vere gzip-komprimering under overføring av filer som standard. F.eks. CSV-fil på 750 KB er berre 350 KB overført.

[+] Førehandsvisning er støtta i GitHub i nettlesaren

[-] Førehandsvisning krev tilrettelegging. Kun førehandsvisning på filer som er maks 512 KB. Støtter ikkje alle CSV-variantar (må vere kommaseparert — ikkje semikolon). Må difor lage ei eiga fil for førehandsvisning som er kutta ned i størrelse og konvertert til kompatibelt CSV-format.

[-] Ikkje noko API for å gjere søk, filtrering o.l. på data

[-] Egnar seg ikkje for store datamengder. Enkeltfiler bør vere under 50 MB. Kan ikkje vere over 100 MB.

[-] Egnar seg ikkje for data som blir oppdatert ofte — repo blir fort stort

[-] Lite statistikk om tal på nedlastingar tilgjengeleg. Er noko tilgjengeleg under Insights --> Traffic

[-] Noko vanskelegare å fjerne data heilt frå historikken. Krev omskriving av historikken i Git-kodelageret. Finst verktøy som hjelper ein med dette.

## Endre CSV-format

Vi anbefaler å endre CSV-format ved overgang frå Datahotellet. Dette for å gjere CSV-filene enklare å konsumere, enten det er ved å programmere eller ved å opne i rekneark-programvare som Excel.

Oppsummert anbefaling: endre til UTF8 med BOM og bruk dobbelt hermeteikn som escape-teikn.

### UTF8 og BOM
UTF8 er teiknsettet brukt som gjer det mogeleg å ha med æøå. UTF8 kjem enten med eller utan Byte Order Mark (BOM). BOM består av tre usynlege teikn (bytes) i begynnelsen av fila som gjer det lettare for program å forstå kva teiknsett ei fil er skriven i.

For at æøå skal visast korrekt når ein opner CSV-fila i Excel, må det vere UTF8 _med_ BOM. Ulempa med å ha med BOM er at utviklarar kanskje må ta eit ekstra steg i å fjerne dei usynlege teikna for å prosessere fila korrekt. Dette avheng av kva verktøy og programvarebibliotek utviklaren nyttar. I Datahotellet er datasett lagra utan BOM. I API-et kan ein velje å laste ned datasettet som fil enten med BOM eller UTAN BOM.

#### Anbefaling

Publiser CSV-filer med UTF8 og BOM. Dette for å gjere det enklare for dei som opnar fila i Excel eller liknande, og dermed støtte både utviklarar og ikkje-utviklarar.

#### Legge på BOM

For å legge til BOM i ei CSV-fil, kan ein bruke kommandolinja. Dette kan gjerast ved å skrive tre spesifikke byte i begynnelsen av fila. Dette kan gjerast med `printf` og `cat`-kommandoar i Unix/Linux. Dersom fila allereie er i UTF8, kan ein bruke følgjande kommandoar:
```
printf '\xEF\xBB\xBF' > newfile.csv
cat originalfile.csv >> newfile.csv
```

### Escape-teikn
Escape-teikn i CSV-filer er teikn som er brukt for å skilje mellom data og kontrollteikn. Dersom data inneheld eit hermeteikn, må dette skiljast frå hermeteikn som er brukt for å omslutte eit felt. Dette gjer ein ved å bruke eit escape-teikn. Datahotellet har brukt slash (\\) som escape-teikn. Dette er ikkje standard i dag. Standard er å bruke dobbelt hermeteikn (") som escape-teikn. Dette er fordi det er meir vanleg i dag og støttast av fleire verktøy.

Eksempel på bruk av slash som escape-teikn:
```
orgnr;foretaksnamn;organisasjonsform
123456789;"Bedriften \"test\" AS";AS
```

Eksempel på bruk av dobbelt hermeteikn som escape-teikn:
```
orgnr;foretaksnamn;organisasjonsform
123456789;"Bedriften ""test"" AS";AS
```

#### Anbefaling
Publiser CSV-filer med dobbelt hermeteikn som escape-teikn sidan dette er den mest utbredte standarden i dag. Dette gjer det lettare å konsumere dataene både for utviklarar og ikkje-utviklarar.

#### Konvertere CSV-fil

csvformat er eit verktøy som kan konvertere CSV-filer. Dette verktøyet er tilgjengeleg i [csvkit](https://csvkit.readthedocs.io/en/latest/)-pakken. For å konvertere ein CSV-fil frå slash til dobbelt hermeteikn, kan ein bruke følgjande kommando:
```
csvformat -d ';' -p '\' -D ';' dataset.csv > fix.csv
```

## Oppsett
Scriptet pack.sh gjer tilrettelegging av data.
0. Forutsetning: har lasta ned heile katalogen "ldir" som ligg på Datahotellet
1. Kopierer filer frå ldir-katalog og fjerner overflødige filer
2. Genererer filer for kvart datasett:
  - sample.csv. For førehandsvisning. Konverterer til CSV-format (kommaseparert) som fungerer med førehandsvisning i GitHub
  - fields.csv. Henta frå fields.xml. Tar med felta shortName, name og content
  - README. Oversikt over filene i datasettet. Tar med tittel og sist-endra-tidspunkt frå meta.xml, samt feltdefinisjonar frå fields.csv

## Ikkje dekka her
Har ikkje sett på korleis Large File Storage (LFS) kan fungere.

## Korleis integrere bruk av GitHub i dataeigar sin arbeidsflyt med publisering av data?

TODO: skriv meir her

Dersom ein opprettar nye datasett som ikkje tidlegare har ligge på Datahotellet, bør ein nytte anledninga til å gjere nokre endringar. Treng ikkje ha med meta.xml og fields.xml lenger.
- dataset.csv : bør gå for eit meir standardisert format (sjå RFC-ar om CSV som format). Datahotellet har brukt slash (\\) som escape-teikn. Bør bruke dobbelt hermeteikn som er meir vanleg i dag.

## Vidare
- Oppskrift for registrering på data.norge.no
- Skriv om til å generere meta.csv og endre generering av README til å hente frå meta.csv. Formålet er å gjere det lettare å oppdatere datasett ved å sleppe å forhalde seg til XML-formatet.
=======
# datahotell
