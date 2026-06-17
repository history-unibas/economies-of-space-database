Dokumentation Projektdatenbank
==============


# 1 Einleitung
Dieses Dokument beschreibt die Entwicklung der relationalen "Projektdatenbank", welche im Rahmen es Projekts "Ökonomien des Raums" zwischen 2022 und 2026 entstanden ist (https://dg.philhist.unibas.ch/de/bereiche/mittelalter/forschung/oekonomien-des-raums/).

Datengrundlage ist das "Historische Grundbuch der Stadt Basel" (HGB). Das HGB umfasst Auszüge aus Archivdokumenten zu Liegenschaften der Basler Altstadt in einer strukturierten Form. Die Einträge reichen von 11. Jahrhundert bis ins 19. Jahrhundert. Ziel bei der Erstellung des HGB war eine möglichst vollständige Sammlung verfügbarer Informationen der Liegenschaften. Insbesondere Angaben zu Verkäufen, Frönungen und Zinszahlungen sind im HGB dokumentiert. Das HGB wurde durch das Staatsarchiv Basel-Stadt digitalisiert und online zur Verfügung gestellt. Die Digitalisate werden durch detailreiche Metadaten bereichert (https://blog.staatsarchiv-bs.ch/neu-online-einsehbar-das-historische-grundbuch/).

Die vorliegenden Daten ermöglichen eine detaillierte raum-zeitliche Analyse der Basler Altstadt vom 11. bis ins 19. Jahrhundert. Da der Datensatz umfassende Informationen über die historischen Akteure dieser Liegenschaften enthält, lassen sich spezifische Muster und Dynamiken auf dem damaligen Immobilienmarkt präzise untersuchen. Auf dieser soliden Datenbasis können in der Folge konkrete, eigene Fragestellungen und theoretische Ansätze fundiert bearbeitet werden.

TODO Verweis auf publizierte Datenbank (Zenodo) folgt

Basierend auf der relationalen Projektdatenbank wird einen Graph entwickelt, so dass diese Daten ebenfalls als Linked Open Data (LOD) zur Verfügung stehen. Weitere Informationen zu diesem Prozess sind im Repository [economies-of-space-lod](https://github.com/history-unibas/economies-of-space-lod) zu finden. 

Diese Dokumentation ist wie folgt aufgebaut: Kapitel [2 Systemarchitektur](#2-systemarchitektur) beschreibt die für die Erstellung der Datenbank verwendeten Daten. Einen Überblick über das Datenmodell liefert Kapitel [3 Modellbeschreibung](#3-modellbeschreibung). Das Kapitel [4 Prozesse](#4-prozesse) beschreibt im Detail die Datenaufbereitung sowie die nun verfügbaren Daten. Schliesslich beschreibt Kapitel [5 Glossar](#5-glossar) im Dokument verwendete Begiffe.


# 2 Systemarchitektur
Für die Befüllung der Projektdatenbank werden Daten aus unterschiedlichen Quellen verarbeitet. Die folgende Grafik stellt der Datenfluss der benutzten Daten dar.

![Systemarchitektur](system_architecture.svg)

Folgende Stellen und Hilfsmittel sind am Datenfluss beteiligt:
- Das Staatsarchiv stellt Digitalisate sowie Metadaten des historischen Grundbuchs Basel (HGB) zur Verfügung.
- Auf der Transkribus Plattform wird das Layout analysiert und semantisch angereichert (Identifikation des Typs einer Textregion, beispielsweise Datumszeile, Quellenverweis oder Haupttext) und Texte automatisiert erkannt.
- Das Grundbuch- und Vermessungsamt des Kantons Basel-Stadt ermittelt auf der Basis der Adresse pro HGB-Dossier einen geographischen Standort, insofern dies ermittelbar ist. 
- In der Projektdatenbank werden die für die Forschung zentralen Informationen gespeichert und zur Verfügung gestellt. Weitere Details zur Datenbank sind im nachfolgenden Kapitel [3 Modellbeschreibung](#3-modellbeschreibung) dokumentiert.

Zentrale Prozesse des Datenflusses sind in Kapitel [4 Prozesse](#4-prozesse) beschrieben.

Für den gesamten Datenfluss ist die Replizierbarkeit sichergestellt, da alle Prozesse über Skripte angestossen werden (u.a. mit Zugriff auf die API von Transkribus). Insbesondere wird die Projektdatenbank mit einem Skript [project_database_update.py](https://github.com/history-unibas/Postgresql-Project-Database/blob/main/project_database_update.py) erstellt, befüllt und aktualisiert .


# 3 Modellbeschreibung
Die Projektdatenbank vereint Daten, die im Aufbereitungsprozess erstellt werden (aus Texterkennung und Layoutanalyse sowie folgenden Schritten) mit bereitgestellten Metadaten aus der archivischen Beschreibung (erarbeitet durch das Staatsarchiv Basel-Stadt).
Die Datenbank ist entsprechend sowohl hierarchisch aufgebaut (Abfragen von der Transkribus-Plattform beeinflussen darauffolgende Schritte) als auch eine Kombination mit Datenabfragen aus SPARQL.
Die Projektdatenbank ist unterteilt in folgende Gruppen von Entitäten:
- **StABS**: Die Entitäten enthalten Metadaten des HGB, basierend auf Daten des [Linked Open Data Portals](https://ld.bs.ch/).
- **Transkribus**: Entitäten dieser Gruppe enthalten ausgewählte Daten der Plattform [Transkribus](https://readcoop.eu/transkribus/), insbesondere Transkriptionen ausgewählter Digitalisate sowie Informationen zur Kategorisierung visueller Einheiten (Textregionen, die als "Paragraph"/"Quellenangabe"/"Titulatur" etc. definiert werden).
- **Geo**: Daten mit geographischer Informationen werden in Entitäten dieser Gruppe gespeichert.
- **Project**: Entitäten enthalten automatisiert als auch händisch aufbereitete Daten basierend auf Daten aus den anderen Gruppen.

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
| [Project_Entry](#442-entität-project_entry) | Aufbereitete Informationen zu HGB-Einträge | 125'405 |
| [Project_Period](#443-entität-project_period) | Gültigkeitszeiträume der Dossier | 4'724 |
| [Project_Relationship](#444-entität-project_relationship) | Beziehungen zwischen Dossier | 2'176 |

Die folgende Grafik zeigt die zentralen Beziehungen zwischen den Entitäten. Lesebeispiel für die Beziehung zwischen StABS_Serie und StABS_Dossier: Ein Element der Entität StABS_Serie ist verbunden zu einem oder mehreren Elementen der Entität StABS_Dossier. Umgekehrt, ein Element der Entität StABS_Dossier ist mit genau einem Element der Entität StABS_Serie verbunden.

![Beziehungen](entityRelations.drawio.svg)

Eine formale Beschreibung der Entitäten in der Datenbank ist im [Readme](https://github.com/history-unibas/Postgresql-Project-Database/blob/main/README.md#database-schema) des Github-Repositories dokumentiert. 


# 4 Prozesse
Dieses Kapitel dokumentiert pro Gruppe von Entitäten die Aufbereitung der Daten aus dem HGB:
- Gruppe StABS: [4.1 HGB-Metadaten in Projektdatenbank integrieren](#41-hgb-metadaten-in-projektdatenbank-integrieren)
- Gruppe Transkribus: [4.2 Verarbeitung der Digitalisate](#42-verarbeitung-der-digitalisate)
- Gruppe Geo: [4.3 HGB-Dossier georeferenzieren](#43-hgb-dossier-georeferenzieren)
- Gruppe Project: [4.4 Anreicherung von Daten](#44-anreicherung-von-daten)


## 4.1 HGB-Metadaten in Projektdatenbank integrieren
Das Staatsarchiv Basel-Stadt (StABS) stellt via des [Linked Open Data Portals](https://ld.bs.ch/) Metadaten zum HGB zur Verfügung. Mithilfe eines Skripts [project_database_update.py](https://github.com/history-unibas/Postgresql-Project-Database/blob/main/project_database_update.py) werden Metadaten des HGB gelesen und in der Projektdatenbank als Entitäten [StABS_Serie](#411-entität-stabs_serie), [StABS_Dossier](#412-entität-stabs_dossier) und [StABS_Page](#413-entität-stabs_page) gespeichert. In diesem Prozess werden Funktionen aus dem Repository [
economies-of-space-metadata
](https://github.com/history-unibas/economies-of-space-metadata) verwendet.

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
Mithilfe der Plattform [Transkribus](https://readcoop.eu/transkribus/) wurden handgeschriebene Texte von ausgewählten Seiten des HGB transkribiert. Die Transkription sowie ausgewählte Metadaten von Transkribus werden anschliessend in der Projektdatenbank gespeichert. Nachfolgende Schritte wurden in diesem Prozess ausgeführt. Für die Schritte 2 bis 4 wurde das Skrript [collection_transcription.py](https://github.com/history-unibas/economies-of-space-transkribus/blob/main/collection_transcription.py) entwickelt.

**1. Schritt: Auswahl der Digitalisate**

Folgende Seiten wurden für die Transkription (Schritte 2 bis 4) ausgeschlossen:
- Titelseiten, das heisst Seitenzahlen 1 und 2
- Seiten zugehörig zu Dossier, welche basierend auf dem Titel (Attribut 'StABS_Dossier_title') als "Personenregister" oder "Situationsplan" identifiziert wurden
- Brandlagerbücher, welche alle aus dem 19. Jahrhundert stammen
- Karteikarten des Reichspfennigs ("Reichspfennigverzeichnisse") von 1497, da auf diesen keine Transaktion festzustellen ist

Die Brandlagerbücher und Reichspfennigverzeichnisse wurden mit einem Bilder-Chlustering [PixPlot](https://github.com/YaleDHLab/pix-plot) identifiziert und mit dem Skript [clusterImages.py](https://github.com/history-unibas/economies-of-space-pix-plot/blob/main/clusterImages.py) validiert. Anschliessend wurde dem Skript [status_change.py](https://github.com/history-unibas/economies-of-space-transkribus/blob/main/status_change.py) der Status der identifizierten Seiten auf "DONE" gesetzt. Seiten mit dem Status "DONE" wurden von der Transkription ausgeschlossen.

**2. Schritt: Erkennen von Textregionen**

- Methode: P2PaLA
- Modell: HGB_M3
    - Structure Types = {Credit, Footer, Header, Marginalia, Paragraph}
    - Min-Error = 0.10
    - N-Train = 1131 (N bezieht sich auf die Anzahl vollständig annotierter Registerkarten)
    - N-Validation = 128 (N bezieht sich auf die Anzahl vollständig annotierter Registerkarten)
    - Min-Error = 0.1044
- usgewählter Parameter: Rectify regions = True

**3. Schritt: Finden von Textlinien**

- Methode: Transkribus Layout Analysis (LA)
- Modell: HGB_Baseline_M1
    - Technology = Baselines
    - nrOfWords = 72'291
    - CER Train = 4.31%
    - CER Validation = 3.79%
- Ausgewählte Parameter
    - Find Text-Regions = False (Textregionen sind bereits durch P2PaLA identifiziert und semantisch angereichter)
    - Find Lines = True
    - Split lines on regions = False 

Teilweise überlappen sich Textregionen, insbesondere bei den Quellennachweisen, weshalb das Splitting problematisch wäre. Bei diesem Modell handelt es sich um ein eigens trainiertes Baseline Model.

**4. Schritt: Texterkennung**

- Methode: HTR (PyLaia)
- Modell: HGB_FT_M5.2 
    - Language = German
    - Technology = PyLaia HTR
    - nrOfWords = 101'992
    - CER Train = 2.50%
    - CER Validation = 4.00%
- Ausgewählte Parameter
    - Language Model = 'Language model from training data'
    - Compute line polygons = True

Das benutzte HTR-Modell wurde nach dem Training mittels Skript [htr_model_validation.py](https://github.com/history-unibas/economies-of-space-transkribus/blob/main/htr_model_validation.py) validiert. Ziel der unterschiedlichen Evaluation ist nachzuweisen, dass zentrale Textregionen (Paragraph und Titel, jedoch nicht Marginalie) zuverlässig erkannt werden. Hintergrund ist, dass häufig Teile falsch/inkorrekt erkannt werden, die von niederer Wichtigkeit sind (insbesondere Marginalien, die Metadaten enthalten, die bereits bekannt sind). Die folgende Tabelle dokumentiert die CER- und WER-Werte pro Textregionsart sowie über alle Textregionen.

| Art der Textregion | CER | WER |
|--------|--------|--------|
| global (alle Textregionen) | 0.042 | 0.173 |
| marginalia | 0.114 | 0.362 |
| header | 0.079 | 0.261 |
| paragraph | 0.036 | 0.160 |
| credit | 0.074 | 0.385 |
| footer | 0.066 | 0.194 |

**5. Schritt: Integration in Projektdatenbank**

Mithilfe der Skripts [project_database_update.py](https://github.com/history-unibas/economies-of-space-database/blob/main/project_database_update.py) und [connect_transkribus.py](https://github.com/history-unibas/economies-of-space-transkribus/blob/main/connect_transkribus.py) wurde der transkribierte Text sowie ausgewählte Metadaten von Transkribus in die Projektdatenbank in Entitäten mit Präfix "Transkribus" übernommen.


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
Es wurden nur ausgewählte Seiten transkribiert. Details siehe Kapitel [4.2 Verarbeitung der Digitalisate, "1. Schritt: Auswahl der Digitalisate"](#42-verarbeitung-der-digitalisate).

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
| htrModel | Für die durchgeführte Trankription oder Bearbeitung der Seite verwendetes Modell. Details zu den verwendeten Modellen siehe [4.2 Verarbeitung der Digitalisate, Schritte 2-4](#42-verarbeitung-der-digitalisate) |

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
- **Daten in Projektdatenbank integrieren**: Mithilfe des Skripts [project_database_update.py](https://github.com/history-unibas/Postgresql-Project-Database/blob/main/project_database_update.py) werden die Daten der Shape-Datei gelesen und in der Projektdatenbank gespeichert.

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

Auf inhaltlicher Ebene erfolgte eine automatisierte Annotation durch Maschine-Learning-Modelle. Diese Annotationen erfassen verschachtelte Entitäten, Ereignisse und Beziehungen, wodurch die komplexen Interaktionen zwischen den verschiedenen Akteuren und Eigenschaften präzise widergespiegelt werden. Die annotierten Daten sind im XML-Format im Attribut 'Project_Entry.annotationAutomated' gespeichert.

### 4.4.1 Entität Project_Dossier

#### 4.4.1.1 Bedeutung
Jedes Element dieser Entität ("Dossier") repräsentiert ein Dossier im Historisches Grundbuch der Stadt Basel (HGB). Alle Dossier sind ebenfalls in der Entität 'StABS_Dossier' abgebildet.

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
| Entstehung | Die Standorte basierend mehrheitlich auf Daten des Grundbuch- und Vermessungsamt des Kantons Basel-Stadt (Entität Geo_Address). Für ausgewählte Dossier sowie für Dossier ohne Standort in der Entität Geo_Address wurde mithilfe eines Skripts Standorte ermittelt basierend auf vorhandenen Standorte (Entität Geo_Address). Um Fehler zu reduzieren und den Datensatz zu vervollständigen, wurden anschliessend Standorte von Dossier manuell geprüft und gesetzt basierend auf dem [Löffelplan](https://www.bs.ch/bvd/grundbuch-und-vermessungsamt/vermessung/historische-plaene) des Jahres 1862. Manuell definiert oder kontrolliert wurden Standorte von Dossier, welche nach der Anwendung des Skripts keinen Standort besassen, sich innerhalb von 20 Meter kein weiteres Dossier derselben Strasse befindet aufgrund des Verdachtes auf einen fehlerhaften Standort, alle Dossier der Hebelstrasse aufgrund fraglicher Geolokalisierung sowie einzelne bei der Durchsicht aufgefallene Dossier. Die Herkunft jedes Dossier-Standortes kann dem Attribut 'Project_Dossier.locationUncorrectedOrigin' entnommen werden. Unterschiedliche Standorte, welche sich weniger als einen Meter voneinander entfernt befinden, wurden harmonisiert. |
| Spezialfälle | 6 Dossier haben keinen definierten Standort (locationUncorrectedAccuracy='nicht lokalisierbar') und beinhalten Informationen zu Gewässer, Teich oder Mauer (siehe Attribut 'Project_Dossier.specialType'). |
| Fehler | Für Dossier, dessen Standort manuell festgelegt wurde, beinhaltet das Attribut 'Project_Dossier.locationUncorrectedAccuracy' eine Aussage über die Genaugikeit des Standortes. Für Standorte, welche vom Grundbuch- und Vermessungsamt übernommen worden sind, können wir keine Angabe machen (locationUncorrectedAccuracy='unbekannt').<br>Bei der Entwicklung des Attributs 'Project_Dossier.location' wurden manuell die Standorte einzelner Dossier verbessert. Diese Korrekturen wurden nicht im Attribut 'Project_Dossier.locationUncorrected' umgesetzt. |
| Statistik | - |

| location |  |
|--------|--------|
| Bedeutung | Dieses Attribut beinhaltet den geschobenen geografischen Standort des Dossiers im Koordinatensystem LV95 (EPSG:2056). Im Vergleich zum Attribut 'Project_Dossier.locationUncorrected' wurden der Standort für ausgewählte Dossier verschoben mit dem Ziel, dass weniger Dossier denselben Standort besitzen und damit die Darstellung der Dossier auf einer Karte zu verbessern. |
| Entstehung | Der geschobene Standort basiert auf dem Attribut 'Project_Dossier.locationUncorrected'. Mithilfe eines Algorithmus wurde für Dossier, in welchen im Titel (Attribut 'StABS_Dossier.title') "neben" erwähnt ist, ein entsprechendes Dossier gesucht. Ist ein "neben-Dossier" verfügbar, wurde der Standort um 1/4 der Distanz in Richtung des neben-Dossier verschoben. Beispiel: "St. Alban-Vorstadt Theil von 17 neben 15" (Dossier: HGB_1_010_041, neben-Dossier: "St. Alban-Vorstadt 15", HGB_1_010_039). Die Distanz von einem Viertel ist willkürlich gewählt, muss aber deutlich weniger als die Hälfte sein, weil sonst «Teil von 15 neben 17» auf den gleichen Punkt zu liegen käme. Für Dossier, welche im Titel mehrere Adressen umfassen ("verbundene Dossier"), wurden entsprechende Dossier gesucht, welche je eine Adresse abbildet. Wurden entsprechende Dossier gefunden, ist als verschobener Standort des verbundenen Dossiers der geometrischer Schwerpunkt der entsprechenden Dossier definiert worden. Beispiel: "St. Alban-Graben 8, 10" (Dossier: HGB_1_005_020, entsprechende Dossier: "St. Alban-Graben 8", HGB_1_005_019 und "St. Alban-Graben 10", HGB_1_005_021).<br>Ausgewählte Dossier wurden anschliessend manuell geprüft und verschoben basierend auf dem [Löffelplan](https://www.bs.ch/bvd/grundbuch-und-vermessungsamt/vermessung/historische-plaene) des Jahres 1862. Manuell geprüft wurden Standorte von Dossier, welche mit dem Skript verschoben worden sind sowie Dossier, welche aufgrund des Titels (Attribut 'StABS_Dossier.title') eine Verschiebung erwartet wurde. Insbesondere wurde die Unterscheidung in Vorder- und Hinterhäuser zusätzlich getroffen, wenn sie ersichtlich war. Bei der abschliessenden manuellen Durchsicht sind viele Dossiers aufgefallen, die nicht ganz korrekt lokalisiert waren. Händische Korrekturen erfolgten vornehmlich (aber nicht nur) dort, wo die Strassen um 1862 stark vom früheren Strassenbild abwichen (Beispiele: Eisengasse, untere Freie Strasse, Fischmarkt). In diesen Fällen wurden die im HGB hinterlegten Planzeichnungen konsultiert. Im Attribut 'Project_Dossier.locationOrigin' ist festgehalten, welche Dossier mit dem Algorithmus respektive manuell verschoben worden sind. Unterschiedliche Standorte, welche sich weniger als einen Meter voneinander entfernt befinden, wurden harmonisiert. |
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
| Fehler | Bei der visuellen Kontrolle des Attributs 'StABS_Dossier.title' von Dossier ohne Zugehörigkeit zu einem Cluster (Skript [dossier_relationship.py](https://github.com/history-unibas/economies-of-space-database/blob/main/dossier_relationship.py)) und ohne Wert im Attribut 'Project_Dossier.specialType' wurden keine weiteren spezielle Dossier gefunden. In dieser Teilmenge existieren jedoch Dossier mit einem Strassennamen aber ohne Hausnummer, beispielsweise in der Freien Strasse. Diese Dossier besitzen teilweise eine alte Hausnummer (Attribut 'StABS_Dossier.oldhousenumber'), es könnte jedoch auch einen Hinweis sein auf weitere "Spezial-Dossier", welche in diesem Attribut nicht berücksichtigt sind. |
| Statistik | [NULL]: 3'937<br>Unbestimmte Liegenschaften: 98 <br>Strassenkörper: 92<br>Brunnen: 50<br>Dohle: 41<br>Unbestimmt: 27<br>Gewässer: 16<br>Keller: 12<br>Garten: 10<br>Laden: 9<br>Sammeldossier: 9<br>Boden: 5<br>Graben: 5<br>Zins: 5<br>Brotbank: 4<br>Bank: 3<br>Graben, Mauer: 3<br>Nachträge: 3<br>Teich: 3<br>Mauer: 2<br>Quartier: 2<br>Tor: 2<br>Abort: 1<br>Allmend : 1<br>Brunnen, Unbestimmte Liegenschaften: 1<br>Brücke: 1<br>Halseisen, Heisserstein: 1<br>Häuserverzeichnis: 1<br>Platz: 1<br>Salzkasten: 1 <br>Schutzrein: 1|


### 4.4.2 Entität Project_Entry

#### 4.4.2.1 Bedeutung
Jedes Element dieser Entität ("Eintrag") repräsentiert einen im HGB erfassten Eintrag. Es können mehrere Einträge auf einer Registerkarte ("Seite") des HGB dokumentiert sein, oder ein Eintrag kann sich über mehrere Seiten erstrecken. Eine Seite im HGB wird durch ein Element in den Entitäten 'StABS_Page' respektive 'Transkribus_Page' repräsentiert. Befinden sich mehrere Einträge auf einer Registerkarte, so werden diese Einträge nicht durch mehrere Elemente in dieser Entität repräsentiert. Eine Seite wird als Folgeseite eines Eintrags betrachtet, wenn die Seite keine Textregion (Entität 'Transkribus_TextRegion') des Typs "header" und "marginalia" besitzt und die vorhergehende Seite keine Textregion des Typs "credit" hat. Einen Eintrag basiert nur auf Seiten desselben Dossiers.

#### 4.4.2.2 Entstehung
Es werden Seiten des HGB betrachtet, welche transkribiert worden sind (Kapitel [4.2 Verarbeitung der Digitalisate](#42-verarbeitung-der-digitalisate)). Für die Generierung der Einträge wird das aktuellste Transkript einer Seite verwendet (Attribut 'Transkribus_Transcript.timestamp'). Hat das aktuellste Transkript den Status "DONE" (Attribut 'Transkribus_Transcript.status'), wird diese Seite nicht berücksichtigt. Ebenfalls werden Seiten ausgeschlossen, welche einem Dossier verbunden ist, welches sich ausserhalb der Stadtmauern befindet. Seiten ohne Textregionen (Entität 'Transkribus_TextRegion') sind ebenfalls ausgenommen. Manuell wurden 3'202 Seiten für die Generierung dieser Entität ausgeschlossen. Darunter fallen insbesondere Karteikarten, die Parzelleninformationen enthalten (Angaben zu zeitweise zusammengelegte Dossiers, oder auch Angaben zur Baugeschichte wie gefundene Jahreszahlen). Identifiziert wurden diese Seiten hauptsächlich bei der händischen Datierung (siehe nächster Abschnitt) oder der händischen Zuweisung von Quellenbelegen von Einträgen (Attribut 'Project_Entry.source').

Manuell wurden Einträge bearbeitet, namentlich die Jahreszahl (Attribut 'Project_Entry.year') und die Definition von Seiten als Folgeseiten. Alle undatierte Einträge wurden händisch durchgesehen. Basierend auf einer Analyse (Skript [year_analysis.py](https://github.com/history-unibas/economies-of-space-database/blob/main/year_analysis.py)) unter der Annahme, dass Einträge chronologisch in einem Dossier abgelegt sind, wurden weitere Jahreszahlen überprüft. Bei weiteren Analysen und Prüfungen wurden weitere Datierungen überprüft und korrigiert. Manuell bearbeitete Einträge haben im Attribut 'Project_Entry.manuallyCorrected' den Wert "True". Weitere Angaben zur Jahreszahl enthaltet das Attribut 'Project_Entry.comment', insbesondere für Einträge ohne eine Jahreszahl.

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
| Bedeutung | Manuell definierte Annotationen des transkribierten Textes im XML-Format (Ground Truth). Diese Daten sind ebenfalls publiziert auf [Hugging Face](https://huggingface.co/datasets/dh-unibe/image-text_historisches-grundbuch-basel_xix-xx_train). |
| Entstehung | Siehe Dokumentation unter [Zenodo](https://zenodo.org/records/16919653) |
| Spezialfälle | - |
| Fehler | - |
| Statistik | Anzahl Einträge ohne XML: 124'576<br>Anzahl Einträge mit XML: 829 |

| annotationAutomated |  |
|--------|--------|
| Bedeutung | Automatisch generierte Annotationen des transkribierten Textes im XML-Format. |
| Entstehung | Es wurden Einträge annotiert, welche eine Datierung im Zeitraum 1400 - 1700 besitzen (Attribut 'project_entry.year') und in deutscher Spracher verfasst sind (Attribut 'project_entry.language'). TODO Link für weitere Informationen folgt |
| Spezialfälle | - |
| Fehler | - |
| Statistik | Anzahl Einträge mit XML: 75'447<br>Anzahl Einträge ohne XML: 49'958 |


### 4.4.3 Entität Project_Period

#### 4.4.3.1 Bedeutung
Elemente der Entität 'Project_Period' repräsentieren "Gültigkeitszeiträume" von Dossiers (Elemente der Entität 'Project_Dossier') an. Mit dieser Angabe ist es möglich zu bestimmen, wie viele Dossier in einem bestimmten Jahr existieren sowie Dossiers über die Zeit zu visualisieren. Ein Dossier kann mehrere Gültigkeitszeiträume besitzen. Es sind nur Dossier abgebildet, welche einen Eintrag in der Entität 'Project_Dossier' besitzen.

#### 4.4.3.2 Entstehung
Basierend auf dem Attribut 'StABS_Dossier.descriptiveNote' werden Gültigkeitszeiträume (Attribute 'Project_Period.{yearFrom,yearTo}') für die häufigsten Muster gesucht. Wurde keine Jahreszahl gefunden, wird die minimale respektive maximale Jahreszahl der diesem Dossier zugehörigen Einträge (Attribut 'Project_Entry.year') verwendet. Basierend auf dem Skripts [year_analysis.py](https://github.com/history-unibas/economies-of-space-database/blob/main/year_analysis.py) und [dossier_validity_range.py](https://github.com/history-unibas/economies-of-space-database/blob/main/dossier_validity_range.py)
 und weiteren Analysen wurden Jahreszahlen manuell korrigiert und Gültigkeitszeiträume definiert.

#### 4.4.3.3 Spezialfälle
Bei der manuellen Bearbeitung wurde unter Berücksichtigung der Beziehungen (Entität 'Project_Relationship') darauf geachtet, dass zeitlich direkt nachfolgende Dossier keine Lücke im Gültigkeitszeitraum existieren und umgekehrt, dass nachfolgende Dossier nicht zum selben Zeitpunkt gültig sind. Wenn eine zeitliche Lücke zwischen Vorgänger und Nachfolger-Dossier existiert, wurde das Attribut 'Project_Period.yearTo' des Vorgängers definiert als 'Project_Period.yearFrom' des Nachfolgers (Zeitraum des Vorgängers ist grösser definiert als Daten vorhanden sind). 

#### 4.4.3.4 Beschreibung der Attribute

| dossierId |  |
|--------|--------|
| Bedeutung | Identifikator des zugehörigen Dossier (Attribut 'Project_Dossier.dossierId') |
| Entstehung | Direkt abgeleitet von 'StABS_Dossier.stabsId' |
| Spezialfälle | - |
| Fehler | - |
| Statistik | - |

| yearFrom |  |
|--------|--------|
| Bedeutung | Jahreszahl, ab welchem Zeitpunkt das Dossier existiert |
| Entstehung | siehe [4.4.3.2 Entstehung](#4432-entstehung) |
| Spezialfälle | In Einzelfällen können Jahreszahlen nicht definiert werden |
| Fehler | Fehler sind aufgrund falsch definierter Beziehungen zu erwarten, siehe [4.4.4.4 Fehler](#4444-fehler). |
| Statistik | Anzahl Werte mit Jahreszahl: 4'695<br>Anzahl Werte ohne Jahreszahl ([NULL]): 29 |

| yearTo |  |
|--------|--------|
| Bedeutung | Jahreszahl, bis zu welchem Zeitpunkt das Dossier existiert |
| Entstehung | siehe [4.4.3.2 Entstehung](#4432-entstehung) |
| Spezialfälle | In Einzelfällen können Jahreszahlen nicht definiert werden |
| Fehler | Fehler sind aufgrund falsch definierter Beziehungen zu erwarten, siehe [4.4.4.4 Fehler](#4444-fehler). |
| Statistik | Anzahl Werte mit Jahreszahl: 4'701<br>Anzahl Werte ohne Jahreszahl ([NULL]): 23 |

| yearFromManuallyCorrected |  |
|--------|--------|
| Bedeutung | Angabe, ob das Attribut 'Project_Period.yearFrom' manuell korrigiert wurde |
| Entstehung | Bei der Generierung der Entität wird der Wert auf "True" gesetzt, wenn die Jahreszahl manuell definiert worden ist. |
| Spezialfälle | - |
| Fehler | - |
| Statistik | False: 3'823<br>True: 901 |

| yearToManuallyCorrected |  |
|--------|--------|
| Bedeutung | Angabe, ob das Attribut 'Project_Period.yearTo' manuell korrigiert wurde |
| Entstehung | Bei der Generierung der Entität wird der Wert auf "True" gesetzt, wenn die Jahreszahl manuell definiert worden ist. |
| Spezialfälle | - |
| Fehler | - |
| Statistik | False: 3'706<br>True: 1'018 |


### 4.4.4 Entität Project_Relationship

#### 4.4.4.1 Bedeutung
Elemente der Entität 'Project_Relationship' repräsentieren "Beziehungen" zwischen Dossiers (Elemente der Entität 'Project_Dossier'). Eine Beziehung besteht zwischen zwei Dossiers, wenn das eine Dossier aus dem anderen Dossier hervorgeht. Beispielsweise existieren zwei Beziehungen, wenn ein Gebäude in zwei Teile geteilt wird. Zeitlich "existiert" zuerst ein Dossier, nach der Teilung existieren zwei Dossiers. Das ursprüngliche Dossier hat somit je eine Beziehung zu den Dossier nach der Teilung. Das Attribut 'Project_Relationship.sourceDossierId' enthaltet die "dossierId" des Dossiers, von welcher die Beziehung ausgeht ("Quelldossier", "Vorgänger"). Das Attribut 'Project_Relationship.targetDossierId' enthaltet die "dossierId" des Dossiers, welche zeitlich auf das Quelldossier folgt ("Zieldossier", "Nachfolger"). Ein Dossier kann mit einem oder mehreren Quelldossier als auch Zieldossier verknüpft sein. Es sind nur Beziehungen zwischen Dossier abgebildet, welche einen Eintrag in der Entität 'Project_Dossier' besitzen.

#### 4.4.4.2 Entstehung
Mithilfe des Skripts [dossier_relationship.py](https://github.com/history-unibas/economies-of-space-database/blob/main/dossier_relationship.py) wurden Beziehungen ermittelt. Einerseits basierend auf den Adressen (Attribut 'StABS_Dossier.title') sowie des Attributs 'StABS_Dossier.deskriptiveNote'. Andererseits basierend auf ermittelte Cluster und der Art des Dossiers. Mit dem Algorithmus konnten viele Beziehungen nicht erkannt werden. Aus diesem Grund wurden anschliessend Beziehungen manuell ermittelt.

#### 4.4.4.3 Spezialfälle
Wenn ein Dossier ab einem bestimmten Zeitpunkt (oder bis zu einem bestimmten Zeitpunkt) bei einem anderen Dossier eingeschlossen war, lässt sich zwar die Beziehung korrekt bestimmen, aber es ist dann eigentlich nicht korrekt, von einem Folgedossier auszugehen (in dem Sinne, dass ein Dossier endet und danach ein anderes beginnt, also keine Überschneidungen bestehen). Deshalb wurden diese Beziehungen separat erfasst.

#### 4.4.4.4 Fehler
Die automatisierte Erkennung von Beziehungen war fehlerhaft, weshalb alle Beziehungen manuell angeschaut wurden. Die Metadaten des HGB (Entität 'StABS_Dossier') sind in vielen Fällen nicht eindeutig und scheinen von unterschiedlicher Qualität zu sein, weshalb diese Beziehungen gar nicht ganz korrekt erfasst werden können. Eine gewisse Fehlerquote ist daher inhärent.

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
| Bezeichnung | Beschreibung |
|--------|--------|
| Attribut | Eine Information oder Eigenschaft eines Eintrages einer Entität. In einer relationalen Datenbank entspricht eine Spalte einem Attribut. |
| Brandlagerbuch | Auszug aus Brandversicherungsakten. Für die Analyse von Transaktionen nicht relevant und deshalb nicht transkribiert. |
| CER | Character Error Rate, Zeichenfehlerrate, siehe Neudecker, Clemens; Baierer, Konstantin; Gerber, Mike u. a.: A survey of OCR evaluation tools and metrics, in: The 6th International Workshop on Historical Document Imaging and Processing, Lausanne Switzerland 2021, S. 13–18. Online: <https://doi.org/10.1145/3476887.3476888>. |
| Entität | Ein Objekt in der Projektdatenbank, welches in einer Datenbank durch eine Tabelle repräsentiert wird. |
| Frönung | Beschlagnahmungsverfahren |
| Graph / Graphdatenbank | Daten werden in einem Netzwerkmodell abgebildet / gespeichert statt in Tabellen. |
| HGB | Abkürzung für Historische Grundbuch der Stadt Basel.<br><br> Weitere Informationen: https://blog.staatsarchiv-bs.ch/neu-online-einsehbar-das-historische-grundbuch  |
| HTR | Handwritten Text Recognition (oder Automatic Text Recognition). Automatisierte Erkennung von textuellen Elementen auf Basis von segmentierten Linien.Basierend auf *machine learning*-Algorithmen |
| Hugging Face | Eine Plattform zur Publikation von Machine Learning-Modellen und Daten.<br><br> Weitere Informationen: https://huggingface.co |
| IIIF Manifest | Eine standardisierte JSON-Datei, die alle Metadaten, Beschreibungen und die logische Reihenfolge der Digitalisate (Bilder) eines Objekts (z. B. eines HGB-Dossiers) enthält. Es dient als universelle Schnittstelle, um hochauflösende Bilder direkt vom Server des Staatsarchivs in externen Tools (wie Dokumenten-Viewern) plattformübergreifend und performant anzuzeigen, ohne die Bilddateien physisch kopieren zu müssen. IIIF steht für International Image Interoperability Framework. |
| Linked Open Data (LOD) | Linked Data ist eine Methode Daten in einer Form verfügbar zu machen, so dass die Nutzbarkeit für Menschen als auch Maschinen möglich ist. Linked Open Data sind Linked Data, welche zur freien Verfügung veröffentlicht werden.<br><br> Weitere Informationen: https://www.stadt-zuerich.ch/de/politik-und-verwaltung/statistik-und-daten/linked-open-data.html |
| Löffelplan | Situationsplan aufgenommen von L.H. Löffel aus dem Jahr 1862<br><br> Weitere Informationen: https://www.bs.ch/bvd/grundbuch-und-vermessungsamt/vermessung/historische-plaene#nachdrucke-historischer-plaene |
| LV95 / EPSG:2056 | Schweizer Koordinatensystem basierend auf der Landervermessung des Jahres 1995 (LV95).<br><br> Weitere Informationen: https://www.swisstopo.admin.ch/de/landesvermessung-lv95, https://epsg.io/2056 |
| P2PaLA | Pixelannotation auf Trainingsbasis (Computer Vision Algorithmus). Entwickelt durch die Technische Universität Valencia (im Rahmen von READ).<br><br> Weitere Informationen: https://blog.transkribus.org/en/transkribus/docu/p2pala |
| Projektdatenbank | Relationale Datenbank (PostgreSQL mit PostGIS-Erweiterung) zur Speicherung und Analyse der für das Projekt relevanten Informationen des HGBs.<br><br> Weitere Informationen: https://github.com/history-unibas/Postgresql-Project-Database/tree/main#postgresql-project-database |
| PyLaia | Toolkit basierend auf PyTorch (Deep Learning) zur Analyse handschriftlicher Dokumente.<br><br> Weitere Informationen: https://github.com/jpuigcerver/PyLaia |
| Reichspfennigverzeichnis | Angabe zur Reichssteuer von 1497. Für die Analyse von Transaktionen nicht relevant und deshalb nicht transkribiert. |
| Shape-Datei | Dateiformat zur Speicherung und Austausch von Daten mit geografischer Information. |
| SPARQL | SPARQL Protocol and RDF Query Language: Sprache für die Abfrage von Linked Open Data.<br><br> Weitere Informationen: https://www.opendata.bs.ch/ld.html |
| StABS | Abkürzung für das Staatsarchiv Basel-Stadt. |
| Stufe (Serie / Dossier) | Die hierarchische Gliederung der Archivdaten basierend auf dem internationalen Standard RiC (Records in Contexts). Im Projekt entspricht eine Serie der übergeordneten Ebene (z. B. einer historischen Strasse), während ein Dossier die darunterliegende, spezifische Einheit abbildet (z. B. ein einzelnes Gebäude oder eine Adresse auf dieser Strasse). |
| Transkribus Plattform | Digitale Plattform zur Erkennung von Dokumenten. Geführt durch die READ COOP.<br><br> Weitere Informationen: https://www.transkribus.org |
| UUID | Abkürzung für Universally Unique Identifier, also einen eindeutiger Identifikator für ein Objekt. |
| WER | Word Error Rate: Wortfehlerrate, siehe Neudecker, Clemens; Baierer, Konstantin; Gerber, Mike u. a.: A survey of OCR evaluation tools and metrics, in: The 6th International Workshop on Historical Document Imaging and Processing, Lausanne Switzerland 2021, S. 13–18. Online: <https://doi.org/10.1145/3476887.3476888>. |
| Zenodo | Open-Access-Repository zur Veröffentlichung und Archivierung von Forschungsdaten.<br><br> Weitere Informationen: https://zenodo.org |
