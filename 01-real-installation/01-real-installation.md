# Módulo 01 — Arch Linux Installation (VM VirtualBox)

**Fase 01 — Instalación real (en VM)**

---

## Objetivos del módulo

- Auditar la instalación de Arch ya existente en la VM `arch_linux` (hecha en el Módulo 06 del curso anterior), documentando exactamente qué se instaló y cómo, en vez de reinstalar desde cero.
- Confirmar el esquema de particionado real (tabla de particiones, tipos de filesystem, punto de montaje de cada uno).
- Verificar que `/etc/fstab` coincide exactamente con lo que el sistema tiene montado — la fuente más común de errores sutiles después de tocar discos.
- Dejar documentado el estado "conocido bueno" de la instalación, para poder comparar contra él si algo se rompe en módulos futuros (Btrfs, bootloader, etc.).

---

## 1. CONCEPTO: auditar una instalación existente

Este módulo, en el curso original, era "instalar Arch desde una ISO en blanco, en hardware físico". Ya adaptamos eso en el Módulo 00: la instalación ya existe, en la VM `arch_linux`, hecha durante el Módulo 06 del curso anterior. Lo que cambia acá no es el objetivo de aprendizaje (entender exactamente cómo quedó armado un sistema Arch desde el disco hacia arriba) sino el método: en vez de tomar decisiones de particionado y verlas aplicarse, las **reconstruimos hacia atrás** a partir del sistema ya instalado.

```
INSTALACIÓN DESDE CERO                    AUDITORÍA DE INSTALACIÓN EXISTENTE
(lo que hiciste en el Módulo 06)          (lo que hacemos acá)

decidís particionado          →           inspeccionás el particionado real
      ↓                                          ↓
formateás filesystems         →           identificás qué filesystem tiene cada partición
      ↓                                          ↓
escribís fstab a mano         →           verificás que fstab coincide con la realidad
      ↓                                          ↓
instalás bootloader           →           confirmás cómo quedó configurado (Módulo 03)
```

El resultado de aprendizaje es el mismo — terminás sabiendo leer un sistema Arch instalado de punta a punta — pero el camino es diagnóstico en vez de constructivo. Es, de hecho, una habilidad más realista: en un trabajo real vas a heredar sistemas ya instalados por otra persona mucho más seguido de lo que vas a instalar uno desde cero.

---

## 2. POR QUÉ EXISTE: por qué esto sigue siendo un módulo completo, no un trámite

Podría parecer que si el sistema "ya funciona", no hay nada que aprender. Es al revés: cuando instalaste en el Módulo 06 del curso anterior, seguías una guía paso a paso — no necesariamente entendías cada decisión en el momento. Auditar ahora, sin guía, te obliga a explicar **por qué** el disco quedó así, no solo **que** quedó así. Es la diferencia entre poder repetir una instalación y poder diagnosticar una que se rompió.

---

## 3. ARQUITECTURA: lo que ya sabemos de esta VM (Módulo 00)

Del inventario del Módulo 00 ya tenemos:

| Disco | Tamaño | Particiones/volúmenes |
|---|---|---|
| `sda` | 50.56 GB | `sda1` → `/boot`, `sda2` → `/` |
| `sdb` | 10.96 GB | LVM: `vg_datos-lv_pruebas` (4G), `vg_datos-lv_cifrado` (2G) |
| `zram0` | 2.36 GB | `[SWAP]` |

Lo que falta confirmar en este módulo: tipo de tabla de partición (MBR, porque la VM está en BIOS legacy — confirmado en el Módulo 00), tipo de filesystem de cada partición, y si `/etc/fstab` referencia estos dispositivos por UUID, label o ruta cruda (`/dev/sda1`) — cada opción tiene implicaciones distintas si el disco se reordena.

---

## 4. HERRAMIENTA: comandos de auditoría

```bash
sudo parted -l              # tabla de particiones: msdos (MBR) o gpt
sudo fdisk -l /dev/sda      # detalle de particiones de sda
lsblk -f                    # filesystem + UUID + punto de montaje, todo junto
df -hT                      # espacio usado, con tipo de filesystem
cat /etc/fstab              # qué dice el sistema que debería montar
blkid                       # UUID real de cada dispositivo, para comparar contra fstab
```

---

## 5. EJEMPLO: leyendo la salida

`sudo parted -l` te va a mostrar algo como:

```
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sda: 54.3GB
Partition Table: msdos
```

`Partition Table: msdos` confirma MBR — coherente con que la VM está en BIOS legacy, no UEFI (una instalación UEFI necesitaría GPT con una partición ESP FAT32).

`lsblk -f` te va a mostrar el filesystem de cada partición (probablemente `ext4` para `/boot` y `/`, dado que no se mencionó Btrfs en el curso anterior) junto a su UUID — comparalo línea por línea contra `cat /etc/fstab`.

---

## 6. PRÁCTICA

1. Correr los 6 comandos de la sección 4.
2. Completar esta tabla:

| Partición/volumen | Filesystem | UUID (de `blkid`) | Punto de montaje (`lsblk`) | ¿En `/etc/fstab`? | ¿UUID coincide? |
|---|---|---|---|---|---|
| `sda1` | | | `/boot` | | |
| `sda2` | | | `/` | | |
| `vg_datos-lv_pruebas` | | | | | |
| `vg_datos-lv_cifrado` | | | | | |
| `zram0` | | | `[SWAP]` | | |

3. Si algún volumen LVM no aparece en `/etc/fstab`, no es necesariamente un error — puede ser un volumen que se monta manualmente o vía script (recordalo del Módulo 11 del curso anterior). Documentá qué encontraste, no lo "arregles" todavía.
4. Confirmar la tabla de particiones (`msdos`/MBR) contra lo que ya sabíamos del Módulo 00 (BIOS legacy).

---

## 7. ERROR INTENCIONAL / DIAGNÓSTICO

Editá `/etc/fstab` (con `sudo`) y cambiá deliberadamente el UUID de la línea de `/boot` por uno inventado (cualquier cadena con el formato correcto pero que no exista). **No reiniciés todavía.**

```bash
sudo mount -a
```

**Diagnóstico esperado:** `mount -a` va a fallar con un error como `mount: /boot: can't find UUID=...` — porque el UUID inventado no corresponde a ningún dispositivo real. Esto demuestra, sin tener que reiniciar y arriesgar un boot roto, por qué `/etc/fstab` con un UUID incorrecto es una de las causas más comunes de sistemas que no arrancan (Break & Fix #5 del curso anterior, `/etc/fstab` corrupto).

Revertí el cambio inmediatamente:

```bash
sudo mount -a   # confirmar que vuelve a funcionar tras revertir
```

---

## 8. RETO

Sin ejecutar nada todavía: basándote en la tabla de la sección 6, predecí qué pasaría si `sdb` (el disco con el volumen LVM) se desconectara de la VM (como hicimos con `sda` en el Módulo 00, pero acá con el disco secundario). ¿El sistema arrancaría igual? ¿Qué falla y en qué momento del arranque? Confirmalo después, con cuidado, apagando la VM y repitiendo el experimento del Módulo 00 pero con `sdb`.

---

## Checklist de cierre del módulo

- [ ] Corrí los 6 comandos de auditoría y completé la tabla de la sección 6.
- [ ] Confirmé que la tabla de particiones es `msdos` (MBR), coherente con BIOS legacy.
- [ ] Provoqué y diagnostiqué el fallo de `mount -a` con un UUID incorrecto en `fstab`, y revertí el cambio.
- [ ] Predije y comprobé qué pasa si se desconecta `sdb`.
- [ ] Puedo explicar, sin mirar la tabla, cómo está armado el disco de esta VM de memoria.

---

## Evidencias

_(pendiente — se agregan capturas reales a medida que se completa el módulo)_

---

**Próximo módulo:** 02 — Manual Installation vs Archinstall
