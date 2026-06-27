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

### 3. Ejecuta el agente

**Windows:**
```cmd
WorkDigest-Agent-windows.exe --token TU_TOKEN_AQUÍ --server wss://workdigest.io/agent-ws
```

**Linux/macOS:**
```bash
chmod +x WorkDigest-Agent-linux  # o WorkDigest-Agent-macos
./WorkDigest-Agent-linux --token TU_TOKEN_AQUÍ --server wss://workdigest.io/agent-ws
```

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

- Verifica que el token es correcto (cópialo de nuevo desde **Mi Puente**)
- Comprueba que tienes conexión a internet
- Asegúrate de que el firewall no bloquea conexiones salientes a `workdigest.io` por el puerto 443

**Error "token inválido"**

- El token solo se muestra una vez al generarlo. Si no lo copiaste, genera uno nuevo desde **Mi Puente** y revoca el anterior.

**El conector no encuentra archivos**

- Verifica que la ruta configurada en el conector existe en el equipo donde corre el agente
- En Windows, usa rutas completas: `C:\Users\jorge\Documentos\Proyectos`
- El agente necesita permisos de lectura sobre esa carpeta

## Versiones

Consulta el historial completo en [Releases](https://github.com/work-digest/agent-releases/releases).
