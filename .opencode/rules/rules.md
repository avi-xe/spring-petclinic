# .opencode/rules/rules.md - Regole Critiche per Agente AI

## Architettura (Clean Architecture)

1. **NESSUN service layer**: i controller parlano direttamente ai repository JPA
2. **Entity solo in package model/**: never OwnerController.java o owner/*.java per le entita'
3. **Repository Pattern**: tutti usano `@Repository` + Spring Data JPA interfaces
4. **Relazioni JPA**: @ManyToOne (Pet←Owner), @OneToMany (Owner→Pet), JOIN FETCH per evitare N+1 queries
5. **NO FlyLiquibase**: migration manuali solo in src/main/resources/db/h2|mysql|postgres/

## Build Commands

```bash
./mvnw clean compile     # Prima compilazione
./mvnw test              # SEVE compresare test prima di ogni commit
./mvnw validate          # Checkstyle + nohttp check obbligatorio
./mvnw spring-javaformat:apply  # Spring JavaFormat validation
./mvnw verify            # Clean+test con H2 o profile DB esterno
```

## Database & Schema

- Default: `src/main/resources/application.properties` (H2 in-memory)
- Switch MySQL: `application-mysql.properties`
- Switch PostgreSQL: `application-postgres.properties`  
- Modificare entita'→aggiornare schema.sql nel db corrispondente

## Frontend/Thymeleaf

- Template in: `src/main/resources/templates/*.html`
- Layout master pattern con file _footer.html, _header.html (prefisso _)
- Input validazione: `@Valid @RequestBody Owner bindingResult` obbligatorio
- CSRF token sempre presente nelle form POST/PUT/DELETE

## Security OWASP

```java
@PostMapping("/owners")  // NO POST a "/" o resource root!
public void add(@Validated Owner owner, BindingResult br) { ... }  // @Valid su input
```

- Mai usare protocolli HTTP in codice (solo JDBC/JPA per DB access)
- No @RequestMapping con path rischiosi come "/crash" fuorché nel CrashController dedicato

## Pre-Commit Checklist

Minima checklist PRIMA di ogni commit:

- [ ] `./mvnw validate` passa senza errori checkstyle/nohttp  
- [ ] `./mvnw test` tutti i test verdi (0 failures)
- [ ] Formattazione Spring JavaFormat conforme (.editorconfig compliant)
- [ ] NO modifiche a pom.xml senza approvazione esplicita
- [ ] Entity in package org.springframework.samples.petclinic.model/  
- [ ] Controller in owner|vet|system package, mai entity nel controller package!
- [ ] @Valid e BindingResult usati su tutti i POST/PUT
- [ ] JOIN FETCH usato per query che evitano N+1 queries
- [ ] Schema DB sincronizzato (file .sql aggiornati)

## NO - Cosa NON fare mai

- ❌ Creare entita' di dominio in package owner/, vet/, system/
- ❌ Usare @GeneratedValue senza @Id generatos da Spring  
- ❌ Modificare pom.xml per solo commenti JavaFormat
- ❌ Chiamare DB con JDBC diretto, usare sempre @Repository + ORM
- ❌ Output non escaped: `th:text="${var}"` mai "th:expr='${exp}'"

## Build Profile Commands

```bash
# Development H2 default - dev mode
./mvnw spring-boot:run

# Test con MySQL real (testcontainers)
./mvnw verify -Dspring.profiles.active=mysql

# Full deployment docker-compose
docker-compose up -d && ./mvnw verify
docker-compose down

# GraalVM native support (se necessario)
./mvnw native:compile
```
