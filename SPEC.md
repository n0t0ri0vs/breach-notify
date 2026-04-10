# BreachNotify — SPEC.md
## Documento de estado del proyecto y roadmap activo

> **Instrucción para Claude Code:** Lee este archivo al inicio de cada sesión antes de tocar ningún archivo. Refleja el estado actual del proyecto, las decisiones de arquitectura tomadas y el trabajo pendiente. El historial de cambios pasados está en `docs/`.

---

## Qué es este proyecto

BreachNotify es una herramienta pública de notificación de incidentes de ciberseguridad. SPA en HTML/CSS/JS puro, sin backend, sin servidor. Todos los datos son volátiles en cliente (privacidad by design — nada sale del navegador del usuario).

**Archivo principal:** `breach-notify.html` en la raíz del repo.  
**Demo pública:** https://n0t0ri0vs.github.io/breach-notify/breach-notify.html  
**Repo:** https://github.com/n0t0ri0vs/breach-notify  
**Versión actual:** v1.4

---

## Historial de versiones

| Versión | Cambios principales |
|---|---|
| v1.0 | Cuestionario base, motor de decisión, timelines por normativa |
| v1.1 | Hardening de seguridad: `escapeHtml()`, `escapeAttr()`, `Object.freeze()` en constantes, SHA-256 fingerprint de inputs |
| v1.2 | Motor NLP de palabras clave + motor de scoring de riesgo por normativa |
| v1.3 | Guía Nacional de Notificación (RD-Ley 12/2018, CCN/INCIBE 2020): peligrosidad, canales exactos, plazos, coexistencia NIS2 |
| v1.4 | Dashboard de impacto at-a-glance (5 tarjetas), exposición legal con jurisprudencia AEPD, renderizado de campo `nota` en obligaciones |

---

## Decisiones de arquitectura — NO cambiar sin consenso

- Stack: HTML/CSS/JS puro. Sin frameworks, sin React, sin Vite, sin dependencias de npm.
- Sin backend. Sin llamadas a APIs externas por defecto.
- Futura opción de IA (Opción C): aplazada. Cuando se implemente será opt-in con aviso explícito al usuario.
- Exportación PDF: pendiente (Capa 2). Usar jsPDF + html2canvas cuando se implemente.
- Todo en un único archivo HTML. No separar CSS o JS en archivos distintos salvo decisión explícita.
- `computeResults()` es `async` (necesario para Web Crypto API).
- Toda interpolación de datos de usuario en innerHTML pasa obligatoriamente por `escapeHtml()` o `escapeAttr()`.

---

## Normativas cubiertas

| Normativa | Organismo | Plazo crítico | Estado |
|---|---|---|---|
| RGPD / LOPD-GDD | AEPD | 72h | ✅ Implementado |
| NIS2 | INCIBE-CERT / CCN-CERT | 24h / 72h / 1 mes | ✅ Implementado |
| DORA | BdE / CNMV / DGSFP | 4h / 72h / 1 mes | ✅ Implementado |
| ENS | CCN-CERT vía LUCIA | Según categoría | ✅ Implementado (capa informativa) |
| ISO 27001 | Interno | Según SGSI | ✅ Implementado (capa informativa) |
| Guía Nacional Notificación (CCN/INCIBE 2020) | INCIBE-CERT / CCN-CERT | Inmediata / 24-72h / 20-40 días | ✅ Implementado en v1.3 |
| PCI-DSS | — | — | ❌ Excluido (futuro) |
| CER | — | — | ❌ Excluido (futuro) |

---

## Estado actual — implementado y funcional (v1.4)

### Cuestionario (9 preguntas)

1. Fecha y hora del incidente
2. Tipo de incidente (taxonomía simplificada con nivel de peligrosidad interno)
3. Tipos de datos afectados (clasificación automática de dato personal/sensible)
4. Sectores NIS2 (Anexo I / Anexo II)
5. Tamaño de la empresa (umbral NIS2)
6. Tipo de entidad financiera (DORA)
7. Certificaciones (ISO 27001 / ENS)
8. Número de afectados
9. Descripción libre del incidente

### Motor de decisión

- Clasificación automática de datos personales y sensibles
- Determinación de aplicabilidad NIS2 (esencial vs importante) y DORA
- Nivel de peligrosidad según tipo de incidente (Guía Nacional: CRÍTICO / MUY_ALTO / ALTO / MEDIO / BAJO / INDETERMINADO)
- Obligatoriedad vs voluntariedad de notificación al CSIRT
- Plazos NIS2 con prioridad sobre Guía Nacional cuando coexisten; coexistencia señalada con campo `nota` visible en la tarjeta
- Selección de canal CSIRT: INCIBE-CERT (general u operadores críticos) vs CCN-CERT/LUCIA según sector ENS
- Timeline con countdown desde fecha de detección; badge pulsante `⚠ PLAZO VENCIDO` si el plazo ha expirado
- Checklists de información obligatoria por fase
- Borrador de comunicación a afectados pre-rellenado (cuando aplica)
- Caso sin obligación: mensaje informativo + opción voluntaria INCIBE-CERT

### Motor NLP (campo descripción libre)

- `analyzeIncidentText(text)`: normalización NFD + eliminación de acentos + lowercase para matching robusto
- Detecta tipos de datos mencionados en la descripción no seleccionados en el formulario → aviso de inconsistencia
- Detecta vectores de ataque (phishing, ransomware, insider, etc.) con etiquetas legibles
- Panel de feedback en tiempo real (debounce 380ms) durante la edición
- Renderizado en pantalla de resultados: sección "Análisis del vector" si hay señales

### Motor de scoring de riesgo

- `computeRiskScore({ selectedDataObjects, selectedDataTypes, affected, incidentDt, sectorSel, size, doraEntity })`
- Tres ejes ponderados: sensibilidad del dato (0-45) + volumen de afectados (0-30) + presión temporal (0-25) = 0-100
- Produce puntuaciones independientes para RGPD/AEPD, NIS2 y DORA
- Etiquetas: CRÍTICO / ALTO / MEDIO / BAJO con colores y barra visual
- Factores descriptivos por puntuación (chips de texto)

### Hardening de seguridad

- `escapeHtml(str)`: sanitiza toda interpolación en innerHTML (entidades HTML)
- `escapeAttr(str)`: sanitiza valores en atributos HTML (comillas, etc.)
- `Object.freeze()` en todas las constantes críticas: `DATA_TYPES`, `SECTORES_NIS2_*`, `ENTIDADES_DORA`, `INCIDENT_TYPES`, `CANALES`, `NLP_*`
- SHA-256 fingerprint de inputs (Web Crypto API) con timestamp en el momento de generación; mostrado en el resumen de entidad para trazabilidad y anti-manipulación

### Dashboard de impacto at-a-glance (v1.4)

Cinco tarjetas visuales en la cabecera de resultados, antes del resumen de entidad:

| Tarjeta | Variante | Cálculo |
|---|---|---|
| Obligaciones activas | danger | `obligations.length` |
| Plazos vencidos | danger / neutral | Cuenta `step.remaining.vencido === true` en todos los pasos |
| Multa máxima | danger | RGPD art. 83.4 → 10 M€; NIS2 Esencial art. 34.4 → 10 M€; NIS2 Importante → 7 M€; DORA art. 50 → 1%/día hasta 6m. Se muestra el máximo aplicable con la fuente legal |
| Envíos externos | info | Pasos `urgent` + `warning` de obligaciones externas |
| Trabajo estimado | neutral | 1.5h/paso urgente + 1h/warning + 0.5h/info-normal + 1h por ISO/ENS |

---

## Pendiente — Capa 2 (próxima sesión de trabajo)

### Exportación PDF por obligación

Cada tarjeta de obligación de notificación debe poder exportarse como PDF independiente con los datos pre-rellenados del usuario. El PDF debe contener:

- Cabecera con nombre de la normativa y organismo destino
- Datos del incidente introducidos por el usuario
- Checklist de información obligatoria con los campos rellenados donde sea posible
- Borrador de texto de notificación listo para enviar o adaptar
- Fecha de generación del documento

**Librería a usar:** jsPDF + html2canvas (cargar desde CDN, no instalar).  
**Formato:** un botón "Descargar PDF" por cada tarjeta de obligación en la pantalla de resultados.  
**Importante:** el PDF se genera localmente en el navegador, nunca se envía a ningún servidor.

---

## Pendiente — Capa 3 (futuro)

- IA externa opt-in con aviso explícito al usuario (Opción C — aplazada)
- Soporte PCI-DSS y Directiva CER
- Internacionalización (EN / FR)

---

## Convenciones de commits

```
feat: nueva funcionalidad
fix: corrección de bug
style: cambios visuales sin impacto en lógica
refactor: reestructuración de código sin cambio de comportamiento
docs: cambios en documentación
```

---

## Cómo actualizar este archivo

Al terminar cada sesión de trabajo con Claude Code, actualizar:
- La tabla de versiones con el nuevo número y los cambios principales
- La tabla de normativas si se añade alguna
- La sección "Estado actual" para reflejar lo completado
- La sección "Pendiente" eliminando lo ya implementado y añadiendo lo nuevo
