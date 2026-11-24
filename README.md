# U3-S03-A03_Modos_de_compilacion

Práctica de modos de compilación (debug, profile, release) y ambientes (`--dart-define=AMBIENTE=dev|qa|prod`) con la app `ecotrack_modos` (Flutter).

**Entorno de prueba:** Flutter en Windows desktop (`flutter build windows --<modo> --dart-define=AMBIENTE=<amb>`), ejecutando el `.exe` y capturando la salida estándar. El comportamiento de `kDebugMode`, `kProfileMode` y `kReleaseMode` es el mismo que en Android/iOS.

Evidencias (capturas y consola) en la carpeta [`evidencias/`](evidencias/).

## Tabla de experimentos

| # | Experimento | Modo / ambiente | Qué vi en UI | Qué vi en consola |
|---|---|---|---|---|
| 1 | Debug base | `debug`, AMBIENTE=dev | Cinta "DEBUG" en la esquina. Título "DEBUG". Chips `kDebugMode=true`, `kProfileMode=false`, `kReleaseMode=false`. Ambiente DEV en verde azulado, API `https://dev.ecotrack.utez.local/api`, logs verbosos "sí", banner "sí". | Al arrancar: `The Dart VM service is listening on http://127.0.0.1:…`, `[INFO][Arranque] … ambiente=dev` y `[DEBUG][SecretoDemo] token=demo-solo-debug-nunca-en-release`. Botón 1: `[DEBUG][Consumo] usuario=alumno@utez.edu.mx litros=12.5` y `[INFO][Consumo] Registro guardado localmente`. |
| 2 | Ambiente qa o prod | `debug` + AMBIENTE=qa, y `release` + AMBIENTE=prod | QA: mismo layout que debug, pero con ambiente QA en naranja y API `https://qa.ecotrack.utez.edu.mx/api`; el color del tema cambia. PROD (release): ambiente PROD en índigo, API `https://api.ecotrack.utez.edu.mx`, logs "no", banner "no", sin cinta DEBUG. | QA: mismas líneas que dev, con `ambiente=qa`. PROD en release: la consola queda vacía al arrancar y al pulsar el botón 1. **Hallazgo:** el ambiente por sí solo no silencia los logs sensibles. `debug()` solo se apaga con `kReleaseMode`, así que `AMBIENTE=prod` en modo debug seguiría imprimiendo el correo. |
| 3 | Profile | `profile`, AMBIENTE=dev | Título "PROFILE". Chips `false / true / false`. Sin cinta DEBUG. | Arranque: VM service y `[INFO][Arranque] … ambiente=dev`. **No** aparece `SecretoDemo` (protegido por `kDebugMode`). Botón 1: **sí** imprime `[DEBUG][Consumo] usuario=alumno@utez.edu.mx …`. Profile filtra menos que release. |
| 4 | Release | `release`, AMBIENTE=dev y prod | Título "RELEASE". Chips `false / false / true`. Sin cinta DEBUG ni VM service. | Release dev: solo `[INFO][Arranque]` al arrancar y `[INFO][Consumo] Registro guardado localmente` en el botón 1; nunca `[DEBUG]`. Release prod: ningún `[INFO]`. |
| 5 | Logs seguros | Comparación de los tres modos | El botón 1 solo agrega "Consumo simulado" a la bitácora; los logs no se ven en pantalla. | El token solo sale en debug. El correo sale en debug y en profile. En release no sale ni token ni correo. `LoggerSeguro` evita la fuga en release, pero no en profile ni en prod ejecutado en debug. |
| 6 | Fallo controlado | **Reproducir:** botón 2 con `personas = 0`. **Diagnosticar:** `StateError` en `Diagnostico.dividirLitros`. **Corregir:** validar antes de llamar y mostrar un mensaje usable. | Diálogo "Análisis del fallo" con reproducción, diagnóstico y corrección. La bitácora muestra "Fallo capturado: Bad state: No se puede repartir 10.0 L entre 0 personas (división inválida)." | Debug y profile: `[ERROR][FalloControlado] Bad state: No se puede repartir 10.0 L entre 0 personas …` más stack trace (`diagnostico.dart:8`, `main.dart:64`). Release (dev y prod): solo `[ERROR][FalloControlado] StateError`, sin mensaje ni stack. |

### Diagnóstico del fallo (experimento 6)

1. **Reproducción:** pulsar "2 · Provocar fallo controlado". `main.dart` llama a `dividirLitros(total: 10, personas: 0)` con el valor fijo `0`.
2. **Diagnóstico:** se lanza `StateError` en `lib/diagnostico.dart` (`dividirLitros`). La causa raíz es que nadie valida el divisor antes de operar. El stack trace de debug apunta a la línea exacta; en release se pierde ese detalle, y por eso conviene reproducir en debug.
3. **Corrección (A04):** validar `personas > 0` en el dominio o en la UI y mostrar un mensaje claro al usuario, en lugar de dejar subir el error.

## Evidencias

Para cada variante hay tres capturas (`_1_inicio`, `_2_consumo`, `_3_fallo`) y un archivo `_consola.txt` con la salida estándar:

`debug_dev`, `debug_qa`, `profile_dev`, `release_dev`, `release_prod`.

Los archivos `_consola.txt` muestran algunos caracteres con acentos deformados (`��`) porque la consola de Windows usa otra codificación que el código de Dart. No afecta a la app: en pantalla el texto se ve bien.

## Cómo reproducir

```bash
cd ecotrack_modos
flutter run                                        # debug, dev
flutter run --dart-define=AMBIENTE=qa              # debug, qa
flutter run --profile                              # profile
flutter run --release --dart-define=AMBIENTE=prod  # release, prod
```
