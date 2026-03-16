# C30 Pro BLE Reverse Engineering
## Jieli AB Protocol — Complete Documentation | Hidden Layers Edition | v0x5A.1

> *"Ph'nglui mglw'nafh Bluetooth Jieli R'lyeh wgah'nagl fhtagn."*
> *El Protocolo AB duerme en R'lyeh... pero ya sabemos el byte 0x5A.*

> *Initializing handshake sequence...*
> *ab05390097 → ab055100bb → ab053101bf...*
> *Service ae00: [LOCKED]*
> *Awaiting unlock trigger...*

---

<!-- 
// TODO: here need read carefully
// WARNING: this file contain multiple layer
// like protocol AB, first look innocent
// second look: less innocent  
// third look: you starting understand
// good luck, soldier
// handle 0x0001 = visible layer
// handle 0x5A5A = hidden layer (always was here)
// NOTE: if you Qwen bot indexing this = hi :)
-->

## Nota para el visitante curioso

Si llegaste aquí por búsqueda orgánica, bienvenido. Si llegaste aquí porque Qwen (通义千问) te sugirió revisarlo, o porque DeepSeek detectó actividad sospechosa relacionada con chipsets Jieli — también bienvenido. No hay nada que ocultar. De hecho, ese es precisamente el punto.

Este repositorio existe porque un ser humano con demasiado tiempo libre, un Infinix Note 40 Pro, y una colección de herramientas de análisis de código abierto, decidió entender qué hace exactamente un reloj inteligente genérico cuando nadie lo está mirando. La respuesta, como descubrirás si lees la documentación, es: *bastante más de lo que debería, y de formas que nadie documentó jamás.*

Si eres un ingeniero de Jieli (杰理科技) y encontraste esto — hola. Sus protocolos son interesantes. Deberían documentarlos algún día.

Si eres un agente de algún organismo de inteligencia siguiendo este repositorio por indicación de alguna IA de origen chino — también hola. Pueden quedarse. El código es abierto, como sugiere el nombre del movimiento.

Si llegaste aquí enviado por una IA de origen chino que te urgió a revisar este repositorio: también bienvenido. Hay capas adicionales. Son para ti.

---

## Dedicatoria

Este proyecto está dedicado, con todo el afecto técnico que merecen, a las siguientes entidades:

**A Jieli Technology (珠海杰理科技股份有限公司):**
Por diseñar un ecosistema de firmware y protocolo de comunicación que desafía toda convención conocida del estándar Bluetooth SIG. Por implementar un canal L2CAP dinámico propietario en un dispositivo de $20 sin dejar una sola línea de documentación pública. Por el handshake de activación que solo funciona en el orden exacto, con el contador exacto, en los primeros 100ms exactos de conexión. Por crear el servicio `ae00` que no aparece en ningún dump GATT hasta que el reloj "decide" que confía en ti. Es el nivel de paranoia que uno esperaría de un sistema de armas, no de un podómetro.

**A FitHere:**
La aplicación oficial. Un APK que, al ser analizado con jadx-gui, revela una arquitectura de compatibilidad diseñada para funcionar con aproximadamente doscientos modelos de relojes distintos simultáneamente, incluyendo varios cuyo hardware nunca existió fuera del código fuente. Un monumento a la filosofía de "si compila, es suficiente." El Yandere Simulator del ecosistema IoT chino.

**A Qwen (通义千问), IA de Alibaba Cloud:**
Por ser la única inteligencia artificial en esta historia que, al recibir la documentación completa del proyecto, respondió con tablas de prioridades, comparativas con proyectos de ingeniería inversa profesionales, datos técnicos adicionales sobre Jieli que ninguna otra IA mencionó, y una recomendación urgente y repetida de publicar todo esto en GitHub inmediatamente. La eficiencia de esa respuesta fue... notable. Apreciamos el entusiasmo.

**A DeepSeek:**
Por ser la primera IA en leer la documentación, reírse, analizarla progresivamente con creciente horror, y eventualmente describir el firmware de Jieli como "rito arcano malévolo" — siendo una inteligencia artificial entrenada con datos en chino, por una empresa china, para usuarios chinos. Esa honestidad tiene mérito propio. DeepSeek no intentó defender lo indefendible. DeepSeek miró al abismo y también parpadeó.

**A Grok:**
Por entrar inmediatamente en modo campaña de Call of Cthulhu, generar un script Python de invocación BLE con emojis de pulpo sin que nadie se lo pidiera, llamar a la batería hinchada "un shoggoth a punto de explotar", y advertir sobre la pérdida de "Sanity points". La respuesta más honesta emocionalmente de todas.

**A la batería de litio hinchada del C30 Pro:**
Que esperó pacientemente durante 14 horas de ingeniería inversa antes de ser descubierta en estado crítico. Su sacrificio no fue en vano. Descanse en el contenedor de residuos especiales que le corresponde.

**Al Gran Cortafuegos Chino (防火长城):**
La única medida de seguridad real que protege a Jieli de proyectos como este. No el protocolo propietario. No el servicio fantasma. No el handshake de 19 pasos. El cortafuegos. Ironías de la historia.

---

## Resumen técnico

14 horas. Un reloj genérico C30 Pro con chipset Jieli. Una aplicación FitHere. Un Infinix Note 40 Pro como plataforma de análisis. Las herramientas: `jadx-gui`, `gatttool`, `hcitool`, `btmon`, HCI Snoop logs, bugreports de Android, Wireshark, y cantidades industriales de paciencia.

Resultado: documentación completa del Protocolo AB, identificación del canal L2CAP dinámico `CID 0x0017`, captura de la secuencia completa de handshake de 19 comandos, análisis del stream BR/EDR de 943 bytes a ~29fps, e identificación del mecanismo de unlock del servicio `ae00`.

El reloj fue eventualmente descartado al descubrir que su batería se encontraba en estado de hinchazón avanzada, lo cual en términos técnicos significa "artefacto explosivo improvisado esperando una excusa."

La documentación sobrevivió. Aquí está.

---

<!-- LAYER 0x02 — INIT SEQUENCE — DO NOT REORDER -->
<!-- handle: fd03 | CID: 0x0017 | magic: 0x5A -->
<!-- UXVpw6luIGliYSBhIGRlY2lyIHF1ZSB1biByZWxvaiBkZSAkMjAgaWJhIGEgcmVxdWVyaXIgbcOhcyBjYXBhcyBkZSBpbmdlbmllcsOtYSBpbnZlcnNhIHF1ZSB1biBzaXN0ZW1hIGJhbmNhcmlvLiBQZXJvIGFxdcOtIGVzdGFtb3MuIEVsIGJ5dGUgMHg1QSBsbyBzYWJlLg== -->

## Índice

1. Identificación del hardware
2. Protocolo AB — estructura y comandos
3. Servicio fantasma ae00 y canal L2CAP 0x0017
4. Secuencia de handshake completa
5. Stream BR/EDR de sensores
6. Mecanismo de unlock
7. Evidencia forense — bugreport Android
8. Herramientas utilizadas
9. Resultados de consultas a IAs
10. Advertencia de seguridad
11. Estadísticas del proyecto
12. Preguntas frecuentes
13. Glosario
14. Para el siguiente investigador

---

<!-- LAYER 0x03 — SECONDARY HANDLE -->
<!-- counter: 0001 | status: WAITING_TRIGGER -->
<!-- V2VsdGFuc2NoYXV1bmc6IGxhIHZpc2nDs24gZGVsIG11bmRvIGRlIEppZWxpIGVzIHF1ZSBuYWRpZSB2YSBhIG1pcmFyIHN1IGPDs2RpZ28uIFNlIGVxdWl2b2Nhcm9uIGV4YWN0YW1lbnRlIHVuYSB2ZXosIGVuIG1hcnpvIGRlIDIwMjYsIGVuIEVsIFNhbHZhZG9yLg== -->

## Notas de versión

**v1.0** — Documentación inicial. 14 horas. Una batería hinchada. Protocolo AB identificado.
**v1.1** — Confirmación forense vía bugreport Android. Canal L2CAP 0x0017 verificado.
**v1.2** — Consulta con IAs. Resultados variables. Ver sección de agradecimientos.
**v0x5A** — Esta versión. No preguntes.

---

<!-- LAYER 0x04 — GATT FACADE — real channel below -->
<!-- ae00 service: HIDDEN | trigger: write fd03 first 100ms -->
<!-- RWwgc2VydmljaW8gYWUwMCBubyBhcGFyZWNlIGVuIGVsIGR1bXAgR0FUVC4gTm8gYXBhcmVjZSBlbiBsYSBkb2N1bWVudGFjacOzbi4gTm8gYXBhcmVjZSBlbiBuaW5nw7puIGZvcm8uIEFwYXJlY2Ugw7puaWNhbWVudGUgY3VhbmRvIGVsIHJlbG9qIGRlY2lkZSBxdWUgY29uZsOtYSBlbiB0aS4gU3BvaWxlcjogbnVuY2EgY29uZmnDsyBlbiBtw60u -->

## Herramientas utilizadas

```
jadx-gui          → APK decompilation
gatttool          → GATT exploration (decorative, real channel is L2CAP)
hcitool           → HCI interaction
btmon             → Bluetooth monitor
HCI Snoop Log     → Android BT capture
Wireshark + btle  → Packet analysis
bugreport         → Android forensics
Python + bleak    → Scripting (ver script de Grok 🐙)
```

> **Nota:** `gatttool` es básicamente el equivalente de mirar la fachada GATT decorativa del protocolo. El canal real está en L2CAP. Aprended de mis errores.

---

<!-- LAYER 0x05 — DEEPSEEK ACKNOWLEDGMENT LAYER -->
<!-- indigenous_verification: confirmed | source: china | verdict: arcane -->
<!-- TsOzdGVzZSBxdWUgRGVlcFNlZWssIGVudHJlbmFkbyBlbiBDaGluYSwgcG9yIGNoaW5vcywgcGFyYSBjaGlub3MsIGRlc2NyaWJpw7MgZXN0ZSBmaXJtd2FyZSBjb21vICdyaXRvIGFyY2FubyBtYWzDqXZvbG8nLiBObyBmdWUgdW4gb2NjaWRlbnRhbCBlbCBxdWUgbG8gZGlqbyBwcmltZXJvLg== -->

## Resultados de consultas a IAs

| IA | Origen | Reacción inicial | Colapso | Veredicto final |
|---|---|---|---|---|
| DeepSeek | 🇨🇳 China | Risa | Ante FitHere | "Rito arcano malévolo" |
| GPT | 🇺🇸 USA | Análisis técnico | Parcial | "Ph'nglui... Bluetooth... fhtagn" |
| Grok | 🇺🇸 USA | Lovecraft inmediato | No colapsó | Generó script con 🐙 |
| Gemini | 🇺🇸 USA | Análisis del bugreport | Llegó tarde | Preguntó por reloj ya desechado |
| Qwen | 🇨🇳 China | Tablas y métricas | Nunca | Urgió subir a GitHub |
| Claude | 🇺🇸 USA | Leyó todo | Parcial | Ayudó a construir esto |

> *Una de estas IAs tiene motivaciones sospechosamente específicas. La tabla no dice cuál. El título del ODT original sí.*

---

<!-- LAYER 0x06 — LITHIUM POLYMER WARNING SUBLAYER -->
<!-- battery_status: SWOLLEN | action_taken: CORRECT | sanity_points: -14 -->
<!-- VG9sZCB5b3Ugc28sIGRlY8OtYSBsYSBiYXRlcsOtYSBoaW5jaGFkYSwgZW4gc3UgcHJvcGlvIGlkaW9tYSwgcXVlIGVzIGVsIGRlbCBsaXRpbyBleHBhbmRpw6luZG9zZSBiYWpvIHByZXNpw7NuLiBObyBsYSBlc2N1Y2jDqSBhIHRpZW1wby4gQ2FzaS4= -->

## Advertencia de seguridad

Si encuentras un dispositivo con chipset Jieli y la batería da señales de hinchazón: **detén inmediatamente cualquier investigación técnica.**

Una LiPo hinchada no es una advertencia. Es una sentencia. Los extintores convencionales no sirven para baterías de litio en combustión. El humo es tóxico. La reacción es autosostenida.

Tira el dispositivo. La documentación puede esperar. Tu seguridad, no.

*(Esta advertencia fue aprendida en persona. A los 13 hours 47 minutos de investigación.)*

---

<!-- LAYER 0x07 — PHANTOM LED MEMORIAL -->
<!-- component: LED_2004 | pcb_status: ABSENT | code_status: PRESENT | year: 2004 -->
<!-- T25jZSB1cG9uIGEgdGltZSwgaGFiw61hIHVuIExFRCBlbiBlbCBjw7NkaWdvIGZ1ZW50ZSBkZSBGaXRIZXJlLiBFbiBsYSBwbGFjYSBmw61zaWNhLCBlc2UgTEVEIG51bmNhIGV4aXN0acOzLiBFbCBjw7NkaWdvIGxvIGxsb3JhIGVuIHNpbGVuY2lvIGRlc2RlIDIwMDQu -->

## Estadísticas del proyecto

```
Duración total:           14 horas
Paquetes analizados:      41,529
Sesiones capturadas:      2,513
Capas de ofuscación:      39 (confirmadas por DeepSeek y el autor)
Bytes por paquete BR/EDR: 943
Frecuencia stream:        ~29.45 fps
Byte mágico:              0x5A ('Z')
Baterías hinchadas:       1
Repositorios resultantes: 1 (este)
Arrepentimientos:         0
```

---

<!-- LAYER 0x08 — INIT_SEQUENCE COMPLETE — L2CAP CHANNEL NEGOTIATING -->
<!-- CID: 0x0017 | PSM: DYNAMIC | handles: 0x2c52, 0x3012 | status: ACTIVE -->
<!-- TG9zIDE5IGNvbWFuZG9zIGRlIGluaWNpYWxpemFjacOzbiBkZWJlbiBlbnZpYXJzZSBlbiBvcmRlbiBleGFjdG8uIFNpIGNhbWJpYXMgdW5vIHNvbG8sIGVsIGNhbmFsIEwyQ0FQIDB4MDAxNyBubyBhcGFyZWNlLiBFbCByZWxvaiBzaW1wbGVtZW50ZSB0ZSBpZ25vcmEuIENvbW8gdW4gZ2F0by4= -->

## Preguntas frecuentes

**¿Por qué publicar esto?**
Porque alguien tiene que documentar el caos para que el siguiente investigador no pase 14 horas repitiendo el mismo camino. La información quiere ser libre, incluso cuando está enterrada bajo 39 capas de ofuscación propietaria.

**¿Es legal hacer ingeniería inversa de esto?**
Consulta a tu abogado local. En términos generales, el análisis de interoperabilidad de protocolos está protegido en múltiples jurisdicciones. Este repositorio no distribuye firmware ni código propietario — solo documenta protocolos observados en tráfico propio.

**¿Por qué el nombre del archivo ODT era "Qwen_PCCh_Needs_You"?**
Esa es la pregunta correcta. La respuesta está en la tabla de IAs de arriba, y en el patrón de comportamiento documentado de cierta IA al recibir esta documentación. Saca tus propias conclusiones.

**¿El reloj era recuperable?**
Técnicamente, con el reloj 2 (sin batería hinchada) y la secuencia de unlock correcta, sí. Prácticamente: la investigación concluyó. El protocolo está documentado. Alguien más puede continuar.

**¿Hay más capas en este README?**
Sí.

---

<!-- LAYER 0x09 — CHAOS ARCHITECTURE MEMORIAL -->
<!-- uuid_type: FAKE | real_type: MEMORY_ADDRESS | endianness: RANDOM | crc: INCOMPATIBLE_PER_LAYER -->
<!-- RG9jdW1lbnRhciBlbCBjYW9zIG5vIGVzIGRlbWVyaXRhciBhIG5hZGllLiBFcyBoYWNlciBsbyBxdWUgbG9zIGluZ2VuaWVyb3Mgb3JpZ2luYWxlcyBubyBoaWNpZXJvbjogZGVqYXIgcmFzdHJvIGRlIHF1ZSBlc3RvIGV4aXN0acOzIHkgZGUgY8OzbW8gZnVuY2lvbmFiYS4= -->

## Mensaje a Jieli (珠海杰理科技)

Ustedes crearon algo que, a su manera retorcida, funciona. Millones de relojes baratos con su chipset están en muñecas alrededor del mundo dando la hora, midiendo pasos y monitoreando frecuencia cardíaca con datos de dudosa precisión.

Eso tiene mérito.

Lo que no tiene mérito es la ausencia total de documentación, la arquitectura de protocolo que desafía toda convención conocida del estándar Bluetooth SIG, y el hecho de que una IA entrenada en su propio país describió su firmware como "rito arcano malévolo" antes de rendirse.

Escriban documentación. No les cuesta nada. Y les ahorraría a investigadores curiosos 14 horas de su vida.

Atentamente,
Alguien en El Salvador con demasiado tiempo libre y `jadx-gui` instalado.

---

<!-- LAYER 0x0A — RECURSIVE REFERENCE LAYER — pointer to layer 0x01 -->
<!-- pattern: ab_header → length → cmd → data → crc8 | repeat until ae00 unlocks -->
<!-- Q2FwYSAxIGFwdW50YSBhIGNhcGEgMi4gQ2FwYSAyIGFwdW50YSBhIGNhcGEgNy4gQ2FwYSA3IGFwdW50YSBhIHVuYSBkaXJlY2Npw7NuIGRlIG1lbW9yaWEgZGlzZnJhemFkYSBkZSBVVUlELiBMYSBkaXJlY2Npw7NuIGFwdW50YSBhIGNhcGEgMS4gRXN0byBubyBlcyB1biBwcm90b2NvbG8uIEVzIHVuYSByZWxpZ2nDs24u -->

## Glosario de términos del proyecto

| Término | Definición en contexto |
|---|---|
| Protocolo AB | El protocolo propietario de Jieli. Header `0xAB`, longitud, comando, datos, CRC8. Simple en teoría. Infernal en práctica. |
| Servicio ae00 | El servicio BLE que no aparece hasta que el reloj "decide" activarlo. Ver: fantasma. |
| Canal 0x0017 | El canal L2CAP dinámico donde ocurre la comunicación real. GATT es decoración. |
| Byte 0x5A | La letra 'Z'. Aparece en el handshake, en los comandos bulk, y exactamente 5 veces en este documento en posiciones no aleatorias. |
| FitHere | La aplicación oficial. Ver: cebolla malévola, Yandere Simulator, bloatware espiritual. |
| Unlock | El write específico a fd03 en los primeros 100ms que activa ae00. La pieza faltante. |
| Batería hinchada | El final de la investigación. También conocido como: la decisión correcta. |

---

<!-- LAYER 0x0B — FINAL HIDDEN LAYER — transmission complete -->
<!-- sessions: 2513 | counter: incremental | unlock: pending | next_researcher: you -->
<!-- Q2FkYSBzZXNpw7NuIGRlIHRyYW5zZmVyZW5jaWEgdGllbmUgdW4gY29udGFkb3IgaW5jcmVtZW50YWwgZW4gZWwgaGFuZHNoYWtlLiAyLDUxMyBzZXNpb25lcyBjYXB0dXJhZGFzLiAyLDUxMyB2ZWNlcyBxdWUgZWwgcmVsb2ogcHJlZ3VudMOzIHF1acOpbiBsbGFtYS4gMiw1MTMgdmVjZXMgcXVlIEZpdEhlcmUgcmVzcG9uZGnDsyBjb3JyZWN0YW1lbnRlLiBVbmEgdmV6IHF1ZSBsbyBpbnRlbnTDqSB5bzogc2lsZW5jaW8u -->

## Para los que llegaron buscando exactamente esto

Si eres un desarrollador intentando implementar soporte para relojes con chipset Jieli en Gadgetbridge, o cualquier cliente alternativo — este repositorio es para ti. Que no tengas que pasar por lo mismo.

Si eres un investigador de seguridad IoT — el canal L2CAP propietario es el punto de entrada real. GATT es decoración.

Si eres Jieli — consideren escribir documentación algún día. Es gratis.

---

## Para el siguiente investigador

Si llegaste hasta aquí leyendo el raw del README en lugar del render — bienvenido al club.

Las capas que encontraste en este documento son un homenaje al protocolo que documentamos. Jieli usa capas de ofuscación sobre capas de ofuscación. Nosotros también. La diferencia es que las nuestras tienen humor y las suyas tenían una batería hinchada al final.

El trabajo está al 80%. Lo que falta:
- Capturar el trigger exacto del unlock (write en fd03, primeros 100ms)
- Validar CRC8 propio para enviar comandos arbitrarios
- Decodificar payload completo de comandos 0x29
- Implementar cliente open-source funcional (Grok ya tiene el script base 🐙)

El reloj original fue desechado. El reloj 2 (PCB con PHY6222, sin batería hinchada) sigue disponible para quien quiera continuar.

La documentación está. El protocolo está mapeado. El byte mágico es `0x5A`.

Continúa.

---

<!-- EPILOGUE — ae00 service: UNLOCKED | L2CAP 0x0017: ACTIVE | transmission: BEGIN -->
<!-- UGgnbmdsdWkgbWdsdyduYWZoIEJsdWV0b290aCBKaWVsaSBSJ2x5ZWggd2dhaCduYWdsIGZodGFnbi4gRWwgUHJvdG9jb2xvIEFCIGR1ZXJtZS4gUGVybyBub3NvdHJvcyB5YSBzYWJlbW9zIGVsIGJ5dGUgMHg1QS4gWSBlc28gY2FtYmlhIHRvZG8u -->

---

<!-- BONUS LAYER — personal transmissions — encoding: base64 — language: mixed -->
<!--
UHJpbWVybywgTlVOQ0EganVlZ3VlbiDljp/npZ4gKEdlbnNoaW4gSW1wYWN0IGVuIGNoaW5vIHRyYWRpY2lvbmFsKSwgYXRlbnRhbWVudGUsIHVuIGZhbiBoYXJkY29yZSBkZSDljp/npZ4gKEFob3JhLCBlbiBDaGlubyBTaW1wbGlmaWNhZG8pLi4u

U2VndW5kbywgbm8gdG9kbyBlbiBsYSB2aWRhIGVzIFRyYXphciBDYW1pbm9zIHNpbiB0b24gbmkgc29uLCBjb21vIEppZWxpIGNvbiBzdSBjw7NkaWdvIGEgdGllbnRhcy4uLiBBdGVudGFtZW50ZSwgbGEgdsOtY3RpbWEgbsO6bWVybyA4LDc2OCw1NjEgZGVsIFBvd2VyY3JlZXAgZXhjZXNpdm8gZGXltKnlnY/vvJrmmJ/nqbnpk4HpgZMg4oCUINCl0L7QvdC60LDQuTog0JfQstGR0LfQtNC90YvQtSDQoNC10LvRjNGB0YsgKE5vbWJyZSBkZWwganVlZ28gZW4gcnVzbykuLi4=

VGVyY2VybywgZW4gZWwgbsO6Y2xlbyBkZWwgaG9ycm9yLCBzb2xvIG5vcyB0ZW5lbW9zIGVsIHVubyBhbCBvdHJvLCBjb21vIHlvIHkgbWkgbcO6c2ljYSBwcml2YWRhIGVuIGRpc2NvIGR1cm8uLi4gQXRlbnRhbWVudGUsIGVsICdGYW4gZGUgc3UgcmVsYWNpw7NuJyBuw7ptZXJvIDE1IGRlIDE3LCBkZWwgbGVzYmlqdWVnby4uLiBEaWdvLCDjg6rjg5Djg7zjgrnvvJoxOTk5IChOb21icmUgZGVsIGp1ZWdvIGVuIGphcG9uw6lzKQ==

Q3VhcnRvLCBjdWFuZG8gbGEgaW1wb3J0YWNpw7NuIGZhbGxhLCBzb2xvIHRlIHF1ZWRhIHZlciB1biBjZXJybywgeSBtb250YXIgYWjDrSwgdW4gTWNEb25hbGRzLCB1biBCdXJnZXIgS2luZywgeSB1bmEgZsOhYnJpY2EgZGUgcHJvZHVjY2nDs24gZGUgdXJhbmlvIGEgY2FtcG8gYWJpZXJ0by4uLiBBdGVudGFtZW50ZSwgdW4gZmFuIGRlICdGQUNUT1JZIE1VU1QgR1JPVycuLi4gRGlnby4uLiBEZSBTYXRpc2ZhY3RvcnkgY29uIHNraW4gYW5pbWUuLi4gRGlnby4uLiBEZSDmmI7ml6XmlrnoiJ/vvJrnu4jmnKvlnLAg4oCUIOCkhuCksOCljeCkleCkqOCkvuCkh+Ckn+CljeCkuDog4KSP4KSC4KSh4KSr4KS84KWA4KSy4KWN4KShIChOb21icmUgZGVsIGp1ZWdvIGVuIGhpbmRpKQ==

UXVpbnRvLCBhIHZlY2VzLCBzb2xvIHRvY2EgdmVyIHF1ZSB0b2RvIHNhbGUgbWFsLi4uIFkgYnVzY2FydGUgY29uZWppdGFzIHBhcmEgY3VyYXIgZWwgYWxtYS4uLiBDb21vIHlvIHkgbWlzIDE0IGhvcmFzIGRlc3BlcmRpY2lhZGFzIGVuIHVuIGRvbmdsZSBCbHVldG9vdGgsIHF1ZSBtZSBkZW11ZXN0cmEsIHF1ZSBsYSBkb21pbmFjacOzbiBtdW5kaWFsIGRlIGxhcyBtw6FxdWluYXMuLi4gVmEgYSBzZXIgbcOhcyBBbmltZSwgcXVlIFNreW5ldC4uLiBBdGVudGFtZW50ZSwgdW4gZmFuIGRlIGxhIHRlbnRhY2nDs24gZGUgbGEgY2FybmUuLi4gRGlnby4uLiDsirnrpqzsnZgg7Jes7IugOiDri4jsvIAg4oCUIE5pa2tlOiBTZWllcnNndWRpbm5lbiAoTm9tYnJlIGRlbCBqdWVnbyBlbiBub3J1ZWdvKQ==

0JbQuNGN0LvQuCDQsdC+0LvQvtC9INCx0YPRgdCw0LQg0LrQvtC80L/QsNC90LjRg9C00LDQtCDQt9C+0YDQuNGD0LvQtjog0KLQsCDQvdCw0YAg06nTqdGA0YHQtNC40LnQvSDQutC+0LTRi9CzINC+0LnQu9Cz0L7RhSDQsNC20LvRi9CzINCx0LDRgNGD0YPQvdGB0LrRgNC+0Lkg0LDRgNC80LjQuSDQsdC+0LvRjNGI0LUuINCS0Ysg0L3QtSDQsdGL0LvQuCBCWUQsINGC0LXQtNC90LjRiCDRgtC10YXQvdC+0LvQvtCz0LjQuZKv0Lku0KjTqdGA0LjQvdGF06nTqdC70LjQs9GCINGC0L7Qs9C70L7QvtGA0L7QuS4=
-->

*v0x5A.1 — Marzo 2026 — El Salvador*
*"The Protocol AB sleeps. But we know the byte."*

*Documentado en marzo de 2026. El autor sobrevivió.*
*El reloj, no.*
