# Web Coverage Service mit MapServer

Dieses Repository enthält ein Docker-Compose-Setup für den Betrieb von MapServer 7.4.2.
Die Konfiguration für die MapServer-Instanz kann mit Hilfe des ebenfalls enthaltenen
Gradle-Skripts automatisch auf Grundlage der im `data/`-Verzeichnis vorliegenden Daten
erstellt werden.

Um MapServer in Betrieb zu nehmen, müssen folgende Schritte durchgeführt werden:

* Ablegen der Daten im Verzeichnis `data/`
* Erzeugen der Kachelindizes (falls nicht bereits vorhanden)
* Anlegen der Basiskonfiguration
* Anlegen der Layer-Konfigurationen
* Erzeugen der MapServer-Konfigurationsdatei
* Anpassen der Docker-Compose-Konfiguration
* Starten des Docker-Containers

## Technische Voraussetzungen

Um den MapServer-WCS-Dienst starten zu können, müssen folgende Softwarepakete verfügbar
sein:

* Docker
* Docker Compose
* GDAL (entbehrlich, wenn Kachelindizes bereits existieren)

## Datenablage

Erzeugen Sie für jede auszuliefernde Coverage ein Unterverzeichnis in `data/`. Als
Verzeichnisnamen müssen Sie die Coverage-ID verwenden, also z.B. `rp_dop20_rgbi`.
Legen Sie die GeoTIFF-Daten die Coverage in diesem Verzeichnis ab.

## Kachelindex

Für jede Coverage muss im Verzeichnis `data/` ein Kachelindex als Shapefile vorliegen.
Der Shapefile muss nach der Coverage-ID benannt sein, also z.B. `rp_dop20_rbgi.shp`.

Unter Linux kann der Kachelindex automatisch erzeugt werden, sofern GDAL installiert
ist (das Tool `gdaltindex` muss im Pfad verfügbar sein). Mit Hilfe des folgenden
Befehls können die Kachelindizes automatisch abgeleitet werden:
    
    ./gradlew generate-tileindex

## Basiskonfiguration

Grundlegende Einstellungen für den WCS-Dienst müssen in der Datei `config.json` im
Grundverzeichnis vorgenommen werden. Zur Erstellung kann die Vorlage 
`config.json.example` kopiert werden. Die Konfigurationseigenschaften haben folgende
Bedeutungen:

- `globalEpsg`: EPSG-Code des globalen Koordinatensystems, in dem die der Dienst-Extent angegeben ist (z.B. "25832")
- `globalExtent`: Bounding-Box des Dienstes (z.B. "282000 5265800 603300 5643400")
- `outputFormats`: Definition der Ausgabeformate (s. Abschnitt zu Ausgabeformaten unten)
- `serviceMetadata`: Konfiguration der Dienst-Metadaten (s. Abschnitt Metadaten unten)

### Ausgabeformate

Die automatische Mapfile-Ableitung unterstützt folgende Parameter und bildet sie auf
den angegebenen Mapfile-Parameter ab:

- `driver` -> `DRIVER`
- `mimetype` -> `MIMETYPE`
- `imagemode` -> `IMAGEMODE`
- `extension` -> `EXTENSION`

Weiterführende Informationen, wie Ausgabeformate in MapServer konfiguriert werden können,
sind in der [MapServer-Dokumentation](https://mapserver.org/mapfile/outputformat.html) zu finden.

Beispiele für Ausgabeformate sind in der Datei `config.json.example` zu finden.

### Metadaten

Die automatische Mapfile-Ableitung unterstützt folgende Metadaten-Parameter und bildet sie in die
angegebenen Metadaten-Parameter des Mapfile ab (s. Mapserver-Dokumentation für weiterführende
Informationen zu diesen Parametern):

- `datasetIdentifierCode` -> `ows_inspire_dsid_code` (verpflichtend)
- `title` -> `ows_title`
- `wcs_abstract` -> `ows_abstract`
- `languages` -> `ows_languages` (verpflichtende Angabe mind. einer Sprache)
- `keywords` -> `ows_keywordlist` 
- `fees` -> `ows_fees`
- `accessConstraints` -> `ows_accessconstraints`
- `contactOrganization` - enthält Eigenschaften zur Organisation (s.u.)

Eigenschaften der Kontaktorganisation:

- `name` -> `ows_contactorganisation` (verpflichtend) 
- `onlineResource` -> `ows_service_onlineresorce`
- `contactPerson` -> `ows_contactperson`
- `contactPosition` -> `ows_contactposition`
- `contactPersonEmail` -> `ows_contactelectronicmailaddress`
- `contactPersonPhone` -> `ows_contactvoicetelephone`
- `contactPersonFax` -> `ows_contactfacsimiletelephone`
- `address` -> `ows_address`
- `city` -> `ows_city`
- `stateOrProvice` -> `ows_stateorprovince`
- `postalCode` -> `ows_postcode`
- `country` -> `ows_country`
- `hoursOfService` -> `ows_hoursofservice`
- `contactInstructions` -> `ows_contactinstructions`
- `role` -> `ows_role`

Für INSPIRE-spezifische Metadaten werden zwei Modi unterstüzt: eingebettet oder extern.
Das heißt, die INSPIRE-spezifischen Metadaten können entweder direkt in das Capabilities-
Dokument eingebettet oder extern verlinkt werden.

Um den eingebetteten Modus zu verweden, muss der Konfigurationsabschnitt `embeddedMetadata`
unterhalb von `serviceMetadata` angegeben werden. Folgende Eigenschaften werden dort unterstützt:

- `lastRevisionDate` - Datumswert im Format `YYYY-MM-DD`
- `metadataDate` - Datumswert im Format `YYYY-MM-DD`
- `resourceLocator` - URL
- `datasetIdentifierNamespace` - Namensraum für Datensatzidentifikator

Um der verlinkten Modus zu verwenden, muss der Konfigurationsabschnitt `linkedMetadata`
unterhalb von `serviceMetadata` angegeben werden, der nur eine Eigenschaft benötigt:

- `href` - Link auf den externen Metadatensatz

Es darf nur genau einer der beiden Abschnitte `embeddedMetadata` und `linkedMetadata`
vorhanden sein, anderenfalls scheitert die Mapfile-Erzeugung.

Weiterführende Informationen zu den Metadatenparameter sind in der MapServer-Dokumentation
in den Abschnitten [WCS-Server](https://mapserver.org/ogc/wcs_server.html) und [INSPIRE Download Service](https://mapserver.org/ogc/inspire_dl.html)
zu finden.

## Layerkonfiguration

In jedem Coverage-Verzeichnis muss eine Datei `layerconfig.json` vorhanden sein, in der
Details zur jeweiligen Coverage konfiguriert werden müssen. Zur Erstellung kann die
Vorlage `layerconfig.json.example` im Verzeichnis `data/` in die Coverage-Verzeichnisse
kopiert und dort angepasst werden.

## MapServer-Konfigurationsdatei

Das Ableiten der MapServer-Konfigurationsdatei `etc/mapserver.map` erfolgt über

    ./gradlew generate-mapfile

Eigenschaften, die nicht im Rahmen der automatischen Ableitung aus der Basis- und Layer-
Konfigurationsdateien gesetzt werden, können manuell editiert werden.

Ein vollständiges Beispiel für eine generierte Konfigurationsdatei ist in der Datei
`etc/mapserver.map.example` zu finden.

## Beispiel für Verzeichnisinhalte

Eine Beispiel für einen Verzeichnisbaum, der alle notwendigen Ordner und Dateien für
eine Coverage `rp_dop20_rgbi` enthält:

    <Basisverzeichnis>/
    + config.json
    + data/
    | +-- rp_dop20_rbgi.dbf
    | +-- rp_dop20_rgbi.shp
    | +-- rp_dop20_rgbi.shx
    | +-- rp_dop20_rgbi/
    |     +-- layerconfig.json
    |     +-- DOP20.tif
    |     +-- DOP20.tgw
    + etc/
      +-- mapserver.map
      
## Docker-Compose-Konfiguration

Die Docker-Compose-Konfiguration erfoglt in der Datei `docker-compose.yml`. Das Setup 
bindet die Verzeichnisse `data/` und `etc/` in den MapServer-Container ein.

Standardmäßig verbindet die Docker-Compose-Konfiguration den Port 80 von MapServer mit dem Port 8080
des Hostsystems. Dieses Verhalten kann durch Anpssen der linken Portnummer in der Datei 
`docker-compose.yml` angepasst werden:

    ports:
      - 8080:80

## Starten des Dienstes

Starten des MapServer-Containers:

    docker-compose up -d

Ausgabe der Container-Logs:

    docker-compose logs [-f] [--tail <n>]

