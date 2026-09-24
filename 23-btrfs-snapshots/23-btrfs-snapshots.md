# Módulo 23 — Snapshots

**Fase 07 — Btrfs Desktop**

> **Nota de adaptación:** misma base que los Módulos 21-22 — seguimos sobre ext4 real, así que la práctica corre sobre una imagen de disco de prueba nueva, sin tocar tu sistema.

---

## Objetivos del módulo

- Entender que un snapshot de Btrfs **es** un subvolumen — no una entidad separada, sino un caso particular de lo que ya practicaste en el Módulo 22.
- Ver copy-on-write en acción real: crear un snapshot, modificar el original, y confirmar que el snapshot no cambió.
- Entender la diferencia entre snapshots de solo lectura y de lectura-escritura, y por qué Snapper (Módulo 24) usa los primeros para el sistema.
- Medir cuánto espacio "cuesta" realmente un snapshot recién creado — la prueba más concreta de que COW no copia nada por adelantado.

---

## 1. CONCEPTO: un snapshot ES un subvolumen

En el Módulo 22 creaste subvolúmenes desde cero, vacíos. Un **snapshot** es exactamente lo mismo — un subvolumen nuevo — pero en vez de arrancar vacío, arranca **compartiendo todos los datos** del subvolumen origen en el momento exacto de la creación:

```bash
btrfs subvolume snapshot <origen> <destino>
```

Es, literalmente, el mismo comando `btrfs subvolume` que ya usaste, con un origen que no está vacío. No hay una herramienta ni un concepto separado — `btrfs subvolume list` (Módulo 22) muestra los snapshots exactamente igual que cualquier otro subvolumen, porque técnicamente lo son.

---

## 2. HERRAMIENTA: crear el entorno de prueba y el primer snapshot

```bash
truncate -s 1G /tmp/btrfs-snap.img
mkfs.btrfs /tmp/btrfs-snap.img
sudo mkdir -p /mnt/btrfs-snap
sudo mount /tmp/btrfs-snap.img /mnt/btrfs-snap
```

Creamos un subvolumen con contenido real, simulando un "sistema" con un archivo de configuración:

```bash
sudo btrfs subvolume create /mnt/btrfs-snap/@sistema
echo "version=1.0" | sudo tee /mnt/btrfs-snap/@sistema/config.txt
```

Ahora el snapshot, **de solo lectura** (`-r`) — el tipo que usa Snapper para el sistema:

```bash
sudo btrfs subvolume snapshot -r /mnt/btrfs-snap/@sistema /mnt/btrfs-snap/@sistema-snap1
sudo btrfs subvolume list /mnt/btrfs-snap
```

Fijate que `@sistema-snap1` aparece en la lista exactamente como `@sistema` — un subvolumen más, con un ID propio.

---

## 3. CONCEPTO: copy-on-write en acción real

Ahora modificamos el **original** después de haber sacado el snapshot:

```bash
echo "version=2.0 - ROTO A PROPOSITO" | sudo tee /mnt/btrfs-snap/@sistema/config.txt
```

Comparemos:

```bash
cat /mnt/btrfs-snap/@sistema/config.txt
cat /mnt/btrfs-snap/@sistema-snap1/config.txt
```

**Esto es la prueba concreta de COW (Módulo 21) funcionando:** el original ahora dice `version=2.0`, pero el snapshot **sigue congelado en `version=1.0`** — exactamente como estaba en el instante en que lo creaste. Nada se copió por adelantado; Btrfs simplemente escribió el cambio en bloques nuevos para el original, dejando intactos los bloques viejos a los que el snapshot sigue apuntando. Esto es, en miniatura, exactamente lo que hace posible un rollback con Snapper (Módulo 24): "volver" a este snapshot sería tan simple como hacer que `@sistema` vuelva a apuntar a los datos congelados de `@sistema-snap1`.

---

## 4. CONCEPTO: solo lectura vs. lectura-escritura

```bash
echo "intento de escritura" | sudo tee /mnt/btrfs-snap/@sistema-snap1/nuevo.txt
```

**Resultado esperado:** falla — el snapshot `-r` es inmutable, ni siquiera `root` puede escribir en él directamente. Esto es intencional y central para el caso de uso de Snapper: un snapshot del sistema que pudiera modificarse dejaría de ser una "fotografía" confiable del pasado.

Comparemos creando uno de lectura-escritura:

```bash
sudo btrfs subvolume snapshot /mnt/btrfs-snap/@sistema /mnt/btrfs-snap/@sistema-snap2-rw
echo "esto si deberia funcionar" | sudo tee /mnt/btrfs-snap/@sistema-snap2-rw/nuevo.txt
ls /mnt/btrfs-snap/@sistema-snap2-rw/
```

**Cuándo se usan los de lectura-escritura en la práctica:** son la base de flujos como "probar una actualización de forma segura" — hacés snapshot RW, actualizás sobre esa copia, y si algo sale mal, la descartás sin haber tocado el sistema real. No es el caso de uso de Snapper (que quiere fotografías inmutables), pero sí de herramientas como `snap-pac` que verás mencionadas en el Módulo 24.

---

## 5. HERRAMIENTA: medir el "costo" real de un snapshot

```bash
sudo btrfs filesystem du -s /mnt/btrfs-snap/@sistema /mnt/btrfs-snap/@sistema-snap1
```

**Lo que deberías ver:** el snapshot ocupa prácticamente el mismo espacio que compartía con el original al momento de crearse — no hay una copia duplicada de datos. El espacio real adicional que "cuesta" un snapshot recién creado es, en la práctica, casi cero — solo empieza a crecer a medida que el original (o el snapshot, si es RW) diverge con cambios nuevos.

---

## 6. PRÁCTICA

1. Creá el entorno de prueba y un subvolumen con contenido real.
2. Creá un snapshot de solo lectura, y confirmá que aparece en `btrfs subvolume list`.
3. Modificá el original después del snapshot, y confirmá que el snapshot no cambió — la demostración concreta de COW.
4. Confirmá que el snapshot de solo lectura rechaza escrituras, y creá uno de lectura-escritura como contraste.
5. Medí el espacio real que ocupa el snapshot con `btrfs filesystem du -s`.
6. Reflexión: explicá con tus propias palabras por qué Snapper (Módulo 24) va a usar exclusivamente snapshots de solo lectura para el sistema, y en qué escenario tendría sentido usar uno de lectura-escritura en cambio.

---

## 7. ERROR INTENCIONAL / DIAGNÓSTICO

```bash
sudo btrfs subvolume snapshot /mnt/btrfs-snap/@subvolumen-que-no-existe /mnt/btrfs-snap/@snap-fallido
```

**Diagnóstico esperado:** un error indicando que la ruta de origen no existe o no es un subvolumen válido — `btrfs subvolume snapshot` valida el origen antes de intentar nada, mismo patrón del curso.

Al terminar, limpiá:

```bash
sudo umount /mnt/btrfs-snap
rm /tmp/btrfs-snap.img
```

---

## Checklist de cierre del módulo

- [ ] Entiendo que un snapshot ES un subvolumen, no una entidad separada.
- [ ] Vi copy-on-write funcionando en la práctica: el original cambió, el snapshot no.
- [ ] Entiendo la diferencia entre snapshots de solo lectura y de lectura-escritura, y cuándo se usa cada uno.
- [ ] Medí el espacio real que ocupa un snapshot recién creado, confirmando que COW no duplica datos por adelantado.
- [ ] Provoqué y diagnostiqué el error de snapshotear un subvolumen inexistente.

---

## Evidencias

_(pendiente — se agregan capturas reales a medida que se completa el módulo)_

---

**Próximo módulo:** 24 — Snapper & Rollback.
