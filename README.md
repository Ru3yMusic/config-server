# config-server

Servidor de configuración centralizada para todos los microservicios de **RUBY MUSIC**. Basado en **Spring Cloud Config Server** con perfil `native` (archivos locales en classpath).

---

## Responsabilidad

Distribuye configuración a todos los microservicios en el momento de su arranque. Cada servicio consulta este servidor antes de inicializarse, lo que permite gestionar puertos, credenciales de BD, Kafka, JWT y CORS desde un único punto.

---

## Stack

| Componente | Versión |
|---|---|
| Java | 21 |
| Spring Boot | 3.2.5 |
| Spring Cloud | 2023.0.1 |
| Spring Cloud Config Server | — |
| Eureka Client | — |
| Spring Boot Actuator | — |

---

## Puerto

| Servicio | Puerto |
|---|---|
| config-server | **8888** |

---

## Orden de arranque

```
discovery-service (8761) → config-server (8888) → api-gateway → business services
```

> El config-server debe estar corriendo **antes** que cualquier microservicio de negocio.

---

## Configuraciones gestionadas

Los archivos en `src/main/resources/config/` son servidos automáticamente a cada microservicio por nombre:

| Archivo | Servicio destino | Puerto |
|---|---|---|
| `application.yml` | Defaults compartidos (todos) | — |
| `auth-service.yml` | auth-service | 8081 |
| `catalog-service.yml` | catalog-service | 8082 |
| `interaction-service.yml` | interaction-service | 8083 |
| `playlist-service.yml` | playlist-service | 8084 |
| `social-service.yml` | social-service | 8085 |
| `api-gateway.yml` | api-gateway | 8080 |

### Defaults compartidos (`config/application.yml`)
- JPA/Hibernate: PostgreSQL dialect, `ddl-auto: update`, batch size 20
- Actuator: expone `health`, `info`, `metrics`, `prometheus`
- Logging: INFO para `com.rubymusic`, WARN para Spring Security e Hibernate SQL

---

## Variables de entorno requeridas

Estas variables deben estar definidas en el entorno donde corran los microservicios (no en el config-server mismo):

### auth-service
| Variable | Descripción |
|---|---|
| `DB_USERNAME` | Usuario PostgreSQL (default: `postgres`) |
| `DB_PASSWORD` | Contraseña PostgreSQL (default: `password`) |
| `MAIL_USERNAME` | Cuenta Gmail para envío de OTP |
| `MAIL_PASSWORD` | Contraseña de aplicación Gmail |
| `JWT_PRIVATE_KEY` | Clave RSA privada para firmar tokens |
| `JWT_PUBLIC_KEY` | Clave RSA pública para validar tokens |

### api-gateway
| Variable | Descripción |
|---|---|
| `JWT_PUBLIC_KEY` | Clave RSA pública para validar JWT en el filtro |

### catalog-service / interaction-service / social-service / playlist-service
| Variable | Descripción |
|---|---|
| `DB_USERNAME` | Usuario PostgreSQL |
| `DB_PASSWORD` | Contraseña PostgreSQL |

---

## Estructura del proyecto

```
config-server/
├── src/
│   ├── main/
│   │   ├── java/com/rubymusic/configserver/
│   │   │   └── ConfigServerApplication.java   ← @EnableConfigServer
│   │   └── resources/
│   │       ├── application.yml                ← config del propio servidor (puerto 8888, Eureka)
│   │       └── config/                        ← configuraciones para los clientes
│   │           ├── application.yml            ← defaults compartidos
│   │           ├── auth-service.yml
│   │           ├── catalog-service.yml
│   │           ├── interaction-service.yml
│   │           ├── playlist-service.yml
│   │           ├── social-service.yml
│   │           └── api-gateway.yml
│   └── test/
│       └── java/com/rubymusic/configserver/
│           └── ConfigServerApplicationTests.java
└── pom.xml
```

---

## Build & Run

```bash
# Build
mvn clean package -DskipTests

# Run
mvn spring-boot:run

# Test
mvn test -Dtest=ConfigServerApplicationTests
```

---

## Endpoints útiles (Actuator)

```
GET http://localhost:8888/actuator/health
GET http://localhost:8888/actuator/info

# Ver configuración que recibirá un servicio
GET http://localhost:8888/{service-name}/default
# Ejemplo:
GET http://localhost:8888/auth-service/default
GET http://localhost:8888/api-gateway/default
```

---

## Registro en Eureka

El config-server se registra como cliente en Eureka para ser descubrible:

```
Service ID : config-server
Eureka URL : http://localhost:8761/eureka/
```

> Aunque el config-server también es cliente de Eureka, los microservicios lo consultan directamente por su URL (`http://localhost:8888`) antes de que Eureka esté disponible para ellos.
