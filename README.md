# AION2 Macro

Utilidad de **comodidad** para **AION2** (no es AFK, no automatiza el juego).

Mientras **sostienes un botón del mouse** (izquierdo o derecho) apuntando a un enemigo, el programa pulsa automáticamente **2 teclas** que tú elijas (por ejemplo `R` y `Q`) y las mantiene mientras el botón siga pulsado. Así el *"click → target → skill"* funciona sin tener que pulsar las skills a mano.

> **Filosofía / ToS:** pensado para usarse **mientras juegas activamente**. La política de NCSoft **tolera** el *skill-weaving* manual asistido; lo que **banea** es AFK, scripts, automatización total y RMT. **Nunca lo dejes en AFK.**

![Captura del programa](screenshot.png?v=0.3.0)

---

## Características

- **CLICK sostenido → teclas** (configurables), con umbral de tiempo y separación entre teclas.
- **Clics como teclas**: además de teclas, puedes asignar **CLICK IZQ.** / **CLICK DER.** en la secuencia (mismo driver HID).
- **Modos por tecla**: *Mantenido* / *Repetir* / *Una vez*, con **delay** configurable hacia la siguiente tecla.
- **Tooltips y guardado automático**: los botones muestran su función al pasar el mouse y los cambios se guardan solos.
- **Idioma en vivo**: **Español / English** (cambia sin reiniciar).
- **Salida por FakerInput** (driver HID firmado): el input sale como dispositivo real (`flags = 0x0`), por eso el anticheat de AION2 lo acepta (a diferencia de AHK / `SendInput`).
- **Solo en combate**: opción para que las teclas salgan **solo cuando el cursor está oculto** (combate), y no en menús o chat.
- **Perfiles** ilimitados (uno por clase) + **compartir perfiles** (Exportar / Cargar `profiles.json`).
- **Detección automática de clase** (experimental, **off** por defecto): lee tu clase de los **paquetes de AION2** (solo lectura, sin inyectar) y cambia solo al perfil de esa clase.
- **Actualizaciones automáticas**: al abrir, comprueba si hay versión nueva y se actualiza **sola**, conservando tus perfiles.
- Tema claro/oscuro, tecla de pánico (**END** por defecto).

---

## Requisitos

1. **Windows 10/11 x64** y permisos de **Administrador**.
2. **FakerInput** (driver firmado) — *no* viene incluido: https://github.com/Ryochan7/FakerInput
3. **Npcap** — solo para la detección automática de clase: https://npcap.com/ (suele venir ya con los medidores de DPS).

---

## Instalación

1. Descarga el ZIP de la **[última versión](https://github.com/crialero/aion2-macro/releases/latest)**.
2. Descomprímelo.
3. Entra a la carpeta **`AION2Macro`** y ejecuta **`AION2Macro.exe`** (acepta el UAC).
   - **No** saques el `.exe` de la carpeta (necesita la subcarpeta `_internal`).
   - La primera vez se crea solo `profiles.json` (tus ajustes).

---

## Uso

1. Elige un **perfil** (o crea/renombra).
2. Ajusta: botón del mouse, teclas (1ª y 2ª), umbral de click, separación, etc.
3. Pulsa **Guardar** y luego **▶ Iniciar**.
4. Juega: al sostener el botón del mouse, salen las teclas.
5. **Pánico: END** (activa/desactiva el macro).

> **El host debe ejecutarse como Administrador** para poder leer el estado del click cuando AION2 tiene el foco (el juego corre en integridad alta y UIPI lo bloquea si no). El `.exe` ya trae el manifiesto `--uac-admin`.

---

## Perfiles

- Se guardan en **`profiles.json`** junto al `.exe` (o en `%APPDATA%\AION2Macro` si la carpeta es de solo lectura).
- **Regla:** cada clase real (Ranger, Cleric, …) solo puede estar en **un** perfil.
- **Compartir ajustes:** botón **Exportar perfiles** (genera un `profiles.json`) y **Cargar perfiles** (importa el de un amigo).

---

## Detección automática de clase

- Casilla **"EXPERIMENTAL: detección automática de clase"** (viene **desactivada**).
- **No depende de ningún otro programa** (ni de medidores de DPS): lee tu clase **directamente de los paquetes de AION2**, igual que un medidor, pero sin inyectar ni leer memoria.
- **Detecta solo**: el proceso del juego, el **adaptador** de red por el que sale y los **puertos** que usa (no asume un puerto fijo). Por eso funciona aunque uses **ExitLag/VPN** (captura el adaptador **Loopback**) o aunque el puerto sea distinto en tu PC.
- Para que detecte: entra a AION2 y **cambia de zona, teletranspórtate o cambia de personaje** una vez.
- **Fuente de captura:** `Automático (detecta el juego)` (recomendado), `Red (NIC)`, `Loopback (ExitLag/VPN)`, `Ambas`.

---

## Actualizaciones automáticas

- Al **abrir**, el macro consulta un manifiesto y, si hay versión nueva, la **descarga**, verifica su **SHA-256**, la **instala sola** y se reinicia.
- **Tus perfiles se conservan** (el actualizador excluye `profiles.json`).
- Puedes activar/desactivar con la casilla **"Buscar actualizaciones al iniciar"** y forzarlo con **"Buscar actualización ahora"**.

---

## Problemas frecuentes

- **El macro no hace nada al hacer click:** asegúrate de ejecutarlo **como Administrador** (si no, no lee el click con AION2 en foco).
- **No salen las teclas en el juego:** revisa que **FakerInput** esté instalado y que el perfil esté en **▶ Iniciar**.
- **No detecta mi clase:** necesita **Npcap**; cambia de zona/personaje una vez; si usas VPN/ExitLag deja la fuente en **Automático**.
- **Windows Defender marca el `.exe`:** es un falso positivo típico de PyInstaller. La **versión carpeta** casi nunca se marca. Si pasa, "Más información → Ejecutar de todas formas".

---

## Cómo funciona (técnico, resumen)

- **GUI:** CustomTkinter. **Motor:** hilo a 8 ms que lee el click (`GetAsyncKeyState`), la ventana en foco (`GetForegroundWindow`) y la visibilidad del cursor (`GetCursorInfo`).
- **Salida:** FakerInput (HID real, `VID 0xFE0F / PID 0x00FF`).
- **Detección:** Npcap **pasivo** + API de Windows (`Toolhelp32` + `GetExtendedTcpTable`) para localizar el proceso/puertos del juego; parsea el protocolo binario de AION2 y lee el *self record* (opcode `33 36`) para obtener nombre/servidor/clase.
- **Empaquetado:** PyInstaller (`--onedir`, sin cifrado ni UPX, para minimizar falsos positivos de AV).

---

## Estructura del repositorio

```
aion2-macro/
├─ version.json        <- manifiesto para las actualizaciones automáticas
├─ screenshot.png      <- captura del programa
└─ README.md
```

Las versiones (ZIP) se publican en la pestaña **Releases**.

---

## Descargas

- **Última versión:** https://github.com/crialero/aion2-macro/releases/latest
- Historial de versiones: pestaña **Releases**.

---

## Versionado

`0.MENOR.PARCHE`
- Cambios **grandes** → sube el segundo número (`0.1` → `0.2`).
- Cambios **pequeños** → sube el parche (`0.1.0`, `0.1.1`, `0.1.2`…).

---

## Créditos

Desarrollado por **Crialero**. Herramienta de comodidad para uso personal mientras se juega activamente.

- FakerInput: https://github.com/Ryochan7/FakerInput
- Npcap: https://npcap.com/
