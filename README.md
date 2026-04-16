# Case 4 – ESB and Web Services  
## MuleSoft Integration Project

Dieses Repository enthält die technische Umsetzung des MuleSoft-Projekts für **Case 4** im Modul **Software Architecture** an der **FHNW**.  
Im Fokus steht die Integration eines externen SOAP-Webservices über einen ESB-basierten Ansatz sowie die Weiterleitung der empfangenen Aufträge über **Apache ActiveMQ** an die bestehende Messaging-Lösung aus **Case 3**.

---

## Zielsetzung

Ziel der Umsetzung ist es, neue Aussendienstaufträge automatisiert von einem externen Callcenter-Service abzurufen, aufzubereiten und abhängig vom Auftragstyp an die passende JMS-Destination weiterzuleiten.

Die Lösung übernimmt dabei insbesondere folgende Aufgaben:

- periodischer Abruf neuer Aufträge über einen **SOAP-Webservice**
- Transformation der empfangenen SOAP-Daten in das erwartete Nachrichtenformat
- Klassifikation der Aufträge nach Typ
- Publikation von **Routineaufträgen** auf ein JMS-Topic
- Weiterleitung von **dringenden Aufträgen** an eine JMS-Queue
- Behandlung unbekannter Auftragstypen über eine **Dead Letter Queue**
- Vorbereitung der Nachrichten für die bestehende **Spring-Boot-Client-Lösung aus Case 3**

---

## Technologiestack

Für die Umsetzung wurden folgende Technologien verwendet:

- **MuleSoft Anypoint Studio**
- **Mule 4**
- **SOAP / Web Service Consumer**
- **Apache ActiveMQ**
- **JMS**
- **DataWeave 2.0**
- **Spring Boot Client (aus Case 3)**

---

## Projektstruktur

Die zentrale Logik befindet sich im Mule-Flow:

- `sa_case4_group6Flow`

Wichtige Projektbestandteile:

- `src/main/mule/` – Mule-Flows und Integrationslogik
- `src/main/resources/` – Konfigurationsdateien und Properties
- `pom.xml` – Maven-Projektdefinition

---

## Funktionsweise der Lösung

Der Mule-Flow wird periodisch durch einen **Scheduler** ausgelöst.  
Anschliessend ruft ein **Web Service Consumer** den externen SOAP-Service auf und führt die Operation `getNewJobs` aus.

Da der Webservice innerhalb einer Antwort mehrere Aufträge zurückliefern kann, werden die empfangenen Einträge in einem **For Each**-Scope einzeln verarbeitet.

Für jeden Auftrag erfolgt anschliessend:

1. Protokollierung der empfangenen Rohdaten
2. Transformation in das erwartete interne Nachrichtenformat
3. Klassifikation anhand des Feldes `jobType`
4. Weiterleitung an die passende JMS-Destination

Die Verzweigungslogik erfolgt über einen **Choice-Connector**:

- `Maintanence` → Topic `group6.dispo.jobs.new`
- `Repair` → Queue `group6.dispo.urgent.orders`
- unbekannter Typ → Queue `group6.dispo.dead.letter`

---

## Nachrichtenformat und JMS-Integration

Die transformierten Daten werden vor dem Publizieren explizit in **JSON** serialisiert, da der Spring-Boot-Client aus Case 3 die Nachrichten als JSON-Text verarbeitet.

Zusätzlich wird der Typ nicht im JSON-Body, sondern als **JMS-Property** gesetzt:

- `_type = ch.fhnw.digi.demo.JobMessage`

Dies ist notwendig, damit der auf Empfängerseite verwendete `MappingJackson2MessageConverter` die Nachricht korrekt in das passende Java-Objekt deserialisieren kann.

---

## Weiterverwendung der Case-3-Lösung

Für die Umsetzung von Case 4 wurde die bestehende Lösung aus **Case 3** grundsätzlich weiterverwendet.  
Angepasst wurde lediglich das Java-Objekt `JobMessage`, welches um das Attribut `scheduledDateTime` ergänzt wurde, damit die von MuleSoft publizierten Nachrichten korrekt gemappt werden können.

Die bestehende Dispositionslogik aus Case 3 wurde bewusst beibehalten.  
Dadurch werden auch die vom ESB gelieferten Routineaufträge weiterhin durch die Disposition verarbeitet und nach Eingang entsprechender Anfragen den Mitarbeitenden zugewiesen.

---

## Service-Virtualisierung

Die physische Adresse des externen SOAP-Services wird nicht direkt im Flow hardcodiert, sondern über Properties referenziert.  
Dadurch übernimmt der ESB die Rolle einer vermittelnden Zugriffsschicht zwischen interner Integrationslogik und externem Service.

Eine weitergehende Lösung über eine **Service Registry** wurde konzeptionell betrachtet, für den vorliegenden Case jedoch bewusst nicht umgesetzt.  
Aufgrund der überschaubaren Anzahl an Services wäre der zusätzliche Einführungsaufwand unverhältnismässig gewesen.  
Die gewählte Property-basierte Lösung reduziert die direkte Adressabhängigkeit mit geringem Aufwand und bleibt gleichzeitig grundsätzlich erweiterbar.

---

## Hinweise zur Ausführung

Für die Ausführung der Lösung werden folgende abhängige Systeme benötigt:

- erreichbarer **SOAP-Webservice**
- laufender **ActiveMQ-Broker**
- bestehender **Spring-Boot-Client aus Case 3** zur Nachrichtenverarbeitung

Je nach Umgebung müssen insbesondere die folgenden Verbindungsparameter angepasst werden:

- SOAP-Service-URL
- ActiveMQ-Broker-URL
- Benutzername / Passwort für JMS

---

## Bekannte Vereinfachungen

Im Rahmen des Cases wurden einzelne Aspekte bewusst vereinfacht oder nicht umgesetzt:

- die technische Behandlung dringender Aufträge durch die Disposition ist nicht Bestandteil dieses Cases
- weiterführende Retry-, Timeout- oder Fehlerbehandlungsmechanismen wurden gemäss Coaching nicht vertieft implementiert
- die bestehende Dispositionslogik aus Case 3 wurde übernommen und nicht grundlegend neu gestaltet

---

## Kontext des Projekts

Dieses Repository wurde im Rahmen des Moduls **Software Architecture** erstellt.  
Der Case behandelt die Themen:

- **Enterprise Service Bus (ESB)**
- **Service-Virtualisierung**
- **SOAP-Webservices**
- **Messaging-basierte Weiterverarbeitung**
- **Integration bestehender Systemkomponenten**

---

## Autoren

- Kapischan Sriganthan  
- Mladen Radovanovic  
- Loic Bösch  

FHNW – Software Architecture  
FS 2026
