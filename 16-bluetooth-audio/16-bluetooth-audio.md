# Módulo 16 — Bluetooth Audio

**Fase 05 — Audio**

Con este módulo cerramos la Fase 05.

> **Nota de adaptación importante:** a diferencia de la batería (Módulo 11), VirtualBox **no emula un adaptador Bluetooth virtual**. Para tener Bluetooth real dentro de la VM hace falta pasar un dongle USB Bluetooth físico del host directamente al huésped (USB passthrough) — algo que puede que no tengas disponible. Este módulo está diseñado para que aprendas la arquitectura completa igual, documentando honestamente hasta dónde llega tu entorno concreto.

---

## Objetivos del módulo

- Entender la arquitectura Bluetooth en Linux: `BlueZ` (el stack del kernel/sistema) + el módulo `bluez5` de PipeWire (la integración de audio).
- Entender los perfiles de audio Bluetooth: A2DP (alta calidad, solo salida) vs. HSP/HFP (llamadas, calidad menor, bidireccional).
- Usar `bluetoothctl` para inspeccionar (y, si el hardware lo permite, emparejar) dispositivos.
- Documentar honestamente qué tan lejos llega el Bluetooth en tu VM concreta.

---

## 1. CONCEPTO: la arquitectura Bluetooth en Linux

```
Hardware Bluetooth (adaptador USB o integrado)
        ↓
Kernel (driver del adaptador + subsistema Bluetooth del kernel)
        ↓
BlueZ (bluetoothd — el demonio de sistema que implementa el protocolo Bluetooth completo)
        ↓
Módulo bluez5 de PipeWire (traduce dispositivos Bluetooth en nodos/sinks de audio, Módulo 15)
        ↓
wpctl / pactl / aplicaciones (ven un dispositivo Bluetooth exactamente igual que uno con cable)
```

**Por qué existe esta separación:** `BlueZ` no sabe nada de audio específicamente — implementa el protocolo Bluetooth genérico (parejamiento, seguridad, perfiles de todo tipo: teclados, archivos, audio). El módulo `bluez5` de PipeWire es el puente que **traduce** un dispositivo de audio Bluetooth ya emparejado por BlueZ en un nodo más del grafo que viste en el Módulo 15 — mismo patrón de "capas especializadas hablando protocolos estándar" que ya es un tema recurrente en el curso.

---

## 2. CONCEPTO: A2DP vs. HSP/HFP — dos perfiles, un trade-off

| Perfil | Uso | Calidad | Dirección |
|---|---|---|---|
| **A2DP** (Advanced Audio Distribution Profile) | Música, reproducción | Alta (códecs SBC/AAC/aptX/LDAC según el dispositivo) | Solo salida (el micrófono del dispositivo, si tiene, no se usa) |
| **HSP/HFP** (Headset/Hands-Free Profile) | Llamadas | Baja (optimizado para voz, no música) | Bidireccional (salida + micrófono) |

**El trade-off real que vas a ver en cualquier auricular Bluetooth:** cuando hacés una videollamada con auriculares Bluetooth, el audio "empeora" notablemente — es porque el sistema cambia automáticamente de A2DP a HFP para poder usar el micrófono, sacrificando calidad de música a cambio de tener canal de voz. No es un bug, es el protocolo mismo forzando esa elección.

```bash
wpctl status    # con un dispositivo Bluetooth conectado, deberías ver ambos perfiles disponibles para elegir
```

---

## 3. HERRAMIENTA: instalar el stack Bluetooth

```bash
sudo pacman -S bluez bluez-utils pipewire-bluez
sudo systemctl enable --now bluetooth.service
systemctl status bluetooth.service
```

**Conexión con el Módulo 07/11:** `bluetooth.service` es un servicio de **sistema** (`sudo systemctl`, no `--user`) — a diferencia de PipeWire/WirePlumber (Módulo 14), porque el hardware Bluetooth es un recurso compartido a nivel de máquina, no de sesión de usuario individual. El puente hacia el audio de tu sesión lo hace específicamente el módulo `bluez5` dentro de PipeWire.

---

## 4. HERRAMIENTA: `bluetoothctl` — inspección y emparejamiento

```bash
bluetoothctl
```

Dentro del prompt interactivo (`[bluetooth]#`):

```
power on          # encender el adaptador (si existe)
show                 # info del adaptador detectado
scan on           # empezar a buscar dispositivos cercanos
devices                  # listar los detectados
pair XX:XX:XX:XX:XX:XX     # emparejar con la dirección MAC de un dispositivo
connect XX:XX:XX:XX:XX:XX    # conectar
exit
```

**Resultado esperado según tu hardware — documentá honestamente cuál te tocó:**
1. Si tu VM tiene un dongle Bluetooth USB pasado por passthrough: `show` debería listar un adaptador real, y podés intentar emparejar contra tu celular u otro dispositivo.
2. Si no tenés passthrough configurado (el caso más probable): `bluetoothctl` corre sin error, pero `show` no lista ningún adaptador (`No default controller available`) — el software está completo y correcto, simplemente no hay hardware que controlar.

---

## 5. CONCEPTO: qué SÍ y qué NO podés aprender de Bluetooth en esta VM (cierre de fase)

| Se puede aprender en la VM | Requiere hardware Bluetooth real (con o sin passthrough) |
|---|---|
| Arquitectura completa: BlueZ → bluez5 → PipeWire → apps | Emparejamiento y conexión reales |
| Diferencia conceptual A2DP vs. HSP/HFP | Escuchar el cambio de calidad real al activar el micrófono |
| Sintaxis de `bluetoothctl` | Latencia/estabilidad reales de una conexión Bluetooth |
| Por qué `bluetooth.service` es de sistema, no de usuario | Compatibilidad real de códecs (SBC/AAC/aptX) con tu hardware específico |

**El mismo patrón que ya viste en toda la Fase 04 (Módulos 11-13):** entender la arquitectura completa no depende de tener el hardware — depende de la VM (o, en este caso, de si hiciste o no el passthrough).

---

## 6. PRÁCTICA

1. Instalá el stack Bluetooth y confirmá que `bluetooth.service` está activo.
2. Corré `bluetoothctl` y documentá honestamente qué resultado te tocó (adaptador presente o `No default controller available`).
3. Si tenés adaptador: intentá `scan on` y `devices`, documentando qué se detecta.
4. Explicá con tus propias palabras la diferencia entre A2DP y HSP/HFP, y por qué el cambio de perfil ocurre automáticamente al activar el micrófono en una llamada.
5. Completá la tabla de la sección 5 con tu propia experiencia concreta.
6. Reflexión de cierre de fase: de los tres módulos de Audio (14-16), ¿cuál mostró la integración más limpia entre kernel, servicio de usuario/sistema, y aplicación?

---

## 7. ERROR INTENCIONAL / DIAGNÓSTICO

```bash
bluetoothctl connect AA:BB:CC:DD:EE:FF
```

(una dirección MAC inventada, sin emparejar previamente)

**Diagnóstico esperado:** un error indicando que el dispositivo no existe o no está emparejado (`Device AA:BB:CC:DD:EE:FF not available`) — `bluetoothctl` no intenta conectar a direcciones que nunca pasaron por `scan`/`pair`, mismo patrón de validación estricta que ya viste en `pw-link` (Módulo 15) con nodos inexistentes.

---

## Checklist de cierre del módulo (y de la Fase 05 completa)

- [ ] Entiendo la cadena BlueZ → módulo bluez5 de PipeWire → aplicaciones.
- [ ] Entiendo la diferencia A2DP vs. HSP/HFP y por qué el perfil cambia automáticamente según el uso.
- [ ] Instalé el stack Bluetooth y confirmé el estado de `bluetooth.service`.
- [ ] Usé `bluetoothctl` y documenté honestamente el resultado según mi hardware disponible.
- [ ] Completé la tabla de cierre de fase, distinguiendo qué es aprendible en VM y qué requiere hardware real.
- [ ] Provoqué y diagnostiqué el error de conexión a un dispositivo no emparejado.

---

## Evidencias

_(pendiente — se agregan capturas reales a medida que se completa el módulo)_

---

## Cierre de Fase 05 — Audio

Con este módulo termina la Fase 05: la arquitectura completa del audio en Linux moderno (Módulo 14: ALSA → PipeWire → WirePlumber), el detalle interno del grafo de nodos/puertos/enlaces con troubleshooting real incluido (Módulo 15), y la extensión de esa misma arquitectura hacia dispositivos inalámbricos vía BlueZ (Módulo 16) — mismo patrón de capas especializadas y protocolos estándar que el curso viene repitiendo desde gráficos (Fase 02) hasta acá.

---

**Próximo módulo:** 17 — NetworkManager (inicio de la Fase 06 — Desktop Networking).
