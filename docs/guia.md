# ARCH LINUX DESKTOP MASTER COURSE
## De Administrador Linux a Ingeniero de Sistemas Desktop

**Nivel:** Intermedio → Avanzado → Experto
**Plataforma:** Arch Linux
**Entorno:** Máquina virtual en VirtualBox (misma herramienta del curso anterior)
**Modalidad:** Laboratorio práctico
**Enfoque:** Desktop Linux + Ecosistema Arch + Troubleshooting + Personalización + Recuperación

> **Nota sobre el entorno:** este curso se cursa íntegramente en una VM de VirtualBox, no en hardware físico — la laptop del estudiante corre Ubuntu como sistema anfitrión y no puede reinstalarse solo con Arch. Esto es una decisión consciente, no una limitación oculta: cada módulo que dependa de hardware que VirtualBox no puede virtualizar de forma realista (Wi-Fi real, Bluetooth real, batería/ACPI real, GPU dedicada real) queda marcado explícitamente como **adaptado** o **opcional**, explicando qué parte del concepto sigue siendo aprendible en la VM y qué parte requeriría hardware real para completarse al 100%.

---

## 1. Propósito del curso

Este curso es la continuación directa de:

**Arch Linux Master Course — De Cero a Ingeniero de Sistemas Linux**

El curso anterior se concentró principalmente en:

- administración Linux
- servidores
- systemd
- networking
- seguridad
- almacenamiento
- Bash
- programación
- DevOps
- contenedores
- kernel
- OS development

Este segundo curso cambia deliberadamente el escenario, pero manteniendo el mismo entorno técnico del primero (VirtualBox).

Ahora el objetivo es aprender:

> Cómo construir, sobre una VM de Arch Linux, un sistema de escritorio completo, estable, mantenible, personalizado y recuperable — todo lo que un servidor no necesita pero un puesto de trabajo diario sí.

Una VM oculta algunos problemas reales de hardware físico:

```
LIMITACIONES DE VIRTUALBOX
│
├── GPU virtual (sin driver dedicado real)
├── Sin Wi-Fi real (la red se virtualiza como Ethernet)
├── Sin Bluetooth real (sin passthrough nativo)
├── ACPI/batería simulados, no reales
└── Suspend/hibernate no se comportan igual que en hardware físico
```

Pero deja intacto, y perfectamente practicable, todo lo que realmente define "un escritorio Linux completo" como sistema de software:

```
SE APRENDE IGUAL DE BIEN EN LA VM
│
├── Entornos de escritorio (GNOME, KDE, tiling WMs)
├── Audio (PipeWire/WirePlumber, sí se virtualiza)
├── Btrfs, subvolúmenes, snapshots, rollback
├── AUR, PKGBUILD, mantenimiento de rolling release
├── Dotfiles, personalización, reproducibilidad
├── Seguridad de escritorio, sandboxing
└── Troubleshooting y recuperación de desastres
```

Por eso este curso trata el escritorio como un sistema completo — con las limitaciones de la VM señaladas con honestidad en cada módulo donde corresponda.

---

## 2. Filosofía del curso

La regla principal es:

> No memorizar comandos. Entender el sistema que hay detrás del comando.

Cada tecnología se estudiará mediante:

```
CONCEPTO → POR QUÉ EXISTE → CÓMO FUNCIONA → ARQUITECTURA →
HERRAMIENTA → EJEMPLO → PRÁCTICA →
ERROR INTENCIONAL → DIAGNÓSTICO → SOLUCIÓN → VALIDACIÓN → RETO
```

Por ejemplo, no aprender:

```
systemctl enable bluetooth
```

como una receta. Primero entender:

```
Bluetooth → kernel → driver → BlueZ → systemd → D-Bus → desktop environment → aplicación
```

Después utilizar la herramienta.

---

## 3. Principios

### Principio 1 — Entender antes de modificar

Antes de instalar una herramienta:

- saber qué problema resuelve
- saber qué componente reemplaza
- saber qué dependencias tiene
- saber cómo verificar que funciona
- saber cómo eliminarla

### Principio 2 — Preferir componentes oficiales

La progresión será:

```
Kernel / systemd → Arch official repositories → ArchWiki → Software upstream → AUR → Herramientas auxiliares
```

El AUR no será utilizado simplemente porque "es más fácil".

### Principio 3 — Cada cambio debe ser reversible

Siempre que sea posible:

```
ANTES → BACKUP → CAMBIO → PRUEBA → VALIDACIÓN
```

### Principio 4 — El sistema debe poder recuperarse

Un sistema profesional no es solamente uno que funciona. Es uno que puede:

- actualizarse
- romperse
- diagnosticarse
- recuperarse
- restaurarse

---

## 4. Prerrequisitos

Se asume que el estudiante ya terminó el primer curso. Por tanto, ya debe conocer:

- terminal
- Bash
- filesystem
- usuarios
- permisos
- procesos
- systemd
- pacman
- networking
- SSH
- firewall
- logs
- almacenamiento
- LUKS
- LVM
- Btrfs básico
- Git
- Python
- C
- Rust
- Docker
- virtualización
- troubleshooting Linux

No se volverán a enseñar desde cero. Se utilizarán como conocimientos previos.

---

## 5. Nuevo entorno de laboratorio

### Máquina virtual

La práctica se realiza reutilizando la **misma VM `arch_linux`** del curso anterior (no se crea una VM nueva separada) — ya tiene Arch instalado y configurado desde el Módulo 06 del primer curso, así que este curso arranca directamente desde ahí en vez de repetir una instalación desde cero. Esto significa que la Fase 01 (instalación) se estudia igual en profundidad, pero validando/ajustando la instalación existente en vez de partir de una ISO en blanco — ver el Módulo 01 para el detalle de qué se reutiliza y qué se revisa.

**Configuración recomendada de la VM (revisar y ajustar la existente):**

- Tipo: Arch Linux (64-bit)
- RAM: mínimo 4 GB, ideal 8 GB (un entorno gráfico completo con GNOME/KDE pesa más que una terminal)
- CPU: mínimo 2 núcleos
- Video Memory: máximo permitido, con **aceleración 3D activada** (`Settings → Display → Enable 3D Acceleration`) — necesaria para que el compositor de Wayland/GNOME/KDE no vaya a software rendering puro
- Firmware: **EFI habilitado** (`Settings → System → Enable EFI`), para practicar systemd-boot/GRUB en UEFI igual que en el Módulo 03
- Disco: suficiente para Btrfs + snapshots (mínimo 40 GB, ideal 60 GB+, ya que los snapshots acumulan espacio)
- Audio: controlador de audio host habilitado (para poder validar PipeWire de verdad)
- Guest Additions: instalar apenas se tenga entorno gráfico, para resolución dinámica y mejor rendimiento — pero entender primero qué reemplazan a nivel de drivers

**Registrar el "hardware" virtual antes de comenzar** (igual que se haría en físico, pero documentando qué es virtual):

```bash
lscpu
lsmem
lsblk
lspci
lsusb
```

```bash
inxi -Fxxxz
```

Crear un inventario, marcando explícitamente qué es simulado:

| Componente | En esta VM |
|---|---|
| CPU | Virtualizado, pero expone las instrucciones reales del host |
| GPU | Virtual (VirtualBox VMSVGA) — no hay driver Intel/AMD/NVIDIA real |
| RAM / Disco | Virtuales, asignados desde el host |
| Audio | Virtualizado, pero PipeWire corre real dentro del guest |
| Wi-Fi | No existe — la red llega como adaptador Ethernet virtual |
| Bluetooth | No existe (sin passthrough nativo en VirtualBox) |
| Batería/ACPI | Simulados de forma muy limitada |
| USB | Se puede pasar dispositivos reales del host con passthrough de VirtualBox |

---

## 6. Regla de seguridad del laboratorio

Antes de cualquier cambio importante en la VM: **snapshot de VirtualBox**, no backup de datos de un sistema físico.

```
ANTES DE UN MÓDULO RIESGOSO → SNAPSHOT DE VIRTUALBOX → CAMBIO → PRUEBA → VALIDACIÓN
```

Esto reemplaza directamente la regla de "backup físico" del curso anterior — la ventaja de trabajar en VM es que el rollback es instantáneo (restaurar snapshot) en vez de depender de un backup externo. Aun así, para los módulos de Btrfs/Snapper (Fase 07), no usar el snapshot de VirtualBox como atajo: ahí el objetivo es aprender a recuperarse *desde adentro* del sistema, con las herramientas de Arch, no reiniciando la VM desde afuera.

---

## 7. Mapa general del curso

| Fase | Módulo | Nombre |
|---|---|---|
| 00 | 00 | Hardware Discovery & Preinstallation |
| 01 | 01 | Arch Linux Installation on Real Hardware |
| 01 | 02 | Manual Installation vs Archinstall |
| 01 | 03 | UEFI, Bootloaders & Secure Boot |
| 02 | 04 | Graphics Architecture |
| 02 | 05 | Xorg vs Wayland |
| 02 | 06 | GPU Drivers, Mesa & Vulkan |
| 03 | 07 | Desktop Architecture |
| 03 | 08 | GNOME |
| 03 | 09 | KDE Plasma |
| 03 | 10 | Window Managers & Compositors |
| 04 | 11 | Laptop Power Management |
| 04 | 12 | Battery, ACPI, Suspend & Hibernate |
| 04 | 13 | CPU/GPU Power Optimization |
| 05 | 14 | Audio Architecture |
| 05 | 15 | PipeWire, WirePlumber & ALSA |
| 05 | 16 | Bluetooth & Audio Devices |
| 06 | 17 | Desktop Networking |
| 06 | 18 | Wi-Fi |
| 06 | 19 | Bluetooth |
| 06 | 20 | Printing & CUPS |
| 07 | 21 | Btrfs Desktop Architecture |
| 07 | 22 | Subvolumes |
| 07 | 23 | Snapshots |
| 07 | 24 | Snapper & Rollback |
| 08 | 25 | AUR Deep Dive |
| 08 | 26 | makepkg & PKGBUILD |
| 08 | 27 | AUR Helpers |
| 08 | 28 | Package Cache & Maintenance |
| 09 | 29 | Rolling Release Maintenance |
| 09 | 30 | Pacman Hooks |
| 09 | 31 | Mirrors & Reflector |
| 09 | 32 | Journald & Maintenance |
| 10 | 33 | Dotfiles |
| 10 | 34 | GNU Stow / chezmoi |
| 10 | 35 | Themes, Fonts & Desktop Configuration |
| 10 | 36 | Linux Rice Engineering |
| 11 | 37 | Multimedia |
| 11 | 38 | Codecs & Hardware Acceleration |
| 11 | 39 | Productivity Applications |
| 12 | 40 | External Displays & Docking |
| 12 | 41 | Cameras, Microphones & Peripherals |
| 12 | 42 | USB & udev |
| 13 | 43 | Desktop Security |
| 13 | 44 | Secrets, Keyrings & Credentials |
| 13 | 45 | Sandboxing & Desktop Isolation |
| 14 | 46 | Desktop Troubleshooting |
| 14 | 47 | Break & Fix Laboratory |
| 15 | 48 | Recovery & Disaster Recovery |
| 15 | 49 | Btrfs Recovery |
| 15 | 50 | Boot Recovery |
| 16 | 51 | Advanced Desktop Automation |
| 16 | 52 | System Integration |
| 17 | 53 | Desktop Engineering |
| 17 | 54 | Final Integrated Project |

**Total: 55 módulos.**

---

## Fase 00 — Hardware Discovery

### Módulo 00 — Hardware Discovery & Preinstallation

**Objetivos**

Aprender a investigar una laptop antes de instalar Arch.

**Conceptos**

- CPU
- GPU
- firmware
- UEFI
- ACPI
- PCI
- USB
- NVMe
- Wi-Fi
- Bluetooth
- audio
- batería

**Herramientas**

- `lscpu`
- `lsmem`
- `lspci`
- `lsusb`
- `lsblk`
- `lsmod`
- `dmesg`

**Laboratorio**

Crear un inventario completo del hardware.

**Reto**

Identificar, sin utilizar una aplicación gráfica:

- GPU
- Wi-Fi chipset
- Bluetooth controller
- Audio controller
- NVMe controller
- Touchpad
- Battery

---

## Fase 01 — Instalación real

### Módulo 01 — Arch Linux sobre Hardware Físico

Aprender:

- ISO
- USB boot
- UEFI
- Secure Boot
- particionado
- instalación
- bootloader
- microcode
- firmware

La página oficial de descargas publica la ISO vigente y remite a la guía de instalación; para sistemas existentes, Arch indica que no es necesario reinstalar mediante una ISO para actualizar.

### Módulo 02 — Manual Installation vs archinstall

Comparar:

```
MANUAL
│
├── máxima comprensión
├── máximo control
└── máximo aprendizaje

ARCHINSTALL
│
├── instalación guiada
├── repetibilidad
└── rapidez
```

No tratar archinstall como una "instalación para principiantes". Analizar qué hace realmente.

La documentación de archinstall registra pasos, configuración y comandos utilizados, lo cual se aprovechará como herramienta pedagógica para comparar una instalación automatizada con una manual.

**Laboratorio**

Realizar:

- Instalación A → manual
- Instalación B → archinstall

Comparar:

- particiones
- paquetes
- bootloader
- red
- configuración
- servicios

### Módulo 03 — UEFI, Bootloaders & Secure Boot

Profundizar en:

- UEFI
- EFI System Partition
- boot entries
- systemd-boot
- GRUB
- Secure Boot
- firmware keys

**Proyecto**

```
UEFI → Bootloader → Kernel → initramfs → systemd → Graphical session
```

---

## Fase 02 — Gráficos

### Módulo 04 — Arquitectura Gráfica Linux

Estudiar:

```
Application → Toolkit → Desktop → Compositor → Wayland/X11 → DRM/KMS → Mesa / NVIDIA driver → GPU
```

### Módulo 05 — Xorg vs Wayland

Comparar:

| Característica | Xorg | Wayland |
|---|---|---|
| Arquitectura | servidor X | protocolo/compositor |
| Antigüedad | histórica | moderna |
| Compositing | externo | integrado |
| Seguridad | modelo histórico | aislamiento más fuerte |
| Compatibilidad | enorme legado | creciente |
| Desktop moderno | posible | principal enfoque |

No enseñar esto como una guerra. El objetivo es entender por qué existen ambos.

### Módulo 06 — GPU Drivers, Mesa & Vulkan *(adaptado — VirtualBox no expone GPU dedicada real)*

> En esta VM el driver activo va a ser el de VirtualBox (`vmwgfx`/VMSVGA vía Mesa), no Intel/AMD/NVIDIA real. Se estudia igual el concepto de la cadena kernel driver → Mesa → Vulkan/OpenGL, y se valida aceleración 3D básica (la que habilita el compositor), pero **no** hay hybrid graphics ni PRIME que practicar — eso queda documentado como "pendiente para cuando se tenga acceso a hardware físico", no simulado a la fuerza.

Separar:

- Intel
- AMD
- NVIDIA

Estudiar:

- kernel driver
- Mesa
- Vulkan
- OpenGL
- firmware
- hardware acceleration
- PRIME
- hybrid graphics

**Laboratorio**

```bash
lspci -k
```

Identificar: GPU, kernel driver, kernel modules. Después validar aceleración gráfica.

---

## Fase 03 — Desktop Environments

### Módulo 07 — Arquitectura Desktop Linux

Estudiar:

- display manager
- session
- compositor
- desktop environment
- toolkit
- portals
- D-Bus
- polkit
- keyring
- notification daemon

### Módulo 08 — GNOME

Instalar y estudiar:

- GNOME Shell
- Mutter
- GNOME Settings
- Extensions
- portals
- applications
- Wayland

**Objetivo:** no solamente "usar GNOME". Entender sus componentes.

### Módulo 09 — KDE Plasma

Estudiar:

- Plasma
- KWin
- KDE applications
- Wayland
- X11
- configuration system

Comparar arquitectura con GNOME.

### Módulo 10 — Tiling Window Managers & Compositors

Estudiar:

- i3
- Sway
- Hyprland

Diferenciar: X11 window manager vs Wayland compositor.

Crear configuraciones mínimas. Después construir un entorno completo.

---

## Fase 04 — Power Management *(opcional/teórica en VM — sin batería ni ACPI reales)*

> VirtualBox no expone batería ni sensores térmicos reales, así que estos tres módulos se estudian a nivel conceptual y de herramientas (instalar y configurar TLP/powertop, leer su documentación y salida aunque los valores sean simulados o inexistentes), dejando la validación real ("¿la laptop dura más con este perfil?") pendiente para cuando se practique en hardware físico más adelante.

### Módulo 11 — Arquitectura de Energía

Estudiar:

- ACPI
- CPU frequency
- CPU idle
- GPU power
- PCI power
- runtime PM
- thermal management

### Módulo 12 — Battery, Suspend & Hibernate

Trabajar con:

- suspensión
- hibernación
- hybrid sleep
- lid switch
- power button
- battery states

**Diagnosticar:**

```
Suspend funciona pero Wi-Fi no vuelve
```

### Módulo 13 — TLP, Power Profiles & powertop

Comparar herramientas y enfoques. Estudiar:

- TLP
- power-profiles-daemon
- powertop
- systemd
- kernel parameters

No activar múltiples gestores de energía sin comprender sus posibles interacciones.

**Proyecto**

Crear un perfil de energía documentado:

```
AC → performance
Battery → balanced/power saving
```

---

## Fase 05 — Audio

### Módulo 14 — Linux Audio Architecture

Comprender:

```
Application → PipeWire → WirePlumber → ALSA → Kernel → Hardware
```

### Módulo 15 — PipeWire, WirePlumber & ALSA

Estudiar:

- ALSA
- PipeWire
- WirePlumber
- PulseAudio compatibility
- JACK compatibility
- audio routing

**Herramientas:** `wpctl`, `pw-cli`, `pw-top`, `aplay`, `arecord`

### Módulo 16 — Bluetooth Audio

Trabajar con:

- BlueZ
- PipeWire
- perfiles Bluetooth
- micrófono
- auriculares
- codecs

---

## Fase 06 — Desktop Networking

### Módulo 17 — NetworkManager

Estudiar: `nmcli`, `nmtui`

Comprender: connections, profiles, DNS, routing, Wi-Fi, Ethernet, VPN.

### Módulo 18 — Wi-Fi *(adaptado — VirtualBox no expone chipset Wi-Fi real)*

> VirtualBox presenta la red al guest siempre como adaptador Ethernet virtual (NAT o Bridged), incluso si el host se conecta por Wi-Fi. No hay `iw`/`rfkill` con un dispositivo inalámbrico real que diagnosticar dentro de la VM. Se estudia la teoría (NetworkManager, `nmcli`, cómo se vería el diagnóstico) usando la conexión Ethernet virtual como sustituto práctico, y el Break & Fix específico de Wi-Fi real queda documentado para resolver en hardware físico.

**Diagnóstico (sobre el adaptador Ethernet virtual):** `ip`, `nmcli`, `journalctl`

### Módulo 19 — Bluetooth *(opcional — sin passthrough nativo en VirtualBox)*

> VirtualBox no virtualiza un adaptador Bluetooth para el guest. Este módulo queda como teoría (arquitectura de BlueZ, `bluetoothctl`, pairing) sin laboratorio práctico, salvo que se disponga de un adaptador Bluetooth USB dedicado para pasar por passthrough — opción avanzada, no requerida para avanzar en el curso.

Estudiar: BlueZ, `bluetoothctl`, pairing, trusted devices, profiles.

### Módulo 20 — Printing & CUPS

Aprender: CUPS, IPP, drivers, printer discovery, print queues.

**Proyecto:** configurar una impresora de red.

---

## Fase 07 — Btrfs Desktop

### Módulo 21 — Btrfs Architecture

Estudiar: copy-on-write, subvolumes, compression, checksums, snapshots, scrub, balance, send/receive.

### Módulo 22 — Subvolumes

Diseñar una estructura:

```
BTRFS
│
├── @
├── @home
├── @snapshots
└── @var
```

El diseño exacto debe justificarse según el objetivo.

### Módulo 23 — Snapshots

Comprender:

```
Sistema → Snapshot → Actualización → Problema → Rollback
```

Diferenciar: `snapshot ≠ backup`. Esto debe ser una lección obligatoria.

### Módulo 24 — Snapper & Rollback

Aprender: Snapper, timeline snapshots, pre/post snapshots, cleanup, rollback, integración con boot cuando corresponda.

**Proyecto**

Realizar una actualización deliberadamente problemática en una VM/laboratorio y recuperar mediante snapshot. Después trasladar el procedimiento al sistema físico con mucho cuidado.

---

## Fase 08 — AUR Profesional

### Módulo 25 — AUR Deep Dive

Estudiar: PKGBUILD, fuentes, checksums, maintainer, comments, votes/popularity, flagged packages, reproducibilidad.

### Módulo 26 — makepkg & PKGBUILD

Crear:

```
hello-desktop/
└── PKGBUILD
```

Construir un paquete:

```
source → build → package → install
```

### Módulo 27 — AUR Helpers

Comparar:

```
manual makepkg → yay → paru
```

No convertir yay o paru en sustitutos de pacman. El estudiante debe saber construir e instalar un paquete manualmente antes de utilizar un helper.

### Módulo 28 — Cache Management

Estudiar: `/var/cache/pacman/pkg`

Herramientas como: `paccache`

Aprender: caché, versiones antiguas, limpieza, recuperación, espacio en disco.

---

## Fase 09 — Rolling Release Engineering

### Módulo 29 — Maintenance Philosophy

Arch exige entender el concepto:

```
UPDATE ≠ CLICK AND FORGET
```

Antes de actualizar:

```
READ → BACKUP → CHECK → UPDATE → VALIDATE
```

### Módulo 30 — Pacman Hooks

Estudiar: `/usr/share/libalpm/hooks/`, `/etc/pacman.d/hooks/`

Comprender: pre-transaction, post-transaction, triggers, automation.

Arch documenta los hooks de pacman como mecanismos para ejecutar acciones antes o después de transacciones.

### Módulo 31 — Mirrors & Reflector

Estudiar: mirrorlist, sincronización, latencia, velocidad, HTTPS, reflector.

La guía actual de instalación contempla la selección de mirrors y describe el uso de reflector para generar listas según criterios como sincronización y ubicación.

### Módulo 32 — Journald & Maintenance

Crear una rutina:

```
Weekly
│
├── system update
├── journal review
├── disk usage
├── cache review
├── failed services
└── snapshot validation
```

---

## Fase 10 — Dotfiles & Rice

### Módulo 33 — Dotfiles

Estudiar: `~/.config`, `~/.local`, `~/.bashrc`, `~/.zshrc`

Comprender configuración por usuario.

### Módulo 34 — GNU Stow / chezmoi

Comparar: Git + Stow vs Git + chezmoi

Crear repositorio:

```
dotfiles/
├── bash
├── git
├── nvim
├── hypr
├── waybar
└── scripts
```

### Módulo 35 — Fonts, Themes & Desktop Configuration

Estudiar: fonts, Nerd Fonts, GTK themes, Qt themes, icon themes, cursor themes, wallpapers, terminal themes.

### Módulo 36 — Linux Rice Engineering

El objetivo no es solamente hacer un escritorio bonito. El estudiante debe aprender:

```
CONFIGURATION + REPRODUCIBILITY + VERSION CONTROL + DOCUMENTATION = ENGINEERED DESKTOP
```

**Proyecto:** crear un entorno completamente reproducible.

---

## Fase 11 — Multimedia & Productivity

### Módulo 37 — Multimedia

Estudiar: audio, video, codecs, containers, subtitles, hardware acceleration.

### Módulo 38 — Hardware Video Acceleration

Comprender:

```
Application → FFmpeg / media framework → VA-API / Vulkan / hardware backend → GPU
```

Validar aceleración en: navegador, reproductor multimedia, aplicaciones compatibles.

### Módulo 39 — Productivity Stack

Construir una selección consciente:

- Browser
- Terminal
- Editor
- Office
- PDF
- Image editor
- Video player
- Archive manager
- Screenshot
- Clipboard manager
- Password manager
- Cloud tools
- Development tools

No convertir esto en una lista de 500 aplicaciones. El objetivo es aprender cómo se integran las aplicaciones con el desktop.

---

## Fase 12 — Periféricos *(requiere USB passthrough de VirtualBox para practicar con hardware real del host)*

### Módulo 40 — External Displays *(teórico en VM — VirtualBox solo simula un monitor)*

Estudiar: HDMI, DisplayPort, USB-C, EDID, refresh rate, scaling, fractional scaling, multi-monitor. La práctica de multi-monitor real queda pendiente para hardware físico; en VM se puede simular una segunda pantalla virtual desde la configuración de VirtualBox para al menos practicar la configuración del compositor.

### Módulo 41 — Cameras, Microphones & Peripherals *(vía USB passthrough del host)*

**Diagnóstico:** `lsusb`, `v4l2-ctl`, `arecord`, `wpctl`

Trabajar con: webcam, micrófono, headset, touchpad, mouse, keyboard — pasando el dispositivo USB real del host a la VM (`Settings → USB` en VirtualBox).

### Módulo 42 — USB & udev

Estudiar: udev, rules, device events, permissions, hotplug.

**Proyecto:** crear una regla udev controlada para un dispositivo de laboratorio.

---

## Fase 13 — Desktop Security

### Módulo 43 — Desktop Security

Estudiar: user sessions, polkit, sudo, permissions, firewall, application permissions, sandboxing.

### Módulo 44 — Secrets & Keyrings

Estudiar: GNOME Keyring, KDE Wallet, secret services, SSH keys, GPG, credential storage.

### Módulo 45 — Application Sandboxing

Estudiar: Flatpak, portals, sandbox, filesystem permissions, desktop integration.

Comparar: pacman, AUR, Flatpak, AppImage — y cuándo tiene sentido cada modelo.

---

## Fase 14 — Troubleshooting

### Módulo 46 — Desktop Troubleshooting

Crear una metodología universal:

```
SÍNTOMA → OBSERVACIÓN → REPRODUCCIÓN → LOG → HIPÓTESIS →
PRUEBA → CAUSA RAÍZ → SOLUCIÓN → VALIDACIÓN → DOCUMENTACIÓN
```

### Módulo 47 — Break & Fix Laboratory

**Break #01 — Xorg no inicia**
`SÍNTOMA → Pantalla negra`. Investigar: `journalctl`, `systemctl`, Xorg logs.

**Break #02 — Wayland no inicia**
Investigar: compositor, GPU, drivers, session, environment.

**Break #03 — Audio mudo**
`Audio device visible → No sound`. Investigar: `wpctl`, `pw-top`, `journalctl`, `aplay`.

**Break #04 — Micrófono desaparece**
Investigar: PipeWire, WirePlumber, ALSA, profiles.

**Break #05 — La red (Ethernet virtual) se cae** *(adaptado de "Wi-Fi desaparece")*
Investigar: `ip`, `nmcli`, `dmesg`, `journalctl`.

**Break #06 — Bluetooth no conecta** *(solo si se usa passthrough USB; opcional en VM)*
Investigar: BlueZ, service, controller, pairing, profile.

**Break #07 — Batería no detectada** *(teórico en VM, real en hardware físico)*
Investigar: ACPI, `/sys/class/power_supply`, kernel, udev.

**Break #08 — Suspend/resume pierde la red** *(adaptado — sin Wi-Fi real que "no despierte")*
Investigar: suspend, resume, driver, NetworkManager.

**Break #09 — GPU acceleration desaparece**
Investigar: kernel driver, Mesa, Vulkan, environment, browser.

**Break #10 — Monitor externo no funciona**
Investigar: connector, DRM, compositor, EDID, cable, driver.

**Break #11 — Btrfs snapshot no aparece**
Investigar: subvolume, Snapper, mountpoints, configuration.

**Break #12 — Rollback falla**
Investigar: snapshot, subvolumes, boot, `/boot`, initramfs.

**Break #13 — AUR package fails**
Investigar: PKGBUILD, source, checksum, dependency, build.

**Break #14 — AUR helper conflicts**
Determinar: pacman, AUR, helper, package database.

**Break #15 — Disk full**
Investigar: `df -h`, `du`, `journalctl`, `paccache`.

**Break #16 — Desktop tarda demasiado en iniciar**
Investigar: `systemd-analyze`, `systemd-analyze blame`.

**Break #17 — Broken pacman transaction**
Aprender: identificar estado, revisar logs, no borrar archivos aleatoriamente, reconstruir correctamente.

**Break #18 — Bootloader roto**
Recuperar desde:

```
Arch ISO → mount → arch-chroot → repair → rebuild → reboot
```

---

## Fase 15 — Recovery

### Módulo 48 — Disaster Recovery

Diseñar procedimientos para:

- BOOT FAILURE
- GRAPHICS FAILURE
- AUDIO FAILURE
- NETWORK FAILURE
- PACKAGE FAILURE
- FILESYSTEM FAILURE
- CONFIGURATION FAILURE

### Módulo 49 — Btrfs Recovery

Practicar:

```
snapshot → update → break → rollback → validate
```

Después investigar: scrub, filesystem checks, snapshots, recovery.

### Módulo 50 — Boot Recovery

Practicar desde el ISO:

```
Identify disk → Mount → arch-chroot → Inspect →
Repair → Rebuild initramfs → Repair bootloader → Reboot
```

---

## Fase 16 — Automatización Desktop

### Módulo 51 — Desktop Automation

Utilizar: Bash, Python, systemd user services, timers, udev.

**Proyectos:**

- battery monitor
- backup launcher
- snapshot helper
- desktop health check
- maintenance report

### Módulo 52 — System Integration

Construir integraciones:

```
hardware → kernel → udev → systemd → desktop → application
```

Ejemplo:

```
USB device connected → udev event → rule → script → desktop notification
```

---

## Fase 17 — Desktop Engineering

### Módulo 53 — Desktop Engineering

Aquí se integran todos los conocimientos. Crear un sistema:

```
ARCH LAPTOP
│
├── UEFI
├── Kernel
├── Btrfs
├── Snapshots
├── systemd
├── NetworkManager
├── PipeWire
├── GPU stack
├── Desktop
├── AUR
├── Dotfiles
├── Security
├── Backups
└── Monitoring
```

### Módulo 54 — Final Integrated Project: Arch Professional Desktop (VM)

Construir una instalación completa en la VM de VirtualBox. Debe incluir:

**Sistema:** Arch Linux, UEFI, bootloader, microcode, kernel, firmware.

**Storage:** Btrfs, subvolumes, snapshots, rollback.

**Desktop:** elegir uno (GNOME, KDE Plasma, Hyprland, Sway, i3) pero justificar la elección.

**"Hardware" (lo que aplica en VM):** debe funcionar GPU virtual con aceleración 3D, red, audio, touchpad/mouse del host vía Guest Additions. Wi-Fi/Bluetooth/batería/cámara real quedan fuera de alcance en este proyecto — documentados explícitamente como pendientes para una futura instalación en hardware físico.

**Software:** browser, terminal, editor, Git, herramientas de desarrollo, multimedia, productividad.

**AUR:** al menos un PKGBUILD propio, uso consciente de AUR, documentación del paquete.

**Dotfiles:** Git + Stow/chezmoi.

**Seguridad:** firewall, usuario normal, sudo, keyring, SSH keys, actualizaciones.

**Recovery:** backup + snapshot + rollback procedure + Arch ISO recovery procedure.

---

## 8. Metodología de cada laboratorio

Cada laboratorio debe utilizar exactamente esta estructura:

```markdown
# Laboratorio XX — Nombre

## Objetivo
¿Qué vamos a conseguir?

## Requisitos
¿Qué necesitamos?

## Preparación
Estado inicial.

## Concepto aplicado
Explicación técnica.

## Procedimiento
Paso a paso.

## Validación
¿Cómo comprobamos que funcionó?

## Errores comunes
Problemas conocidos.

## Troubleshooting
Cómo investigar.

## Rollback
Cómo deshacer los cambios.

## Reto
Problema sin solución inmediata.

## Preguntas de reflexión
Preguntas conceptuales.
```

---

## 9. Break & Fix — Metodología

Todos los laboratorios de errores deben seguir:

```
SÍNTOMA → OBSERVACIÓN → LOG → HIPÓTESIS → PRUEBA → CAUSA RAÍZ → SOLUCIÓN → VALIDACIÓN
```

Nunca entregar directamente "Ejecuta este comando". Primero enseñar a encontrar el problema.

---

## 10. Sistema de evaluación

**Nivel 1 — Básico.** El estudiante puede: utilizar el escritorio, instalar aplicaciones, configurar Wi-Fi, administrar archivos, cambiar configuración, comprender componentes básicos.

**Nivel 2 — Intermedio.** Puede: diagnosticar audio, diagnosticar Wi-Fi, administrar GPU, configurar Bluetooth, manejar snapshots, mantener paquetes, gestionar dotfiles.

**Nivel 3 — Avanzado.** Puede: diagnosticar fallos complejos, recuperar desktop, realizar rollback, crear PKGBUILD, administrar Btrfs, construir un rice reproducible, administrar power management, diagnosticar hardware.

**Nivel 4 — Experto.** Puede: explicar toda la arquitectura, encontrar root cause, diseñar una instalación, automatizar mantenimiento, recuperar sistemas rotos, reproducir su configuración, documentar incidentes, diseñar un entorno desktop completo.

---

## 11. Proyectos progresivos

| # | Proyecto |
|---|---|
| 01 | Inventario completo de hardware |
| 02 | Instalación Arch manual en laptop |
| 03 | Instalación mediante archinstall |
| 04 | Configuración gráfica completa |
| 05 | Laptop con GNOME |
| 06 | Laptop con KDE Plasma |
| 07 | Tiling environment |
| 08 | Sistema de energía optimizado |
| 09 | Audio completo |
| 10 | Wi-Fi + Bluetooth + CUPS |
| 11 | Btrfs + Snapper |
| 12 | AUR + PKGBUILD propio |
| 13 | Dotfiles reproducibles |
| 14 | Rice completo |
| 15 | Multimedia + hardware acceleration |
| 16 | Multi-monitor |
| 17 | Desktop security |
| 18 | Break & Fix challenge |
| 19 | Recovery challenge |
| 20 | **FINAL** — Professional Arch Linux Laptop |

---

## 12. Estructura del repositorio

```
arch-linux-desktop-master-course/
│
├── README.md
├── ROADMAP.md
├── PROGRESS.md
├── HARDWARE.md
├── TROUBLESHOOTING.md
│
├── 00-hardware-discovery/
├── 01-real-installation/
├── 02-archinstall-vs-manual/
├── 03-uefi-bootloaders/
├── 04-graphics-architecture/
├── 05-xorg-wayland/
├── 06-gpu-drivers/
├── 07-desktop-architecture/
├── 08-gnome/
├── 09-kde-plasma/
├── 10-window-managers/
├── 11-power-management/
├── 12-battery-suspend/
├── 13-power-optimization/
├── 14-audio-architecture/
├── 15-pipewire-alsa/
├── 16-bluetooth-audio/
├── 17-networkmanager/
├── 18-wifi/
├── 19-bluetooth/
├── 20-cups-printing/
├── 21-btrfs/
├── 22-btrfs-subvolumes/
├── 23-btrfs-snapshots/
├── 24-snapper-rollback/
├── 25-aur/
├── 26-makepkg-pkgbuild/
├── 27-aur-helpers/
├── 28-package-cache/
├── 29-rolling-release/
├── 30-pacman-hooks/
├── 31-mirrors-reflector/
├── 32-journald-maintenance/
├── 33-dotfiles/
├── 34-stow-chezmoi/
├── 35-fonts-themes/
├── 36-linux-rice/
├── 37-multimedia/
├── 38-hardware-acceleration/
├── 39-productivity/
├── 40-external-displays/
├── 41-camera-microphone/
├── 42-usb-udev/
├── 43-desktop-security/
├── 44-keyrings-secrets/
├── 45-application-sandboxing/
├── 46-desktop-troubleshooting/
├── 47-break-fix/
├── 48-disaster-recovery/
├── 49-btrfs-recovery/
├── 50-boot-recovery/
├── 51-desktop-automation/
├── 52-system-integration/
├── 53-desktop-engineering/
├── 54-final-project/
│
├── labs/
├── projects/
├── scripts/
├── dotfiles/
├── pkgbuilds/
├── cheatsheets/
└── incident-reports/
```

---

## 13. Documentación de incidentes

Cada problema real debe convertirse en documentación. Crear `incident-reports/`, por ejemplo:

```
INC-001-audio-missing-after-update.md
INC-002-wifi-resume-failure.md
INC-003-wayland-black-screen.md
INC-004-btrfs-rollback.md
INC-005-aur-build-failure.md
```

**Formato:**

```markdown
# Incident

## Date
## System
## Symptoms
## Impact
## Investigation
## Logs
## Root Cause
## Solution
## Validation
## Prevention
## Lessons Learned
```

Esto convierte los errores reales del estudiante en conocimiento reutilizable.

---

## 14. Cheatsheets

Crear:

```
cheatsheets/
├── pacman.md
├── aur.md
├── btrfs.md
├── snapper.md
├── pipewire.md
├── networkmanager.md
├── bluetooth.md
├── gpu.md
├── wayland.md
├── systemd-desktop.md
├── power-management.md
├── troubleshooting.md
└── recovery.md
```

Las cheatsheets se crean después de comprender los temas, no antes.

---

## 15. Fuentes de referencia

**Prioridad 1 — ArchWiki.** Será siempre la referencia principal. Especialmente: Installation guide, General recommendations, Pacman, Archinstall, systemd, Btrfs, Wayland, Xorg, GNOME, KDE, NetworkManager, PipeWire, power management, AUR.

La guía oficial de instalación actualmente recomienda seleccionar adecuadamente los mirrors y recuerda que la configuración del entorno live no se transfiere automáticamente al sistema final salvo elementos concretos como la mirrorlist.

**Prioridad 2 — Arch Linux.** Utilizar documentación oficial, paquetes oficiales, ISO, páginas de paquetes, anuncios, noticias. El buscador oficial de paquetes permite comprobar qué paquetes existen actualmente en los repositorios de Arch.

**Prioridad 3 — Manual pages.** `man pacman`, `man systemctl`, `man btrfs`, `man mount`, `man NetworkManager`

**Prioridad 4 — Upstream.** Consultar directamente los proyectos: Linux, systemd, GNOME, KDE, Mesa, PipeWire, BlueZ, NetworkManager, Btrfs, Snapper.

---

## 16. Regla sobre internet

Cuando exista una duda:

```
ArchWiki → man page → official upstream documentation → Arch package information → community discussion
```

No utilizar como fuente principal: tutoriales aleatorios, blogs antiguos, vídeos desactualizados, scripts de instalación desconocidos.

---

## 17. Cronograma

| Fase | Duración | Contenido |
|---|---|---|
| 00 | 1 semana | Hardware discovery |
| 01 | 1–2 semanas | Instalación real |
| 02 | 1–2 semanas | Gráficos |
| 03 | 2 semanas | GNOME, KDE y tiling |
| 04 | 1–2 semanas | Power management |
| 05 | 1 semana | Audio |
| 06 | 1–2 semanas | Networking desktop |
| 07 | 2 semanas | Btrfs y Snapper |
| 08 | 2 semanas | AUR y PKGBUILD |
| 09 | 2 semanas | Rolling release maintenance |
| 10 | 2–3 semanas | Dotfiles y rice |
| 11 | 1 semana | Multimedia |
| 12 | 1 semana | Periféricos |
| 13 | 1 semana | Desktop security |
| 14 | 2 semanas | Troubleshooting |
| 15 | 1–2 semanas | Recovery |
| 16 | 1 semana | Automation |
| 17 | 2 semanas | Desktop engineering + final project |

---

## 18. Duración total

- Ritmo normal: ≈ 20–30 semanas
- Ritmo intensivo: ≈ 12–16 semanas
- Ritmo profundo: ≈ 6–8 meses

La duración no debe medirse solamente por horas estudiadas. Debe medirse por:

```
CONCEPTOS DOMINADOS + LABORATORIOS COMPLETADOS + ERRORES RESUELTOS + PROYECTOS TERMINADOS
```

---

## 19. Matriz de dominio final

Al terminar:

| Área | Básico | Intermedio | Avanzado | Experto |
|---|---|---|---|---|
| Instalación física | ✓ | ✓ | ✓ | ✓ |
| UEFI/Boot | ✓ | ✓ | ✓ | ✓ |
| GPU | ✓ | ✓ | ✓ | ✓ |
| Wayland/Xorg | ✓ | ✓ | ✓ | ✓ |
| GNOME/KDE | ✓ | ✓ | ✓ | |
| Tiling | ✓ | ✓ | ✓ | ✓ |
| Power Management | ✓ | ✓ | ✓ | ✓ |
| Audio | ✓ | ✓ | ✓ | ✓ |
| Networking desktop | ✓ | ✓ | ✓ | ✓ |
| Btrfs | ✓ | ✓ | ✓ | ✓ |
| Snapper | ✓ | ✓ | ✓ | ✓ |
| AUR | ✓ | ✓ | ✓ | ✓ |
| PKGBUILD | | ✓ | ✓ | ✓ |
| Dotfiles | ✓ | ✓ | ✓ | ✓ |
| Rice | ✓ | ✓ | ✓ | ✓ |
| Multimedia | ✓ | ✓ | ✓ | |
| Security | ✓ | ✓ | ✓ | ✓ |
| Troubleshooting | | ✓ | ✓ | ✓ |
| Recovery | | ✓ | ✓ | ✓ |
| Automation | | ✓ | ✓ | ✓ |
| Desktop Engineering | | | ✓ | ✓ |

---

## 20. Criterio de graduación

El estudiante no se considera graduado simplemente por completar los módulos. Debe poder realizar sin tutorial paso a paso:

```
1. Analizar hardware
2. Diseñar instalación
3. Instalar Arch
4. Configurar boot
5. Configurar GPU
6. Configurar desktop
7. Configurar red
8. Configurar audio
9. Configurar energía
10. Configurar Btrfs
11. Configurar snapshots
12. Configurar AUR
13. Crear dotfiles
14. Personalizar sistema
15. Mantener rolling release
16. Diagnosticar errores
17. Recuperar sistema
18. Documentar todo
```

---

## 21. Resultado final

Al terminar este segundo curso, el conocimiento acumulado debe quedar así:

```
                 LINUX
                   │
        ┌──────────┴──────────┐
        │                     │
    SERVER                  DESKTOP
        │                     │
    Curso 1                Curso 2
        │                     │
        ├── systemd           ├── GNOME
        ├── networking        ├── KDE
        ├── security          ├── Wayland
        ├── containers        ├── Xorg
        ├── DevOps            ├── GPU
        ├── kernel            ├── Audio
        ├── OSDev             ├── Battery
        └── programming       ├── Bluetooth
                              ├── Btrfs
                              ├── Snapper
                              ├── AUR
                              ├── Dotfiles
                              ├── Multimedia
                              ├── Hardware
                              └── Recovery
```

Los dos cursos juntos forman:

```
                    ARCH LINUX
                        │
          ┌─────────────┴─────────────┐
          │                           │
     INFRASTRUCTURE                 DESKTOP
          │                           │
          ↓                           ↓
       SERVERS                    DESKTOP (VM)
          │                           │
       DEVOPS                    HARDWARE
          │                           │
      SECURITY                    GRAPHICS
          │                           │
       KERNEL                      AUDIO
          │                           │
       OSDEV                       POWER
          │                           │
    PROGRAMMING                    BTRFS
          │                           │
     CONTAINERS                     AUR
          │                           │
     AUTOMATION                  DOTFILES
          │                           │
          └─────────────┬─────────────┘
                        ↓
              LINUX SYSTEM ENGINEER
```

---

## 22. Principio final del curso

El objetivo no es terminar diciendo "Tengo un Arch bonito." Ni siquiera "Sé instalar Arch."

El objetivo final es poder decir:

> "Entiendo mi sistema desde el hardware hasta el entorno gráfico, sé por qué funciona cada componente, puedo modificarlo de forma controlada, mantenerlo durante el rolling release, diagnosticarlo cuando falla y recuperarlo cuando se rompe."

Ese es el objetivo de **Arch Linux Desktop Master Course**.
