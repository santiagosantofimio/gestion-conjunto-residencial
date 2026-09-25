# Proyecto Final — Bases de Datos I

Base de datos para la gestión de cartera, reservas de zonas comunes y control de visitantes en un conjunto residencial sometido al régimen de propiedad horizontal.

**Universidad El Bosque** · Ingeniería de Sistemas · Bases de Datos I

**Integrantes:** Santiago Santofimio, Jairo Esteban, Santiago Silva, Andres Hernandez, Jorge Pinilla

**SGBD:** MySQL 8

## Estructura

```
docs/        propuesta, requerimientos, normalización, diccionario de datos
diagramas/   modelo entidad-relación y modelo relacional
sql/         scripts de creación, datos de prueba y consultas
informe/     informe final y capturas de resultados
```

## Ejecución

Los scripts se ejecutan en orden sobre un servidor MySQL sin la base de datos creada:

```bash
mysql -u root -p < sql/01_schema.sql
mysql -u root -p < sql/02_datos_prueba.sql
mysql -u root -p < sql/03_consultas.sql
```

## Plan de trabajo

- [x] Fase 1 — Propuesta: tema, problema, objetivos, alcance y entidades candidatas
- [ ] Fase 2 — Análisis de requerimientos: usuarios, procesos y reglas de negocio
- [ ] Fase 3 — Diseño conceptual: diagrama entidad-relación
- [ ] Fase 4 — Normalización: planilla original, 1FN, 2FN y 3FN
- [ ] Fase 5 — Diccionario de datos y modelo relacional
- [ ] Fase 6 — Implementación del esquema en MySQL
- [ ] Fase 7 — Datos de prueba
- [ ] Fase 8 — Consultas SQL y capturas de resultados
- [ ] Fase 9 — Informe escrito en formato APA 7
- [ ] Fase 10 — Preparación de la sustentación
