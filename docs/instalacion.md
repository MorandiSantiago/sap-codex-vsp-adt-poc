# Instalación y configuración: Codex + VSP + SAP ADT

Este procedimiento reconstruye la prueba documentada en la guía privada del autor. Los comandos y nombres de parámetros se limitan a los registrados allí. Sustituí los valores ficticios por datos aprobados de tu propio entorno; no copies configuraciones de terceros ni guardes secretos en el repositorio. La configuración pública de partida está en [`examples/config.toml.example`](../examples/config.toml.example).

## 1. Preparar VS Code

1. Instalá Visual Studio Code y, desde Extensions (`Ctrl + Shift + X`), la extensión **Codex** publicada por OpenAI. Abrí su panel e iniciá sesión.
2. Instalá **ABAP Remote Filesystem** (Marcello Urbani). La guía también instaló **ABAP CDS Language Support** (Frederik Hudák); ABAP Cleaner, ABAP Snippets y ABAP Syntax Highlighting quedaron como opcionales.
3. Abrí un workspace de trabajo separado. Definí reglas de revisión y autorización antes de permitir escrituras: comenzar en solo lectura, mostrar los cambios propuestos, no modificar objetos estándar o productivos y no liberar transportes desde el asistente.

## 2. Instalar VSP

1. Descargá el ejecutable para tu plataforma desde las [releases oficiales de VSP](https://github.com/oisee/vibing-steampunk/releases). En la prueba histórica se usó el binario de Windows x64, se renombró `vsp.exe` y se ubicó en una carpeta local de herramientas.
2. Agregá esa carpeta al `Path` de usuario, reiniciá VS Code y verificá desde PowerShell o CMD:

   ```powershell
   vsp --version
   ```

3. Confirmá la versión mostrada. Si Windows oculta extensiones de archivo, comprobá que el ejecutable no haya quedado con una extensión duplicada.

## 3. Preparar acceso SAP y ADT

Solicitá un endpoint ADT, usuario, mandante, conectividad de red y autorizaciones aprobados para un entorno de desarrollo o capacitación. La guía usó una red privada y un certificado interno. Para una instalación nueva, pedí al equipo responsable una cadena de confianza válida y configurá cada cliente según su soporte documentado; verificá TLS antes de usar datos reales.

ABAP Remote Filesystem se configuró en las preferencias de usuario de VS Code mediante `abapfs.remote`, con los campos `url`, `username`, `password`, `client`, `language` y `customCA`. Se dejó `password` vacío para que la extensión solicitara la contraseña al conectar. La guía también registró `http.proxySupport = "off"` y `allowSelfSigned = true` por las condiciones de ese entorno: son decisiones históricas, **no ajustes generales recomendados**. `customCA` correspondía a ABAP Remote Filesystem; la guía no confirma un parámetro equivalente de CA para VSP.

Para comprobar la conectividad de red, la guía utilizó `Test-NetConnection` con el host y puerto de ADT. Usá solo valores autorizados de tu entorno:

```powershell
Test-NetConnection sap.example.invalid -Port 44300
```

En la prueba, un error de autorización sobre un recurso ADT confirmó que la conexión y autenticación llegaban a SAP, pero faltaba un permiso. La lectura se repitió tras la habilitación por el equipo responsable y resultó exitosa.

## 4. Registrar VSP como servidor MCP de Codex

La guía registró el servidor en el archivo de configuración de Codex del usuario (`config.toml`) y usó `stdio` como transporte. Copiá el [ejemplo público](../examples/config.toml.example) a tu configuración personal y reemplazá **solo** las rutas y valores ficticios por los autorizados. El ejemplo mantiene:

- `--mode hyperfocused` y `--transport stdio`, tal como figuran en la guía.
- `--read-only=true`, `--allow-transportable-edits=false` y `--enable-transports=false` para comenzar sin escritura ni gestión de transportes.
- `env_vars = ["SAP_PASSWORD"]` para pasar una variable de entorno a VSP sin escribir el valor en TOML.

En Windows, la guía creó `SAP_PASSWORD` como variable de entorno de usuario, reinició VS Code y comprobó únicamente su presencia:

```powershell
[bool]$env:SAP_PASSWORD
```

El resultado esperado es `True`; no imprimas ni registres el valor. En una instalación nueva, seguí las reglas de gestión de secretos de tu organización. No pongas contraseñas en `config.toml`, `settings.json`, el workspace ni este repositorio.

La prueba histórica añadió `--insecure` porque VSP no reconocía el certificado interno. **El ejemplo público omite esa opción.** La guía no demuestra un método seguro concreto para configurar la confianza TLS de VSP, así que ese paso debe resolverse con el equipo de infraestructura y la documentación vigente de VSP antes de continuar.

## 5. Validar lectura antes de escribir

Reiniciá VS Code para que tome el `Path`, la variable de entorno y la configuración MCP. Confirmá que el servidor VSP esté disponible y pedí a Codex una consulta mínima de lectura, con una instrucción explícita de no crear, activar ni modificar objetos. Después, buscá y leé un objeto permitido. La guía registró búsqueda de objetos ABAP y lectura de un programa existente; no hubo cambios en esta fase.

ABAP Remote Filesystem también permitió navegar objetos como un sistema de archivos virtual en VS Code. Esos objetos representan contenido SAP activo; no son copias locales. La extensión y VSP usan sus propias configuraciones de conexión, aunque ambos acceden a ADT.

## 6. Habilitar escritura restringida solo con autorización

La guía cambió `--read-only` a `false` para una primera prueba limitada a un paquete temporal mediante `--allowed-packages`, conservando `--allow-transportable-edits=false` y `--enable-transports=false`. Esa prueba quedó **parcial**: faltó autorización de desarrollo para guardar una instrucción adicional.

Una segunda prueba usó un paquete de capacitación y una orden de transporte previamente autorizados. La configuración restringió la escritura con `--allowed-packages`, `--allow-transportable-edits=true` y `--allowed-transports`, mientras `--enable-transports=false` mantuvo deshabilitada la gestión de transportes desde VSP. El valor permitido fue la **orden principal**: al usar una tarea asociada, SAP rechazó el guardado. No reproduzcas ningún paquete, objeto u orden de la prueba histórica; obtené los tuyos mediante el proceso de autorización de tu organización.

Antes de cualquier escritura, revisá el objeto, paquete, orden, permisos y diff propuesto. La habilitación de opciones MCP no concede por sí misma autorizaciones SAP.

## 7. Verificar el resultado

En la segunda prueba se creó un programa ABAP didáctico. Se ejecutó la verificación de sintaxis sin errores, se activó el objeto y se volvió a leer el código activo para confirmar que coincidía con lo aprobado. No se modificaron otros objetos ni se crearon o liberaron transportes desde Codex.

La guía no documenta una modificación completa de código empresarial existente, ejecución de ATC o tests automatizados, ni un despliegue productivo. Esas actividades requieren sus propios permisos, controles y evidencias.

## Resumen de configuración histórica y criterio actual

| Tema | En la prueba documentada | Para una instalación nueva |
| --- | --- | --- |
| Acceso inicial | Solo lectura | Empezar en solo lectura y validar consultas mínimas. |
| Credencial | Variable de entorno para VSP; solicitud interactiva en ABAP Remote Filesystem | Aplicar la gestión de secretos aprobada; no guardarla en archivos publicables. |
| Certificado | Excepción TLS para VSP y ajuste de certificado en ABAP Remote Filesystem | Establecer confianza TLS con infraestructura; no copiar la excepción histórica. |
| Escritura | Primero un intento parcial; después un paquete y una orden permitidos | Habilitarla solo con autorización, alcance preciso y revisión. |
| Transportes | Edición transportable restringida; gestión de transportes deshabilitada | Seguir el proceso SAP de la organización. |
