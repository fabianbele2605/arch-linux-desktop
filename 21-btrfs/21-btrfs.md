# Módulo 21 — Btrfs Desktop Architecture

**Fase 07 — Btrfs Desktop**

> **Nota de adaptación importante:** en el curso anterior instalaste tu sistema con LVM + ext4 (Módulo 11 de `arch-linux-mastery`) — el filesystem tradicional de servidor. Este módulo asume `Btrfs`, así que el primer paso es **confirmar qué tenés realmente** en tu VM actual. Si tu partición raíz es ext4, este módulo (y toda la Fase 07) va a ser mayormente conceptual sobre esta instalación existente — documentalo honestamente. Si en algún momento querés practicar Btrfs de verdad, la opción más simple es crear una imagen de disco nueva de prueba con `truncate`/`losetup` sin tocar tu sistema real, algo que vamos a ver en la sección 5.

---

## Objetivos del módulo

- Confirmar qué filesystem usa realmente tu VM, y entender la diferencia arquitectónica entre LVM+ext4 (lo que ya conocés) y Btrfs.
- Entender **copy-on-write (COW)**, el mecanismo que hace posible todo lo que vas a ver en los Módulos 22-24 (subvolúmenes, snapshots).
- Entender **subvolúmenes** como la unidad organizativa nativa de Btrfs, reemplazando el rol que jugaban las particiones LVM.
- Crear un filesystem Btrfs de prueba en una imagen de disco, sin tocar tu instalación real.

---

## 1. CONCEPTO: confirmá qué tenés

```bash
findmnt -T /
lsblk -f
cat /etc/fstab | grep -v "^#"
```

Si ves `ext4` como tipo de filesystem en la raíz, confirmá que efectivamente instalaste con LVM (Módulo 11 del curso anterior) — es el resultado esperado para la mayoría, y **no es un problema**: seguimos igual, aprendiendo Btrfs sobre una imagen de prueba en esta misma VM.

---

## 2. CONCEPTO: por qué Btrfs es una capa distinta, no "otro ext4"

```
LVM + ext4 (lo que ya conocés, curso anterior)
─────────────────────────────────────────────
Volumen físico (LVM) → Grupo de volúmenes → Volúmenes lógicos → ext4 encima de cada uno
  (gestión de espacio y filesystem son DOS capas separadas, con herramientas distintas: pvcreate/vgcreate/lvcreate + mkfs.ext4)

Btrfs (este módulo)
─────────────────────────────────────────────
Un solo filesystem Btrfs → subvolúmenes dentro de él (gestión de espacio y filesystem son LA MISMA capa, una sola herramienta: btrfs)
```

**La diferencia que más importa para el resto de esta fase:** en LVM, crear un snapshot de un volumen lógico requiere reservar espacio por adelantado y tiene overhead de rendimiento notable. En Btrfs, los snapshots (Módulo 23) son casi instantáneos y no requieren reservar nada de antemano — porque están construidos directamente sobre el mecanismo de copy-on-write del filesystem mismo, no agregados como una capa extra.

---

## 3. CONCEPTO: copy-on-write (COW) — la base de todo Btrfs

En un filesystem tradicional (ext4), modificar un archivo sobrescribe sus bloques de disco directamente. En Btrfs, modificar un archivo **nunca sobrescribe los bloques originales** — escribe los cambios en bloques nuevos, y solo después actualiza los punteros de metadata para que apunten a la versión nueva. Los bloques viejos quedan intactos hasta que nada los referencia.

```
ext4: escribís sobre el bloque original → el dato viejo se pierde inmediatamente
Btrfs: escribís en un bloque NUEVO → el puntero de metadata se actualiza → el bloque viejo sigue existiendo hasta que se libera
```

**Por qué esto es la base de los snapshots (Módulo 23):** un snapshot en Btrfs no es "copiar todos los archivos" — es simplemente **congelar el estado actual de los punteros de metadata**. Como los datos ya son inmutables una vez escritos (por el propio diseño COW), un snapshot es prácticamente gratis: no copia nada, solo fija una referencia. Recién cuando modificás archivos *después* del snapshot, Btrfs empieza a escribir bloques nuevos — el snapshot sigue apuntando a los viejos.

---

## 4. CONCEPTO: subvolúmenes — la unidad organizativa nativa

Un **subvolumen** en Btrfs es, conceptualmente, un filesystem independiente dentro del filesystem principal — con su propio árbol de directorios, pero compartiendo el mismo espacio de almacenamiento subyacente (sin la partición fija y el overhead que tenía un volumen lógico LVM).

**Patrón estándar que vas a instalar en el Módulo 24 (Snapper):** separar `/` y `/home` en subvolúmenes distintos, específicamente para poder hacer rollback del sistema **sin perder los datos del usuario** — un snapshot de `@` (subvolumen raíz) no toca `@home` en absoluto, son independientes pese a compartir el mismo disco físico.

---

## 5. HERRAMIENTA: crear un Btrfs de prueba, sin tocar tu sistema real

```bash
sudo pacman -S btrfs-progs
```

Creamos una imagen de disco de 1GB y la formateamos como Btrfs — completamente aislada de tu instalación real:

```bash
truncate -s 1G /tmp/btrfs-practica.img
mkfs.btrfs /tmp/btrfs-practica.img
sudo mkdir -p /mnt/btrfs-practica
sudo mount -o loop /tmp/btrfs-practica.img /mnt/btrfs-practica
```

```bash
df -hT /mnt/btrfs-practica
sudo btrfs filesystem show /mnt/btrfs-practica
```

---

## 6. HERRAMIENTA: crear tu primer subvolumen

```bash
sudo btrfs subvolume create /mnt/btrfs-practica/@subvol-prueba
sudo btrfs subvolume list /mnt/btrfs-practica
```

```bash
echo "archivo de prueba en el subvolumen" | sudo tee /mnt/btrfs-practica/@subvol-prueba/archivo.txt
ls /mnt/btrfs-practica/@subvol-prueba/
```

---

## 7. PRÁCTICA

1. Confirmá qué filesystem usa realmente tu partición raíz (`findmnt -T /`), y documentá el resultado honestamente.
2. Explicá con tus propias palabras la diferencia entre la arquitectura de dos capas de LVM+ext4 y la capa única de Btrfs.
3. Explicá con tus propias palabras por qué copy-on-write hace que los snapshots sean casi instantáneos, en vez de una copia completa de archivos.
4. Creá el Btrfs de prueba en una imagen de disco, montalo, y confirmá con `btrfs filesystem show`.
5. Creá un subvolumen, escribí un archivo dentro, y confirmalo con `btrfs subvolume list`.

---

## 8. ERROR INTENCIONAL / DIAGNÓSTICO

```bash
sudo btrfs subvolume delete /mnt/btrfs-practica/@subvol-que-no-existe
```

**Diagnóstico esperado:** un error indicando que la ruta no existe o no es un subvolumen válido — `btrfs` valida el subvolumen antes de intentar eliminarlo, mismo patrón de validación estricta que el curso viene repitiendo desde `pw-link` (Módulo 15).

Al terminar, desmontá y limpiá la práctica (no afecta tu sistema real, pero es buena costumbre):

```bash
sudo umount /mnt/btrfs-practica
rm /tmp/btrfs-practica.img
```

---

## Checklist de cierre del módulo

- [ ] Confirmé qué filesystem usa realmente mi VM (ext4/LVM del curso anterior, lo más probable).
- [ ] Entiendo la diferencia arquitectónica entre LVM+ext4 (dos capas) y Btrfs (una capa única).
- [ ] Entiendo copy-on-write y por qué hace que los snapshots sean casi instantáneos.
- [ ] Entiendo qué es un subvolumen y por qué separar `/` y `/home` en subvolúmenes distintos importa para rollback (anticipo del Módulo 24).
- [ ] Creé un Btrfs de prueba en una imagen de disco, sin tocar mi sistema real.
- [ ] Creé un subvolumen y confirmé su contenido.
- [ ] Provoqué y diagnostiqué el error de eliminar un subvolumen inexistente.

---

## Evidencias

_(pendiente — se agregan capturas reales a medida que se completa el módulo)_

---

**Próximo módulo:** 22 — Subvolumes.
