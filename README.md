# WorkDigest Agent

El agente portable de WorkDigest conecta tu instancia de WorkDigest con sistemas internos de tu red local o VPN: Redmine on-premise, sistemas de ficheros locales, SharePoint corporativo, y más.

## Descarga

| Sistema operativo | Descarga |
|-------------------|----------|
| Windows (x64) | [WorkDigest-Agent-windows.exe](https://github.com/work-digest/agent-releases/releases/latest/download/WorkDigest-Agent-windows.exe) |
| Linux (x64) | [WorkDigest-Agent-linux](https://github.com/work-digest/agent-releases/releases/latest/download/WorkDigest-Agent-linux) |
| macOS (x64) | [WorkDigest-Agent-macos](https://github.com/work-digest/agent-releases/releases/latest/download/WorkDigest-Agent-macos) |

## Instalación y configuración

### 1. Descarga el agente para tu sistema operativo

Usa los enlaces de la tabla anterior o descárgalo directamente desde [la última release](https://github.com/work-digest/agent-releases/releases/latest).

### 2. Genera un token en WorkDigest

1. Entra en tu cuenta de [WorkDigest](https://workdigest.io)
2. Ve a **Mi Puente** en el menú lateral
3. En la sección **Crear nuevo token**, escribe un nombre descriptivo para este dispositivo (ej. "Mi PC", "Servidor corporativo")
4. Haz clic en **Generar token**
5. **Copia el token** — solo se muestra una vez

### 3. Ejecuta el agente y pega el token

**Windows:** haz doble clic en `WorkDigest-Agent-windows.exe`.

**Linux/macOS:**
```bash
chmod +x WorkDigest-Agent-linux  # o WorkDigest-Agent-macos
./WorkDigest-Agent-linux
```

La primera vez te pide el token: pégalo y pulsa Intro. Se guarda en `~/.workdigest-agent.json`,
con permisos solo para tu usuario, y **no hay que volver a escribirlo**, ni siquiera cuando el
agente se actualiza solo.

**En un servidor sin pantalla o como servicio**, guarda el token en un fichero que solo pueda
leer el usuario que ejecuta el agente y arráncalo así (desde la versión 0.2.16):

```bash
nano /etc/workdigest/token          # pega el token, guarda y sal
chmod 600 /etc/workdigest/token
./WorkDigest-Agent-linux --token-file /etc/workdigest/token
```

> **No escribas el token con `--token`**: lo que va en la línea de órdenes lo ve cualquier
> usuario del equipo en la lista de procesos y se queda en el historial de la terminal. Se sigue
> aceptando por compatibilidad, pero el agente avisa, lo guarda en la configuración y lo quita de
> sus argumentos.

### Tú decides qué ve WorkDigest (desde la versión 0.2.17)

Al configurarlo por primera vez, el agente te pregunta **qué carpetas puede leer** y **con qué
servidores puede conectar** (tu Redmine, tu Jira…). Lo que no esté en esa lista se rechaza, aunque
alguien lo configure en WorkDigest: la lista se guarda en `~/.workdigest-agent.json` y **solo se
cambia en este equipo**.

```bash
WorkDigest-Agent-windows.exe --permisos                                  # ver la lista
WorkDigest-Agent-windows.exe --permitir-carpeta "D:\Proyectos"           # incluye sus subcarpetas
WorkDigest-Agent-windows.exe --permitir-host redmine.miempresa.local
WorkDigest-Agent-windows.exe --permitir-host "*.sharepoint.com"          # todos sus subdominios
WorkDigest-Agent-windows.exe --quitar-carpeta "D:\Proyectos"
WorkDigest-Agent-windows.exe --quitar-host redmine.miempresa.local
```

(En Linux y macOS, igual con `./WorkDigest-Agent-linux` o `./WorkDigest-Agent-macos`.) Los cambios
valen al momento, sin reiniciar el agente. Para SharePoint en la nube permite `graph.microsoft.com`
y `*.sharepoint.com`; para GitHub, `api.github.com`.

Si instalaste el agente antes de la 0.2.17 no tiene lista: sigue leyendo lo que se le pida y lo
avisa al arrancar y en **Mi Puente**. En cuanto permitas la primera carpeta o servidor, empieza a
rechazar todo lo demás.

### Redes con inspección TLS (cortafuegos corporativo)

Si tu organización inspecciona el tráfico HTTPS —FortiGate, Zscaler, Palo Alto y similares—,
el cortafuegos presenta certificados firmados por la autoridad de certificación (CA) de la
empresa. El agente ya confía en los certificados instalados en el sistema operativo, así que si
esa CA está desplegada en el equipo normalmente no hay que hacer nada.

Si aun así falla con errores de certificado, indícale la CA de tu organización (pídesela a tu
departamento de sistemas):

```cmd
WorkDigest-Agent-windows.exe --ca "C:\ruta\ca-corporativa.pem"
```

También se puede indicar con la variable `WD_CA_CERT`, o con `"caCert"` en el fichero de
configuración `~/.workdigest-agent.json` para no repetirla en cada arranque.

Si la ruta no es correcta, el agente **arranca igualmente** y lo dice en el registro: así se
distingue un problema de certificados de cualquier otro fallo de conexión.

### 4. Verifica la conexión

1. Vuelve a **Mi Puente** en WorkDigest
2. Haz clic en **Verificar conexión**
3. El estado debe cambiar a 🟢 Conectado
4. Tu token aparecerá con el indicador verde en la lista de **Mis tokens**

## Mantener el agente activo

El agente debe estar en ejecución mientras uses WorkDigest con fuentes internas. Si el agente se desconecta, los conectores de tipo Filesystem, Redmine on-premise, SharePoint, etc. no podrán sincronizarse.

Para ejecutarlo en segundo plano en Windows puedes usar el Programador de tareas, o en Linux/macOS un servicio systemd o launchd.

## Uso con conectores

Una vez el agente está conectado, puedes configurar conectores en WorkDigest que usen tu agente:

1. Ve a **Conectores** en tu proyecto
2. Crea o edita un conector de tipo Filesystem, Redmine (on-premise), SharePoint, etc.
3. En el campo **Agente a utilizar**, selecciona el agente que acabas de conectar
4. Guarda y usa **Probar conexión** para verificar

## Solución de problemas

**El estado sigue como Desconectado tras ejecutar el agente**

- Verifica que el token es correcto. Para cambiarlo, borra `~/.workdigest-agent.json` y vuelve a abrir el agente: te lo pedirá de nuevo
- Comprueba que tienes conexión a internet
- Asegúrate de que el firewall no bloquea conexiones salientes a `workdigest.io` por el puerto 443

**Error "token inválido"**

- El token solo se muestra una vez al generarlo. Si no lo copiaste, genera uno nuevo desde **Mi Puente** y revoca el anterior.

**«Mi puente no tiene permitido leer la carpeta…» o «…conectar con…»**

- Esa carpeta o ese servidor no están en la lista de este equipo. Si quieres que WorkDigest los
  use, ejecuta en el equipo la orden que aparece en el mensaje (`--permitir-carpeta` o
  `--permitir-host`).

**El conector no encuentra archivos**

- Verifica que la ruta configurada en el conector existe en el equipo donde corre el agente
- En Windows, usa rutas completas: `D:\Proyectos\Cliente A`
- El agente necesita permisos de lectura sobre esa carpeta

## Versiones

Consulta el historial completo en [Releases](https://github.com/work-digest/agent-releases/releases).
