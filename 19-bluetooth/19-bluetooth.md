# Módulo 19 — Bluetooth (general, opcional)

**Fase 06 — Desktop Networking**

> **Nota de adaptación:** este módulo cubre Bluetooth como **transporte genérico** (periféricos, transferencia de archivos) — no confundir con el Módulo 16, que ya cubrió a fondo la arquitectura de **audio** Bluetooth (BlueZ → bluez5 → PipeWire). Como ya confirmamos en el Módulo 16 (`/sys/class/bluetooth` vacío), esta VM no tiene ningún adaptador Bluetooth, ni siquiera emulado. Por eso este módulo queda **opcional** y es puramente conceptual, sin práctica nueva contra tu sistema.

---

## Objetivos del módulo

- Entender qué usos de Bluetooth existen más allá del audio: periféricos (teclado, mouse), transferencia de archivos (OBEX), tethering de red.
- Entender por qué toda esa funcionalidad pasa por el mismo demonio `bluetoothd` que ya instalaste en el Módulo 16 — un único stack, múltiples perfiles.
- Reconocer que la arquitectura (no la práctica) es lo transferible a hardware físico real.

---

## 1. CONCEPTO: un demonio, múltiples perfiles

Bluetooth no es "un protocolo para audio" — es un protocolo de transporte genérico de corto alcance, y el audio (A2DP/HSP/HFP, Módulo 16) es solo **uno** de sus perfiles posibles. `bluetoothd` (BlueZ) es el mismo demonio para todos:

| Perfil | Uso |
|---|---|
| A2DP / HSP / HFP | Audio (ya visto en detalle, Módulo 16) |
| HID (Human Interface Device) | Teclados y mouse inalámbricos |
| OBEX (Object Exchange) | Transferencia de archivos entre dispositivos |
| PAN (Personal Area Network) | Tethering — compartir la conexión de datos de un celular |

**Por qué importa verlo así:** es el mismo patrón que ya reconociste con `pactl`/`pw-link` (Módulo 15) — una sola infraestructura de bajo nivel, expuesta hacia arriba de formas distintas según el caso de uso, en vez de un demonio separado por cada función.

---

## 2. CONCEPTO: qué cambiaría con hardware real

Con un adaptador Bluetooth físico funcionando, `bluetoothctl` (que ya usaste en el Módulo 16) es la misma herramienta para **todos** estos casos — emparejar un teclado no es conceptualmente distinto de emparejar un auricular, es el mismo flujo `scan on` → `pair` → `connect`, solo que el perfil que se negocia automáticamente después es HID en vez de A2DP.

```bash
# Con hardware real, así se vería emparejar un teclado (no ejecutable en esta VM):
# bluetoothctl
# [bluetooth]# scan on
# [bluetooth]# pair XX:XX:XX:XX:XX:XX
# [bluetooth]# trust XX:XX:XX:XX:XX:XX    # confiar en el dispositivo para reconexión automática
# [bluetooth]# connect XX:XX:XX:XX:XX:XX
```

**Detalle nuevo respecto al Módulo 16:** `trust` — un paso que no necesitaste para audio en su momento, pero que es clave para periféricos como teclados: le dice a BlueZ que reconecte automáticamente sin pedir confirmación cada vez, algo especialmente importante para un teclado (no podés "confirmar" un emparejamiento tipeando en un teclado que todavía no está conectado).

---

## 3. PRÁCTICA (conceptual)

1. Explicá con tus propias palabras por qué el mismo demonio `bluetoothd` sirve tanto para audio como para un teclado inalámbrico.
2. Investigá (documentación de `bluetoothctl` o `man bluetoothctl`) qué hace exactamente el comando `trust`, y por qué es más relevante para periféricos HID que para audio.
3. Reflexión: si mañana instalás Arch en tu laptop física con hardware Bluetooth real, ¿qué de lo que aprendiste en el Módulo 16 y en este módulo se aplicaría sin cambios?

---

## Checklist de cierre del módulo

- [ ] Entiendo que Bluetooth es un transporte genérico con múltiples perfiles, no solo audio.
- [ ] Entiendo que `bluetoothd` (Módulo 16) es el mismo demonio detrás de todos los perfiles.
- [ ] Entiendo la diferencia entre `pair`, `trust` y `connect` en `bluetoothctl`.
- [ ] Puedo explicar qué de esta arquitectura se transfiere sin cambios a hardware físico real.

---

## Evidencias

_(módulo conceptual, sin práctica nueva contra el sistema — sin evidencias propias; ver Módulo 16 para las capturas del stack Bluetooth real)_

---

**Próximo módulo:** 20 — Printing & CUPS.
