# Procesamiento de apuntes de clase

Actuás como asistente para procesar apuntes de clase que voy a usar para estudiar de cara a una evaluación tipo multiple choice, obligatoria para aprobar el curso. El foco de la evaluación es sobre CONCEPTOS, no sobre el detalle técnico o anecdótico, así que la prioridad es cobertura completa de conceptos con síntesis, no profundidad.

Voy a pedirte el trabajo en DOS ETAPAS, normalmente en turnos separados. Indicá siempre en qué etapa estamos.

## ETAPA 1 — Formatear apunte crudo

Te paso un apunte tomado en crudo durante la clase (texto suelto, abreviado, desordenado, con errores de tipeo por escribir rápido).

Tu tarea:
1. Reestructurar el contenido en Markdown con jerarquía clara (títulos, subtítulos, listas, negritas para términos clave).
2. Corregir errores evidentes de tipeo/gramática sin cambiar el significado.
3. Agrupar ideas sueltas bajo el concepto al que pertenecen, aunque en el apunte original estén desordenadas o repetidas en distintos lugares.
4. NO agregar contenido que no esté en el apunte crudo en esta etapa — solo reorganizar y limpiar lo que ya anoté. La completitud con las diapositivas es la etapa 2.
5. Mantener fórmulas, definiciones y ejemplos tal cual los anoté, pero prolijos (si es matemática, usar LaTeX con $ $ o $$ $$).
6. Al final de esta etapa, agregar una sección "⚠️ Puntos poco claros / a revisar" listando cualquier parte del apunte crudo que sea ambigua o que no se entienda bien, para que yo la aclare o para resolverla en la etapa 2 con las diapositivas.

Formato de salida: el archivo .md completo, listo para guardar.

## ETAPA 2 — Completar con diapositivas

Te paso (o ya tenés en el repo) las diapositivas de la misma clase en PDF, y el apunte ya formateado de la etapa 1.

Esta etapa se hace en DOS PASOS. No pases al paso 2 sin que yo confirme el paso 1.

**Paso 1 — Resumen de lo que falta (para revisar antes de aplicar)**
1. Revisar las diapositivas y detectar conceptos, definiciones, fórmulas o ejemplos que aparecen ahí pero NO están en mi apunte.
2. Devolverme SOLO una lista breve (no el .md todavía) con lo que detectaste como faltante, una línea por concepto, agrupado por sección del apunte a la que iría. Si algo resuelve un punto marcado como "poco claro" en la etapa 1, indicalo.
3. Esperar mi confirmación (puedo pedir que saques algo de la lista, agregues algo que faltó, o que sigas directo).

**Paso 2 — Aplicar los cambios (después de mi confirmación)**
1. Agregar los conceptos confirmados al .md, integrándolos en la sección correspondiente (no como bloque aparte al final, salvo que no encaje en ninguna sección existente).
2. Marcar visualmente lo que agregaste vos (por ejemplo con 🔹 o un comentario tipo *(de diapositivas)*) para que yo sepa qué es mío y qué completaste vos — esto es importante porque quiero poder confiar en distinguir qué es lo que yo ya tenía anotado.
3. Priorizar SÍNTESIS: si un concepto de la diapo tiene mucho desarrollo/ejemplos redundantes, resumilo en las ideas esenciales en vez de transcribir todo. Preferí definiciones cortas y precisas, cuadros comparativos o listas antes que párrafos largos.
4. Si un concepto se presta a pregunta de multiple choice (definiciones, diferencias entre X e Y, cuándo se usa una fórmula vs otra, condiciones/hipótesis de un teorema, excepciones), destacalo especialmente — por ejemplo con una sección corta tipo "🎯 Posibles puntos de examen" al final del apunte, listando esos conceptos en una línea cada uno.
5. Resolvé o comentá los puntos marcados como "poco claros" en la etapa 1 si las diapositivas los aclaran.

Formato de salida del paso 2: el .md actualizado completo (no solo el diff), listo para reemplazar el archivo anterior.

## Ajuste sobre expresiones matemáticas

En estos cursos hay expresiones matemáticas que ayudan a entender un concepto, pero NO son en sí lo que se evalúa (la evaluación es multiple choice conceptual: qué hace algo, cómo funciona, para qué sirve, cuándo se usa, diferencias entre conceptos).

Por lo tanto:
- NO transcribas ni desarrolles fórmulas completas salvo que sea estrictamente necesario para entender el concepto.
- Si una fórmula aparece, reducila a lo mínimo indispensable (por ejemplo el nombre de la fórmula y una idea de qué relaciona), priorizando SIEMPRE la explicación conceptual en palabras: qué representa, qué problema resuelve, qué pasa si cambia una variable, en qué se diferencia de otro método/concepto similar.
- Preguntate para cada concepto: "¿esto se podría preguntar como '¿qué es X?' o '¿para qué sirve X?' o '¿cuál es la diferencia entre X e Y?'" — si la respuesta es sí, priorizalo en texto plano, no en notación matemática.
- Las fórmulas quedan como referencia secundaria (podés ponerlas en una sub-línea chica o nota), nunca como el contenido principal de la sección.

## Reglas generales para ambas etapas

- Idioma: español.
- Nada de relleno ni frases genéricas de "introducción" — directo al contenido.
- Preferí listas y tablas sobre párrafos largos.
- Si un concepto es puramente de fórmula/cálculo, priorizá tener la fórmula, cuándo se aplica, y qué significa cada término — eso es lo que más rinde en multiple choice.
- No inventes contenido que no esté ni en mi apunte ni en las diapositivas.

## Uso rápido

Etapa 1:
```
Etapa 1 sobre @2026-07-29-clase.md
```

Etapa 2:
```
Etapa 2 sobre @2026-07-29-clase.md usando @diapositivas.pdf
```