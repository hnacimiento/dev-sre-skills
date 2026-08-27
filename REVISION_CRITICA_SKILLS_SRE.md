# Revisión SRE de los skills en C:\Tools\skills\SRE

## Contexto de la revisión

Leí los siete skills completos (`sre-engineering-mindset`, `sre-bash`, `sre-security`, `sre-observability`, `sre-release-deployment`, `sre-incident-review`, `sre-documentation`, `sre-testing` — son ocho archivos pero uno de ellos, testing, es notablemente más corto que el resto) y el archivo `Preguntas para validar y optimizar los skills.txt`, que contiene una batería de prompts de prueba ya diseñada pero sin resultados registrados en la carpeta: no hay ningún archivo de salida, log o transcript que indique que esos tests se hayan corrido. Esa batería de validación es, dicho sea de paso, excelente — está construida exactamente con el criterio correcto (activación explícita vs. implícita, trampa de happy-path, falso éxito, priorización con criterio, anti-overengineering). Vale la pena ejecutarla contra los skills reales antes de darlos por cerrados; ahora mismo son un plan de pruebas sin evidencia.

La comparación es contra el corpus formal que compartiste (los libros de Google SRE, Building Secure and Reliable Systems, el Site Reliability Workbook, Chaos Engineering, y el hilo de podcasts/reflexiones sobre SRE agéntico, STPA, Digital Twin, etc.) y contra la arquitectura de tres capas que vos mismo derivaste al final de esa conversación (razonamiento probabilístico / herramientas deterministas / invariantes de seguridad).

## Conclusión general primero

Estos skills no son un intento inicial. Son, en varios sentidos, una implementación más completa de la filosofía que discutiste con la otra IA que el "Manifiesto" que terminaron esbozando en prosa. `sre-engineering-mindset` ya contiene, con nombres casi idénticos, el modelo de control sociotécnico, la distinción "component correctness != system reliability", el principio determinista/probabilístico, la autoridad del agente separada de la ejecución, el threat model del agente, la evaluación de drift, y el bucle humano de aprendizaje. No es una guía inspirada vagamente en SRE: es SRE de Google aplicado con bastante fidelidad al dominio específico de "un asistente de IA que edita, automatiza y opera sistemas". Esa es la fortaleza principal y hay que decirlo sin rodeos antes de entrar en lo que falta.

Dicho eso, hay brechas reales, algo de duplicación que se va a volver un problema de mantenimiento, y algunas decisiones de diseño de los skills en sí (no del contenido SRE) que conviene revisar.

## Skill por skill

### sre-engineering-mindset

Es el más fuerte de los siete y funciona bien como capa "padre": 43 secciones que van desde toil y work-as-imagined-vs-work-as-done hasta STPA y agentic drift. La sección 38 ("Avoid Absolute Rules Without Context") y la 37 ("Do Not Overfit to Google") son exactamente el antídoto contra el dogmatismo que vos mismo señalaste como fricción en tu propio razonamiento con la otra IA — el skill ya sabe distinguir invariante duro de heurística de default de mecanismo contextual. Eso significa que debería pasar sin problema el Test 10 de tu propio archivo de validación (el script chico y manual que no necesita MPA ni canary).

Lo que le falta, comparado con el corpus: no hay ninguna sección dedicada a SLOs ni error budgets como mecanismo de decisión organizacional. Se menciona "error budget" tangencialmente en `sre-release-deployment` (sección 28) y "SLO-oriented observability" en `sre-observability` (sección 22), pero el principio número uno de la guía ejecutiva que pegaste — "SRE necesita SLOs con consecuencias" — no tiene un hogar propio en ningún skill. Si el objetivo es que estos skills cubran razonamiento SRE de punta a punta, un noveno skill (`sre-slo` o extender observability) que trate definición de SLOs, ventanas rodantes, burn-rate multiwindow, y la regla de congelamiento de features por presupuesto agotado, cerraría ese hueco. Hoy el conjunto sabe razonar sobre *cómo* falla un sistema pero no sobre *cuánta falla es aceptable y qué gatilla la política* — que es el corazón del capítulo 1 y 3 de tu guía ejecutiva.

### sre-bash

Es el más largo (78 secciones) y el más "de trinchera": errexit, traps, subshells, TOCTOU, generación de scripts, todo con el nivel de detalle que un ingeniero de Bash necesita. La cobertura de "generated scripts are security boundaries" y "recovery artifacts must be self-contained" es exactamente lo que discutiste sobre turn-down asimétrico y sobre no confiar en el proceso original para la recuperación. No tengo objeciones de fondo acá; el único riesgo es de volumen — 44KB es mucho contexto para cargar cada vez que alguien pide "escribime un script", y buena parte de las secciones 1-20 son variaciones del mismo principio (exit status != estado, contratos explícitos) que ya están en `sre-engineering-mindset`. Hay duplicación intencional (el skill dice explícitamente que es una especialización), pero convendría revisar si las primeras ~15 secciones podrían compactarse remitiendo al mindset padre en vez de re-explicar el mismo argumento con ejemplos de Bash.

### sre-security

Muy sólido y alineado casi 1:1 con Building Secure and Reliable Systems: trust boundaries, least privilege, blast radius, TOCTOU, deterministic exclusion principle, agent threat model, prompt injection, tool boundaries para agentes. La sección 33 ("Deterministic exclusion principle") es literalmente el argumento de Ramón Medrano Llamas que citaste ("la matemática no está deprecada") convertido en regla operativa. Buena señal de que el corpus se digirió bien y no se copió como cita suelta.

Un matiz que vale la pena marcar: el skill trata SSH/sudo como antipatrón casi siempre ("Do not treat SSH, sudo, root... as inherently safe merely because the operator is trusted"), lo cual es más estricto que tu propia posición intermedia en la reflexión 5-6 (donde defendías que SSH puede ser legítimo si el control es proporcional al riesgo). El skill no dice "SSH está prohibido" — dice "evaluá el boundary real" — así que en rigor no contradice tu postura, pero conviene confirmar que ese matiz quede claro cuando lo uses para revisar código real, porque un lector apurado puede leerlo como un veto absoluto a SSH.

### sre-observability

Muy completo, con una idea que aparece pocas veces en guías de observabilidad convencionales y que sí está en el corpus que compartiste: "Agent Output Is Not Evidence" (sección 32) y "Agent Observability Must Not Depend on Agent Honesty" (sección 60) — esto es exactamente la advertencia de Shannon Brady sobre sobre-confianza en lo que el modelo dice que hizo. Buena cobertura de UNKNOWN como estado legítimo, que es uno de los ejes más fuertes de tu propio análisis en las reflexiones 4 y 5.

### sre-release-deployment

Cubre canary como experimento (no solo "menos máquinas"), expand/contract para migraciones, y la asimetría turn-up/turn-down que Pierre Palatin señala en el podcast que citaste — está explícitamente en la sección 36 ("Destructive Changes"). Buena traducción del principio a release engineering específicamente.

### sre-incident-review

Este es, junto con engineering-mindset, el que mejor traduce el hilo de reflexiones a instrucciones operables. "Blameless does not mean causeless" (sección 1) ataca directamente la trampa de "causa raíz: error del operador" que vos mismo identificaste como antipatrón cultural en tu guía ejecutiva original. La sección de "AI Learning Loop" (39-40) es prácticamente la respuesta a tu propio punto ciego identificado en la Reflexión 6 sobre el "Postmortem Nutrition Loop": ya distingue explícitamente "this worked once" de "this should always be done", que es justo la corrección que la otra IA te señaló que te faltaba.

### sre-documentation

Fuerte en "documentation must match reality" y en el checklist de publicación. Nada que objetar de fondo; es el skill más "aburrido" de los siete en el buen sentido — hace bien un trabajo acotado.

### sre-testing

Es el más corto (17KB contra 30-44KB de los demás) y se nota: llega a la sección 30 mientras los demás pasan de 50-77. Cubre bien fault injection, testing de generated code, e idempotencia, pero no menciona GameDays, DiRT, ni Wheel of Misfortune — que son mecanismos centrales en el corpus que compartiste (Beyer et al., capítulo 28) para validar exactamente lo que este skill se propone: que la automatización sobreviva a fallos no anticipados. Dado que `sre-engineering-mindset` sí los menciona de pasada (sección 4), sería razonable que `sre-testing` los desarrollara con más profundidad, ya que es el skill donde un ingeniero buscaría "cómo diseño una prueba de esto". También es el único de los siete sin una sección de invariantes numerada tipo "Bash Reliability Invariants" (B1-B12) o "Security Invariants" — podría beneficiarse de un bloque similar con invariantes de testing.

## Problemas transversales

Primero, duplicación textual real, no solo temática. El bloque "Postmortem Nutrition Loop" aparece casi palabra por palabra en `sre-observability` (58), `sre-release-deployment` (57), `sre-incident-review` (37-38) y `sre-documentation` (48-51). Lo mismo pasa con el bloque de "Agent Threat Model / Tool Boundaries / Deterministic Exclusion", repetido con variaciones menores en `sre-engineering-mindset`, `sre-security`, `sre-observability`, `sre-release-deployment` y `sre-testing`. Esto no es necesariamente un error de diseño — cada skill se pensó standalone — pero significa que si mañana corregís o profundizás el concepto (por ejemplo, agregás el matiz de STPA sobre que invariantes locales no garantizan seguridad global, que ya identificaste vos mismo como punto ciego), vas a tener que tocar cinco o seis archivos y hay riesgo real de que queden desincronizados. Con `skill-creator` podrías extraer estos bloques recurrentes a un fragmento de referencia único y que cada skill lo cite en vez de reescribirlo.

Segundo, ningún skill trata el patrón "Digital Twin" (Sal Furino) ni la idea de degradar disponibilidad mediante una réplica probabilística del servicio real — lo cual está bien como omisión, en realidad: en tu propia Reflexión 6 vos mismo identificaste ese patrón como arquitectónicamente cuestionable (mezcla probabilístico donde necesitás garantías de integridad transaccional). Que los skills no lo mencionen es consistente con esa autocorrección, así que no lo marcaría como gap sino como coherencia.

Tercero, ninguno de los ocho aborda de forma directa el "50% rule" ni la regla de guardia sostenible (dos incidentes por turno) de tu guía ejecutiva. Esto es esperable — son reglas de gestión de personas y turnos de guardia, no de ingeniería de software — pero si el objetivo final es que estos skills cubran el corpus completo y no solo la porción "técnica", falta un ángulo de gestión operativa (dimensionamiento de guardia, rotación, definición de toil ceiling) que hoy no tiene dueño en ningún skill.

Cuarto, a nivel de metadatos: la mayoría de los `SKILL.md` usa el bloque YAML `description: >` multilínea, pero `sre-documentation` lo escribe en una sola línea larga sin el `>`. Es un detalle cosmético, no funcional, pero conviene unificar el formato antes de considerar el set "terminado".

## Qué haría primero si fuera vos

Correr la batería de `Preguntas para validar y optimizar los skills.txt` contra los skills reales y guardar las respuestas — hoy ese archivo es un plan de examen sin resultados, y es la evidencia empírica que el propio corpus que citaste (Work-as-Imagined vs Work-as-Done) exige antes de confiar en cualquier control de diseño. Después, decidiría si vale la pena invertir en un noveno skill de SLO/error-budget, porque es el único principio "innegociable" de tu propia guía ejecutiva que hoy no tiene una casa clara. Por último, consolidaría los bloques duplicados (postmortem nutrition loop, agent threat model) en un solo lugar de referencia para que corregir el concepto no implique editar seis archivos.
