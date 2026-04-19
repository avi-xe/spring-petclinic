# AGENTS.md - Architettura del Progetto Spring PetClinic v4.0.0-SNAPSHOT

## Panoramica del Progetto

**Nome:** Spring PetClinic Sample Application  
**Versione:** 4.0.0-SNAPSHOT  
**Lingua:** Java 17  
**Build Tool:** Maven (con anche Gradle disponibile)  
**Licenza:** Apache License 2.0  

---

## Architettura a Livelli

Il progetto segue la **Clean Architecture** con separazione netta tra:
- **Modello** (`model/`) - Entità e relazioni di dominio
- **Controllori** (`owner/`, `vet/`, `system/`) - Gestione HTTP requests  
- **Repository** (*`*Repository.java`) - Strato accessso dati JPA
- **Configurazioni** (`system/`) - Setup infrastrutturale

---

## Package Structure

```
org.springframework.samples.petclinic
├── model       ← Entità, BaseEntity,NamedEntity, Person
├── owner       ← Gesto proprietari e relativi Pet, Visit  
├── vet         ← Vets e Specialty gestione
├── system      ← CacheConfig, CrashController, WebConfig, WelcomeController
└── package-info.java  ← Documentation Javadoc
```

---

## Stack Tecnologico

### Spring Framework
- **spring-boot-starter-webmvc** - RESTful web layers  
- **spring-boot-starter-data-jpa** - Persistenza ORM  
- **spring-boot-starter-validation** - Validazione input  
- **spring-boot-starter-cache** - Caching infrastructure  
- **spring-boot-starter-actuator** - Health checks e metrics  

### Database
- **H2** - In-memory development (runtime scope)  
- **MySQL**, **PostgreSQL** - Persistence per deployments  
- **Caffeine** - Cache runtime  

### Web & UI
- **Thymeleaf** - Server-side templates  
- **Bootstrap 5.3.8** + **Font Awesome 4.7.0** - Frontend styling  

---

## Flusso delle Richieste

```
HTTP Request → Controller (REST endpoints) → Repository (JPA/Hibernate) 
     ↓                                                    ↓
   View/HTML Response                                  Database (H2/MySQL/PostgreSQL)
```

---

## Repository Management

### Repository Pattern
Tutti i repository implementano interfacce con JPA:
- `OwnerRepository`, `PetTypeRepository`, `VetRepository`  
- Usa `@Repository` per mappatura AOP delle eccezioni  

### Entity Relationship
```java
// Esempio relazionale Owner - Pet
public class Pet {
    private Long ownerId;     // Relazione inversa
    @ManyToOne 
    private Owner owner;      // Foreign key JPA
}
```

---

## Deployment Targets

- **Local Development** - H2 in-memory (default)
- **Production MySQL/PostgreSQL** - switch via `application.properties`
- **Docker Compose** - `docker-compose.yml` per orchestrare tutti i servizi  
- **Kubernetes** - manifesti in `k8s/`  

---

## Testing Strategy

### Test Containner
- **Testcontainers** con profili database reali (MySQL/PostgreSQL)
- Isolamento test completo senza setup manuale

### Unit Tests
- **JUnit Jupiter**, **Spring Boot Test**
- Mocking tramite Spring TestContext

### Code Coverage
- **JaCoCo** - Report generati in `prepare-package` phase  
- Target coverage raggiungibile su tutti i package  

---

## Build & Deployment Commands (Maven)

### Sviluppo Locale
```bash
# Clean + Compile con H2 default
./mvnw clean compile

# Test solo
./mvnw test

# Full build + pack
./mvnw clean package

# Dev mode hot-reload (devtools activated)
./mvnw spring-boot:run
```

### Profili Database
```bash
# Switch a PostgreSQL per test
./mvnw verify -Dspring.profiles.active=postgres

# Docker compose full stack
docker-compose up -d
```

### Code Quality
```bash
# Checkstyle validation  
./mvnw validate

# Formattazione codice Spring JavaFormat
./mvnw spring-javaformat:validate

# JaCoCo coverage report  
./mvnw package exec:java -Dexec=report
```

---

## Configurazione Ambiente

### File Properties Principali
- `src/main/resources/application.properties` - Default H2  
- `application-mysql.properties` - MySQL config
- `application-postgres.properties` - PostgreSQL config  

### Dev Container
- `.devcontainer/devcontainer.json` - VSCode remote settings
- `.gradle/` - Gradle wrapper (fallback per builds)

---

## Security Considerations

### OWASP Compliance
- **CSRF Protection** - Enabled su Thymeleaf views
- **XSS Prevention** - Content-Security-Policy headers
- **Input Validation** - Bean Validation (`@Valid`)  

### Database Migrations
- SQL schemi versionati in `src/main/resources/db/<dbtype>/`
- Non usa FlyLiquibase - manual migrations  

---

## Performance Optimization

### Caching Strategy
- **Caffeine** per cache L1 (JPA PersistenceContext)  
- Configurato in `CacheConfiguration.java`  

### Database Connections
- HikariCP default pool su H2  
- Configurabile via profile properties  

---

## Documentazione Interna

- **package-info.java** - Javadoc per ogni package  
- **README.md** - Guide complete di setup  
- `.editorconfig` - Convenzioni formattazione uniformi  

---

## Estensioni Agente Supportate

Questo progetto supporta agenti AI su:
- ✅ Java code analysis e refactoring
- ✅ Test generation e maintenance
- ✅ Code review static analysis
- ✅ Build troubleshooting
- ✅ Database schema inspections 

---

## Regole di Commit (DCO)

Il progetto usa **DCO (Developer Certificate of Origin)** in `.github/dco.yml`.  
Prima del merge, assicurati che ogni commit abbia firma DCO valida.
