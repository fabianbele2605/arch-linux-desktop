# Módulo 08 — GNOME

**Fase 03 — Desktop Environments**

---

## Objetivos del módulo

- Entender la arquitectura de GNOME: GNOME Shell, Mutter (compositor), GTK, GSettings/dconf.
- Instalar GNOME completo y arrancarlo por primera vez con `sddm` (ya preparado en el Módulo 07).
- Explorar la configuración vía `dconf`/`gsettings` en vez de archivos de texto sueltos.
- Entender extensiones de GNOME Shell como mecanismo de personalización.

---

## 1. CONCEPTO: arquitectura de GNOME

```
GNOME Shell (la interfaz: panel superior, vista de actividades, notificaciones)
        ↓ construido sobre
Mutter (el compositor — gestiona ventanas, efectos, y también es un cliente de Wayland/DRM)
        ↓
GTK (el toolkit gráfico — cómo se dibujan botones, menús, ventanas de todas las apps GNOME)
        ↓
GSettings / dconf (el sistema de configuración — reemplaza archivos de config sueltos)
```

**Por qué GNOME Shell y Mutter están tan integrados (a diferencia de, por ejemplo, KDE que separa más las piezas):** GNOME apuesta por una experiencia muy cohesiva y opinada — menos piezas intercambiables, pero una integración más pulida entre ellas. Esto es una decisión de diseño, no una limitación técnica.

---

## 2. HERRAMIENTA: instalar GNOME

```bash
sudo pacman -S gnome
```

**Nota:** este es un grupo de paquetes grande (te va a preguntar qué paquetes opcionales instalar dentro del grupo — para este módulo, podés aceptar todos con Enter, o revisar la lista si preferís algo más liviano). Va a tardar bastante y descargar varios GB — es normal.

```bash
sudo systemctl start sddm
```

Esto debería mostrarte, por primera vez en toda esta VM, una **pantalla de login gráfica real** (SDDM), en vez de una consola de texto. Iniciá sesión con tu usuario.

---

## 3. EJEMPLO: primera exploración de GNOME

Una vez dentro de tu primera sesión GNOME real:

```bash
echo $XDG_SESSION_TYPE           # ahora sí debería confirmar "wayland" (GNOME moderno usa Wayland por defecto)
echo $XDG_CURRENT_DESKTOP          # "GNOME"
glxinfo | grep -i "opengl renderer"  # confirmá que sigue siendo SVGA3D, como en el Módulo 05/06
```

Explorá brevemente la interfaz: el panel superior, la vista de "Actividades" (tecla Super o esquina superior izquierda), y el menú de configuración rápida (esquina superior derecha).

---

## 4. CONCEPTO: GSettings y dconf — configuración sin archivos sueltos

GNOME no guarda su configuración en archivos de texto tipo `.ini` esparcidos — usa una **base de datos de configuración centralizada** llamada `dconf`, accedida mediante la API `GSettings`.

```bash
gsettings list-schemas | head -10        # esquemas de configuración disponibles
gsettings get org.gnome.desktop.interface color-scheme    # tema claro/oscuro actual
gsettings set org.gnome.desktop.interface color-scheme "prefer-dark"   # cambialo
```

**Por qué este modelo en vez de archivos:** permite validación de tipos (no podés poner texto donde se espera un número), valores por defecto centralizados, y que herramientas gráficas (Configuración del Sistema) y la línea de comandos lean/escriban exactamente el mismo lugar sin desincronizarse.

```bash
sudo pacman -S dconf-editor
dconf-editor    # explorador gráfico completo del árbol de configuración
```

---

## 5. CONCEPTO: extensiones de GNOME Shell

GNOME Shell tiene una interfaz deliberadamente minimalista "de fábrica" — la personalización profunda (barras de tareas alternativas, indicadores extra) se logra vía **extensiones**, pequeños paquetes de JavaScript que modifican el Shell en tiempo de ejecución.

```bash
sudo pacman -S gnome-shell-extensions gnome-extensions-app
gnome-extensions list
```

Podés explorar e instalar más en `extensions.gnome.org` desde el navegador (si tenés uno instalado — sino, lo vas a instalar en el Módulo 39, Productividad).

---

## 6. PRÁCTICA

1. Instalá el grupo `gnome` completo y arrancá `sddm`.
2. Iniciá sesión gráficamente por primera vez, y confirmá `$XDG_SESSION_TYPE` y `$XDG_CURRENT_DESKTOP`.
3. Cambiá el tema claro/oscuro con `gsettings set`, y confirmá visualmente el cambio en la interfaz.
4. Instalá `dconf-editor` y navegá el árbol `org.gnome.desktop` explorando 2-3 categorías de configuración.
5. Listá las extensiones instaladas con `gnome-extensions list`.

---

## 7. ERROR INTENCIONAL / DIAGNÓSTICO

```bash
gsettings set org.gnome.desktop.interface color-scheme "un-valor-invalido"
```

**Diagnóstico esperado:** `gsettings` debería rechazar el valor (`GLib.GError` o similar), porque ese esquema define un **enum** de valores válidos (`default`, `prefer-dark`, `prefer-light`) — no acepta texto arbitrario. Esto ilustra la validación de tipos que mencionamos en la sección 4, algo que un archivo de texto plano nunca podría garantizar por sí solo.

---

## Checklist de cierre del módulo

- [ ] Entiendo la arquitectura GNOME Shell + Mutter + GTK + GSettings/dconf.
- [ ] Instalé GNOME completo y logré mi primera sesión gráfica real vía SDDM.
- [ ] Confirmé `$XDG_SESSION_TYPE` (wayland) y `$XDG_CURRENT_DESKTOP` (GNOME) desde dentro de la sesión.
- [ ] Cambié una configuración con `gsettings` y vi el efecto real en la interfaz.
- [ ] Entiendo el rol de las extensiones de GNOME Shell.

---

## Evidencias

_(pendiente — se agregan capturas reales a medida que se completa el módulo)_

---

**Próximo módulo:** 09 — KDE Plasma.
