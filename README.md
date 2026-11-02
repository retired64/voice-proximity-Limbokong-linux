# Limbokong's Proximity Voice — Wine portable (Linux x86_64)

Paquete **no oficial** para Linux que ejecuta el cliente original de Windows de
**Limbokong's Proximity Voice** con un runtime **Wine 11.21 `amd64-wow64`**
incluido. No hace falta instalar Wine en el sistema.

> Este repositorio sólo contiene este README. El ejecutable está en
> **[Releases](../../releases)** como `LimbokongVoice-1.2.0-wine-x86_64.tar.zst`
> (175 MB).
>
> No es el port nativo en C++. Es el cliente Windows original bajo Wine.
> No está afiliado a Limbokong, Nintendo, Unity, Photon ni Firebase.

## Descarga

- Asset: `LimbokongVoice-1.2.0-wine-x86_64.tar.zst`
- Tamaño: **175 MB**
- SHA-256:

```text
fd28ba4b8f68665088c3b7b91a69a4495cebbdc129a53f3f44f934c0a0448d85
```

Verificación:

```sh
sha256sum -c LimbokongVoice-1.2.0-wine-x86_64.tar.zst.sha256
```

## Qué incluye

- Cliente original `Limbokong's Proximity Voice.exe` y sus datos de Unity.
- Wine 11.21 `amd64-wow64` (sin multilib).
- Lanzador `run.sh` que:
  - resuelve la carpeta de guardados del juego;
  - espeja el `.sav` del mod dentro del prefijo Wine;
  - siembra la ruta personalizada del cliente;
  - filtra el ruido de Unity en la consola y escribe logs.

**No incluye** el juego sm64coopdx, el mod, ni tu partida.

## Requisitos

- Linux **x86_64** con glibc 2.25 o superior (Ubuntu 18.04+, Debian 10+,
  Fedora 28+).
- Sesión gráfica **X11** o **Wayland con XWayland**.
- **PulseAudio** o **PipeWire** con compatibilidad PulseAudio.
- Driver gráfico con OpenGL/Vulkan.
- `sm64coopdx` con el mod **Limbokong's Proximity Voice** activo.
- **~1.5 GB libres**: el primer arranque crea un prefijo Wine de ~700 MB.

Probado en Fedora 44 con Xvfb y en escritorio Wayland/XWayland.

## Instalación

Extrae en una carpeta **escribible** (por ejemplo, tu home). No lo extraigas en
`/opt` ni en una ruta de sólo lectura: Wine necesita escribir su prefijo dentro
de la carpeta.

```sh
mkdir -p ~/Juegos
tar --zstd -xf LimbokongVoice-1.2.0-wine-x86_64.tar.zst -C ~/Juegos
cd ~/Juegos/LimbokongVoice-1.2.0-wine-x86_64
```

## Uso

```sh
./run.sh --game-data "$HOME/.local/share/sm64coopdx" --windowed
```

- La primera vez crea el prefijo Wine (puede tardar 1–2 minutos).
- Las siguientes veces arranca directo.
- `--windowed` evita problemas de pantalla completa en XWayland.
- Sin `--game-data`, el lanzador busca en
  `~/.local/share/{sm64coopdx,sm64ex-coop,coopdx}/sav/`.

Opciones útiles:

```text
--game-data PATH       raíz del juego o el .sav
--windowed             Unity en ventana
--print-paths          muestra las rutas resueltas y sale
--prepare-only         crea el prefijo y sale (sin abrir ventana)
--recreate-prefix      borra y recrea el prefijo (no toca el juego)
--allow-missing-save   arranca aunque falte el .sav del mod
--strict-save          aborta si falta el .sav del mod
--raw-output           consola sin filtro de Unity
--help                 ayuda completa
```

## El guardado: por qué la UI muestra una ruta de Windows

El juego de Linux guarda en:

```text
~/.local/share/sm64coopdx/sav/limbokongs-voicechat.sav
```

El cliente es una aplicación Windows y no puede leer esa ruta tal cual. Por eso
el lanzador:

1. Lee el `.sav` real del juego.
2. Lo copia cada 100 ms a las tres carpetas que el cliente conoce dentro del
   prefijo:
   - `%APPDATA%\sm64ex-coop\sav\`
   - `%APPDATA%\sm64coopdx\sav\`
   - `%APPDATA%\coopdx\sav\`
3. Siembra `CustomFilePath` apuntando a la copia de `sm64coopdx`.

Por eso SETTINGS → GENERAL muestra algo como:

```text
C:\users\<usuario>\AppData\Roaming\sm64coopdx\sav\limbokongs-voicechat.sav
```

**Eso es normal**: es el espejo dentro del prefijo, no la ruta real de Linux.

- El `.sav` real **no se borra ni se modifica**.
- Si el cliente borra su copia, el lanzador la recrea.
- Si el `.sav` no existe, el cliente abre igual pero sin voz. Para crearlo,
  juega una partida en sm64coopdx con el mod activo y vuelve a lanzar.

## Voz

- **HOST** crea la sala de voz de Photon; **JOIN** entra a una existente.
- Todos los que quieran hablar deben ejecutar este cliente.
- Las distancias, nombres y efectos (cap/wall) los aporta el mod a través del
  `.sav`; el cliente de voz sólo los lee.
- Wine expone el micrófono a 48 kHz y el cliente remuestrea a 24 kHz.

## Problemas frecuentes

| Síntoma | Solución |
|---|---|
| No aparece la ventana | Añade `--windowed` |
| “falta el guardado del mod” | Juega una partida con el mod activo; luego relanza |
| No hay audio | Comprueba PulseAudio/PipeWire y el dispositivo por defecto |
| Quiero ver las rutas | `./run.sh --print-paths` |
| Prefijo corrupto | `./run.sh --recreate-prefix` |
| Ver logs | `var/logs/` y `var/logs/Player-last.log` |

Defectos conocidos del cliente original, no del paquete:

- `NullReferenceException` en `MicrophoneSelector.Awake` (campos vacíos del
  prefab). El `Recorder` principal sigue funcionando.
- `Player 0 not found!` con un guardado de un solo jugador.
- El cliente borra el `.sav` que encuentra al entrar a `Game`; el espejo evita
  que eso afecte a tu partida real.

## Desinstalar

Borra la carpeta extraída. El prefijo Wine vive dentro de ella; no se toca el
juego ni tu `.sav` real.

## Aviso legal

- Proyecto **no oficial**, sin afiliación con el autor del mod, Nintendo, Unity,
  Photon o Firebase.
- El paquete incluye el cliente original de Windows y runtimes de terceros. Los
  derechos pertenecen a sus autores.
- No se incluyen ROMs, assets del juego ni partidas.
- Se distribuye **tal cual**, sin garantía.
- Si eres titular de derechos y quieres que se retire algún contenido, abre un
  issue en este repositorio.

## Créditos

- Cliente y mod originales: **Limbokong**.
- Wine: proyecto Wine / build portátil de Kron4ek.
- Empaquetado y lanzador: este repositorio.
