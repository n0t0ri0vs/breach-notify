# BreachNotify

**Herramienta pública de apoyo a la notificación de incidentes de ciberseguridad.**  
Sin servidor. Sin base de datos. Todo se procesa localmente en tu navegador.

🔗 **[Demo en vivo](https://n0t0ri0vs.github.io/breach-notify/breach-notify.html)**

---

## Atribución

BreachNotify was created by [Sergio Henestrosa](https://github.com/n0t0ri0vs). 
If you build on this project or use it as inspiration, a mention of the original is appreciated.

## Qué hace

En menos de 5 minutos determina qué obligaciones de notificación aplican a tu incidente y genera un plan de acción con plazos, checklists y borradores de comunicación.

Normativas cubiertas:

- **RGPD / LOPD-GDD** — Notificación a AEPD (72h) y comunicación a afectados
- **NIS2** — Notificación a INCIBE-CERT / CCN-CERT (24h / 72h / 1 mes)
- **DORA** — Notificación a BdE, CNMV o DGSFP (4h / 72h / 1 mes)
- **ENS / ISO 27001** — Capa informativa de contexto

## Privacidad by design

Ningún dato introducido sale del navegador. No hay llamadas a servidor, no hay logs, no hay cookies. Los datos desaparecen al cerrar la pestaña.

## Estado del proyecto

| Capa | Descripción | Estado |
|------|-------------|--------|
| 1 | Cuestionario + motor de decisión + timeline + checklists + borrador comunicación | ✅ Completada |
| 2 | Exportación PDF por obligación con plantillas pre-rellenadas | 🔄 En desarrollo |
| 3 | Clasificador semántico local + IA opt-in | 📋 Planificada |

## Stack

HTML / CSS / JavaScript puro — sin frameworks, sin dependencias, sin build step.

## Autor

Sergio Henestrosa · [LinkedIn](https://www.linkedin.com/in/sergio-henestrosa-garc%C3%ADa-696531167/) · GRC & Ciberseguridad
