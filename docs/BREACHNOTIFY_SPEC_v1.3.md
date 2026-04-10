# BreachNotify — Spec de cambios v1.1
## Basado en: Guía Nacional de Notificación y Gestión de Ciberincidentes (CCN/INCIBE/CNPIC, 2020)

> **Instrucción para Claude Code:** Lee este documento completo antes de tocar ningún archivo. Contiene contexto normativo crítico que afecta a decisiones de implementación. El archivo a modificar es `breach-notify.html`.

---

## CONTEXTO NORMATIVO — LEE ESTO PRIMERO

### Relación entre la Guía Nacional y NIS2 (no son lo mismo)

La Guía Nacional de Notificación (2020) y NIS2 son dos marcos distintos que coexisten y usan el mismo canal de notificación pero con criterios diferentes:

| | Guía Nacional (2020) | NIS2 (transposición española) |
|---|---|---|
| Base legal | RD-Ley 12/2018 (NIS1) | Directiva UE 2022/2555 |
| Ámbito | Todo operador obligado (sector público, infraestructuras críticas, OSE/PSD) | Entidades esenciales e importantes según Anexo I/II |
| Canal | INCIBE-CERT o CCN-CERT | INCIBE-CERT o CCN-CERT (mismo canal) |
| Criterio de notificación | Nivel de **peligrosidad/impacto** del incidente (CRÍTICO/MUY ALTO/ALTO) | Incidente "significativo" según definición NIS2 |
| Plazos | Basados en nivel de peligrosidad (ver tabla abajo) | Fases fijas: 24h / 72h / 1 mes |

**Regla de precedencia para el motor de decisión:**

```
Si entidad bajo NIS2 → usar plazos NIS2 (más estrictos, tienen precedencia)
Si entidad bajo ENS (sector público) → usar plazos Guía Nacional vía CCN-CERT/LUCIA
Si entidad obligada por RD-Ley 12/2018 pero fuera de NIS2 → usar plazos Guía Nacional
Si ninguno de los anteriores → notificación voluntaria (informar pero no obligar)
Si entidad bajo NIS2 Y bajo Guía Nacional simultáneamente → mostrar NIS2 como prioritario, Guía Nacional como complementario
```

**Los plazos de la Guía Nacional NO son una alternativa a NIS2. Son el marco que aplica cuando NIS2 no aplica.**

### Plazos Guía Nacional (Tabla 7 de la guía)

| Nivel peligrosidad/impacto | Notificación inicial | Notificación intermedia | Notificación final |
|---|---|---|---|
| CRÍTICO | Inmediata (sin demora) | 24/48 horas | 20 días |
| MUY ALTO | Inmediata (sin demora) | 72 horas | 40 días |
| ALTO | Inmediata (sin demora) | — | — |
| MEDIO | No obligatoria | — | — |
| BAJO | No obligatoria | — | — |

"Inmediata" = en cuanto se tiene conocimiento, sin plazo numérico explícito. En la práctica: lo antes posible en las primeras horas.

### Niveles de peligrosidad por tipo de incidente (Tabla 4 de la guía)

| Peligrosidad | Tipos de incidente |
|---|---|
| CRÍTICO | APT (amenaza persistente avanzada) |
| MUY ALTO | Distribución/configuración de malware, robo físico, sabotaje, interrupciones |
| ALTO | Sistema infectado, C&C, compromiso de aplicaciones/cuentas con privilegios, DoS/DDoS, acceso/modificación/pérdida no autorizada de datos, phishing |
| MEDIO | Ingeniería social, explotación de vulnerabilidades conocidas, compromiso de cuentas sin privilegios, suplantación, sistemas vulnerables |
| BAJO | Spam, scanning, sniffing |

---

## CAMBIOS A IMPLEMENTAR EN breach-notify.html

### CAMBIO 1 — Nueva pregunta: Tipo de incidente

**Posición:** Insertar como pregunta 3 del cuestionario, entre "datos afectados" (Q2) y "sector NIS2" (Q3 actual, que pasa a ser Q4).

**Propósito:** Determinar el nivel de peligrosidad del incidente para saber si la notificación a INCIBE-CERT/CCN-CERT es obligatoria o voluntaria bajo la Guía Nacional.

```javascript
{
  id: 'incident_type',
  type: 'radio',
  title: '¿Qué tipo de incidente ha ocurrido?',
  subtitle: 'Si no estás seguro, selecciona el que mejor se aproxime o el más grave.',
  options: [
    {
      id: 'apt_ransomware',
      label: 'Ataque dirigido o ransomware',
      desc: 'APT, ransomware, sabotaje deliberado y organizado',
      peligrosidad: 'CRITICO'
    },
    {
      id: 'malware_infeccion',
      label: 'Infección por malware o sistema comprometido',
      desc: 'Virus, troyano, gusano, rootkit, servidor de mando y control',
      peligrosidad: 'ALTO'
    },
    {
      id: 'acceso_datos',
      label: 'Acceso no autorizado o robo de información',
      desc: 'Exfiltración de datos, acceso indebido a información, pérdida de datos',
      peligrosidad: 'ALTO'
    },
    {
      id: 'dos_ddos',
      label: 'Denegación de servicio (DoS / DDoS)',
      desc: 'Interrupción o degradación del servicio por ataque',
      peligrosidad: 'ALTO'
    },
    {
      id: 'phishing_fraude',
      label: 'Phishing, fraude o suplantación de identidad',
      desc: 'Engaño a usuarios, robo de credenciales mediante ingeniería social',
      peligrosidad: 'ALTO'
    },
    {
      id: 'vulnerabilidad',
      label: 'Explotación de vulnerabilidad',
      desc: 'Aprovechamiento de un fallo conocido o desconocido en sistemas o aplicaciones',
      peligrosidad: 'MEDIO'
    },
    {
      id: 'otro',
      label: 'Otro tipo de incidente / No estoy seguro',
      desc: 'Incidente no clasificable claramente en las categorías anteriores',
      peligrosidad: 'INDETERMINADO'
    }
  ]
}
```

El atributo `peligrosidad` es interno al motor de decisión, no se muestra al usuario.

---

### CAMBIO 2 — Motor de decisión: variable peligrosidad y obligatoriedad

Añadir al motor la evaluación del nivel de peligrosidad para determinar si la notificación a CSIRT es obligatoria o voluntaria cuando NIS2 no aplica:

```javascript
// Niveles con notificación OBLIGATORIA bajo Guía Nacional
const PELIGROSIDAD_OBLIGATORIA = ['CRITICO', 'MUY_ALTO', 'ALTO'];

// Obtener peligrosidad desde la respuesta de tipo de incidente
const incidentTypeObj = questions[state.currentQ]... // buscar en el array de opciones
const peligrosidad = INCIDENT_TYPES.find(t => t.id === a.incident_type)?.peligrosidad || 'INDETERMINADO';

const notificacionCSIRT_obligatoria = bajoNIS2 || (PELIGROSIDAD_OBLIGATORIA.includes(peligrosidad));
const notificacionCSIRT_voluntaria = !notificacionCSIRT_obligatoria;
```

Usar `notificacionCSIRT_obligatoria` y `peligrosidad` en la generación de obligaciones para:
- Determinar si mostrar la obligación INCIBE/CCN como obligatoria o como recomendada
- Seleccionar qué rama de plazos mostrar (NIS2 o Guía Nacional)

---

### CAMBIO 3 — Plazos: nueva rama para Guía Nacional

Cuando la entidad NO está bajo NIS2 pero sí bajo la Guía Nacional (peligrosidad CRÍTICO/MUY ALTO/ALTO), mostrar el timeline con los plazos de la Tabla 7:

```javascript
// Rama Guía Nacional — usar cuando bajoNIS2 === false y notificacionCSIRT_obligatoria === true
{
  id: 'csirt_guia_nacional',
  icon: '🛡️',
  title: `Notificación a ${csirt} — Guía Nacional`,
  dest: `${csirt} · RD-Ley 12/2018 · Guía Nacional de Notificación de Ciberincidentes`,
  timeline: [
    {
      cls: 'urgent',
      deadline: 'INMEDIATA — en cuanto tengas conocimiento del incidente',
      title: 'Notificación inicial',
      desc: `Notifica a ${csirt} sin demora.`,
      checklist: [
        'Descripción general del incidente',
        'Fecha y hora del incidente (en UTC)',
        'Fecha y hora de detección (en UTC) — campo separado',
        'Clasificación/tipo de incidente',
        `Nivel de peligrosidad estimado: ${peligrosidad}`,
        'Recursos tecnológicos afectados (número, tipo, IPs, SO, aplicaciones)',
        'Origen del incidente si se conoce (fichero, USB, web maliciosa, etc.)',
        '¿Tiene impacto transfronterizo en otros países UE? (sí/no)',
        'Regulación afectada: [autogenerado por la herramienta]',
        'Plan de acción y contramedidas adoptadas hasta el momento',
        '¿Requiere intervención de Fuerzas y Cuerpos de Seguridad del Estado? (sí/no)',
      ]
    },
    // Fase intermedia solo si peligrosidad es CRÍTICO o MUY_ALTO
    ...(peligrosidad === 'CRITICO' ? [{
      cls: 'warning',
      deadline: peligrosidad === 'CRITICO' ? '24/48 horas desde notificación inicial' : '72 horas desde notificación inicial',
      title: 'Notificación intermedia',
      desc: 'Actualización del estado del incidente con información adicional disponible.',
      checklist: [
        'Actualización del estado de contención',
        'Nuevos datos sobre el alcance',
        'Medidas adicionales adoptadas',
      ]
    }] : []),
    // Informe final solo si peligrosidad es CRÍTICO o MUY_ALTO
    ...(peligrosidad === 'CRITICO' || peligrosidad === 'MUY_ALTO' ? [{
      cls: 'normal',
      deadline: peligrosidad === 'CRITICO' ? '20 días desde notificación inicial' : '40 días desde notificación inicial',
      title: 'Informe final',
      desc: 'Cierre del incidente con análisis completo.',
      checklist: [
        'Causa raíz del incidente',
        'Descripción completa de lo ocurrido',
        'Medidas definitivas adoptadas',
        'Lecciones aprendidas',
      ]
    }] : [])
  ]
}
```

Cuando NIS2 y Guía Nacional coinciden en la misma entidad, mostrar NIS2 primero con una nota: *"Los plazos NIS2 son más estrictos y tienen precedencia. Usa el mismo canal (INCIBE-CERT) para cumplir ambos marcos."*

---

### CAMBIO 4 — Canales de notificación exactos en los resultados

Sustituir los textos genéricos actuales ("notifica a INCIBE-CERT") por los canales concretos según el tipo de entidad:

```javascript
const CANALES = {
  INCIBE_CERT: {
    nombre: 'INCIBE-CERT',
    aplica: 'Entidades privadas',
    email_general: 'incidencias@incibe-cert.es',
    email_operadores_criticos: 'pic@incibe-cert.es',
    portal: 'https://www.incibe.es/incibe-cert/respuesta-incidentes',
    nota: 'Enviar preferiblemente con cifrado PGP. El email es complementario al portal web.'
  },
  CCN_CERT: {
    nombre: 'CCN-CERT',
    aplica: 'Sector público, AA.PP., entidades Ley 40/2015',
    herramienta_principal: 'LUCIA',
    url_lucia: 'https://www.ccn-cert.cni.es/gestion-de-incidentes/lucia.html',
    email_alternativo: 'incidentes@ccn-cert.cni.es',
    nota: 'Canal preferente: LUCIA. El email solo como alternativa si LUCIA no está disponible.'
  }
};
```

**Lógica de selección de canal:**
- Sector privado → INCIBE-CERT (`incidencias@incibe-cert.es`)
- Sector privado + infraestructura crítica → INCIBE-CERT (`pic@incibe-cert.es`)
- Sector público / ENS / AA.PP. → CCN-CERT vía LUCIA
- Ambos (entidad pública con actividad mixta) → mostrar ambos

Mostrar siempre la URL y el email en el resultado, no solo el nombre del organismo.

---

### CAMBIO 5 — Checklist de notificación inicial al CSIRT: campos adicionales

El checklist de la fase de notificación inicial al CSIRT debe añadir los campos de la Tabla 6 de la Guía Nacional que actualmente faltan:

**Añadir a todos los checklists de notificación inicial INCIBE/CCN:**

- Separar "fecha/hora del incidente" de "fecha/hora de detección" — son dos campos distintos según la guía. La herramienta solo tiene uno actualmente.
- Recursos tecnológicos afectados: número y tipo de activos, IPs, sistemas operativos, aplicaciones y versiones
- Origen conocido del incidente (apertura de fichero sospechoso, USB, acceso a web maliciosa, credenciales comprometidas, etc.)
- Impacto transfronterizo: ¿afecta a sistemas o usuarios en otros países UE? (sí/no)
- Regulación afectada (campo autogenerado: RGPD / NIS2 / DORA / ENS según lo que haya determinado el motor)
- ¿Requiere actuación de Fuerzas y Cuerpos de Seguridad del Estado? (sí/no) — relevante si hay indicios de delito

---

### CAMBIO 6 — Borrador de notificación: añadir campo "Regulación afectada"

En el borrador de comunicación al CSIRT que genera la herramienta, añadir el campo "Regulación afectada" autogenerado con las normativas que el motor haya determinado como aplicables:

```
Regulación afectada: [RGPD] [NIS2] [DORA] [ENS] [RD-Ley 12/2018]
```

Se rellena automáticamente — no requiere input adicional del usuario.

---

### CAMBIO 7 — Caso sin obligación: no cortar el flujo

Cuando el motor determine que la notificación es voluntaria (peligrosidad MEDIO/BAJO y fuera de NIS2/DORA), el mensaje actual debe cambiarse. No eliminar la sección, sino informar:

**Mensaje actual (a eliminar):** texto que sugiere que no hay nada que hacer.

**Mensaje nuevo:**

```
Sin obligación legal de notificación externa en este momento.

Sin embargo, puedes notificar voluntariamente a INCIBE-CERT:
- Te da acceso a su capacidad de respuesta técnica sin coste
- Contribuye a la inteligencia colectiva de ciberseguridad nacional
- Si la situación evoluciona y el impacto aumenta, ya tienes el canal abierto

Canal: incidencias@incibe-cert.es / incibe.es/incibe-cert/respuesta-incidentes

En cualquier caso, documenta internamente el incidente. Si hay datos personales afectados, revisa si aplica notificación a la AEPD (72h).
```

---

## RESUMEN DE CAMBIOS — CHECKLIST DE IMPLEMENTACIÓN

```
[ ] 1. Nueva pregunta "tipo de incidente" en posición 3 del cuestionario
        → Radio, 7 opciones, atributo interno peligrosidad
        → Renumerar preguntas actuales 3-8 → 4-9 (el total pasa de 8 a 9 preguntas)

[ ] 2. Motor de decisión: añadir variable peligrosidad
        → Leer desde respuesta incident_type
        → Calcular notificacionCSIRT_obligatoria (CRÍTICO/MUY_ALTO/ALTO) vs voluntaria (MEDIO/BAJO)

[ ] 3. Timeline: nueva rama para Guía Nacional
        → Activar cuando peligrosidad obligatoria Y bajoNIS2 === false
        → Plazos: inmediata / 24-72h / 20-40 días según nivel
        → Cuando coexiste con NIS2: NIS2 primero, nota de coexistencia

[ ] 4. Canales exactos en resultados
        → Sector privado: incidencias@incibe-cert.es + URL portal
        → Operadores críticos privados: pic@incibe-cert.es
        → Sector público/ENS: LUCIA (URL) + email alternativo CCN-CERT

[ ] 5. Checklist notificación inicial CSIRT: añadir 6 campos
        → Separar fecha incidente / fecha detección
        → Recursos tecnológicos afectados
        → Origen del incidente
        → Impacto transfronterizo
        → Regulación afectada (autogenerado)
        → ¿Requiere FFCCSE? (sí/no)

[ ] 6. Borrador notificación CSIRT: añadir campo "Regulación afectada"
        → Autogenerado con normativas aplicables según motor

[ ] 7. Mensaje caso sin obligación: reemplazar por texto informativo
        → Incluir opción voluntaria + canal + recomendación de documentar
```

---

## NOTAS TÉCNICAS PARA LA IMPLEMENTACIÓN

- El total de preguntas pasa de 8 a 9. Actualizar `state.totalQ` y la lógica de `progressLabel`.
- La nueva pregunta es de tipo `radio` (selección única), igual que `company_size` y `dora_entity`.
- El atributo `peligrosidad` de cada opción de `incident_type` debe ser accesible desde el motor de decisión. Guardarlo en `DATA_INCIDENT_TYPES` como array paralelo a las opciones, o incluirlo directamente en el objeto de opciones y leerlo al calcular resultados.
- Los canales exactos (emails, URLs) deben aparecer como enlaces clicables en el HTML resultante.
- La coexistencia NIS2 + Guía Nacional en el mismo resultado debe presentarse visualmente de forma que no genere confusión — sugerencia: mostrar NIS2 primero con badge "PRIORITARIO" y Guía Nacional con badge "COMPLEMENTARIO".
