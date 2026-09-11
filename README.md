# Biblioteca

Proyecto de microservicios desarrollado con **Spring Boot**, desplegado en **AWS EC2** y conectado a una base de datos **Amazon RDS MySQL**. El proyecto utiliza **Eureka** para descubrimiento de servicios, **API Gateway** como punto de entrada y **GitHub Actions** para automatizar la compilación y el despliegue.

## Arquitectura general

El proyecto está compuesto por los siguientes módulos:

- `common`: módulo compartido entre microservicios.
- `eureka`: servidor de descubrimiento de servicios.
- `ms-usuarios`: microservicio de usuarios y autenticación.
- `ms-catalogo`: microservicio de catálogo.
- `ms-recursos`: microservicio de recursos.
- `api-gateway`: puerta de entrada a los microservicios.
- `init-multi-db`: scripts de inicialización de bases de datos.
- `postman`: recursos para pruebas de API.

## Prerrequisitos

Para ejecutar el proyecto localmente se requiere:

- Java 21.
- Maven Wrapper incluido en el repositorio.
- MySQL 8 o compatible.
- Git.
- Acceso a las variables de entorno necesarias para conexión a base de datos y seguridad JWT.

Para el despliegue en AWS se requiere:

- Una instancia EC2 con Ubuntu.
- Java 21 instalado en la EC2.
- Una instancia Amazon RDS MySQL.
- Acceso SSH a la instancia EC2.
- GitHub Secrets configurados en el repositorio.
- Servicios `systemd` configurados para cada microservicio.

## Variables de entorno

Los microservicios utilizan variables de entorno para evitar almacenar credenciales directamente en el código.

Variables utilizadas:

```text
DB_URL
DB_USERNAME
DB_PASSWORD
EUREKA_URL
JWT_SECRET
```

En el entorno desplegado, estas variables se gestionan mediante **GitHub Secrets** y se escriben en archivos de entorno dentro de la EC2.

Los principales secretos configurados en GitHub son:

```text
DB_USERNAME
DB_PASSWORD
DB_URL_USUARIOS
DB_URL_CATALOGO
DB_URL_RECURSOS
JWT_SECRET
EC2_HOST
EC2_USER
EC2_PORT
EC2_SSH_KEY
```

> Importante: nunca se deben subir contraseñas, llaves privadas ni secretos JWT directamente al repositorio.

## Instalación y puesta en marcha local

Clonar el repositorio:

```bash
git clone https://github.com/belruz/Biblioteca-Springboot.git
cd Biblioteca-Springboot
```

En Windows, compilar el proyecto con:

```powershell
.\mvnw.cmd clean install -DskipTests
```

En Linux o macOS:

```bash
./mvnw clean install -DskipTests
```

Para ejecutar un módulo individual, ingresar a su carpeta y ejecutar el Maven Wrapper.

Ejemplo para Eureka:

```powershell
cd eureka
..\mvnw.cmd spring-boot:run
```

Ejemplo para `ms-usuarios`:

```powershell
cd ms-usuarios
..\mvnw.cmd spring-boot:run
```

Los servicios deben iniciarse respetando el siguiente orden:

```text
Eureka
→ MS Usuarios
→ MS Catálogo
→ MS Recursos
→ API Gateway
```

## Bases de datos

El proyecto utiliza tres bases de datos principales:

```text
usuarios
catalogo
recursos
```

En MySQL se pueden revisar con:

```sql
SHOW DATABASES;
```

Para visualizar las tablas de una base de datos:

```sql
USE usuarios;
SHOW TABLES;
```

## Puesta en marcha en AWS

El despliegue se realiza sobre una instancia **AWS EC2 con Ubuntu**.

Los archivos JAR generados se copian a:

```text
/home/ubuntu/micros
```

Los servicios son administrados mediante `systemd`.

Servicios configurados:

```text
biblioteca-eureka.service
biblioteca-usuarios.service
biblioteca-catalogo.service
biblioteca-recursos.service
biblioteca-gateway.service
```

Para consultar el estado de un servicio:

```bash
sudo systemctl status biblioteca-eureka.service
```

Para reiniciarlo:

```bash
sudo systemctl restart biblioteca-eureka.service
```

## CI/CD con GitHub Actions

El proyecto incluye un pipeline de despliegue automatizado mediante **GitHub Actions**.

El flujo general es:

```text
Push a main
→ Checkout del código
→ Configuración de JDK 21
→ Compilación con Maven
→ Generación de JARs
→ Copia de JARs a EC2 mediante SCP
→ Configuración de variables de entorno
→ Reinicio de servicios systemd
→ Health checks automáticos
```

El pipeline verifica que cada servicio quede disponible antes de continuar con el siguiente.

## Estrategia de trabajo con Git

Para esta evaluación se trabajó únicamente sobre la rama:

```text
main
```

No se utilizaron ramas `develop`, `feature` ni `hotfix`, ya que para esta actividad se definió trabajar directamente sobre `main`.

Los cambios se fueron registrando mediante commits con mensajes descriptivos siguiendo una convención similar a:

```text
fix: corrige scripts SQL y externaliza secreto JWT
ci: agrega workflow de compilacion
ci: agrega copia de jars a EC2
ci: configura variables de entorno desde GitHub Secrets
ci: agrega reinicio y health checks al despliegue
```

## Proyecto en funcionamiento

Reemplazar `ip_publica` por la IP pública vigente de la instancia EC2.

### Eureka

```text
http://ip_publica:8761/
```

Health:

```text
http://ip_publica:8761/actuator/health
```

### Usuarios

Swagger:

```text
http://ip_publica:9001/swagger-ui/index.html
```

Health:

```text
http://ip_publica:9001/actuator/health
```

### Catálogo

Swagger:

```text
http://ip_publica:9002/swagger-ui/index.html
```

Health:

```text
http://ip_publica:9002/actuator/health
```

### Recursos

Swagger:

```text
http://ip_publica:9003/swagger-ui/index.html
```

Health:

```text
http://ip_publica:9003/actuator/health
```

### API Gateway

Health:

```text
http://ip_publica:9000/actuator/health
```

## Pruebas realizadas

Se validó correctamente:

- Registro de servicios en Eureka.
- Health checks en estado `UP`.
- Swagger disponible en los tres microservicios.
- Login en `ms-usuarios`.
- Generación de token JWT.
- Uso del token JWT en `ms-catalogo` y `ms-recursos`.
- Respuestas HTTP `200`.
- Conexión de los microservicios con Amazon RDS.
- Despliegue automático mediante GitHub Actions.

## Seguridad

Se aplicaron las siguientes medidas:

- Credenciales fuera del repositorio.
- Uso de GitHub Secrets.
- JWT externalizado mediante variable de entorno.
- Archivos de entorno con permisos restringidos en EC2.
- Uso de Security Groups para controlar acceso a EC2 y RDS.
- Administración de servicios mediante `systemd`.


## Uso de Inteligencia Artificial

Se utilizó inteligencia artificial como apoyo para redacción del README, revisión de comandos y organización del proceso. Las decisiones, validaciones, pruebas y conclusiones técnicas fueron revisadas por el equipo.
