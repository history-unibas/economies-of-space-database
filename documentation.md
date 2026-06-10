Dokumentation Projektdatenbank
==============


# 1 Einleitung
TODO
- Hinweis auf Projekt, in welchem diese Datenbank erstellt worden ist
- Ziel von diesem Dokument: Dokumentiert die im Projekt entwickelte Datenbank ("Projektdatenbank").
- Inhalt von diesem Dokument (Übersicht) 


# 2 Systemarchitektur
Für die Befüllung der Projektdatenbank werden Daten aus unterschiedlichen Quellen verarbeitet. Die folgende Grafik stellt der Datenfluss der benutzten Daten dar.

![Systemarchitektur](system_architecture.svg)

Folgende Stellen und Hilfsmittel sind am Datenfluss beteiligt:
- Das Staatsarchiv stellt Digitalisate sowie Metadaten des historischen Grundbuchs Basel (HGB) zur Verfügung.
- Auf der Transkribus Plattform wird das Layout analysiert und semantisch angereichert (Identifikation des Typs einer Textregion, beispielsweise Datumszeile, Quellenverweis oder Haupttext) und Texte automatisiert erkannt.
- Das Grundbuch- und Vermessungsamt ermittelt auf der Basis der Adresse pro HGB-Dossier einen geographischen Standort, insofern dies ermittelbar ist. 
- In der Projektdatenbank werden die für die Forschung zentralen Informationen gespeichert und zur Verfügung gestellt. Weitere Details zur Datenbank sind im nachfolgenden Kapitel [3 Modellbeschreibung](#3-modellbeschreibung) dokumentiert.

Zentrale Prozesse des Datenflusses sind in Kapitel [4 Prozesse](#4-prozesse) beschrieben.

Für den gesamten Datenfluss ist die Replizierbarkeit sichergestellt, da alle Prozesse über Skripte angestossen werden (u.a. mit Zugriff auf die API von Transkribus). Insbesondere wird die Projektdatenbank mit einem [Skript](https://github.com/history-unibas/Postgresql-Project-Database/blob/main/project_database_update.py) erstellt, befüllt und aktualisiert .


# 3 Modellbeschreibung
Die Projektdatenbank vereint Daten, die im Aufbereitungsprozess erstellt werden (aus Texterkennung und Layoutanalyse sowie folgenden Schritten) mit bereitgestellten Metadaten aus der archivischen Beschreibung (erarbeitet durch das Staatsarchiv Basel-Stadt).
Die Datenbank ist entsprechend sowohl hierarchisch aufgebaut (Abfragen von der Transkribus-Plattform beeinflussen darauffolgende Schritte) als auch eine Kombination mit Datenabfragen aus SPARQL.
Die Projektdatenbank ist unterteilt in folgende Gruppen von Entitäten:
- **StABS**: Die Entitäten enthalten Metadaten des HGB, basierend auf Daten des [Linked Open Data Portals](https://ld.bs.ch/).
- **Transkribus**: Entitäten dieser Gruppe enthalten ausgewählte Daten der Plattform [Transkribus](https://readcoop.eu/transkribus/), insbesondere Transkriptionen ausgewählter Digitalisate sowie Informationen zur Kategorisierung visueller Einheiten (Textregionen, die als "Paragraph"/"Quellenangabe"/"Titulatur" etc. definiert werden).
- **Geo**: Daten mit geographischer Informationen werden in Entitäten dieser Gruppe gespeichert.
- **Project**: Entitäten enthalten aufbereitete Daten basierend auf Daten aus den anderen Gruppen.

Das Präfix der Bezeichnung jeder Entität entspricht der Gruppenbezeichnung. In der nachfolgender Tabelle sind alle Entitäten aufgeführt. In Kapitel [4 Prozesse](#4-prozesse) werden die Entitäten genauer umschrieben.
| Bezeichnung | Bedeutung | Anzahl Elemente |
|--------|--------|--------|
| [StABS_Serie](#411-entität-stabs_serie) | Metadaten des HGB einer Strasse | 231 |
| [StABS_Dossier](#412-entität-stabs_dossier) | Metadaten des HGB einer Adresse, Gebäude oder weitere Informationen | 6'056 |
| [StABS_Page](#413-entität-stabs_page) | Metadaten des HGB einer Seite (Digitalisats) | 193'406 |
| [Transkribus_Collection](#421-entität-transkribus_collection) | Daten von Transkribus einer Strasse | 207 |
| [Transkribus_Document](#422-entität-transkribus_document) | Daten von Transkribus einer Adresse, Gebäude oder weitere Informationen | 6'057 |
| [Transkribus_Page](#423-entität-transkribus_page) | Daten von Transkribus eines Digitalisats | 193'409 |
| [Transkribus_Transcript](#424-entität-transkribus_transcript) | Daten von Transkribus einer Transkriptions-Version | 751'079 |
| [Transkribus_TextRegion](#425-entität-transkribus_textregion) | Daten von Transkribus einer transkribierten Textregion | 607'771 |
| [Geo_Address](#431-entität-geo_address) | Geographische Standorte von HGB-Dossier | 3'785 |
| [Project_Dossier](#441-entität-project_dossier) | Aufbereitete Informationen zu Adressen | 4'347 |
| [Project_Entry](#442-entität-project_entry) | Aufbereitete Informationen zu HGB-Einträgen | 125'405 |
| [Project_Period](#443-entität-project_period) | Gültigkeitszeiträume der Dossier | 4'724 |
| [Project_Relationship](#444-entität-project_relationship) | Aufbereitete Beziehungen zwischen Dossier | 2'176 |

Die folgende Grafik zeigt, wie diese Entitäten in Beziehung gesetzt werden können. Lesebeispiel für die Beziehung zwischen StABS_Serie und StABS_Dossier: Ein Element der Entität StABS_Serie ist verbunden zu einem oder mehreren Elementen der Entität StABS_Dossier. Umgekehrt, ein Element der Entität StABS_Dossier ist mit genau einem Element der Entität StABS_Serie verbunden.

![Beziehungen](entityRelations.drawio.svg)

Eine formale Beschreibung der Entitäten in der Datenbank ist im [Readme des Github-Repositories](https://github.com/history-unibas/Postgresql-Project-Database/blob/main/README.md#database-schema) dokumentiert. 


# 4 Prozesse
Dieses Kapitel dokumentiert pro Gruppe von Entitäten die Aufbereitung der Daten aus dem HGB:
- Gruppe StABS: [4.1 HGB-Metadaten in Projektdatenbank integrieren](#41_hgb-metadaten_in_projektdatenbank_integrieren)
- Gruppe Transkribus: [4.2 Verarbeitung der Digitalisate](#42_verarbeitung_der_digitalisate)
- Gruppe Geo: [4.3 HGB-Dossier georeferenzieren](#43_hgb-dossier_georeferenzieren)
- Gruppe Project: [4.4 Anreicherung von Daten](#44_anreicherung_von_daten)


## 4.1 HGB-Metadaten in Projektdatenbank integrieren
Das Staatsarchiv Basel-Stadt (StABS) stellt via des [Linked Open Data Portals](https://ld.bs.ch/) Metadaten zum HGB zur Verfügung. Mithilfe eines [Skripts](https://github.com/history-unibas/Postgresql-Project-Database/blob/main/project_database_update.py) werden Metadaten des HGB gelesen und in der Projektdatenbank als Entitäten [StABS_Serie](#411-entität-stabs_serie), [StABS_Dossier](#412-entität-stabs_dossier) und [StABS_Page](#413-entität-stabs_page) gespeichert.


### 4.1.1 Entität StABS_Serie

#### 4.1.1.1 Bedeutung
Elemente der Entität StABS_Serie ("Serie") repräsentieren Strassen. Das Staatsarchiv hat diese Elemente als Stufe "Serie" klassiert.

#### 4.1.1.2 Entstehung
Metadaten zu Serien wurden mithilfe folgender Query bezogen:

```
PREFIX rico: <https://www.ica.org/standards/RiC/ontology#>
        SELECT ?link ?identifier ?title
        WHERE {
            {
            ?link rico:identifier ?identifier ;
            rico:title ?title ;
            rico:type "Akte"@ger ;
            rico:isDirectlyIncludedIn <https://ld.bs.ch/ais/Record/1027330> .
            }
        }
```
Dabei entspricht die URI https://ld.bs.ch/ais/Record/1027330 dem HGB 1-Bestand.

#### 4.1.1.3 Spezialfälle
Gewisse Serien haben keine zugehörige Dossiers (Objekte der Entität [StABS_Dossier](#412-entität-stabs_dossier)). Im Attribut "title" haben diese Serien den Präfix "[leer]".

#### 4.1.1.4 Fehler
Bei der Datenaufbereitung wurden einzelne Fehler entdeckt und durch das Staatsarchiv in der Zwischenzeit korrigiert.

#### 4.1.1.5 Beschreibung der Attribute
| Attribut | Bedeutung |
|--------|--------|
| serieId | Projekt-Identifikator einer Serie. Direkt abgeleitet aus der stabsId, ohne Leerzeichen und Vereinheitlichung der Anzahl Charakter |
| stabsId | Durch das Staatsarchiv definierter Identifikator einer Serie |
| title | Titel der Serie, üblicherweise eine oder mehrere Strassen- oder Ortsbezeichnung |
| linkRecord | URI des entsprechenden Eintrags im Linked Data Portal Basel-Stadt |


### 4.1.2 Entität StABS_Dossier

#### 4.1.2.1 Bedeutung
Elemente der Entität StABS_Dossier ("Dossier") repräsentieren Gebäude, Adressen, Teile eines Gebäudes sowie weitere Objekte oder Informationen zu einer bestimmten Strasse. Das Staatsarchiv hat diese Elemente als Stufe "Dossier" klassiert.

#### 4.1.2.2 Entstehung
Metadaten zu Dossier wurden mithilfe folgender Query bezogen, wobei der Parameter LINK_SERIE die URI der verknüpften Serie eingesetzt wird:

```
PREFIX rico: <https://www.ica.org/standards/RiC/ontology#>
PREFIX stabs-rico: <https://ld.bs.ch/ontologies/StABS-RiC/>
PREFIX schema: <http://schema.org/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT ?link ?identifier ?title ?note ?housenamebs ?oldhousenumber
    ?owner1862 ?instantiation_url ?manifest_url ?viewer_url
WHERE {{
    ?link rico:identifier ?identifier ;
    rico:title ?title ;
    rico:type "Akte"@ger ;
    rico:isDirectlyIncludedIn <{LINK_SERIE}> ;
    ^rico:isOrWasDigitalInstantiationOf ?instantiation_url .
    ?instantiation_url schema:url ?manifest_url ;
    rdfs:seeAlso ?viewer_url .
    OPTIONAL {{?link rico:note ?note .}}
    OPTIONAL {{?link stabs-rico:houseNameBS ?housenamebs .}}
    OPTIONAL {{?link stabs-rico:oldHousenumber ?oldhousenumber .}}
    OPTIONAL {{?link stabs-rico:owner1862 ?owner1862 .}}
}}
```

#### 4.1.2.3 Spezialfälle
keine Anmerkung

#### 4.1.2.4 Fehler
Bei der Datenaufbereitung wurden einzelne Fehler entdeckt und durch das Staatsarchiv korrigiert.

#### 4.1.2.5 Beschreibung der Attribute
| Attribut | Bedeutung |
|--------|--------|
| dossierId | Projekt-Identifikator eines Dossier. Direkt abgeleitet aus 'StABS_Dossier.stabsId', ohne Leerzeichen und Vereinheitlichung der Anzahl Charakter |
| serieId | Identifikator der zugehörigen Serie (Attribut 'StABS_Serie.serieId') |
| stabsId | Durch das Staatsarchiv definierter Identifikator eines Dossier |
| title | Titel des Dossiers, häufig entsprechend der Adresse gemäss dem Adressbuch des Jahres 1862 |
| linkRecord | URI des entsprechenden Eintrags im Linked Data Portal Basel-Stadt |
| linkInstantiation | URI zur Instantiation im Linked Data Portal Basel-Stadt |
| linkManifest | URL zum IIIF Manifest (REST-Schnittstelle) |
| linkViewer | URL zum Dokumenten-Viewer des Staatsarchivs Basel-Stadt |
| houseName | Name des Hauses |
| oldHousenumber | Alte Hausnummer |
| owner1862 | Besitzer des Hauses im Jahr 1862 |
| descriptiveNote | Auf der Titelseite des Dossier angebrachte Bemerkung |


### 4.1.3 Entität StABS_Page

#### 4.1.3.1 Bedeutung
Die Elemente der Entität StABS_Page ("Seite") repräsentieren jeweils ein Digitalisat und entsprechen der Vorder- oder Rückseite einer HGB-Karteikarte.

#### 4.1.3.2 Entstehung
Die Metadaten der Seiten werden aus dem IIIF Manifest des zugehörigen Dossier extrahiert sowie aus den Metadaten des entsprechenden Dossiers abgeleitet.

#### 4.1.3.3 Spezialfälle
Auf einer Seite befindet sich häufig einen Auszug aus einem Archivdokument ("Eintrag"). Teilweise befinden sich mehrere Einträge auf einer einzelnen Seite, teils erstreckt sich ein einzelner Eintrag über mehrere Seiten.

#### 4.1.3.4 Fehler
keine Anmerkung

#### 4.1.3.5 Beschreibung der Attribute
| Attribut | Bedeutung |
|--------|--------|
| pageId | Projekt-Identifikator einer Seite. ID wird zusammengesetzt aus 'StABS_Dossier.dossierId' und 'StABS_Page.pageNr' nach dem folgendem Schema: [dossierId]_{int([pageNr]):03} |
| dossierId | Identifikator des zugehörigen Dossiers (Attribut 'StABS_Dossier.dossierId') |
| pageNr | Seitenzahl der Seite im Dossier |
| linkViewer | URL zum Dokumenten-Viewer des Staatsarchivs Basel-Stadt |


## 4.2 Verarbeitung der Digitalisate
TODO
- Prozess beschreiben (https://drive.switch.ch/index.php/f/6566052981) -> Transkribus
- inkl. Paper, Kapitel 5
- Auf ausgeschlossene Seiten hinweisen -> #### 4.2.4.3 Spezialfälle


### 4.2.1 Entität Transkribus_Collection

#### 4.2.1.1 Bedeutung
Elemente der Entität Transkribus_Collection ("Kollektion") repräsentieren Strassen. In der Transkribus Plattform sind diese Objekte als Collections gespeichert. 

#### 4.2.1.2 Entstehung
Auf Basis der Serien (Elemente der Entität StABS_Serie) wurden die Kollektionen auf Transkribus definiert.

#### 4.2.1.3 Spezialfälle
Es wurden nur für Serien entsprechende Kollektionen auf Transkribus generiert, für welche Dossier existieren. Aus diesem Grund existieren folgende 24 Serien nicht als Kollektionen:
- HGB_1_011
- HGB_1_019
- HGB_1_021
- HGB_1_022
- HGB_1_030
- HGB_1_033
- HGB_1_045
- HGB_1_046
- HGB_1_049
- HGB_1_054
- HGB_1_055
- HGB_1_057
- HGB_1_067
- HGB_1_077
- HGB_1_081
- HGB_1_102
- HGB_1_107
- HGB_1_110
- HGB_1_127
- HGB_1_140
- HGB_1_158
- HGB_1_186
- HGB_1_187
- HGB_1_200

#### 4.2.1.4 Beschreibung der Attribute
| Attribut | Bedeutung |
|--------|--------|
| colId | Identifikation der Kollektion |
| colName | Name der Kollektion, entspricht dem Identifikation der Serie (Attribut 'StABS_Serie.serieId') |
| nrOfDocuments | Anzahl mit der Kollektion verknüpfter Dokumente |


### 4.2.2 Entität Transkribus_Document

#### 4.2.2.1 Bedeutung
Elemente der Entität Transkribus_Document ("Dokument") repräsentieren Gebäude, Adressen, Teile eines Gebäudes sowie weitere Objekte oder Informationen zu einer bestimmten Strasse. In der Transkribus Plattform sind diese Objekte als Documents gespeichert.

#### 4.2.2.2 Entstehung
Auf Basis der Dossier (Elemente der Entität StABS_Dossier) wurden Dokumente auf Transkribus definiert.

#### 4.2.2.3 Spezialfälle
Das Dokument HGB_1_003_001 wird in StABS_Dossier nicht als Dossier betrachtet. Aus diesem Grund existiert in Transkribus_Document ein Element mehr als in StABS_Dossier.

#### 4.2.2.4 Beschreibung der Attribute
| Attribut | Bedeutung |
|--------|--------|
| docId | Identifikation des Dokuments |
| colId | Identifikator der zugehörigen Kollektion (Attribut 'Transkribus_Collection.colId') |
| title | Name des Dokument, entspricht dem Identifikation des Dossiers (Attribut 'StABS_Dossier.dossierId') |
| nrOfPages | Anzahl im Dokument vorhandener Seiten |


### 4.2.3 Entität Transkribus_Page

#### 4.2.3.1 Bedeutung
Dokument auf Transkribus bestehen aus einer oder mehreren Pages ("Seiten"). Elemente der Entität Transkribus_Page repräsentieren eine Digitalisat auf Transkribus. Mithilfe von Transkribus_Document.title ("Dossier-ID") und Transkribus_Page.pageNr ("Seitenzahl") ist eine Verknüpfung zu den Elementen in StABS_Page möglich mithilfe der Attribute 'StABS_Page.dossierId' und 'StABS_Page.pageNr'.

#### 4.2.3.2 Entstehung
Die vom Staatsarchiv erhaltene Digitalisate wurden auf die Transkribus Plattform hochgeladen.

#### 4.2.3.3 Spezialfälle
In Transkribus_Page existieren drei Seiten im Dokument HGB_1_003_001, welche nicht in der Entität StABS_Page verfügbar sind. Grund: Dieses Dokument ist beim Staatsarchiv nicht als Dossier abgebildet.

#### 4.2.3.4 Beschreibung der Attribute
| Attribut | Bedeutung |
|--------|--------|
| pageId | Identifikation der Seite |
| key | Schlüssel der Seite |
| docId | Identifikator des zugehörigen Dokuments (Attribut 'Transkribus_Document.docId') |
| pageNr | Seitenzahl der Seite im Dokument |
| urlImage | URI des Digitalisats auf der Transkribus Plattform |
| entryId | Identifikator des zugehörigen Eintrags (Attribut 'Project_Entry.entryId') |


### 4.2.4 Entität Transkribus_Transcript

#### 4.2.4.1 Bedeutung
Jeder auf Transkribus durchgeführte Schritt einer Seite wird als "pageXML"s gespeichert ("Transkript"). In der Entität Transkribus_Transcript wird jede pageXML-Version zusammen mit Metadaten gespeichert.

#### 4.2.4.2 Entstehung
Jede Bearbeitung einer Seite auf Transkribus wie beispielsweise eine Layoutanalyse oder Durchführung einer Transkription erzeugt eine neue Version eines pageXMLs.

#### 4.2.4.3 Spezialfälle
TODO: Nicht bearbeitete Seiten auflisten -> gekennzeichnet mit Status DONE
- Brandlagerbücher
- Reichspfennigverzeichnisse
- Titelseiten (erste zwei Seiten eines Dossiers) -> Status NEW
- Dossier ausserhalb Stadtmauern


Anmerkung Benjamin: Grundsätzlich ausgeschlossen wurden Brandlagerbücher, die alle aus dem 19. Jahrhundert stammen, sowie die Karteikarten des Reichspfennigs von 1497 (weil hier keine Transaktion festzustellen ist). Die zwei Typen von Karten wurden mit dem Bilder-Clustering identifiziert und ausgeschlossen


#### 4.2.4.4 Beschreibung der Attribute
| Attribut | Bedeutung |
|--------|--------|
| key | Schlüssel des Transkripts |
| tsId | Identifikator des Transkripts |
| pageId | Identifikator der zugehörigen Seite (Attribut 'Transkribus_Page.pageId') |
| parentTsId | Identifikator der vorheriger Transkript-Version (Wert ist "-1", wenn keine vorherige Version existiert) |
| pageXML | pageXML des Transkripts |
| status | Definierter Status der Transkripts. Bedeutung der Werte:<br>- NEW: Seite wurde auf Transkribus hochgeladen <br>- IN_PROGRESS: Seite wurde bearbeitet <br>- DONE: Seite wird von weiterer Verarbeitung ausgeschlossen |
| timestamp | Zeitpunkt der Erstellung des Transkripts |
| htrModel | Für die durchgeführte Trankription oder Bearbeitung der Seite verwendetes Modell. <br> TODO: Beschreiben, welche Modelle benutzt worden sind (oder verlinken) |


### 4.2.5 Entität Transkribus_TextRegion

#### 4.2.5.1 Bedeutung
Elemente der Entität Transkribus_TextRegion repräsentieren eine "Textregion" eines bestimmten Transkrips.

#### 4.2.5.2 Entstehung
Für jede Seite wird vor der Transkription der Texte eine Layouterkennung durchgeführt. Die dadurch erhaltenen Textregionen werden in den pageXML abgebildet und die Text-Transkriptionen pro Textregion ermittelt.

#### 4.2.5.3 Spezialfälle
Textregionen ohne transkribierten Text sind von dieser Entität ausgeschlossen. Basierend auf den im pageXML verfügbaren Koordinaten werden automatisiert Zeilenreihenfolgen korrigiert.

#### 4.2.5.4 Beschreibung der Attribute
| Attribut | Bedeutung |
|--------|--------|
| textRegionId | Generierter Identifikator nach dem folgendem Schema: [key]_{int([index]):02} |
| key | Schlüssel der Textregion |
| index | Index der Textregion |
| type | Identifizierte Art der Textregion. Mögliche Kategorien: paragraph, header, marginalia, credit, footer |
| textLine | Transkribierter Text pro Zeile als Liste |
| text | Transkribierter Text der Textregion |


## 4.3 HGB-Dossier georeferenzieren
Um Erkenntnisse aus dem HGB räumlich analysieren zu können und Ergebnisse im Raum darzustellen, soll jedes für das Projekt relevante Dossier räumlich verortet werden. Zur Erreichung dieses Ziels, wurde in einem ersten Schritt HGB-Dossier durch das [Grundbuch- und Vermessungsamt des Kantons Basel-Stadt](https://www.bs.ch/bvd/grundbuch-und-vermessungsamt) georeferenziert. Dieser Prozess wird in diesem Kapitel beschrieben. Im zweiten Schritt wurde im Rahmen des Projekts weitere geographische Standorte ermittelt sowie Standorte verbessert. Dieser Prozess ist in Kapitel [4.4.1.4 Beschreibung der Attribute](#4414-beschreibung-der-attribute) in den Attributen 'locationUncorrected' und 'location' dokumentiert.


### 4.3.1 Entität Geo_Address

#### 4.3.1.1 Bedeutung
Jeder Eintrag der Entität Geo_Address repräsentiert ein HGB-Dossier und beinhaltet der geographische Standort des im Dossier abgebildeten Gebäudes als Punktgeometrie. 

#### 4.3.1.2 Entstehung
Die Georeferenzierung von HGB-Dossiers kann durch folgende Teilprozesse beschrieben werden.

![Prozess Geo](prozess_geo.svg)

- **HGB-Metadaten beziehen**: Mithilfe des [Linked Data Portal Basel-Stadt](https://ld.bs.ch/) werden Metadaten zum HGB bezogen.
- **Dossier georeferenzieren**: Auf Basis eines Datensatzes von geolokalisierten historischen Adressen der Stadt Basel und den HGB-Metadaten ("Titel" und "Alte Hausnummer") hat das Grundbuch- und Vermessungsamt mit einem [FME-Skript](https://fme.safe.com/) eine Shape-Datei erstellt. Ein HGB-Dossier wird durch einen Punkt im Koordinatensystem LV95 ([EPSG:2056](https://epsg.io/2056)) repräsentiert.
- **Daten in Projektdatenbank integrieren**: Mithilfe eines [Skript](https://github.com/history-unibas/Postgresql-Project-Database/blob/main/project_database_update.py) werden die Daten der Shape-Datei gelesen und in der Projektdatenbank gespeichert.

#### 4.3.1.3 Spezialfälle
Umfasst ein HGB-Dossier mehr als eine Hausnummer, wird für die Geolokalisierung die erste Hausnummer verwendet.

Viele HGB-Dossier konnten in diesem Prozess nicht geolokalisiert werden:
- Für 997 Dossier existiert keine Hausnummer in den Metadaten, beispielsweise weil das Dossier Personenregister enthaltet.
- 632 Dossier konnten nicht einer alten Hausnummer zugeordnet werden. Dies betrifft teilweise alte Hausnummern, welche im Adressverzeichnis von 1862 nicht mehr existieren oder Eigenkreationen von den Verfassern des HGB sind.
- Für 416 Dossier fehlte eine "neue" Adresse, das heisst Adressen nach 1862. Diese Dossier beinhalten meistens nur Brandlagerbücher.
- Für 227 Dossier ist der Grund nicht bekannt.

#### 4.3.1.4 Fehler
Bei einzelnen Metadaten wurden Fehler in den Adressen festgestellt. Beispiele: falsch erfasste Attribute, Fehler in Gross-/Kleinschreibung, Hausnummer "4/6" statt "416". Das Staatsarchiv wurde über diese Fehler informiert.

Die Georeferenzierung durch das Vermessungsamt basiert hauptsächlich auf den alten Hausnummern. Deshalb ist die zum Teil feinere Gliederung, die das Adressverzeichnis von 1862 aufweist, nicht abgebildet.

#### 4.3.1.5 Statistik
Von den 6'057 HGB-Dossier, über welche Metadaten verfügbar sind, konnten im Rahmen dieser Geolokalisierung für 3'785 Dossier einen Standort ermittelt werden. Es existieren Dossier, welche denselben Standort besitzen.

#### 4.3.1.6 Beschreibung der Attribute
Die vom Grundbuch- und Vermessungsamt erhaltene Shape-Datei wird ohne Veränderung der Attribute in die Projektdatenbank übernommen. Diese Attribute enthalten Metadaten des Staatsarchivs sowie während der Georeferenzierung gespeicherte Zwischenschritte. Im Projekt wird ausschliesslich die Signatur (entspricht dem Attribut 'StABS_Dossier.stabsId') sowie die Punktgeometrie (im Koordinatensystem EPSG:2056) weiterverwendet. Aus diesem Grund wird auf eine genaue Beschreibung der Attribute verzichtet.


## 4.4 Anreicherung von Daten
Auf Basis der Entitäten der Gruppen "StABS", "Transkribus" und "Geo" ([3 Modellbeschreibung](#3-modellbeschreibung)) wurden die Entitäten der Gruppe "Project" iterativ mithilfe von Skripts und manueller Bearbeitung entwickelt.

### 4.4.1 Entität Project_Dossier

#### 4.4.1.1 Bedeutung
Jedes Element dieser Entität ("Dossier") repräsentiert ein Dossier im Historisches Grundbuch der Stadt Basel (HGB). Alle Dossier sind ebenfalls in der Tabelle StABS_Dossier abgebildet.

#### 4.4.1.2 Entstehung
Es werden ausschliesslich Dossier in dieser Entität abgebildet, welche mindestens einen Eintrag in der Entität Project_Entry mit Bezug zum entsprechenden Dossier besitzt.

#### 4.4.1.3 Spezialfälle
Üblicherweise repräsentiert ein Dossier ein Gebäude. Spezielle Dossier enthalten im Attribut "specialType" einen Wert.

#### 4.4.1.4 Beschreibung der Attribute
| dossierId |  |
|--------|--------|
| Bedeutung | Projekt-Identifikator eines Dossier |
| Entstehung | Entspricht dem Attribut 'StABS_Dossier.dossierId' |
| Spezialfälle | - |
| Fehler | - |
| Statistik | - |

| locationUncorrectedAccuracy |  |
|--------|--------|
| Bedeutung | Für in diesem Projekt manuell gesetzte Standorte (Attribut 'Project_Dossier.locationUncorrected') wird in diesem Attribut eine Aussage gemacht, wie genau dieser Standort definiert werden konnte. Standorte auf Basis von Standorten des Grundbuch- und Vermessungsamt sind als "unbekannt" gekennzeichnet, Dossier ohne Standorte als "nicht lokalisierbar". |
| Entstehung | siehe Entstehung des Attributs 'Project_Dossier.locationUncorrected' |
| Spezialfälle | - |
| Fehler | - |
| Statistik | unbekannt: 3'475<br>genau gesetzt: 397<br>ungefähre Ortsangabe: 328<br>ungefähr gesetzt: 123<br>grob geschätzt: 18<br>nicht lokalisierbar: 6 |

| locationUncorrectedOrigin |  |
|--------|--------|
| Bedeutung | Dieses Attribut beschreibt, auf welcher Basis der Standort des Dossier (Attribut 'Project_Dossier.locationUncorrected') entstanden ist. |
| Entstehung | siehe Entstehung des Attributs 'Project_Dossier.locationUncorrected' |
| Spezialfälle | Dossier ohne Standort besitzen keinen Wert |
| Fehler | - |
| Statistik | Grundbuch- und Vermessungsamt Basel-Stadt: 3'089<br>manuell gesetzt: 866<br>mithilfe von Skript generiert basierend auf Standorte von Grundbuch- und Vermessungsamt: 306<br>manuell geprüft: 80<br>[NULL]: 6 |

| locationUncorrected |  |
|--------|--------|
| Bedeutung | Dieses Attribut beinhaltet der geografische Standort des Dossiers im Koordinatensystem LV95 (EPSG:2056). |
| Entstehung | Die Standorte basierend mehrheitlich auf Daten des Grundbuch- und Vermessungsamt des Kantons Basel-Stadt (Entität Geo_Address). Für ausgewählte Dossier sowie für Dossier ohne Standort in der Entität Geo_Address wurde mithilfe eines Skripts Standorte ermittelt basierend auf vorhandenen Standorte (Entität Geo_Address). Um Fehler zu reduzieren und den Datensatz zu vervollständigen, wurden anschliessend Standorte von Dossier manuell geprüft und gesetzt basierend auf dem Löffelplan von 1862 (https://www.bs.ch/bvd/grundbuch-und-vermessungsamt/vermessung/historische-plaene). Manuell definiert oder kontrolliert wurden Standorte von Dossier, welche nach der Anwendung des Skripts keinen Standort besassen, sich innerhalb von 20 Meter kein weiteres Dossier derselben Strasse befindet aufgrund des Verdachtes auf einen fehlerhaften Standort, alle Dossier der Hebelstrasse aufgrund fraglicher Geolokalisierung sowie einzelne bei der Durchsicht aufgefallene Dossier. Die Herkunft jedes Dossier-Standortes kann dem Attribut 'Project_Dossier.locationUncorrectedOrigin' entnommen werden. Unterschiedliche Standorte, welche sich weniger als einen Meter voneinander entfernt befinden, wurden harmonisiert. |
| Spezialfälle | 6 Dossier haben keinen definierten Standort (locationUncorrectedAccuracy='nicht lokalisierbar') und beinhalten Informationen zu Gewässer, Teich oder Mauer (siehe Attribut 'Project_Dossier.specialType'). |
| Fehler | Für Dossier, dessen Standort manuell festgelegt wurde, beinhaltet das Attribut 'Project_Dossier.locationUncorrectedAccuracy' eine Aussage über die Genaugikeit des Standortes. Für Standorte, welche vom Grundbuch- und Vermessungsamt übernommen worden sind, können wir keine Angabe machen (locationUncorrectedAccuracy='unbekannt').<br>Bei der Entwicklung des Attributs 'Project_Dossier.location' wurden manuell die Standorte einzelner Dossier verbessert. Diese Korrekturen wurden nicht im Attribut 'Project_Dossier.locationUncorrected' umgesetzt. |
| Statistik | - |

| location |  |
|--------|--------|
| Bedeutung | Dieses Attribut beinhaltet den geschobenen geografischen Standort des Dossiers im Koordinatensystem LV95 (EPSG:2056). Im Vergleich zum Attribut 'Project_Dossier.locationUncorrected' wurden der Standort für ausgewählte Dossier verschoben mit dem Ziel, dass weniger Dossier denselben Standort besitzen und damit die Darstellung der Dossier auf einer Karte zu verbessern. |
| Entstehung | Der geschobene Standort basiert auf dem Attribut 'Project_Dossier.locationUncorrected'. Mithilfe eines Algorithmus wurde für Dossier, in welchen im Titel (Attribut 'StABS_Dossier.title') "neben" erwähnt ist, ein entsprechendes Dossier gesucht. Ist ein "neben-Dossier" verfügbar, wurde der Standort um 1/4 der Distanz in Richtung des neben-Dossier verschoben. Beispiel: "St. Alban-Vorstadt Theil von 17 neben 15" (Dossier: HGB_1_010_041, neben-Dossier: "St. Alban-Vorstadt 15", HGB_1_010_039). Die Distanz von einem Viertel ist willkürlich gewählt, muss aber deutlich weniger als die Hälfte sein, weil sonst «Teil von 15 neben 17» auf den gleichen Punkt zu liegen käme. Für Dossier, welche im Titel mehrere Adressen umfassen ("verbundene Dossier"), wurden entsprechende Dossier gesucht, welche je eine Adresse abbildet. Wurden entsprechende Dossier gefunden, ist als verschobener Standort des verbundenen Dossiers der geometrischer Schwerpunkt der entsprechenden Dossier definiert worden. Beispiel: "St. Alban-Graben 8, 10" (Dossier: HGB_1_005_020, entsprechende Dossier: "St. Alban-Graben 8", HGB_1_005_019 und "St. Alban-Graben 10", HGB_1_005_021).<br>Ausgewählte Dossier wurden anschliessend manuell geprüft und verschoben basierend auf dem Löffelplan von 1862 (https://www.bs.ch/bvd/grundbuch-und-vermessungsamt/vermessung/historische-plaene). Manuell geprüft wurden Standorte von Dossier, welche mit dem Skript verschoben worden sind sowie Dossier, welche aufgrund des Titels (Attribut 'StABS_Dossier.title') eine Verschiebung erwartet wurde. Insbesondere wurde die Unterscheidung in Vorder- und Hinterhäuser zusätzlich getroffen, wenn sie ersichtlich war. Bei der abschliessenden manuellen Durchsicht sind viele Dossiers aufgefallen, die nicht ganz korrekt lokalisiert waren. Händische Korrekturen erfolgten vornehmlich (aber nicht nur) dort, wo die Strassen um 1862 stark vom früheren Strassenbild abwichen (Beispiele: Eisengasse, untere Freie Strasse, Fischmarkt). In diesen Fällen wurden die im HGB hinterlegten Planzeichnungen konsultiert. Im Attribut 'Project_Dossier.locationOrigin' ist festgehalten, welche Dossier mit dem Algorithmus respektive manuell verschoben worden sind. Unterschiedliche Standorte, welche sich weniger als einen Meter voneinander entfernt befinden, wurden harmonisiert. |
| Spezialfälle | 6 Dossier haben keinen definierten Standort und beinhalten Informationen zu Gewässer, Teich oder Mauer (siehe Attribut 'Project_Dossier.specialType'). |
| Fehler | Die Genauigkeit war bei der Defintion der Standorte kein eigentliches Ziel. Die Dossiers wurden zwar in eine plausible Richtung verschoben und sind in den meisten Fällen genauer platziert als zuvor (Attribut 'Project_Dossier.locationUncorrected'), das lässt sich aber nicht messen. Die händische Verschiebung wurde nach Augenmass und nicht mit einer korrekten Messung vorgenommen. |
| Statistik | - |

| locationOrigin |  |
|--------|--------|
| Bedeutung | Dieses Attribut dokumentiert die Verschiebung des verschobenen Standortes (Attribut 'Project_Dossier.location') im Vergleich zum Standort (Attribut 'Project_Dossier.locationUncorrected'). |
| Entstehung | siehe Entstehung des Attributs 'Project_Dossier.location' |
| Spezialfälle | Dossier ohne Standort haben keinen Wert. |
| Fehler | - |
| Statistik | keine Verschiebung: 2'753<br>Verschiebung mit Algorithmus: 807<br>manuelle Verschiebung: 781<br>[NULL]: 6 |

| specialType |  |
|--------|--------|
| Bedeutung | Mit diesem Attribut werden Dossier markiert, welche nicht ein Gebäude repräsentieren, zum Beispiel ein Gewässer. |
| Entstehung | Mit einfachen Stichwortsuchen (z.B. nach dem Begriff "Laden") im Attribut 'StABS_Dossier.title' wurden Dossier identifiziert, welche kein Gebäude abbildet. |
| Spezialfälle | - |
| Fehler | Bei der visuellen Kontrolle des Attributs 'StABS_Dossier.title' von Dossier ohne Zugehörigkeit zu einem Cluster (Skript dossier_relationship.py) und ohne Wert im Attribut 'Project_Dossier.specialType' wurden keine weiteren spezielle Dossier gefunden. In dieser Teilmenge existieren jedoch Dossier mit einem Strassennamen aber ohne Hausnummer, beispielsweise in der Freien Strasse. Diese Dossier besitzen teilweise eine alte Hausnummer (Attribut 'StABS_Dossier.oldhousenumber'), es könnte jedoch auch einen Hinweis sein auf weitere "Spezial-Dossier", welche in diesem Attribut nicht berücksichtigt sind. |
| Statistik | [NULL]: 3'937<br>Unbestimmte Liegenschaften: 98 <br>Strassenkörper: 92<br>Brunnen: 50<br>Dohle: 41<br>Unbestimmt: 27<br>Gewässer: 16<br>Keller: 12<br>Garten: 10<br>Laden: 9<br>Sammeldossier: 9<br>Boden: 5<br>Graben: 5<br>Zins: 5<br>Brotbank: 4<br>Bank: 3<br>Graben, Mauer: 3<br>Nachträge: 3<br>Teich: 3<br>Mauer: 2<br>Quartier: 2<br>Tor: 2<br>Abort: 1<br>Allmend : 1<br>Brunnen, Unbestimmte Liegenschaften: 1<br>Brücke: 1<br>Halseisen, Heisserstein: 1<br>Häuserverzeichnis: 1<br>Platz: 1<br>Salzkasten: 1 <br>Schutzrein: 1|


### 4.4.2 Entität Project_Entry

#### 4.4.2.1 Bedeutung
Jedes Element dieser Entität ("Eintrag") repräsentiert einen im HGB erfassten Eintrag. Es können mehrere Einträge auf einer Registerkarte ("Seite") des HGB dokumentiert sein, oder ein Eintrag kann sich über mehrere Seiten erstrecken. Eine Seite im HGB wird durch ein Element in den Entitäten 'StABS_Page' respektive 'Transkribus_Page' repräsentiert. Befinden sich mehrere Einträge auf einer Registerkarte, so werden diese Einträge nicht durch mehrere Elemente in dieser Entität repräsentiert. Eine Seite wird als Folgeseite eines Eintrags betrachtet, wenn die Seite keine Textregion (Entität 'Transkribus_TextRegion') des Typs "header" und "marginalia" besitzt und die vorhergehende Seite keine Textregion des Typs "credit" hat. Einen Eintrag basiert nur auf Seiten desselben Dossiers.

#### 4.4.2.2 Entstehung
Es werden Seiten des HGB betrachtet, welche transkribiert worden sind (Entität 'Transkribus_Transcript'). Für die Generierung der Einträge wird das aktuellste Transkript einer Seite verwendet (Attribut 'Transkribus_Transcript.timestamp'). Hat das aktuellste Transkript den Status "DONE" (Attribut 'Transkribus_Transcript.status'), wird diese Seite nicht berücksichtigt. Ebenfalls werden Seiten ausgeschlossen, welche einem Dossier verbunden ist, welches sich ausserhalb der Stadtmauern befindet. Seiten ohne Textregionen (Entität 'Transkribus_TextRegion') sind ebenfalls ausgenommen. Manuell wurden 3'202 Seiten für die Generierung dieser Entität ausgeschlossen. Darunter fallen insbesondere Karteikarten, die Parzelleninformationen enthalten (Angaben zu zeitweise zusammengelegte Dossiers, oder auch Angaben zur Baugeschichte wie gefundene Jahreszahlen). Identifiziert wurden diese Seiten hauptsächlich bei der händischen Datierung (siehe nächster Abschnitt) oder der händischen Zuweisung von Quellenbelegen von Einträgen (Attribut 'Project_Entry.source').

Manuell wurden Einträge bearbeitet, namentlich die Jahreszahl (Attribut 'Project_Entry.year') und die Definition von Seiten als Folgeseiten. Alle undatierte Einträge wurden händisch durchgesehen. Basierend auf einer Analyse unter der Annahme, dass Einträge chronologisch in einem Dossier abgelegt sind, wurden weitere Jahreszahlen überprüft. Bei weiteren Analysen und Prüfungen wurden weitere Datierungen überprüft und korrigiert. Manuell bearbeitete Einträge haben im Attribut 'Project_Entry.manuallyCorrected' den Wert "True". Weitere Angaben zur Jahreszahl enthaltet das Attribut 'Project_Entry.comment', insbesondere für Einträge ohne eine Jahreszahl.

#### 4.4.2.3 Spezialfälle
keine Anmerkung

#### 4.4.2.4 Beschreibung der Attribute
| entryId |  |
|--------|--------|
| Bedeutung | Identifikator des Eintrags |
| Entstehung | Bei der Erstellung oder Aktualsierung dieser Entität wird eine zufällige UUID mit dem aktuellen Datum (Format "YYYYMMDD") als Postfix erzeugt. |
| Spezialfälle | - |
| Fehler | - |
| Statistik | - |

| dossierId |  |
|--------|--------|
| Bedeutung | Identifikator des zugehörigen Dossier (Attribut 'Project_Dossier.dossierId') |
| Entstehung | Direkt abgeleitet von 'StABS_Dossier.stabsId' |
| Spezialfälle | - |
| Fehler | - |
| Statistik | - |

| pageId |  |
|--------|--------|
| Bedeutung | Identifikatoren der zum Eintrag gehörender Seiten gespeichert als Liste. |
| Entstehung | Die pageId wurde durch die Transkribus-Plattform generiert. |
| Spezialfälle | - |
| Fehler | Es ist zu erwarten, dass einige Folgeseiten von mehrseitigen Einträge eigentlich Parzelleninformationen sind und ausgeschlossen werden müssten. Auf eine händische Durchsicht aller mehrseitigen Einträge wurde verzichtet, da sich der Aufwand nicht rechtfertigen lässt. Insbesondere enthalten Parzelleninformationen kaum Textelemente, die von der Entitäten- oder Eventerkennung fälschlicherweise annotiert würden. Ansonsten ist die Fehlerwarscheinlichkeit bei der Identifizierung von Folgeseiten klein, da die automatisierte Zuweisung von Folgeseiten sehr konservativen Mustern folgte (erste Seite mit Textregion "header", aber ohne "footer" und "credit", folgende Seite(n) ohne "header", aber mit "credit") und viele Folgeseiten händisch identifiziert wurden. |
| Statistik | Anzahl Einträge mit einer Seite: 119'610<br>Anzahl Einträge mit zwei Seiten: 5'611<br>Anzahl Einträge mit drei Seiten: 149<br>Anzahl Einträge mit vier Seiten: 31<br>Anzahl Einträge mit fünf Seiten: 1<br>Anzahl Einträge mit sechs Seiten: 2<br>Anzahl Einträge mit sieben Seiten: 0<br>Anzahl Einträge mit acht Seiten: 1<br>Anzahl Einträge mit mehr als acht Seiten: 0 |

| year |  |
|--------|--------|
| Bedeutung | Dieses Attribut beschreibt die Jahreszahl des Eintrags. |
| Entstehung | Die Jahreszahl wird auf Basis der aktuellster Transkription jeder Seite ermittelt, welche mit dem Eintrag verknüpft ist (Attribut 'Project_Entry.pageId'). Existieren Textregionen des Typs "header" (Attribut 'Transkribus_TextRegion.type'), dann entspricht die Jahreszahl der ersten Übereinstimmung des Musters "1[0-9]{3}". Der Identifikator der Textregion mit der ersten Übereinstimmung des Musters wird im Attribut 'Project_Entry.yearSource' gespeichert. Falls keine Jahreszahl gefunden wird und in einer Textregion der Suchbegriff "Zins" vorkommt (Muster "[Zz][iü]n[n]?s"), dann wird die erste Jahreszahl nach demselben Muster aus den Textregionen "paragraph" übernommen. Annahme: der entsprechende Eintrag stammt aus dem "Zinsverzeichnis".<br>Ausgewählte Jahreszahlen wurden manuell definiert, siehe [4.4.2.2 Entstehung](#4422-entstehung). Manuell definierte Jahreszahlen haben keinen Wert im Attribut 'Project_Entry.yearSource' und "TRUE" im Attribut 'Project_Entry.manuallyCorrected'. |
| Spezialfälle | Einträge ohne Jahreszahl enthalten weitere Informationen im Attribut 'Project_Entry.comment'. |
| Fehler | Dank der Prüfung der Chronologie der Datierungen innerhalb desselben Dossier sind grössere Fehler weitgehend eliminiert, wenn ein Eintrag nicht am Anfang oder am Ende eines Dossiers steht. Kleinere Fehler, welche die chronologische Ordnung nicht verletzen, sind möglich, meist handelt sich dabei um falsche Lesarten der Einer oder Zehner einer Jahreszahl. Bei Dossiers, die Lücken in der Überlieferung aufweisen, kann der Fehler beträchtlich sein, was jedoch eher selten zu erwarten ist. Einträge am Anfang eines Dossiers, die vor dem Jahr 1300 datiert sind, sowie von Dossiers, mit einer Differenz der Jahreszahlen zweier nachfolgenden Einträgen von mehr als 50 Jahren, wurden händisch überprüft. Bei einer Differenz von 50 Jahren sowie gegen das Jahr 1300 sank die Fehlerquote stark, so dass auf weitere Überprüfungen verzichtet wurde. Insgesamt sind kleinere Fehler möglich, Falschdatierungen mit mehreren Jahrzehnten Differenz aber nur sehr selten zu erwarten. |
| Statistik | Anzahl Einträge mit Jahreszahl: 124'348<br>Anzahl Einträge mit manuell definierter Jahreszahl: 9'474<br>Anzahl Einträge ohne Jahreszahl: 1'057<br>minimale Jahreszahl: 1002<br>maximale Jahreszahl: 1936 |

| yearSource |  |
|--------|--------|
| Bedeutung | Identifikator der Textregion (Attribut 'Transkribus_TextRegion.textRegionId'), aus welcher die Jahreszahl (Attribut 'Project_Entry.year') extrahiert worden ist. |
| Entstehung | siehe Entstehung des Attributs 'Project_Entry.year' |
| Spezialfälle | Einträge ohne Quellenangabe für die Jahreszahl wurden entweder manuell definiert oder keine Jahreszahl ist verfügbar. |
| Fehler | - |
| Statistik | Anzahl Einträge mit Quelle der Jahreszahl: 114'874<br>Anzahl Einträge ohne Quelle der Jahreszahl: 10'531 |

| comment |  |
|--------|--------|
| Bedeutung | Kommentar ergänzend zur Jahreszahl des Eintrags. Die meisten Kommentare können einer der folgenden Kategorien zugewiesen werden:<br>- "undatiert": Auf der Karteikarte ist keine Datierung ersichtlich oder als nicht datiert gekennzeichnet.<br>- "ungefähr": Jahreszahl wird im Eintrag ist mit einer Formulierung "um" oder ähnlich versehen. Im Attribut 'Project_Entry.year' wird die Jahreszahl erfasst. Aus dem Kommentar ist ersichtlich, dass diese Datierung nicht präzise ist.<br> - Angabe eines Jahrhunderts, z.B. 15. Jh.: Karteikarte enthaltet statt einer genauen Jahreszahl als Datierung ein Jahrhundert. Das Jahrhundert wird im Kommentar erwähnt.<br>- Korrektur: Wenn die Karteikarte handschriftlich korrigiert wurde, wurde das ursprüngliche Datum in 'Project_Entry.year' erfasst und die Korrektur als Kommentar. |
| Entstehung | Die Kommentare wurden bei der manuellen Bearbeitung von Jahreszahlen erfasst. |
| Spezialfälle | Einträge ohne Jahreszahl erhalten immer eine Bemerkung. |
| Fehler | - |
| Statistik | Anzahl Einträge ohne Kommentar: 123'545<br>Anzahl Einträge mit Kommentar: 1'860<br>Häufigste Kommentare: undatiert (Anzahl: 733), ungefähr (Anzahl: 627) |

| manuallyCorrected |  |
|--------|--------|
| Bedeutung | Wenn der Wert "True" ist, wurde die Jahreszahl (Attribut 'Project_Entry.year') und/oder Folgeseiten manuell definiert oder bearbeitet, siehe auch [4.4.2.2 Entstehung](#4422-entstehung). |
| Entstehung | Mit der Integration manueller Bearbeitungen wird dieses Attribut generiert |
| Spezialfälle | - |
| Fehler | - |
| Statistik | False: 114'444<br>True: 10'961 |

| language |  |
|--------|--------|
| Bedeutung | Dieses Attribut enthaltet die Sprache des Eintrags der Textregionen (Entität 'Transkribus_TextRegion') vom Typ "paragraph". Folgende Werte sind verfügbar:<br>- "german": Die Textregion(en) umfasst wahrscheinlich Text in deutscher Sprache<br>- "latin": Die Textregion(en) umfasst wahrscheinlich Text in lateinischer Sprache<br>- "mixed": Die Textregion(en) umfasst Texte in deutscher und lateinischer Sprache oder die Sprache konnte nicht eindeutig bestimmt werden.|
| Entstehung | Basierend auf den Textregionen vom Typ "paragraph" wird die Sprache mithilfe eines Algorithmus bestummen. Mithilfe der berechneter Konfidenz wurde der Prozess für dieses Projekt optimiert, um die Anzahl falscher Klassierungen in Deutsch und Latein zu minimieren. |
| Spezialfälle | Fünf Einträge mit pageId in {47446933, 47471458, 47506281, 47573580, 47590707} haben keine Sprache, da für diese pageIds keine Textregion des Typs Paragraph existiert. |
| Fehler | - Stichprobe für Klasse "german" (n=200): 1% der Stichprobe ist "mixed" statt "german". Diese Fälle beinhalten mehrheitlich Text in Deutsch.<br>- Stichprobe für Klasse "latin" (n=100): 3% der Stichprobe ist "mixed" statt "latin". Diese Fälle beinhalten mehrheitlich Text in Latein.<br>- Stichprobe für Klasse "mixed" (n=100): 19% der Stichprobe ist "german" statt "mixed", 65% der Stichprobe ist "latin" statt "mixed". Diese Fälle sind alle kurze Zeichenfolgen. |
| Statistik | german: 114'839<br>latin: 8'269<br>mixed: 2'292<br>[NULL]: 5 |

| source |  |
|--------|--------|
| Bedeutung | Dieses Attribut beschreibt, aus welchem Quellenbestand der Eintrag stammt. |
| Entstehung | Die Quellenverweise wurden in der Layout-Erkennung als eigene Textregion erfasst und trainiert. Allerdings stellte sich heraus, dass die Erkennung in rund 15% der Karteikarten gar nicht funktionierte, und auch sonst teilweise fehlerhaft war. Anhand eines Samples von 2'000 erkannten Quellenverweisen (wovon 1'894 einer Institution bzw. einem Quellenbestand zugewiesen werden konnten) ergänzt um 65 mehrzeilige Quellenverweise (als Spezialfall), wurde ein Modell trainiert, das aus dem ganzen Text eines Eintrags den Quellenverweis herausliest und diesen einem Quellenbestand bzw. einer Institution zuweist.  |
| Spezialfälle | 546 Karteikarten sind mehr als einem Quellenbestand zuweisbar. Die Werte sind mit einem Semikolon getrennt. <br>Für Einträge ohne Quellenangabe wurde der Attributwert "fehlt" benutzt.|
| Fehler | Das Modell erwies sich als sehr zuverlässig, eine Stichprobe von 200 zufällig ausgewählten Einträgen ergab keinen einzigen Fehler.|
| Statistik | Anzahl unterschiedliche Quellenangaben: 112 distinke Quellenangaben in 244 unterschiedlichen Nennungen (aufgrund der mehr als einem Quellenbestand zuweisbaren Einträgen)<br>Häufigste Quellenangaben:<br>- Gerichtsarchiv (Anzahl: 41'317)<br>- Notariatsarchiv (9'500)<br>- St. Peter (7'682)<br>Die 10 häufigsten Nennungen machen 73.2% aller Belege aus.|

| sourceOrigin |  |
|--------|--------|
| Bedeutung | Dieses Attribut gibt an, auf welche Art der Quellenbeleg zugewiesen wurde. |
| Entstehung | Die grosse Mehrheit wurde automatisch erkannt. Die Fälle, in der die automatische Erkennung mehrere Quellenverweise zuwies, wurden durchgeschaut und plausibilisiert, daher der Wert "händisch durchgeschaut". Die restlichen Fälle von Einträgen ohne erkanntem Quellenverweis oder erkanntem Quellenverweis ohne zugewiesener Institution wurden händisch ergänzt. Es handelt sich hier einerseits um schwer lesbare Verweise (aufgrund des schlechten Zustandes vieler Karteikarten) und um Verweise auf selten vorkommende Bestände, die im Trainingsmaterial nicht vorkamen und deshalb korrekterweise nicht zugewiesen wurden. |
| Spezialfälle | keine |
| Fehler | keine |
| Statistik | automatisch erkannt: 110'251<br>händisch erfasst: 14'547<br>händisch durchgeschaut: 607 |

| keyLatestTranscript |  |
|--------|--------|
| Bedeutung | Identifikatoren der aktuellster Transkription der zum Eintrag gehörender Seiten (Attribut 'Transkribus_Transcript.key'), gespeichert als Liste. |
| Entstehung | Das Attribut 'Transkribus_Transcript.key' wurde durch die Transkribus-Plattform generiert. |
| Spezialfälle | - |
| Fehler | - |
| Statistik | - |

| annotationManual |  |
|--------|--------|
| Bedeutung | Manuell definierte Annotationen des transkribierten Textes im XML-Format (Ground Truth). |
| Entstehung | Siehe Dokumentation unter https://zenodo.org/records/16919653 |
| Spezialfälle | - |
| Fehler | - |
| Statistik | - |

| annotationAutomated |  |
|--------|--------|
| Bedeutung | Automatisch generierte Annotationen des transkribierten Textes im XML-Format. |
| Entstehung | TODO |
| Spezialfälle | - |
| Fehler | - |
| Statistik | - |


### 4.4.3 Entität Project_Period

#### 4.4.3.1 Bedeutung
TODO

The elements of the Project_Period entity represent the validity period of the dossier that exists in the Project_Dossier entity. A dossier can have several entries or validity periods.

#### 4.4.3.2 Entstehung
TODO

#### 4.4.3.3 Spezialfälle
TODO

#### 4.4.3.4 Fehler
TODO

#### 4.4.3.5 Statistik
TODO

#### 4.4.3.6 Beschreibung der Attribute
TODO

| dossierId |  |
|--------|--------|
| Bedeutung | Identifier to the linked project dossier |
| Entstehung |  |
| Spezialfälle |  |
| Fehler |  |
| Statistik |  |

| yearFrom |  |
|--------|--------|
| Bedeutung | Year from when dossier is valid |
| Entstehung |  |
| Spezialfälle |  |
| Fehler |  |
| Statistik |  |

| yearTo |  |
|--------|--------|
| Bedeutung | Year until when dossier is valid |
| Entstehung |  |
| Spezialfälle |  |
| Fehler |  |
| Statistik |  |

| yearFromManuallyCorrected |  |
|--------|--------|
| Bedeutung | Indication if the attribute yearFrom is manually corrected |
| Entstehung |  |
| Spezialfälle |  |
| Fehler |  |
| Statistik |  |

| yearToManuallyCorrected |  |
|--------|--------|
| Bedeutung | Indication if the attribute yearFrom is manually corrected |
| Entstehung |  |
| Spezialfälle |  |
| Fehler |  |
| Statistik |  |


### 4.4.4 Entität Project_Relationship

#### 4.4.4.1 Bedeutung
Elemente der Entität 'Project_Relationship' repräsentieren "Beziehungen" zwischen Dossiers (Elemente der Entität 'Project_Dossier'). Eine Beziehung besteht zwischen zwei Dossiers, wenn das eine Dossier aus dem anderen Dossier hervorgeht. Beispielsweise existieren zwei Beziehungen, wenn ein Gebäude in zwei Teile geteilt wird. Zeitlich "existiert" zuerst ein Dossier, nach der Teilung existieren zwei Dossiers. Das ursprüngliche Dossier hat somit je eine Beziehung zu den Dossier nach der Teilung.
Das Attribut 'Project_Relationship.sourceDossierId' enthaltet die "dossierId" des Dossiers, von welcher die Beziehung ausgeht ("Quelldossier", "Vorgänger"). Das Attribut 'Project_Relationship.targetDossierId' enthaltet die "dossierId" des Dossiers, welche zeitlich auf das Quelldossier folgt ("Zieldossier", "Nachfolger"). Ein Dossier kann mit einem oder mehreren Quelldossier als auch Zieldossier verknüpft sein.

TODO
This entity maps direct temporal relationships between HGB dossiers (represented as a direct edge list). The relationships were determined on the basis of the cluster information (see dossier_relationship.py) using a rule-based approach and manual editing. Dossier represented by identifier in sourceDossierId has as descendant dossier with identifier in targetDossierId. Conversely, dossier represented by identifier in targetDossierId has dossier with identifier in sourceDossierId as previous dossier. Dossier can have several descendants or preceding dossiers due to a split or merge.

#### 4.4.4.2 Entstehung
TODO Benjamin:
- Kannst du bitte ergänzen, wie der type des Dossiers entstanden ist? -> kommt doch woanders vor...
- Gibt es ein Muster, für welche Dossier die Beziehungen manuell erstellt worden sind? -> Wenn nicht alle Dossiers eines Clusters eine Beziehung aufwiesen, wurde manuell geprüft. Später haben wir auch noch die restlichen Cluster einer manuellen Prüfung unterzogen.

Mithilfe des Skripts 'dossier_relationship.py' wurden Beziehungen ermittelt. Einerseits basierend auf den Adressen (Attribut 'StABS_Dossier.title') sowie des Attributs 'StABS_Dossier.deskriptiveNote'. Andererseits basierend auf ermittelte Cluster und der Art des Dossiers. Mit dem Algorithmus konnten viele Beziehungen nicht erkannt werden. Aus diesem Grund wurden anschliessend Beziehungen manuell ermittelt.

#### 4.4.4.3 Spezialfälle
TODO Benjamin: Existieren Spezialfälle? -> Wenn ein Dossier ab einem bestimmten Zeitpunkt (oder bis zu einem bestimmten Zeitpunkt) bei einem anderen Dossier eingeschlossen war, lässt sich zwar die Beziehung korrekt bestimmen, aber es ist dann eigentlich nicht korrekt, von einem Folgedossier auszugehen (in dem Sinne, dass ein Dossier endet und danach ein anderes beginnt, also keine Überschneidungen bestehen). Deshalb wurden diese Beziehungen separat erfasst.

#### 4.4.4.4 Fehler
TODO Benjamin: Welche Aussage können wir über Fehler machen? -> Die automatisierte Erkennung war durchaus fehlerhaft, weshalb schlussendlich alle Beziehungen manuell angeschaut wurden. Die Metadaten des HGB sind aber in vielen Fällen nicht eindeutig, weshalb diese Beziehungen gar nicht ganz korrekt erfasst werden können. Eine gewisse Fehlerquote ist also inhärent.

Nachricht von Benjamin 23.1.2025 (Ausschnitt): Insgesamt sind diese Dossierbeziehungen mit zweierlei Unsicherheit behaftet: Wir können die HGB-Entscheide oft nicht nachvollziehen, und sie scheinen auch von unterschiedlicher Qualität zu sein. Und wir machen selbst weitere Fehler (wo das HGB eigentlich verständlich wäre). Weil das Ziel bloss eine ungefähre Abschätzung der Anzahl Dossiers zu einem Zeitpunkt war, würde ich hier nicht mehr allzu viel investieren. 

#### 4.4.4.5 Statistik
Anzahl Dossier mit
- einem Nachfolger: 1'161
- zwei Nachfolger: 388
- drei Nachfolger: 54
- vier Nachfolger: 11
- fünf Nachfolger: 4
- sechs Nachfolger: 1
- sieben Nachfolger: 1

Anzahl Dossier mit
- einem Vorgänger: 902
- zwei Vorgänger: 467
- drei Vorgänger: 77
- vier Vorgänger: 13
- fünf Vorgänger: 6
- sechs Vorgänger: 3
- sieben Vorgänger: 0
- acht Vorgänger: 0
- neun Vorgänger: 1


# 5 Glossar
TODO: prüfen, ob einzelne Einträge überflüssig sind oder weitere in der Tabelle ergänzt werden müssen.

| Bezeichnung | Beschreibung |
|--------|--------|
| Attribut | Eine Information oder Eigenschaft eines Eintrages einer Entität. In einer relationalen Datenbank entspricht eine Spalte einem Attribut. |
| CER | Character Error Rate, Zeichenfehlerrate, siehe Neudecker, Clemens; Baierer, Konstantin; Gerber, Mike u. a.: A survey of OCR evaluation tools and metrics, in: The 6th International Workshop on Historical Document Imaging and Processing, Lausanne Switzerland 2021, S. 13–18. Online: <https://doi.org/10.1145/3476887.3476888>. |
| Entität | Ein Objekt in der Projektdatenbank, welches in einer Datenbank durch eine Tabelle repräsentiert wird. |
| Falknerplan | Historischer Grundbuchplan der Basler Innenstadt. Erstellt durch Geometer Rudolf Falkner in den Jahren 1865-1872.<br><br> Weitere Informationen: https://www.gva.bs.ch/vermessung/historische-plaene/georeferenzierte-plaene.html |
| HGB | Historisches Grundbuch des Stadt Basel.<br><br> Weitere Informationen: https://www.staatsarchiv.bs.ch/benutzung/recherche/suche-gedruckte-kataloge/historisches-grundbuch.html |
| HTR | Handwritten Text Recognition (oder Automatic Text Recognition). Automatisierte Erkennung von textuellen Elementen auf Basis von segmentierten Linien.<br>Basierend auf *machine learning* Algorithmen |
| P2PaLA | Pixelannotation auf Trainingsbasis (Computer Vision Algorithmus). Entwickelt durch die Technische Universität Valencia (im Rahmen von READ).<br><br> Weitere Informationen: https://readcoop.eu/transkribus/docu/p2pala/ |
| Projektdatenbank | Relationale Datenbank (PostgreSQL mit PostGIS-Erweiterung) zur Speicherung und Analyse der für das Projekt relevanten Informationen des HGBs.<br><br> Weitere Informationen: https://github.com/history-unibas/Postgresql-Project-Database/tree/main#postgresql-project-database |
| Shape-Datei | Dateiformat zur Speicherung und Austausch von Daten mit geografischer Information. |
| SPARQL | SPARQL Protocol and RDF Query Language: Sprache für die Abfrage von Linked Open Data.<br><br> Weitere Informationen: https://www.opendata.bs.ch/ld.html |
| SQL-Abfrage | Structured Query Language (SQL) ist eine Sprache, um Daten in einer Datenbank aufzurufen oder bearbeiten. Eine Abfrage ist eine (konkrete) Aussage, welche auf einer Datenbank ausgeführt werden kann. |
| Transkribus Plattform | Digitale Plattform zur Erkennung von Dokumenten. Geführt durch die READ COOP.<br><br> Weitere Informationen: https://readcoop.eu/transkribus/ |
| WER | Word Error Rate: Wortfehlerrate, siehe Neudecker, Clemens; Baierer, Konstantin; Gerber, Mike u. a.: A survey of OCR evaluation tools and metrics, in: The 6th International Workshop on Historical Document Imaging and Processing, Lausanne Switzerland 2021, S. 13–18. Online: <https://doi.org/10.1145/3476887.3476888>. |
