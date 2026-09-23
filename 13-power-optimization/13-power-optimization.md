# Módulo 13 — CPU/GPU Power Optimization

**Fase 04 — Power Management**

Con este módulo cerramos la Fase 04.

---

## Objetivos del módulo

- Entender CPU frequency scaling: governors, y por qué la CPU no corre siempre a su velocidad máxima.
- Usar `cpupower` para inspeccionar y cambiar el comportamiento de energía de la CPU.
- Instalar y entender `TLP`, el demonio de optimización de energía más usado en Arch.
- Entender qué de esto es real y qué está limitado por la virtualización (cierre de la Fase 04, conectando los Módulos 11-13).

---

## 1. CONCEPTO: CPU frequency scaling y governors

Las CPU modernas no corren siempre a su velocidad máxima — ajustan dinámicamente su frecuencia según la carga, para ahorrar energía cuando no hace falta rendimiento máximo. El kernel gestiona esto mediante **governors** (políticas de escalado):

| Governor | Comportamiento |
|---|---|
| `performance` | Siempre a la frecuencia máxima — máximo rendimiento, máximo consumo |
| `powersave` | Siempre a la frecuencia mínima — mínimo consumo, mínimo rendimiento |
| `schedutil` | El más moderno: el propio scheduler del kernel decide la frecuencia según la carga real, en tiempo real |
| `ondemand` / `conservative` | Governors más antiguos, hoy generalmente reemplazados por `schedutil` |

**Por qué existe esto:** una CPU a máxima frecuencia constante gasta mucha más batería innecesariamente cuando estás, por ejemplo, solo leyendo texto en pantalla. El escalado dinámico es la diferencia entre horas y minutos de autonomía en una laptop real.

---

## 2. HERRAMIENTA: `cpupower`

```bash
sudo pacman -S cpupower
cpupower frequency-info
```

```bash
cpupower frequency-info | grep "governor"    # ¿qué governor está activo ahora?
sudo cpupower frequency-set -g performance      # cambiar a máximo rendimiento
sudo cpupower frequency-set -g powersave          # cambiar a mínimo consumo
```

**Nota de adaptación importante:** en tu VM, la CPU que ves (`vCPU`) es una **capa virtual** sobre los núcleos físicos reales de tu laptop host (Ubuntu) — el host es quien realmente controla el escalado de frecuencia del hardware físico. Es probable que `cpupower frequency-info` muestre información limitada, genérica, o directamente indique que no puede consultar/cambiar la frecuencia real. **Documentá exactamente lo que veas** — es información real sobre los límites de la virtualización, no un fallo tuyo.

---

## 3. HERRAMIENTA: `TLP` — optimización automática integral

```bash
sudo pacman -S tlp
sudo systemctl enable --now tlp
sudo tlp-stat -s     # resumen general del estado
```

`TLP` va mucho más allá de solo el governor de CPU: ajusta automáticamente USB autosuspend, energía de discos, comportamiento de red inalámbrica (Módulo 18, más adelante), brillo, y docenas de parámetros más — todo con perfiles sensibles por defecto, sin que tengas que tocar cada uno a mano.

```bash
sudo tlp-stat -p    # estado detallado de energía de procesador
cat /etc/tlp.conf | grep -v "^#" | grep -v "^$" | head -20
```

**Conexión con el Módulo 07 (curso actual) y el Módulo 09 (curso anterior):** `TLP` corre como un servicio `systemd` más — mismo patrón de gestión (`enable`, `status`, archivo de configuración en `/etc/`) que ya dominás de sobra a esta altura de ambos cursos.

---

## 4. CONCEPTO: cierre de la Fase 04 — qué aprendiste realmente sobre energía en una VM

| Módulo | Qué se confirmó real | Qué quedó limitado por virtualización |
|---|---|---|
| 11 | Batería virtual funcional (`acpi`, `upower`, D-Bus) | `powertop` sin datos de consumo significativos |
| 12 | `zram` incompatible con hibernate (conceptual, 100% real) | `systemctl suspend` no resumía correctamente |
| 13 | Sintaxis y herramientas (`cpupower`, `TLP`) | Control real de frecuencia de CPU (pertenece al host) |

**La lección de toda la fase, dicha una sola vez con claridad:** no aprendiste "gestión de energía falsa" — aprendiste la **arquitectura completa y las herramientas reales** que vas a usar sin cambios el día que instales Arch en hardware físico (algo que la guía original de este curso preveía con la nota "adaptado" en cada módulo). Lo que cambia entre VM y hardware real no es el conocimiento, es solo cuánto control final tiene el kernel sobre el hardware de abajo.

---

## 5. PRÁCTICA

1. Instalá `cpupower`, corré `frequency-info`, y documentá exactamente qué información real (o limitada) te muestra tu VM.
2. Probá cambiar el governor a `performance` y `powersave`, confirmando si el cambio se aplica o si el sistema lo rechaza/ignora.
3. Instalá y habilitá `TLP`, corré `tlp-stat -s` y `tlp-stat -p`.
4. Completá, con tus propias palabras, la fila del Módulo 13 en la tabla de la sección 4.
5. Reflexión de cierre de fase: de los 3 módulos de Power Management, ¿cuál te pareció el más valioso para entender la arquitectura real de Linux, independientemente de la limitación de estar en una VM?

---

## 6. ERROR INTENCIONAL / DIAGNÓSTICO

```bash
sudo cpupower frequency-set -g governor_que_no_existe
```

**Diagnóstico esperado:** un error indicando que ese governor no es válido — el kernel expone una lista fija de governors disponibles (`cpupower frequency-info` te la muestra), y no acepta valores arbitrarios. Mismo patrón de validación estricta que ya viste en `gsettings` (Módulo 08) y `plasma-apply-colorscheme` (Módulo 09), aplicado ahora al kernel mismo.

---

## Checklist de cierre del módulo (y de la Fase 04 completa)

- [ ] Entiendo qué son los governors de CPU y por qué existe el escalado dinámico de frecuencia.
- [ ] Usé `cpupower` y documenté honestamente las limitaciones de virtualización que encontré.
- [ ] Instalé y exploré `TLP` como demonio integral de optimización.
- [ ] Completé la tabla de cierre de fase, distinguiendo qué fue real y qué estuvo limitado.
- [ ] Puedo explicar por qué el conocimiento de esta fase se transfiere completo a hardware físico real, aunque algunos resultados numéricos no.

---

## Evidencias

_(pendiente — se agregan capturas reales a medida que se completa el módulo)_

---

## Cierre de Fase 04 — Power Management

Con este módulo termina la Fase 04: ACPI y la batería virtual (Módulo 11), suspend/hibernate y el hallazgo real de la incompatibilidad `zram` (Módulo 12), y optimización de CPU/GPU con sus límites honestos de virtualización (Módulo 13). Cada módulo distinguió explícitamente entre "arquitectura y herramientas reales" y "resultados numéricos limitados por estar en una VM" — la habilidad de saber cuál es cuál es, en sí misma, parte de lo que este curso buscaba enseñar.

---

**Próximo módulo:** 14 — Audio Architecture (inicio de la Fase 05 — Audio).
