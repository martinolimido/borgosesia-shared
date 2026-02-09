# borgosesia-shared

Libreria Java condivisa per i servizi Borgosesia con componenti comuni di:
- sicurezza Spring Security con validazione JWT remota
- configurazione Jackson
- entità base JPA
- proprietà applicative condivise

Il progetto è un modulo **Maven `jar`** (non un'applicazione standalone).

## Requisiti

- Java 17
- Maven 3.9+

## Build e installazione locale

```bash
mvn clean install
```

Questo comando produce il JAR e lo installa nel repository Maven locale, così può essere usato dagli altri servizi.

## Utilizzo in un altro progetto

Aggiungi la dipendenza nel `pom.xml` del servizio che la consuma:

```xml
<dependency>
  <groupId>it.borgosesiaspa</groupId>
  <artifactId>borgosesia-shared</artifactId>
  <version>1.0.0-SNAPSHOT</version>
</dependency>
```

## Configurazione richiesta

Le proprietà principali esposte/usate dal modulo:

- `remote.auth.url`: endpoint remoto usato dal filtro JWT per validare il bearer token
- `bgs.default.repository`: proprietà bindata da `BgsDefaultProperties`

Esempio (`application.yml`):

```yaml
remote:
  auth:
    url: "https://<host>/.../authenticate"

bgs:
  default:
    repository: "<valore-default>"
```

## Componenti inclusi

- `it.borgosesiaspa.shared.config.SecurityConfig`
  - configura Spring Security stateless
  - aggiunge `RemoteJwtAuthFilter`
  - permette accesso anonimo a:
    - `/v3/api-docs/**`
    - `/swagger-ui/**`
    - `/swagger-ui.html`
    - `/api/images/**`
  - abilita CORS per:
    - `http://localhost:5173`
    - `https://borgosesia.xplants.net`

- `it.borgosesiaspa.shared.security.RemoteJwtAuthFilter`
  - legge `Authorization: Bearer <token>`
  - valida il token tramite chiamata HTTP a `remote.auth.url`
  - popola il `SecurityContext` con username e authorities (da `utente.extra.permessi`)

- `it.borgosesiaspa.shared.config.JacksonConfig`
  - registra `JavaTimeModule`
  - configura serializzazione/deserializzazione JSON condivisa

- `it.borgosesiaspa.shared.util.BaseEntity`
  - `id`, `dataCreazione`, `dataModifica`, `idUtenteCreatore`
  - auditing base con `@PrePersist` e `@PreUpdate`

- `it.borgosesiaspa.shared.service.BgsDefaultProperties`
  - binding delle proprietà `bgs.default.*`

## Note

- Il modulo include `spring-boot-starter-security`, `spring-boot-starter-data-jpa`, `spring-boot-starter-web` e dipendenze Jackson/JJWT.
- Essendo una libreria condivisa, eventuali cambiamenti breaking richiedono aggiornamento versione e coordinamento con i servizi che la consumano.
