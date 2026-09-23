# Módulo 18 — Wi-Fi

**Fase 06 — Desktop Networking (adaptado)**

> **Nota de adaptación:** ya lo confirmamos en el Módulo 17 — `nmcli general status` mostró `WIFI: enabled` pero `WIFI-HW: missing`. VirtualBox no emula ningún adaptador Wi-Fi (a diferencia de la batería, Módulo 11), así que este módulo es mayormente teórico. Si tenés un adaptador Wi-Fi USB y podés pasarlo por passthrough a la VM (igual que se mencionó para Bluetooth, Módulo 16), las secciones prácticas van a funcionar contra hardware real.

---

## Objetivos del módulo

- Entender el rol de `wpa_supplicant`/`iwd` como motor de autenticación Wi-Fi, y cómo NetworkManager (Módulo 17) lo orquesta.
- Entender WPA2 vs. WPA3, y por qué el segundo no es simplemente "una versión más nueva del mismo protocolo".
- Entender redes ocultas y roaming entre puntos de acceso.
- Documentar honestamente hasta dónde llega el Wi-Fi en tu entorno concreto.

---

## 1. CONCEPTO: dónde encaja Wi-Fi en la arquitectura que ya conocés

```
Hardware Wi-Fi (adaptador USB o integrado)
        ↓
Kernel (driver del chipset — ej. ath9k, iwlwifi, rtl8xxxu)
        ↓
wpa_supplicant / iwd (negocia WPA2/WPA3 con el punto de acceso — YA LO USASTE en el curso anterior)
        ↓
NetworkManager (Módulo 17 — orquesta, guarda perfiles, decide a cuál red conectarse)
        ↓
nmcli / nmtui / applet gráfico
```

**Nada nuevo en la cadena, solo una rama distinta:** es exactamente la misma arquitectura del Módulo 17, con Wi-Fi ocupando el lugar que ocupaba Ethernet. La diferencia real es que Wi-Fi necesita **negociación de seguridad** antes de poder pasar tráfico — de ahí el rol central de `wpa_supplicant`/`iwd`.

```bash
nmcli device status | grep -i wifi
rfkill list    # confirma si hay algún adaptador inalámbrico bloqueado por software o hardware
```

**Resultado esperado en esta VM:** `rfkill list` probablemente no liste nada relacionado a Wi-Fi (ningún dispositivo detectado) — coherente con `WIFI-HW: missing` del Módulo 17.

---

## 2. CONCEPTO: WPA2 vs. WPA3 — no es solo "más nuevo"

| | WPA2 | WPA3 |
|---|---|---|
| Intercambio de claves | 4-way handshake (vulnerable a ataques offline de fuerza bruta contra la contraseña capturada) | SAE (*Simultaneous Authentication of Equals*) — resistente a ataques offline, cada intento de adivinar la clave requiere interacción en vivo con el punto de acceso |
| Forward secrecy | No — si alguien captura tráfico cifrado y después obtiene la contraseña, puede descifrar tráfico pasado | Sí — cada sesión tiene su propia clave derivada, comprometer la contraseña no compromete capturas anteriores |
| Redes abiertas | Sin cifrado real | OWE (*Opportunistic Wireless Encryption*) — cifra incluso redes "abiertas" sin contraseña |

**Por qué esto importa más allá de la sigla:** SAE es la razón real por la que WPA3 es más difícil de atacar — no es una mejora incremental de WPA2, es un protocolo de intercambio de claves fundamentalmente distinto (conexión directa con lo que viste de criptografía y protocolos en el curso anterior, Módulo de seguridad).

```bash
iw list 2>/dev/null | grep -A5 "Supported extended features" || echo "sin adaptador Wi-Fi para consultar capacidades"
```

---

## 3. CONCEPTO: redes ocultas y roaming

**Redes ocultas:** un punto de acceso puede desactivar el broadcast de su SSID (nombre de red) en las tramas beacon. Esto **no es seguridad real** — cualquier dispositivo que ya esté conectado sigue enviando el SSID en texto plano al intentar reconectarse, capturable con herramientas pasivas. Es "seguridad por oscuridad", el mismo antipatrón que ya identificaste conceptualmente en el curso de seguridad del curso anterior.

**Roaming:** cuando te movés físicamente entre puntos de acceso de la misma red (mismo SSID, múltiples APs — típico en oficinas grandes), el dispositivo decide cuándo "saltar" de un AP a otro según la intensidad de señal. `wpa_supplicant`/`iwd` implementan la lógica de decisión; el estándar 802.11k/v/r existe específicamente para hacer ese salto más rápido y sin cortes.

```bash
nmcli connection show "conexion-wifi-ejemplo" 2>/dev/null | grep -i "hidden\|802-11-wireless"
```

---

## 4. HERRAMIENTA: conectar a una red Wi-Fi con `nmcli` (si tenés adaptador)

```bash
nmcli device wifi list
nmcli device wifi connect "NOMBRE_DE_RED" password "CONTRASEÑA"
```

Esto crea automáticamente un perfil nuevo en NetworkManager (recordá el Módulo 17: quedaría guardado como conexión, a diferencia del perfil Ethernet autogenerado que vimos en memoria).

---

## 5. CONCEPTO: qué SÍ y qué NO podés aprender de Wi-Fi en esta VM (cierre parcial de fase)

| Se puede aprender en la VM | Requiere adaptador Wi-Fi real (con o sin passthrough) |
|---|---|
| Arquitectura completa: kernel → wpa_supplicant/iwd → NetworkManager → apps | Conexión real a una red |
| Diferencia conceptual WPA2 vs. WPA3 (SAE, forward secrecy) | Confirmar qué protocolo negocia tu adaptador específico |
| Por qué las redes ocultas no son seguridad real | Comportamiento real de roaming entre APs físicos |
| Sintaxis de `nmcli device wifi` | Rendimiento/estabilidad reales de una señal inalámbrica |

---

## 6. PRÁCTICA

1. Corré `nmcli device status` y `rfkill list`, confirmando la ausencia (o presencia) de hardware Wi-Fi en tu VM.
2. Explicá con tus propias palabras por qué SAE hace que WPA3 sea más resistente a ataques offline que el 4-way handshake de WPA2.
3. Explicá por qué una red Wi-Fi "oculta" no es una medida de seguridad real.
4. Si tenés adaptador disponible: conectate a una red real con `nmcli device wifi connect` y confirmá que el perfil se guardó en `/etc/NetworkManager/system-connections/` (a diferencia del perfil Ethernet en memoria del Módulo 17).
5. Completá la tabla de la sección 5 con tu resultado concreto.

---

## 7. ERROR INTENCIONAL / DIAGNÓSTICO

```bash
nmcli device wifi connect "red-que-no-existe" password "loquesea"
```

**Diagnóstico esperado según tu hardware:**
1. Sin adaptador Wi-Fi: un error indicando que no hay ningún dispositivo Wi-Fi disponible (`Error: No Wi-Fi device found`).
2. Con adaptador Wi-Fi: un error indicando que la red no fue encontrada en el escaneo (`Error: No network with SSID 'red-que-no-existe' found`).

Documentá cuál de los dos te tocó — ambos son formas válidas de la misma validación estricta que el curso viene repitiendo desde `pw-link` (Módulo 15).

---

## Checklist de cierre del módulo

- [ ] Entiendo que Wi-Fi usa la misma arquitectura de orquestación del Módulo 17, con `wpa_supplicant`/`iwd` como motor de autenticación.
- [ ] Entiendo la diferencia real entre WPA2 y WPA3 (SAE, forward secrecy), no solo que "uno es más nuevo".
- [ ] Entiendo por qué las redes ocultas no son una medida de seguridad real.
- [ ] Entiendo el concepto de roaming entre puntos de acceso.
- [ ] Documenté honestamente el estado de hardware Wi-Fi disponible en mi VM.
- [ ] Provoqué y diagnostiqué el error de `nmcli device wifi connect` según mi caso concreto.

---

## Evidencias

_(pendiente — se agregan capturas reales a medida que se completa el módulo)_

---

**Próximo módulo:** 19 — Bluetooth (opcional).
