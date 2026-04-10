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

## Contexto del proyecto
Este es BreachNotify. Lee siempre `SPEC.md` al inicio de cada sesión.
Archivo principal: `breach-notify.html` en la raíz.
Stack: HTML/CSS/JS puro. Sin frameworks. Sin backend. Sin dependencias npm.
