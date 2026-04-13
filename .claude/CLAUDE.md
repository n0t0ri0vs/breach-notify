# Instrucciones permanentes para Claude Code

## Regla obligatoria — siempre activa
Después de cada commit, actualiza `SPEC.md` si el trabajo realizado:
- Completa algo que estaba en la sección "Pendiente"
- Añade una nueva normativa o funcionalidad
- Cambia una decisión de arquitectura
- Introduce una nueva convención

La actualización del SPEC.md debe ir en el mismo commit o en uno inmediatamente 
posterior con mensaje `docs: actualizar SPEC.md`.

## Versionado semántico — reglas obligatorias

El footer y SPEC.md deben reflejar siempre la versión actual en formato `X.Y.Z`.
Incrementar automáticamente al hacer commit, sin esperar instrucción del usuario.

| Tipo | Cuándo usarlo | Ejemplo |
|---|---|---|
| **MAJOR** `X.0.0` | Rediseño completo de UI, cambio de stack/arquitectura, ruptura de flujo existente | Separar en múltiples archivos, añadir backend |
| **MINOR** `X.Y.0` | Nueva normativa cubierta, nueva funcionalidad visible al usuario, nuevo motor o componente UI significativo | Nueva sección de resultados, soporte PCI-DSS |
| **PATCH** `X.Y.Z` | Corrección de URLs/datos normativos, fix de bug visual, ajuste de estilos, actualización de contactos/canales | Corregir email de CSIRT, fix de color |

Actualizar en dos sitios siempre:
1. Footer en `breach-notify.html`: `BreachNotify vX.Y.Z`
2. `SPEC.md`: campo "Versión actual" + fila en la tabla de historial

## Seguridad — reglas obligatorias en todo cambio de código

Estas reglas se aplican siempre, sin que el usuario las solicite, en cualquier modificación de `breach-notify.html`.

### Prevención de XSS
- Toda interpolación de datos de usuario en `innerHTML` debe pasar por `escapeHtml()`.
- Toda interpolación en atributos HTML debe pasar por `escapeAttr()`.
- Los strings hardcodeados internos que se inserten en `innerHTML` deben también pasar por `escapeHtml()` como medida defensiva ante futuros cambios.
- Nunca usar `innerHTML` con concatenación directa de variables, independientemente de su origen.

### Content Security Policy
- El `<head>` debe contener siempre una meta tag CSP con política restrictiva: `default-src 'self'`, `object-src 'none'`, `base-uri 'self'`. No relajar esta política sin justificación explícita.

### Persistencia de datos
- Ningún dato introducido por el usuario puede escribirse en `localStorage`, `sessionStorage` ni cookies. La herramienta es volátil por diseño (privacidad by design). Si en algún momento se necesita persistencia, consultar antes de implementar.

### Dependencias externas
- Cualquier recurso cargado desde CDN externo debe incluir el atributo `integrity` con su hash SRI y `crossorigin="anonymous"`.
- Al añadir una nueva dependencia externa, generar o consultar el hash SRI antes del commit.

### Exposición de información
- No dejar `console.log` activos con datos de usuario o lógica interna en código de producción.
- No incluir comentarios que revelen lógica de negocio sensible o vectores de ataque.

### Constantes críticas
- Las constantes de configuración normativa (`DATA_TYPES`, `SECTORES_NIS2_*`, `ENTIDADES_DORA`, `INCIDENT_TYPES`, `CANALES`, `NLP_*`) deben mantenerse congeladas con `Object.freeze()`.

## Auditoría periódica de seguridad y funcionalidad — cada 10 commits

Cada vez que el número de commits en `main` sea múltiplo de 10, ejecutar automáticamente la siguiente auditoría antes del siguiente commit de funcionalidad:

### Seguridad (análisis estático)
1. Verificar que no existe ninguna interpolación en `innerHTML` sin `escapeHtml()` o `escapeAttr()`.
2. Verificar que la meta tag CSP sigue presente y no ha sido relajada.
3. Verificar que no hay escrituras en `localStorage`, `sessionStorage` ni cookies.
4. Verificar que todos los recursos externos tienen atributo `integrity` con hash SRI válido.
5. Verificar que no hay `console.log` activos con datos de usuario.

### Funcionalidad (casos de prueba del motor de decisión)
Ejecutar mentalmente (análisis estático del código) los siguientes casos y verificar que el motor produce el resultado esperado:

| Caso | Entrada | Resultado esperado |
|---|---|---|
| Financiero + datos bancarios + >500 afectados | sector financiero DORA, datos IBAN/tarjeta, 501 afectados | RGPD + DORA + NIS2 activos |
| Sanitario + datos de salud + <50 afectados | sector sanitario, datos salud art.9, 30 afectados | RGPD activo, NIS2 evaluar umbral |
| Genérico + solo contacto + <10 afectados | sector no NIS2, solo email/teléfono, 5 afectados | Riesgo bajo, sin obligación comunicar afectados |
| Incidente crítico + fecha pasada >72h | peligrosidad CRÍTICO, incidentDt hace 80h | Badge "PLAZO VENCIDO" en timeline RGPD |

Si se detecta cualquier desviación, corregirla antes de continuar con el trabajo de la sesión. Registrar el resultado de la auditoría en el commit con mensaje `audit: revisión periódica seguridad y funcionalidad vX.Y.Z`.

## Git — rama de trabajo
Trabaja siempre directamente sobre `main`. No crees ramas auxiliares ni worktrees salvo que el usuario lo pida explícitamente. Todos los commits van a `main`.

## Contexto del proyecto
Este es BreachNotify. Lee siempre `.claude/SPEC.md` al inicio de cada sesión.
Archivo principal: `breach-notify.html` en la raíz.
Stack: HTML/CSS/JS puro. Sin frameworks. Sin backend. Sin dependencias npm.
