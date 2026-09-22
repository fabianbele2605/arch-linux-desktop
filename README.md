# Arch Linux Desktop Master Course

**De Administrador Linux a Ingeniero de Sistemas Desktop**

![Progreso](https://img.shields.io/badge/progreso-0%2F55%20m%C3%B3dulos-lightgrey)
![Fases](https://img.shields.io/badge/fases-0%2F18%20completas-blue)
![Método](https://img.shields.io/badge/m%C3%A9todo-hands--on%20en%20VirtualBox-informational)

---

## 🎯 De qué se trata esto

Segundo curso propio, continuación directa de [**Arch Linux Master Course**](https://github.com/fabianbele2605/arch-linux-mastery) — que cubrió administración, servidores, networking, seguridad, DevOps y kernel. Este curso completa lo que quedó afuera: **Arch Linux como sistema de escritorio completo**, con su ecosistema entero de herramientas — entornos gráficos, audio, Btrfs/Snapper, AUR a fondo, dotfiles, mantenimiento de rolling release, y troubleshooting/recovery de escritorio.

Se cursa en una VM de **VirtualBox** (mismo entorno que el curso anterior), no en hardware físico — la laptop del estudiante corre Ubuntu como sistema anfitrión y se usa activamente para trabajar. Esta decisión está documentada y cada módulo que depende de hardware que VirtualBox no puede virtualizar de forma realista (Wi-Fi real, Bluetooth real, batería/ACPI real, GPU dedicada) queda marcado explícitamente como **adaptado** u **opcional** en la guía, explicando qué parte del concepto sigue siendo aprendible en la VM.

📖 **[Ver la guía completa del curso](docs/guia.md)** — filosofía, fases, metodología, evaluación, proyectos y cronograma.

---

## Progreso

### Fase 00 — Hardware Discovery
- [ ] [00 — Hardware Discovery & Preinstallation](00-hardware-discovery/00-hardware-discovery.md)

### Fase 01 — Instalación real (en VM)
- [ ] [01 — Arch Linux Installation](01-real-installation/01-real-installation.md)
- [ ] [02 — Manual Installation vs Archinstall](02-archinstall-vs-manual/02-archinstall-vs-manual.md)
- [ ] [03 — UEFI, Bootloaders & Secure Boot](03-uefi-bootloaders/03-uefi-bootloaders.md)

### Fase 02 — Gráficos
- [ ] [04 — Graphics Architecture](04-graphics-architecture/04-graphics-architecture.md)
- [ ] [05 — Xorg vs Wayland](05-xorg-wayland/05-xorg-wayland.md)
- [ ] [06 — GPU Drivers, Mesa & Vulkan](06-gpu-drivers/06-gpu-drivers.md)

### Fase 03 — Desktop Environments
- [ ] [07 — Desktop Architecture](07-desktop-architecture/07-desktop-architecture.md)
- [ ] [08 — GNOME](08-gnome/08-gnome.md)
- [ ] [09 — KDE Plasma](09-kde-plasma/09-kde-plasma.md)
- [ ] [10 — Window Managers & Compositors](10-window-managers/10-window-managers.md)

### Fase 04 — Power Management (teórico en VM)
- [ ] [11 — Laptop Power Management](11-power-management/11-power-management.md)
- [ ] [12 — Battery, ACPI, Suspend & Hibernate](12-battery-suspend/12-battery-suspend.md)
- [ ] [13 — CPU/GPU Power Optimization](13-power-optimization/13-power-optimization.md)

### Fase 05 — Audio
- [ ] [14 — Audio Architecture](14-audio-architecture/14-audio-architecture.md)
- [ ] [15 — PipeWire, WirePlumber & ALSA](15-pipewire-alsa/15-pipewire-alsa.md)
- [ ] [16 — Bluetooth Audio](16-bluetooth-audio/16-bluetooth-audio.md)

### Fase 06 — Desktop Networking
- [ ] [17 — NetworkManager](17-networkmanager/17-networkmanager.md)
- [ ] [18 — Wi-Fi (adaptado)](18-wifi/18-wifi.md)
- [ ] [19 — Bluetooth (opcional)](19-bluetooth/19-bluetooth.md)
- [ ] [20 — Printing & CUPS](20-cups-printing/20-cups-printing.md)

### Fase 07 — Btrfs Desktop
- [ ] [21 — Btrfs Desktop Architecture](21-btrfs/21-btrfs.md)
- [ ] [22 — Subvolumes](22-btrfs-subvolumes/22-btrfs-subvolumes.md)
- [ ] [23 — Snapshots](23-btrfs-snapshots/23-btrfs-snapshots.md)
- [ ] [24 — Snapper & Rollback](24-snapper-rollback/24-snapper-rollback.md)

### Fase 08 — AUR Profesional
- [ ] [25 — AUR Deep Dive](25-aur/25-aur.md)
- [ ] [26 — makepkg & PKGBUILD](26-makepkg-pkgbuild/26-makepkg-pkgbuild.md)
- [ ] [27 — AUR Helpers](27-aur-helpers/27-aur-helpers.md)
- [ ] [28 — Package Cache & Maintenance](28-package-cache/28-package-cache.md)

### Fase 09 — Rolling Release Engineering
- [ ] [29 — Rolling Release Maintenance](29-rolling-release/29-rolling-release.md)
- [ ] [30 — Pacman Hooks](30-pacman-hooks/30-pacman-hooks.md)
- [ ] [31 — Mirrors & Reflector](31-mirrors-reflector/31-mirrors-reflector.md)
- [ ] [32 — Journald & Maintenance](32-journald-maintenance/32-journald-maintenance.md)

### Fase 10 — Dotfiles & Rice
- [ ] [33 — Dotfiles](33-dotfiles/33-dotfiles.md)
- [ ] [34 — GNU Stow / chezmoi](34-stow-chezmoi/34-stow-chezmoi.md)
- [ ] [35 — Themes, Fonts & Desktop Configuration](35-fonts-themes/35-fonts-themes.md)
- [ ] [36 — Linux Rice Engineering](36-linux-rice/36-linux-rice.md)

### Fase 11 — Multimedia & Productivity
- [ ] [37 — Multimedia](37-multimedia/37-multimedia.md)
- [ ] [38 — Codecs & Hardware Acceleration](38-hardware-acceleration/38-hardware-acceleration.md)
- [ ] [39 — Productivity Applications](39-productivity/39-productivity.md)

### Fase 12 — Periféricos (vía USB passthrough)
- [ ] [40 — External Displays & Docking](40-external-displays/40-external-displays.md)
- [ ] [41 — Cameras, Microphones & Peripherals](41-camera-microphone/41-camera-microphone.md)
- [ ] [42 — USB & udev](42-usb-udev/42-usb-udev.md)

### Fase 13 — Desktop Security
- [ ] [43 — Desktop Security](43-desktop-security/43-desktop-security.md)
- [ ] [44 — Secrets, Keyrings & Credentials](44-keyrings-secrets/44-keyrings-secrets.md)
- [ ] [45 — Sandboxing & Desktop Isolation](45-application-sandboxing/45-application-sandboxing.md)

### Fase 14 — Troubleshooting
- [ ] [46 — Desktop Troubleshooting](46-desktop-troubleshooting/46-desktop-troubleshooting.md)
- [ ] [47 — Break & Fix Laboratory](47-break-fix/47-break-fix.md)

### Fase 15 — Recovery
- [ ] [48 — Recovery & Disaster Recovery](48-disaster-recovery/48-disaster-recovery.md)
- [ ] [49 — Btrfs Recovery](49-btrfs-recovery/49-btrfs-recovery.md)
- [ ] [50 — Boot Recovery](50-boot-recovery/50-boot-recovery.md)

### Fase 16 — Automatización Desktop
- [ ] [51 — Advanced Desktop Automation](51-desktop-automation/51-desktop-automation.md)
- [ ] [52 — System Integration](52-system-integration/52-system-integration.md)

### Fase 17 — Desktop Engineering
- [ ] [53 — Desktop Engineering](53-desktop-engineering/53-desktop-engineering.md)
- [ ] [54 — Final Integrated Project](54-final-project/54-final-project.md)

---

## Break & Fix Laboratory (Módulo 47)

Incidentes simulados de escritorio, misma metodología SÍNTOMA → OBSERVACIÓN → LOG → CAUSA RAÍZ → SOLUCIÓN → VALIDACIÓN:

- [ ] #1 — Xorg no inicia
- [ ] #2 — Wayland no inicia
- [ ] #3 — Audio mudo
- [ ] #4 — Micrófono desaparece
- [ ] #5 — La red (Ethernet virtual) se cae
- [ ] #6 — Bluetooth no conecta (opcional, USB passthrough)
- [ ] #7 — Batería no detectada (teórico en VM)
- [ ] #8 — Suspend/resume pierde la red
- [ ] #9 — GPU acceleration desaparece
- [ ] #10 — Monitor externo no funciona
- [ ] #11 — Btrfs snapshot no aparece
- [ ] #12 — Rollback falla
- [ ] #13 — AUR package fails
- [ ] #14 — AUR helper conflicts
- [ ] #15 — Disk full
- [ ] #16 — Desktop tarda demasiado en iniciar
- [ ] #17 — Broken pacman transaction
- [ ] #18 — Bootloader roto

---

## Cómo está documentado cada módulo

Cada carpeta de módulo tiene su archivo `.md` con teoría + práctica, y una carpeta `evidencias/` con capturas reales de la VM, cada una con una descripción de qué comando se corrió y qué pasó — incluyendo los errores reales y cómo se diagnosticaron y resolvieron. Igual metodología que [arch-linux-mastery](https://github.com/fabianbele2605/arch-linux-mastery).

## Estructura del repositorio

```
arch-linux-desktop/
├── docs/guia.md                 ← guía completa del curso
├── 00-hardware-discovery/ ... 54-final-project/    ← un módulo por carpeta, con su evidencias/
├── labs/                          ← laboratorios de incidentes simulados
├── projects/                        ← proyectos progresivos del curso
├── scripts/                           ← automatización (Módulos 51-52)
├── dotfiles/                            ← dotfiles versionados (Módulos 33-36)
├── pkgbuilds/                              ← PKGBUILDs propios (Módulo 26)
├── cheatsheets/                              ← referencias rápidas
└── incident-reports/                            ← incidentes reales documentados
```

## Entorno

- Arch Linux instalado en una VM de **VirtualBox** nueva (distinta de la del curso anterior), con EFI habilitado y aceleración 3D activada.
- La laptop física (host) corre Ubuntu y no se toca — el hardware real (Wi-Fi, Bluetooth, batería) queda fuera de alcance salvo passthrough USB puntual.
- Cada módulo riesgoso se practica con snapshot de VirtualBox tomado de antemano.
