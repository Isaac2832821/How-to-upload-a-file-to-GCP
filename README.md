# Guía de Despliegue: Spring Boot + Aiven MySQL + Google Cloud Run

## Parte 1: Requisitos Previos y Configuración Inicial

### 1. Cuentas Necesarias

* **Cuenta de GitHub**: Para almacenar tu código y realizar despliegues automatizados.
* **Cuenta de Google Cloud Platform (GCP)**: Para desplegar tu aplicación en Cloud Run y Artifact Registry.

  * Asegúrate de tener la facturación habilitada.
  * Configura alertas de presupuesto si te preocupa el costo.
* **Cuenta de Aiven**: Para usar la base de datos MySQL gestionada.

### 2. Herramientas a Instalar Localmente

* Git: [https://git-scm.com/downloads](https://git-scm.com/downloads)
* JDK 17 o superior: [https://adoptium.net/temurin/releases/](https://adoptium.net/temurin/releases/)
* Maven: [https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)
* Docker Desktop *(opcional)*: [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

---

## Parte 2: Configurar la Base de Datos MySQL en Aiven

### Crear Servicio MySQL en Aiven

* Inicia sesión en Aiven.
* Crea un nuevo servicio MySQL (versión 8.0+).
* Elige una región cercana a donde desplegarás en GCP (ej. `us-central1`, `southamerica-east1`).

### Obtener Credenciales

* Ve al panel **Overview** del servicio MySQL.
* Toma nota de:

  * Host
  * Puerto
  * Nombre de base de datos
  * Usuario
  * Contraseña
  * Configuración SSL

### Configurar el Firewall

* Ve a la sección de Redes o Control de Acceso.
* Agrega temporalmente `0.0.0.0/0` para permitir conexiones desde Cloud Run *(no recomendado para producción)*.

### Probar Conexión

* Usa una herramienta como DBeaver.
* Si da error, agrega `allowPublicKeyRetrieval=TRUE` al final del string JDBC.

---

## Parte 3: Configurar el Proyecto Spring Boot

### Generar Proyecto en Spring Initializr

* Proyecto: Maven
* Lenguaje: Java
* Versión: Spring Boot 3.5.0 o superior
* Grupo: `com.example`
* Artefacto: `demo-microservice-spring-boot`
* Java: 17
* Dependencias:

  * Spring Web
  * Spring Data JPA
  * MySQL Driver
  * Thymeleaf

### `application.properties` (solo para pruebas locales)

```properties
server.port=8080
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
spring.thymeleaf.cache=false
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html

# --- SOLO PARA PRUEBAS LOCALES ---
spring.datasource.url=jdbc:mysql://<HOST_AIVEN>:<PUERTO>/<DB>?reconnect=true&useSSL=true&requireSSL=true
spring.datasource.username=<USUARIO>
spring.datasource.password=<CONTRASEÑA>
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# --- FIN ---
```

### Estructura del Código

* `Post.java`: Entidad JPA
* `PostRepository.java`: Repositorio JPA
* `PostController.java`: Controlador Spring MVC
* `index.html`: Plantilla Thymeleaf

### Prueba Local

```bash
mvn spring-boot:run
```

Navega a `http://localhost:8080` y verifica que todo funcione.

---

## Parte 4: Dockerizar la Aplicación

### `Dockerfile`

```dockerfile
FROM eclipse-temurin:17-jdk-alpine
WORKDIR /app
COPY target/demo-microservice-spring-boot-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### `.dockerignore`

```txt
.git
.gitignore
.mvn/
mvnw
mvnw.cmd
target/
.idea/
*.iml
**/.DS_Store
**/node_modules/
```

---

## Parte 5: Configuración en GCP

### Activar APIs Requeridas

* Cloud Run Admin API
* Cloud Build API
* Artifact Registry API
* IAM API

### Crear Cuenta de Servicio

* Nombre: `github-actions-deployer`
* Roles necesarios:

  * Cloud Run Admin
  * Service Account User
  * Artifact Registry Writer
  * Artifact Registry Administrator
  * Cloud Build Editor

### Generar Clave JSON

* Guarda la clave JSON — se usará como secreto en GitHub.

---

## Parte 6: Repositorio GitHub y Secretos

### Inicializar Repositorio

```bash
git init
git add .
git commit -m "Primer commit"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/TU_REPO.git
git push -u origin main
```

### Agregar Secretos en GitHub (Settings > Secrets > Actions)

* `GCP_SA_KEY`: Contenido completo del JSON
* `DB_URL`: Cadena JDBC
* `DB_USER`: Usuario Aiven
* `DB_PASSWORD`: Contraseña Aiven

---

## Parte 7: GitHub Actions Workflow

### `.github/workflows/deploy.yml`

*(ver en la versión original, se mantiene igual solo que debes cambiar IDs y nombres por los tuyos)*

---

## Parte 8: Desplegar y Verificar

### Hacer Push para Desplegar

```bash
git add .
git commit -m "Agregar CI/CD"
git push origin main
```

### Verificar GitHub Actions

* Ve al tab **Actions** en tu repositorio.
* Sigue los logs en tiempo real.

### Revisar en Cloud Run

* Abre Google Cloud Console → Cloud Run.
* Verifica que el servicio esté desplegado.
* Copia la URL del servicio.

### Probar Aplicación

* Visita la URL desplegada.
* Crea un post y verifica que se guarde en la base de datos.

---

## Consejos de Solución de Problemas

* **Logs de Cloud Run** → Revisa errores de arranque o conexión a DB.
* **Firewall de Aiven** → Asegúrate que `0.0.0.0/0` esté habilitado temporalmente.
* **Secretos de GitHub** → Verifica que no tengan errores o espacios extra.
* **Variables de Entorno en Cloud Run** → Revisa la sección "Variables y secretos".

---

> Esta guía cubre todo el flujo de despliegue de una app Spring Boot con MySQL en Aiven usando Cloud Run y GitHub Actions. 🚀
