# Comandos de Gradle / Spring Boot

Todos se ejecutan dentro de `inventario-api`. En Windows es `.\gradlew.bat`; en Linux (AWS, GitHub Actions) es `./gradlew`.

## Equivalencias

| Node / Laravel | Spring Boot (Gradle) | Qué hace |
|---|---|---|
| `npm run dev` / `php artisan serve` | `.\gradlew.bat bootRun` | Levanta la API en http://localhost:8080 |
| `npm install` / `composer install` | (automático) | Gradle descarga las dependencias al compilar |
| `npm test` / `php artisan test` | `.\gradlew.bat test` | Corre las pruebas (JUnit + Mockito) |
| `npm run build` | `.\gradlew.bat build` | Compila, prueba y genera el `.jar` en `build/libs/` |
| `node dist/app.js` | `java -jar build\libs\inventario-api-0.0.1-SNAPSHOT.jar` | Ejecuta el `.jar` (así corre en producción) |
| `rm -rf node_modules dist` | `.\gradlew.bat clean` | Borra la carpeta `build/` |
| `npm ls` | `.\gradlew.bat dependencies` | Árbol de dependencias |
| `composer.json` | `build.gradle.kts` | Dependencias y configuración del build |
| `.env` | `src/main/resources/application.yml` | Configuración por perfiles (dev / prod) |

## Útiles

```powershell
.\gradlew.bat bootRun --args='--spring.profiles.active=prod'   # arrancar con otro perfil
.\gradlew.bat test --tests "*ProductoServiceTest"              # una sola clase de pruebas
.\gradlew.bat build -x test                                    # build sin pruebas (solo si hay apuro)
.\gradlew.bat --stop                                           # matar el proceso de Gradle si bloquea carpetas
```

Para detener el servidor: **Ctrl + C** (si pregunta "¿Desea terminar el trabajo por lotes?", responde **S**).
La barra `80% EXECUTING` es normal: Gradle sigue "ejecutando" mientras el servidor esté encendido.

## Perfiles y base de datos

| Perfil | Base | Cómo arrancar |
|---|---|---|
| `dev` (por defecto) | H2 en memoria (se borra al reiniciar) | `.\gradlew.bat bootRun` |
| `local` | PostgreSQL en Docker (puerto 5433) | `.\gradlew.bat bootRun --args='--spring.profiles.active=local'` |
| `prod` | PostgreSQL en AWS RDS | variables de entorno del servidor |

Un **perfil** es un entorno: el mismo código con otra configuración. Es como tener un `.env` distinto en Laravel para local y producción. Cada bloque separado por `---` en `application.yml` es un perfil; lo que está arriba del primer `---` aplica a todos.

### Cómo activar un perfil

Si no activas ninguno, se usa `dev` (`profiles.default: dev`).

| Dónde | Cómo |
|---|---|
| Terminal (Gradle) | `.\gradlew.bat bootRun --args='--spring.profiles.active=local'` |
| IntelliJ ▶️ | Menú del ▶️ → **Edit Configurations** → **Active profiles**: `local` → OK |
| Variable de entorno | PowerShell: `$env:SPRING_PROFILES_ACTIVE="local"` y luego `.\gradlew.bat bootRun` |
| Producción (`.jar`) | `java -jar inventario-api.jar --spring.profiles.active=prod` (o la variable `SPRING_PROFILES_ACTIVE=prod` en el servidor) |

Para confirmar cuál está activo, busca en el log al arrancar: `The following 1 profile is active: "local"`.

### El `.env`

Las credenciales viven en `inventario-farmacia/.env` (la raíz del monorepo, una carpeta arriba de `inventario-api`). Lo leen Docker Compose y Spring (`config.import` en `application.yml`). **No se sube a GitHub** (está en `.gitignore`); si no existe, copia `.env.example` como `.env` y pon tus valores.

- `local` toma `DB_URL`, `DB_USER` y `DB_PASSWORD` del `.env`.
- `prod` usa las mismas variables, pero en AWS vienen de las variables de entorno del servidor, no de un archivo.
- `dev` no necesita `.env` (H2 en memoria).

```powershell
# desde inventario-farmacia/
docker compose up -d     # levantar Postgres
docker compose stop      # apagarlo (los datos se conservan)
docker compose down -v   # borrarlo TODO, incluidos los datos
docker ps                # ver qué contenedores corren
```

## En IntelliJ

| Atajo | Qué hace |
|---|---|
| ▶️ junto a `main` en `InventarioApiApplication` | Igual que `bootRun`, con logs en pestaña y botón ■ para detener |
| 🐞 (Debug) | Arranca con breakpoints |
| **Ctrl + F9** | Compila; con DevTools la app se reinicia sola (~2 s) |
| **Alt + F12** | Abre la terminal |
| **Shift + F6** | Renombrar (archivo, clase, variable) en todo el proyecto |
| **Alt + Enter** | Arreglo rápido: importar clase, crear método, etc. |
| **Doble Shift** | Buscar cualquier cosa |

## URLs en desarrollo

- http://localhost:8080/actuator/health → `{"status":"UP"}` si la app está viva
- http://localhost:8080/h2-console → base H2 en memoria (JDBC URL `jdbc:h2:mem:inventario`, usuario `sa`, sin contraseña)
