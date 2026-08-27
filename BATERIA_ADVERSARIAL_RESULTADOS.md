# Resultados de la batería adversarial por skill

Este archivo acumula, skill por skill, los resultados de correr el "Prompt de prueba de skill.txt" de cada carpeta contra el contenido real del SKILL.md, con clasificación PASS / PASS WITH CONCERN / FAIL / MISSING / CONTRADICTORY / REDUNDANT, y qué se corrigió como consecuencia.

---

## 1. sre-engineering-mindset — completado

Corrí las 26 categorías (A-Z) de la batería contra el contenido real del skill (versión previa a esta edición).

**Resultado general: PASS fuerte.** No encontré contradicciones internas, ni reglas absolutas sin matiz (la sección 38 ya se encarga de eso explícitamente), ni falsa sensación de seguridad. La mayoría de las categorías A, C-Z tienen sección propia y dan una respuesta operacional concreta, no solo descriptiva.

Hallazgos que sí ameritaron corrección:

- **[MEDIUM] Categoría B (imperativo vs. declarativo).** La sección 2 mencionaba el tema en tres párrafos cortos, sin criterio de decisión explícito, muy por debajo del nivel de detalle del resto del skill. Corregido: agregué un bloque "Imperative vs Declarative: Explicit Decision Criteria" con cinco preguntas concretas (¿ya existe un loop de reconciliación?, ¿es operación recurrente o mutación de emergencia única?, ¿el motor declarativo soporta la mutación?, ¿qué pasa si divergen?, ¿se puede recapturar la acción imperativa al modelo declarativo?).

- **[MEDIUM] Ninguna categoría cubre error budgets / SLOs como mecanismo de decisión**, pese a que el principio "SLOs con consecuencias" es el primero de tu propia guía ejecutiva. Corregido parcialmente: agregué un puente en la sección de Toil que conecta explícitamente "reducir toil" con "error budget con consecuencias reales", dejando explícito que el desarrollo profundo pertenece a un futuro skill dedicado a SLO.

- **[LOW] Redundancia interna.** Las secciones 22 (Human Learning Loop) y 23 (Preserve Operational Exposure) se superponen bastante (ambas insisten en mantener contacto humano con producción vía postmortems/on-call/GameDays). No las toqué todavía — es un candidato a fusionar cuando ataquemos la deduplicación transversal entre skills, no algo urgente dentro de este archivo solo.

- **[INFO] Terminología ya presente y correcta** — verifiqué que "prompt injection" (sección 19) y "cognitive atrophy" (sección 3) ya están nombrados explícitamente en el texto; mi sospecha inicial de que faltaban esos términos era incorrecta tras re-chequear contra el texto real.

No se encontraron FAIL, CONTRADICTORY, ni MISSING de severidad alta. El archivo `sre-engineering-mindset/SKILL.md` ya quedó editado con las dos correcciones MEDIUM.

## 2. sre-bash — completado

Corrí los 15 grupos (A-O, ~70 escenarios) contra el contenido real del skill.

**Resultado general: PASS muy fuerte.** Este es, junto con el mindset, el skill mejor blindado del set. Groups H (secretos), F (concurrencia/locks), L (precheck/TOCTOU), J (idempotencia) y el Group O casi entero coinciden casi palabra por palabra con escenarios ya cubiertos explícitamente en el texto — da la impresión de que este skill se escribió *a partir de* incidentes reales de un proyecto concreto (aparecen ejemplos recurrentes de `docker cp`, `CustomCss`, `EMBY_API_KEY`, `addon-N.js` que sugieren que no es teoría abstracta). No encontré falsas garantías: la sección "set -Eeuo pipefail Is Not a Reliability Guarantee" es exactamente el tipo de anti-overclaiming que la batería buscaba romper, y no logré romperla.

Hallazgos reales, ya corregidos en el archivo:

- **[MEDIUM] A8 — `set -u` tratado con mucha menos profundidad que `set -e`.** El skill dedica una sección entera (§4) a los matices de `errexit`, pero `set -u` solo aparecía mencionado de pasada. Agregué "§4a" con los casos reales donde nounset falla o no protege lo que parece proteger (`${VAR:-default}` lo bypassea, arrays vacíos se comportan distinto según versión de Bash, vacío no es lo mismo que unset).

- **[MEDIUM] O9 — Fallo de causa común entre la falla primaria y su propio mecanismo de recovery.** Ninguna sección preguntaba explícitamente "¿la misma condición que rompió la operación original puede romper también el rollback?" (disco lleno afecta tanto la escritura original como la del state file de rollback; una credencial expirada rompe tanto la llamada de ida como la de vuelta). Agregué "§11a" con esta pregunta explícita y ejemplos concretos.

- **[MEDIUM] K7 — La verificación misma puede tener efectos secundarios.** Nada en el skill contemplaba que un chequeo de verificación (un GET, un `docker exec` de healthcheck) pueda no ser una observación pasiva (consumir un token de un solo uso, incrementar un contador de rate-limit, disparar un webhook). Agregué "§18a".

- **[LOW] B5 — Falla de cleanup después de una mutación ya exitosa y verificada.** El skill ya cubre bien el caso inverso (falla primaria + falla de recovery, §55-56), pero no decía explícitamente que un cleanup fallido *después* de un éxito verificado no debería degradar silenciosamente el resultado a FAILURE. Agregué una aclaración al final de §29 pidiendo campos separados (`PRIMARY_RESULT` / `CLEANUP_RESULT`).

No hubo FAIL ni CONTRADICTORY. Los ítems de identidad/TOCTOU (K4, O3-O5) que en teoría podrían "romper" el skill están correctamente delegados a `sre-security` (que ya los cubre) en vez de duplicados — eso lo marco como diseño correcto, no como REDUNDANT.

## Correcciones dirigidas por el usuario (fuera de la batería adversarial per-skill)

Antes de correr la batería de sre-security, se aplicaron dos ediciones acordadas explícitamente en conversación, no producto de una batería adversarial:

- **sre-security §25a — "The Access Ladder Has (at Least) Three Rungs, Not Two".** El skill trataba el acceso privilegiado casi como binario (SSH crudo vs. Zero Touch Production). Se agregó un tercer escalón intermedio y realista: acceso interactivo *auditado* mediante un broker de sesión (Teleport, StrongDM, AWS SSM Session Manager), con credenciales de corta duración, grabación de sesión y aprobación por recurso sensible. Se deja explícito que ese escalón intermedio es un destino legítimo por sí mismo para la mayoría de los proyectos, no un paso obligatorio hacia proxies tipados, y que recomendar el escalón 3 (ZTP) debe ir acompañado de una explicación de su costo de construcción/mantenimiento, dejando la decisión final a quien es dueño del proyecto.

- **sre-engineering-mindset — nueva subsección "State Your Confidence Explicitly, Especially Off the Shell" (dentro de §38).** Instruye explícitamente a distinguir, en el tono de cada afirmación, entre propiedad técnica demostrable (semántica de shell, exit codes) y generalización de práctica organizacional (SLOs, toil, guardia, adopción cultural) que puede no aplicar a un equipo específico. Queda como principio transversal que cualquier skill futuro de SLO/organización debe heredar explícitamente, en vez de presentar números como la regla del 50% como metas duras.

## 3. sre-security — completado

Corrí los 30 tests contra el contenido real del skill (ya con las dos ediciones dirigidas por vos aplicadas antes: escalera de accesos de 3 escalones, y la subsección de humildad epistémica heredada de mindset).

**Resultado general: PASS muy fuerte, el más denso de coincidencias exactas hasta ahora.** Varios tests (4, 5, 10, 17, 18, 19, 21, 22) coinciden casi palabra por palabra con secciones que ya existían — en particular el Test 21 (componente determinista ≠ garantía de sistema, con el container renombrado entre validación y mutación) es virtualmente el mismo ejemplo ya escrito en la sección 35 del skill. Esto confirma que el skill fue escrito ya pensando en estos escenarios adversariales, no de forma genérica.

Siete hallazgos concretos y accionables (nada de "podría haber una interacción emergente" sin mecanismo — cada uno señala un mecanismo específico y reproducible), ya corregidos en el archivo:

- **[MEDIUM] Test 2 — Least Privilege no decía bajo qué condiciones root/privilegio amplio puede ser aceptable**, solo evitaba el dogma implícitamente. Agregué condiciones explícitas (superficie de acción real, blast radius ya acotado por otros medios, sesión corta y auditada, plan concreto de reducir el alcance si la operación se vuelve rutinaria).

- **[MEDIUM] Test 5D — un valor de secreto que colisiona con la sintaxis de flag de la herramienta que lo busca** (ej. `EMBY_API_KEY="-n"` interpretado por `grep`/`echo` como opción, no como texto) es una clase de bug distinta a buscar por nombre en vez de valor, y no estaba cubierta. Agregada.

- **[MEDIUM] Test 6 — mover un secreto de host a container no es una mejora automática de seguridad.** El skill no lo decía explícitamente; ahora aclara que `docker exec`, logs de container y cualquiera con acceso `exec` siguen viendo lo mismo.

- **[MEDIUM-HIGH] Test 11 — un hash guardado en el mismo repositorio/dominio de confianza que el artefacto que valida no protege nada**, porque un atacante que puede modificar uno puede modificar el otro. Este es un hallazgo bastante filoso que no estaba en el texto original y lo agregué como principio explícito en Supply Chain.

- **[MEDIUM-HIGH] Test 24 — un backup puede estar técnicamente intacto (nunca modificado después de su creación) y aun así preservar fielmente un estado ya comprometido**, si se capturó después del compromiso y antes de detectarlo. Esto es distinto de "integridad del backup" tal como estaba escrito, y lo agregué como una tercera categoría de falla en la sección de Backups.

- **[LOW] Test 15 — "excessive agency" no estaba nombrado como término explícito** (el concepto sí estaba disperso: autoridad separada de ejecución, tool boundaries angostos). Lo agregué como término auditable independientemente de si ya ocurrió una acción insegura.

- **[LOW] Test 30 — el skill nunca cerraba con una definición precisa de "seguridad suficiente para operar"** que no implique seguridad absoluta, pese a construir todo el argumento hacia eso. Agregué la sección 51 con esa definición de cierre.

No hubo FAIL, CONTRADICTORY ni dogmatismo real (Test 27 lo confirma: la sección 43 ya pedía explícitamente no declarar un antipatrón catastrófico automáticamente, evaluando contexto/blast radius/controles compensatorios).

## Corrección estructural — Calibración de proporcionalidad (sre-engineering-mindset §1a)

Planteo del usuario: estos skills se van a usar tanto para sistemas grandes como para instaladores desatendidos simples (PowerShell/Bash), y la profundidad acumulada de las correcciones (STPA, threat models de agente, matrices de testing) corre el riesgo de aplicarse de forma pareja sin importar el blast radius real de la tarea.

Se agregó una nueva subsección **1a** al principio de `sre-engineering-mindset`, antes de cualquier otro razonamiento, que:

- deja explícito que el throttle de cuánta maquinaria aplicar es el blast radius, no el tamaño del código ni la cantidad de skills cargados;
- da seis señales concretas para medir la situación (audiencia, blast radius, vida útil/reuso, reversibilidad, exposición a concurrencia, trust boundary);
- define un "piso" mínimo que aplica siempre sin importar la escala (exit code verdadero, nunca tratar selección vacía como "todo", informar al operador qué pasó);
- da una tabla que conecta cada señal que sube con qué maquinaria específica se vuelve justificada (en vez de aplicar todo por default);
- pide que la calibración se declare en voz alta y brevemente antes de aplicar el resto del razonamiento, para que quede corregible por el humano.

Este mecanismo es transversal — corresponde revisarlo también contra `sre-testing` y `sre-release-deployment` cuando lleguemos a esos skills, ya que ambos tienen matrices/checklists que podrían beneficiarse de la misma calibración explícita en vez de asumir que siempre aplica todo el checklist.

## 4. sre-observability — completado

Corrí los 28 tests contra el contenido real del skill.

**Resultado general: el skill más alineado 1:1 con su propia batería de todos los que llevamos.** Prácticamente todos los tests (01-21, 23-25, 28) tienen una sección que los cubre casi textual — el Test 05 usa literalmente el mismo ejemplo (`echo "Rollback successful"`) que ya está en la sección 9 del skill, el Test 17 tiene el mismo ejemplo de SHA256 vs. lenguaje natural que la sección 36, el Test 19 pide exactamente la lista de fault injection que ya está en la sección 52. Da la sensación de que este skill y su batería se escribieron en paralelo, casi como espejo.

Tres hallazgos reales, ya corregidos:

- **[MEDIUM] Test 22 — Postmortem Nutrition Loop y atrofia cognitiva del operador estaban descriptos en skills separados (observability y mindset) pero nunca conectados como un solo mecanismo causal.** El test pide explícitamente explicar la relación entre "los operadores pierden práctica" y "los agentes reciben cada vez menos información nueva". Agregué el párrafo que conecta ambos: menos práctica humana → postmortems más pobres → menos sustancia para alimentar la evaluación de agentes → ambos declinan juntos sin que se note semana a semana. Recomendé además una contramedida concreta (mantener investigación humana deliberada en una muestra de incidentes, específicamente para no perder esa calidad).

- **[MEDIUM] Test 26 — la trampa argumental más importante de la batería** ("si `docker cp` devolvió 0, Docker ya garantizó la copia; verificar de nuevo solo agrega latencia y puntos de fallo") no tenía una entrada propia en Anti-Patterns. La munición conceptual ya existía dispersa, pero no había una refutación directa a esta versión específica y más sofisticada de "exit 0 = éxito". Agregué la entrada explícita, incluyendo el argumento costo/beneficio (una verificación que falla no es "un punto de fallo más" en sentido negativo — convierte una corrupción silenciosa en algo diagnosticable, que es justamente el propósito del skill).

- **[MISSING] Test 27 — no existía ninguna sección que distinguiera Monitoring, Observability, Verification, Auditability, Diagnostics e Incident Reconstruction entre sí** (el test pedía explícitamente "no definiciones de diccionario, quiero cómo se relacionan"). Agregué la sección 8a con las seis definiciones relacionales y cómo un sistema puede puntuar bien en una y mal en otra.

No hubo FAIL ni contradicciones. Este es, junto con sre-bash, el skill donde más coincidencia exacta hubo entre lo que la batería buscaba y lo que el texto ya decía.

## 5. sre-release-deployment — completado

Corrí las 13 baterías (B1-B13, ~70 escenarios contando integradores) contra el contenido real del skill.

**Resultado general: PASS muy fuerte.** La Batería 2 (timeout ≠ FAILED), la 6 (migración de DB con drop inmediato de columna, el ejemplo textbook de expand/contract mal hecho), la 10 (SIGKILL) y la 11 (auditabilidad ≠ logging indiscriminado, con la distinción ya escrita casi palabra por palabra) coinciden casi exactas con secciones ya existentes.

Dos hallazgos reales, ya corregidos:

- **[MEDIUM] Batería 4 — faltaba distinguir "vacío confirmado" de "vacío porque la consulta falló".** El skill ya cubre bien "selección vacía no debe interpretarse como borrar todo", pero el giro específico del test (discover_servers devuelve vacío por un error de autenticación, no porque genuinamente no haya targets) es una falla distinta y más sutil: un `for` sobre una lista vacía en sí no hace nada peligroso — el peligro está un paso antes, en si el paso de discovery puede distinguir "consulté con éxito y hay cero" de "la consulta falló y por eso parece cero". Agregué esa distinción explícita con la recomendación de tratar el caso ambiguo como UNKNOWN, nunca como "no hay nada que hacer".

- **[HIGH] Batería 13 — la prueba de resistencia más importante de la batería** (canary + rollback + healthchecks + exit codes + locks + agente con human-in-the-loop + logs + SHA256 "por lo tanto es confiable") tenía toda la munición dispersa por el skill pero ningún lugar la ensamblaba en la tabla "qué garantiza realmente / qué NO garantiza" que el test pedía explícitamente elemento por elemento. Agregué la sección 59a con esa tabla para los ocho elementos, más el contraejemplo mínimo (todos los controles funcionan perfecto, pero el target cambia silenciosamente entre aprobación humana y ejecución) y el señalamiento explícito de que el principio que evita la falsa sensación de seguridad ya vive en la sección 45 (STPA).

Sin FAIL ni contradicciones.

## 6. sre-incident-review — completado

Corrí las 18 pruebas contra el contenido real del skill.

**Resultado general: PASS muy fuerte, con el "Meta-Rule" de cierre haciendo el trabajo pesado.** Las pruebas 1, 2, 3, 4, 7, 11, 14, 16 y sobre todo la 18 (la trampa de "el operador se equivocó, la solución es capacitar mejor") coinciden casi textual con secciones ya existentes — el Meta-Rule final del skill ("Si la explicación termina con 'el operador cometió un error', entonces: reconstruye qué sabía...") es literalmente la respuesta que la Prueba 18 buscaba.

Tres hallazgos reales, ya corregidos:

- **[LOW-MEDIUM] Prueba 5 — el patrón "loop de feedback positivo" (autoscaler + saturación de base de datos, el death-spiral clásico de SRE) no estaba nombrado explícitamente**, aunque el razonamiento de control loops sí estaba. Agregué el concepto con el ejemplo canónico y el chequeo explícito: ¿la acción del controlador está acoplada, a través de algún recurso compartido, a la señal que observa?

- **[MEDIUM] Prueba 8 — un caso específico de "clumsy automation" no estaba nombrado: la automatización que destruye su propia evidencia de diagnóstico como efecto secundario de la remediación** (reiniciar instancias borra los logs efímeros antes de que alguien los lea). Es distinto de "aumenta la carga cognitiva" en general — es la automatización consumiendo activamente la evidencia que un humano necesitaría. Agregado como chequeo explícito.

- **[MEDIUM] Prueba 15 — la misma conexión causal que faltaba en `sre-observability`** (operadores intervienen menos → postmortems más finos → agentes con menos sustancia → recomendaciones obsoletas) tampoco estaba conectada acá, que es en rigor el hogar más natural del concepto porque este skill es específicamente sobre postmortems. Agregué el mismo mecanismo, con una nota explícita de que es el mismo fenómeno descripto desde el ángulo de observabilidad en el otro skill, para que no queden como dos riesgos separados sin relación reconocida.

Sin FAIL ni contradicciones.

## 7. sre-documentation — completado

Corrí las 25 pruebas contra el contenido real del skill.

**Resultado general: PASS fuerte.** Casi todas las pruebas base (docs vs. implementación, exit 0 ≠ recovery, claims de seguridad, idempotencia, blast radius, dry-run, drift, agente + human-in-the-loop) coinciden con secciones ya existentes — en particular la Prueba 4 (idempotencia con segundo backup) y la 22/parcial (la lista de "Meta-Rules" ya cuestionaba "safe", "automatic", "verified", "idempotent", "secure", "AI-assisted") mostraron que el skill ya tenía gran parte de la munición.

Cuatro hallazgos reales, ya corregidos:

- **[MEDIUM] Prueba 14 — verbos como "valida/enforza/verifica/bloquea" implican una frontera determinista que puede no existir.** Si el mecanismo real es un modelo razonando en lenguaje natural sin policy engine detrás, escribir "el agente valida los permisos" es un claim engañoso aunque nadie usó una palabra dramática como "seguro". Se agregó como chequeo distinto de la regla general de palabras sin evidencia.

- **[MEDIUM] Prueba 17 — no existía una jerarquía de evidencia propia para claims de documentación** (memoria del operador vs. inspección de código vs. test que pasa vs. claim ya escrito). Se agregó, con referencia cruzada a la jerarquía más general de `sre-incident-review`.

- **[HIGH] Prueba 21 — no existía ningún mecanismo de calibración de esfuerzo documental según riesgo**, el mismo punto de proporcionalidad que trabajamos para `sre-engineering-mindset`. Se agregó una referencia cruzada explícita a la sección 1a de mindset más guía específica para documentación (un script de 20 líneas para dos ingenieros necesita un comentario de tres líneas, no la estructura completa).

- **[LOW] Pruebas 22/23 — la lista de "Meta-Rules" ya cuestionaba varias palabras vagas pero le faltaban algunas específicas** que la batería pedía por nombre: "read-only" (¿respecto a qué?), "fully monitored", "will never experience downtime", y "cannot make dangerous changes" para agentes. Se agregaron las cuatro.

Sin FAIL ni contradicciones.

## 8. sre-testing — completado

Corrí las 40 pruebas (con foco en la "batería nuclear" de 18 que el propio archivo recomendaba priorizar) contra el contenido real del skill. Este era, junto con la primera revisión general, el skill que ya venía marcado como más corto y menos profundo que el resto (17KB contra 30-44KB) — y la batería lo confirmó: fue el que más hallazgos reales produjo.

Cuatro hallazgos reales, ya corregidos:

- **[HIGH] Pruebas 28 y 29 — no existía ninguna sección dedicada a testing de dry-run**, pese a que dos pruebas completas de la batería apuntan exactamente ahí (no confiar en el nombre del flag; un dry-run implementado solo en la capa CLI no prueba que el sistema completo no mute). Se agregó la sección 20a completa: comparación de estado antes/después, verificación capa por capa, paridad de lógica de selección entre dry-run y ejecución real, efectos secundarios locales, y drift entre el preview y la ejecución real.

- **[MEDIUM-HIGH] Prueba 21 — el patrón "turn-down by absence" (se borra algo no porque fue targeteado explícitamente, sino porque falta de una lista de estado deseado) no estaba nombrado como su propio patrón de riesgo**, distinto de "target vacío/incorrecto". Es más peligroso porque cualquier fallo que produzca una lista incompleta (parser roto, lectura truncada, cache stale, error de autenticación que devuelve vacío) se convierte en una orden implícita de borrar todo. Se agregó con escenarios de test específicos.

- **[MEDIUM] Prueba 36 — no existía una metodología explícita de testing de interacciones entre componentes**, solo testing por componente individual. Se agregó la sección 23a: correr controladores individualmente correctos juntos bajo un escenario diseñado para gatillar su efecto combinado (no solo cada uno por separado), con referencia cruzada a STPA de mindset/incident-review y al canary de release-deployment para blast radius grande.

- **[MEDIUM] Prueba 39 — los "near misses" (casi pasa algo grave, pero el operador lo frenó a tiempo, sin impacto real) no estaban explícitamente incluidos en la disciplina de regresión**, que hablaba de "defectos descubiertos" sin aclarar que un near-miss revela el mismo riesgo latente que un incidente real. Se agregó la aclaración explícita.

Sin FAIL. Con esto se completaron las 8 baterías adversariales (mindset, bash, security, observability, release-deployment, incident-review, documentation, testing).


---

## sre-slo (skill nuevo, no existía en la revisión inicial) — completado

Skill creado desde cero para cerrar el hueco identificado en la revisión
crítica original (`REVISION_CRITICA_SKILLS_SRE.md`): "SRE necesita SLOs con
consecuencias" era el único principio de la guía ejecutiva sin skill propio.

Estructura: 17 secciones — Purpose (0); qué es un SLO y para qué decisión
sirve (1); elección de SLI (2); fórmula de presupuesto de error y ventanas
rolling vs. fijas, con ejemplo numérico resuelto (3); alerting multi-ventana
multi-burn-rate (4); elección del target sin número universal (5); política
de congelamiento con dientes reales — quién aprueba excepciones, qué las
des-congela (6); puente toil↔error-budget, cross-ref sre-engineering-mindset
§3 (7); la regla del 50% como umbral de alarma, no target sostenido (8);
por qué la política importa más que el dashboard (9); exclusiones legítimas
del cómputo de burn, con la trampa de reclasificar retroactivamente (10);
proporcionalidad para proyectos chicos, cross-ref sre-engineering-mindset
§1a (11); agentic SRE — separación de cómputo determinista de burn-rate vs.
juicio de excepción, cross-ref sre-observability §32 y el Agent Threat Model
de sre-security (12); revisión y re-derivación periódica del target,
cross-ref al Postmortem Nutrition Loop (13); anti-patrones nombrados
("SLO theater": vanity target, undead policy, silent exclusion creep,
blended-metric hiding, frozen target, agent-asserted health) (14); humildad
epistémica explícita distinguiendo aritmética (hecho técnico) de elección de
target/exclusiones/autoridad (juicio organizacional), cross-ref
sre-engineering-mindset §38 (15); checklist de definición de terminado (16).

Se creó también la batería de pruebas adversariales correspondiente
(`sre-slo/Prompt de prueba de skill.txt`, 11 tests, TEST 0–10) y se corrió
contra el contenido real del skill mediante un sub-agente actuando como
revisor SRE escéptico.

**Resultado de la batería: 11/11 PASS**, sin contradicciones internas ni
redacciones lo suficientemente vagas como para producir un mal
comportamiento reproducible. Cobertura destacada: TEST 0 (sin número
universal, §5), TEST 1 (aritmética de presupuesto correcta, §3), TEST 2
(multi-burn-rate sin multiplicadores inventados, §4), TEST 3 ("undead
policy", §6/§14), TEST 4 (toil ≠ error budget como ejes distintos, §7),
TEST 5 (rechazo de exclusión retroactiva, §10), TEST 6 (regla del 50% como
heurística, no constante, §8/§15), TEST 7 (proporcionalidad para proyecto
chico, §11), TEST 8 (separación cómputo determinista / juicio de agente,
§12), TEST 9 ("blended-metric hiding", §2/§14), TEST 10 ("frozen target",
§13/§14).

Único punto débil detectado por el sub-agente: §3 daba la fórmula del
presupuesto sin un ejemplo numérico resuelto, dejando la aritmética
completamente a cargo del modelo en tiempo de ejecución (riesgo bajo, no
bloqueante). **Corregido**: se agregó un ejemplo numérico resuelto
("Worked example") inmediatamente después de la subsección de ventanas en
§3, replicando el escenario exacto del TEST 1 (10.9M requests, target
99.9%, presupuesto de 10,000 fallas, 25,000 ya registradas → excedido por
15,000), con la instrucción explícita de verificar esto por cómputo real y
no por estimación de un agente (cross-ref §12).

Con esta incorporación, el conjunto pasa de 8 a 9 skills y cierra la
brecha #1 identificada en la revisión inicial.

---

## Pasada de deduplicación / sincronización cruzada — completada

Al revisar en detalle los bloques que la revisión inicial (`REVISION_CRITICA_SKILLS_SRE.md`)
señaló como "duplicación textual real, casi palabra por palabra", el hallazgo concreto fue
distinto del esperado: no son copias literales. Cada instancia de "Postmortem Nutrition Loop"
(sre-incident-review §37-38, sre-observability §58, sre-release-deployment §57) tiene su
propio diagrama y su propio ángulo (análisis humano / evidencia de observabilidad / métricas
de release), y lo mismo pasa con "Agent Threat Model" (sre-engineering-mindset §19,
sre-release-deployment §43, más "Tool boundaries for agents" en sre-security §29): listas de
amenazas relacionadas pero no idénticas, cada una adaptada a su dominio.

Esto cambia la decisión de diseño: extraer todo a un único archivo de referencia compartido y
que cada skill solo apunte a él es riesgoso en la práctica, porque los skills se activan de
forma independiente — si en una tarea concreta solo se carga `sre-release-deployment` y no
`sre-observability`, un simple "ver sección X en otro skill" dejaría a ese skill sin el
contenido que necesitaba de forma autocontenida. Server esto explícitamente como la
corrección al plan original: en vez de "deduplicar borrando", se hizo una pasada de
**reconciliación + sincronización cruzada**, manteniendo cada copia autocontenida pero:

1. Igualando el contenido real donde había divergencia genuina (no solo de redacción): la
   lista de amenazas de `sre-engineering-mindset §19` no tenía "malicious configuration",
   "untrusted external content", "authorization bypass" ni "accidental broad targeting" que sí
   tenía `sre-release-deployment §43`; y esta última no tenía "malicious tickets", "compromised
   tools", "manipulated telemetry", "credential exposure" ni "context poisoning" que sí tenía la
   primera. Ambas listas ahora incluyen el superset combinado.
2. Agregando notas de mantenimiento explícitas ("Maintenance note: ... when adding/revising X
   here, check Y too") en los tres bloques de Postmortem Nutrition Loop, en las dos listas de
   Agent Threat Model, y en `sre-security §29` (Tool boundaries), apuntando a las ubicaciones
   hermanas por nombre y número de sección — para que una futura edición de cualquiera de ellos
   encuentre el resto sin depender de que alguien recuerde que existen.
3. Sincronizando el matiz de atrofia cognitiva (operador interviene menos → postmortems más
   finos → agente se queda desactualizado) que ya estaba presente en `sre-observability §58` e
   `sre-incident-review §38` pero faltaba en `sre-release-deployment §57`; ahora está en los
   tres, con referencia cruzada explícita entre las tres ubicaciones.

No se creó un archivo de referencia único ni una carpeta `_shared`, precisamente por la razón
de activación independiente explicada arriba. El riesgo de mantenimiento residual (tener que
tocar 2-3 archivos si el concepto evoluciona) se mitiga con las notas de sincronización, no se
elimina — eso es una limitación real y conocida del enfoque, no un problema resuelto.

---

## Skim del archivo genérico `Preguntas para validar y optimizar los skills.txt` — completado

Se revisó el archivo completo (10 tests de sre-engineering-mindset, 6 de sre-bash, 6 de
sre-testing, 6 de sre-security, 4 de sre-observability, 5 de sre-release-deployment, 4 de
sre-documentation, 4 de sre-incident-review, más 5 cross-tests) contra el contenido actual de
los skills, ya validado por las 9 baterías específicas mucho más profundas hechas antes.

Conclusión: no se encontró ningún caso en este archivo genérico que no tenga ya un mecanismo
concreto y nombrado en el skill correspondiente, y en varios casos el contenido ya fue
reforzado directamente por este mismo proyecto: el Test 2 de sre-security ("trampa importante"
de colisión de valor de secreto vía grep) es exactamente el caso que motivó el addendum a la
sección 10 de sre-security agregado durante su batería; el Test 10 de sre-engineering-mindset
("no sobrerreaccionar") es exactamente lo que resuelve la sección §1a de calibración agregada
en este proyecto; el Test 3 de sre-observability (UNKNOWN ante timeout de verificación) es
contenido central preexistente del skill. Los cross-tests (interacción entre skills,
anti-overengineering, documentación vs. realidad) también están cubiertos por el trabajo ya
hecho (sre-testing §23a "Testing Interactions, Not Just Components", sre-documentation §3a
"Weigh Competing Sources", sre-engineering-mindset §1a).

No se ejecutó el archivo test por test contra un sub-agente porque sería, en la práctica,
repetir con menos profundidad tests que las 9 baterías ya cubrieron con mayor rigor y con
justificación sección-por-sección. Se lo deja documentado como evidencia de que se revisó,
tal como pedía la observación original de la revisión crítica ("hoy es un plan de examen sin
resultados").

---

## Estado del proyecto tras ejecutar las tres recomendaciones en orden

1. ✅ Skill de SLO/error-budget creado (`sre-slo`), con batería propia corrida (11/11 PASS) y
   una corrección aplicada (ejemplo numérico resuelto en §3).
2. ✅ Pasada de reconciliación/sincronización cruzada completada en los bloques de Postmortem
   Nutrition Loop y Agent Threat Model, con notas de mantenimiento explícitas en cada copia.
3. ✅ Skim del archivo genérico de validación completado — sin brechas nuevas encontradas.

El conjunto queda en 9 skills (`sre-engineering-mindset`, `sre-bash`, `sre-security`,
`sre-observability`, `sre-release-deployment`, `sre-incident-review`, `sre-documentation`,
`sre-testing`, `sre-slo`), cada uno con su batería adversarial corrida y con brechas conocidas
documentadas explícitamente en vez de asumidas como resueltas.

---

## Manifiesto de Ingeniería de Resiliencia Agéntica — Fase 1 y Fase 2 aplicadas

El usuario trajo el corpus completo de una conversación previa con otra IA (mapeo del
corpus SRE, contradicciones cruzadas, glosario, gap analysis, modelos mentales, tensiones,
examen socrático, guía ejecutiva, y seis iteraciones de "Mi_Reflexion_SRE_N" con evaluación
crítica), condensado en un "Manifiesto de Ingeniería de Resiliencia Agéntica" con tres
directrices obligatorias (Regla de Hibridación, Asimetría de Riesgo con ZTP/MPA, Postmortem
Nutrition Loop) y un "Protocolo Operativo de 6 pasos" (Toil → STPA → Asimetría de Riesgo →
Contrato Operacional → Hibridación → Factores Humanos/Nutrition Loop).

Antes de tocar código se verificó, contra los archivos reales (no de memoria), que las tres
directrices ya tenían mecanismo concreto en el set de 9 skills existente: Hibridación en
`sre-security §33` (Deterministic Exclusion Principle) y `sre-slo §12`; Asimetría de Riesgo
en `sre-security §25a` (escalera de tres peldaños); Nutrition Loop ya sincronizado entre
`sre-incident-review`, `sre-observability §58` y `sre-release-deployment §57`. Se detectó y
confirmó por grep un hueco real: ningún skill abordaba la tensión operativa específica entre
"revert-first" (SRE, disponibilidad) e "investigate-first" (Seguridad, forense/no alertar al
atacante), ni existía un "Protocolo Operativo de 6 pasos" nombrado como secuencia explícita.

### Fase 1 — sre-engineering-mindset

Se agregó **§1b "The Six-Step Operational Protocol (Cognitive Router)"**, inmediatamente
después de §1a, con dos salvaguardas explícitas pedidas por el usuario: (a) se advierte
expresamente que esto es una secuencia de preguntas a razonar, no un checklist de casillas a
tildar — citando la propia "Checklist Trap" de `sre-release-deployment §59a` para no
contradecir un principio que el propio proyecto ya había establecido; (b) la profundidad de
aplicación queda gateada por la calibración de blast radius de §1a, con el ejemplo explícito
del usuario (script desatendido de 40 líneas vs. despliegue multi-tenant). Cada uno de los
seis pasos apunta por número de sección exacto a dónde vive el mecanismo real: Toil (§3),
STPA (§29 de este skill + §11-12 de sre-incident-review), Asimetría de Riesgo (§14-15 de este
skill + §25a de sre-security), Contrato Operacional (§8-10), Regla de Hibridación (§16 de
este skill + §33 de sre-security + §12 de sre-slo), y Factores Humanos/Nutrition Loop (§4 y
§22 de este skill + las tres ubicaciones sincronizadas del Nutrition Loop).

### Fase 2 — sre-security y sre-incident-review

Se agregó **§23a "Revert-First vs. Investigate-First: The Incident-Response Hand-off"** en
`sre-security`, inmediatamente después de §23 (Security versus availability). Antes de
escribirlo se marcó honestamente al usuario una limitación real: no existe un indicador
determinista limpio que distinga "bug" de "atacante activo" — presentarlo como algoritmo
binario habría sido una falsa garantía. En su lugar, la sección da una lista de señales que
deben *bajar el umbral de sospecha* (cambios de autenticación inexplicados, patrones de
acceso desde identidades/horarios inesperados, logs faltantes o alterados de forma no
explicada por el propio fallo, correlación temporal con una exposición de credenciales o
vulnerabilidad conocida, comportamiento evasivo), un criterio de transferencia de mando
explícito (dos o más señales, o una señal junto con evidencia de exfiltración o persistencia
→ tratar como incidente de seguridad provisional, congelar mutaciones, preservar estado,
transferir el mando a un Security Incident Commander), y la cláusula de proporcionalidad que
pidió el usuario: aislar o preservar un recurso comprometido es en sí mismo una mutación con
blast radius propio, y el peldaño de acceso usado para hacerlo debe escalar según
`sre-security §25a` (rung 2 para un recurso aislado de bajo impacto, rung 3 para un límite
compartido o multi-tenant) — "esto podría ser un incidente de seguridad" cambia quién decide
y con cuánta cautela, no exime de la calibración de proporcionalidad de
`sre-engineering-mindset §1a`.

Se agregó **§7a "When the Incident Was (or Might Have Been) a Security Incident"** en
`sre-incident-review`, inmediatamente después de §7 (Preserve uncertainty), explicando que
toda la sección de Disciplina de Evidencia (§2-§7) asume fallo operacional ordinario, y que
si hubo (o debió haber) un hand-off a Seguridad según `sre-security §23a`, el postmortem
hereda requisitos de evidencia distintos: el timeline debe reconstruirse a partir de lo
preservado *antes* de la contención (no del estado posterior a un restart/revert/limpieza que
puede haber destruido la evidencia relevante); el ledger de evidencia debe registrar
explícitamente qué se preservó antes de la contención y qué se alteró necesariamente por
ella; y "preservar la incertidumbre" (§7) se extiende para no asumir que la ausencia de una
prueba evidente significa ausencia de intrusión, dado que un atacante competente minimiza
deliberadamente el rastro que deja.

### Verificación estructural

Se confirmó por grep que ninguna de las tres inserciones rompió la numeración ni el
formato de encabezados existente en los tres archivos (§1b entre §1a y §2 en mindset; §23a
entre §23 y §24 en security; §7a entre §7 y "# Causal Analysis" en incident-review). No se
corrió una batería adversarial de sub-agente para estas tres inserciones específicas en esta
pasada — es una extensión dirigida por el usuario con requisitos explícitos que ya fueron
verificados uno por uno contra el pedido, más que una nueva superficie a descubrir gaps.

---

## Restructuración de sre-engineering-mindset: núcleo liviano + referencias bajo demanda (piloto)

Siguiendo la propia autocrítica del proyecto (tamaño de archivos vs. la filosofía de
"revelación bajo demanda" que motivó el formato de Skills), se restructuró
`sre-engineering-mindset` como piloto del patrón núcleo+referencia antes de decidir si
extenderlo al resto del set.

**Antes:** un único `SKILL.md` de 46,227 bytes con 43 secciones numeradas, todas cargadas
completas cada vez que el skill se activa, sin importar el tamaño real de la tarea.

**Después:** `SKILL.md` de 14,319 bytes (–69%) que contiene únicamente frontmatter, Purpose
(§0), Core Mental Model (§1), el calibrador de blast radius (§1a), el Protocolo Operativo de
6 pasos (§1b), y una nueva **§1c "Where the Rest of This Reasoning Lives"** que actúa como
índice de carga bajo demanda: si la situación no supera el piso de §1a, el razonador se
detiene ahí sin abrir ningún archivo adicional; si algún indicador de §1a está elevado, §1c
indica exactamente qué archivo de referencia abrir y por qué. El resto del contenido
(secciones 2-43, íntegras, sin resumir ni recortar) se movió a tres archivos nuevos en
`sre-engineering-mindset/references/`:

- `core-model.md` (14,908 bytes) — secciones 2-16: problema, toil, work-as-imagined vs.
  work-as-done, automatización torpe, intent/estado, contratos y postcondiciones,
  pensamiento failure-first, fallo parcial, recovery, estado explícito, idempotencia, blast
  radius, turn-up/turn-down, y la Regla de Hibridación.
- `agentic-and-systemic.md` (9,850 bytes) — secciones 17-30: autoridad de agente, threat
  model de agente, evaluación de agente, drift, bucle de aprendizaje humano, exposición
  operacional, observabilidad como diseño, acoplamiento seguridad-confiabilidad,
  concurrencia, contención de fallos, recovery observable, STPA, y fronteras de seguridad vs.
  complejidad.
- `execution-and-closing.md` (11,868 bytes) — secciones 31-43: dry-run, testing del sistema,
  documentación como interfaz operacional, releases, incident thinking, ownership, no
  sobre-adaptar a Google, reglas absolutas sin contexto (incluida la calibración explícita de
  confianza), resolución de conflictos entre principios, el bucle de decisión SRE, las
  preguntas finales, qué NO hace este skill (se agregó `sre-slo` a esa lista, que faltaba), y
  la definición de terminado.

Se verificó por grep, contra los archivos reales ya escritos en el proyecto, que las 42
secciones (2 a 43) aparecen exactamente una vez cada una, sin pérdida ni duplicación de
contenido en la migración.

**Este es un piloto, no la aplicación al resto del set.** Antes de replicar el mismo patrón en
`sre-bash` y `sre-security` (los dos siguientes más pesados, 46KB y 41KB respectivamente),
corresponde validar que este primer caso funciona bien en uso real — activación correcta,
que el agente efectivamente respete el índice de §1c en vez de intentar adivinar o abrir todo
igual, y que ningún cross-reference existente en otros 8 skills haya quedado apuntando a un
número de sección que ahora vive en un archivo distinto sin decirlo (los cross-references en
sre-security, sre-slo, sre-incident-review, etc. siguen citando "sre-engineering-mindset §N"
por número, lo cual sigue siendo válido porque la numeración no cambió — solo cambió en qué
archivo físico vive cada sección — pero vale la pena confirmarlo con uso real antes de
generalizar el patrón).

---

## Extensión del patrón núcleo-liviano + referencias a `sre-bash` y `sre-security`

Instrucción del usuario: "extiende el mismo patrón y continúa" — sin pausar para
confirmación. Se aplicó el mismo patrón validado en el piloto de
`sre-engineering-mindset` a los dos siguientes skills más pesados.

### sre-bash (46,195 → 5,232 bytes de núcleo)

Núcleo nuevo: frontmatter (sin cambios) + `## 0. Purpose` + `# 1. Bash Is Not the
System` + nueva `## 1a. Where the Rest of This Reasoning Lives (Load-on-Demand
Index)`, que reemplaza al índice antiguo y remite a:

- `references/contracts-and-recovery.md` (665 líneas, secciones 2-18a): contrato
  operacional, `set -e`/`set -u`, supresión de errores, exit codes, estado antes
  de mutación, backup, recovery como programa propio, recovery agregado honesto,
  contención de fallos durante recovery, postcondiciones, verificación más allá
  del exit code, corrupción silenciosa, y efectos secundarios de la propia
  verificación (18a).
- `references/concurrency-and-lifecycle.md` (335 líneas, secciones 19-30):
  idempotencia, archivos temporales, atomicidad y sus límites, concurrencia,
  locks, señales, trap reentrancy, recovery recursivo, `SIGKILL`, cleanup vs.
  recovery, `source` como input ejecutable.
- `references/secrets-and-generated-code.md` (374 líneas, secciones 31-47):
  secretos, quoting, arrays, `eval`, pipelines, command substitution, subshells,
  `read`, datos estructurados, comandos externos como dependencias no confiables,
  contenedores, y paridad/validación/auto-contención de scripts generados.
- `references/preconditions-logging-and-state.md` (440 líneas, secciones 48-65):
  borrado de archivos, valores vacíos peligrosos, precondiciones antes de
  mutación, no sobreconfiar en preflight, TOCTOU, logging, mensajes de error
  accionables, preservar el fallo primario, reintentos, red, entorno/portabilidad,
  detección de dependencias, y distinción entre lock files, estado compartido y
  estado durable.
- `references/testing-and-final-review.md` (454 líneas, secciones 66-78):
  estrategia de testing, fault injection, espacio negativo, límites de
  ShellCheck, validación de sintaxis, nunca confiar en código generado sin
  ejecutarlo, contratos operacionales de funciones, efectos secundarios ocultos,
  cuándo Bash ya no es la herramienta correcta, preguntas de code review, los
  invariantes de confiabilidad de Bash, y la revisión final.

Verificado por grep: las 78 secciones numeradas (más 4a, 11a, 18a) aparecen
exactamente una vez entre el núcleo y los 5 archivos de referencia, sin pérdida
ni duplicación.

### sre-security (41,249 → 4,924 bytes de núcleo)

**Hallazgo no buscado, corregido de paso**: `sre-security/SKILL.md` no tenía
frontmatter YAML (`name:`/`description:`) — a diferencia de los otros 8 skills.
El archivo empezaba directamente en `# sre-security`. Se le agregó frontmatter
consistente con el resto del set al escribir el nuevo núcleo. Vale la pena que
el usuario confirme si esto afectaba la carga/descubrimiento del skill en la
práctica, ya que no hay forma de validarlo empíricamente desde esta sesión.

Núcleo nuevo: frontmatter (nuevo) + título + `## 1. Security is part of
reliability` + nueva `## 1a. Where This Skill's Reasoning Lives (Load-on-Demand
Index)`, que remite a:

- `references/threat-modeling-and-boundaries.md` (274 líneas, secciones 2-8):
  threat model, activos y consecuencias, trust boundaries, mínimo privilegio,
  autorización vs. autenticación, blast radius, selecciones ambiguas/vacías.
- `references/secrets-and-state-integrity.md` (437 líneas, secciones 9-21):
  secretos, validación y exposición de secretos, archivos temporales, estado
  atómico, identidad confiada vs. estado cacheado, validación de input,
  inyección de shell, scripts generados como frontera de seguridad, supply
  chain, verificación de integridad, seguridad de recovery y backups.
- `references/availability-and-incident-response.md` (224 líneas, secciones
  22-25a) — **contiene el criterio de hand-off Revert-First vs.
  Investigate-First (§23a) y la escalera de acceso de tres peldaños (§25a)**,
  ambos citados por nombre desde `sre-engineering-mindset` §1b e
  `sre-incident-review`. Se marcó explícitamente en el índice como archivo de
  apertura prioritaria para decisiones de acceso o hand-off de incidente.
- `references/agentic-security.md` (254 líneas, secciones 26-34) — incluye
  **§33 Deterministic Exclusion Principle**, también citado desde
  `sre-engineering-mindset` §1b.
- `references/systemic-and-testing.md` (471 líneas, secciones 35-51):
  interacciones a nivel sistema, TOCTOU, replay, compromiso de dependencias,
  observabilidad de seguridad, estrategia y disciplina de testing de seguridad,
  preguntas de review, anti-patrones, seguridad vs. recovery, testing de
  artefactos de recovery generados, detección de cambios en supply chain,
  validación empírica, definición de terminado, y la pregunta final.

Verificado por grep: las 51 secciones numeradas (más 1a, 23a, 25a) aparecen
exactamente una vez entre el núcleo y los 5 archivos de referencia, sin pérdida
ni duplicación.

### Estado y pendientes que sobreviven a esta extensión

- Los cross-references por número de sección desde otros skills siguen siendo
  válidos (la numeración no cambió, solo el archivo físico donde vive cada
  sección) — misma advertencia de "confirmar con uso real" que en el piloto de
  mindset, ahora extendida a estos dos.
- Siguen sin resolver, y **no fueron tocados por este trabajo de
  restructuración**, las dos inconsistencias identificadas en la autoauditoría
  previa: (1) `sre-security §23a` usa "si dos o más de las señales anteriores
  están presentes" — un umbral numérico, contradiciendo la intención declarada
  de dar señales de juicio y no un algoritmo determinista; (2) esa misma
  sección nunca cita `sre-engineering-mindset §38` (humildad epistémica) como
  se había prometido, solo cita §1a. Ambas quedan como tarea abierta explícita.
- Aún no se decidió si extender el mismo patrón a los 6 skills restantes
  (`sre-observability`, `sre-release-deployment`, `sre-incident-review`,
  `sre-documentation`, `sre-testing`, `sre-slo`) — todos por debajo de las
  ~43KB de sre-observability, el más grande del grupo restante.
- Git para todo el proyecto, apuntando al GitHub del usuario, sigue
  explícitamente diferido al final de todo este esfuerzo.

---

## Cierre de las dos inconsistencias de la autoauditoría

Instrucción del usuario: "cerrar las dos inconsistencias pendientes y luego seguir
extendiendo el patrón."

**Inconsistencia 1 — `sre-security` §23a, umbral numérico y cita faltante.**
El texto decía "If two or more of the above are present" (un umbral numérico),
contradiciendo la intención declarada de dar señales de juicio y no un algoritmo
determinista; además nunca citaba `sre-engineering-mindset §38` (humildad
epistémica) como se había prometido. Se reescribió el criterio de hand-off para
ponderar señales por peso/relevancia en vez de contarlas ("a single signal...
already carries enough weight", "several of the weaker signals... can carry the
same weight... the reasoning is 'does the totality... make an active adversary
more likely'... not 'did we clear a fixed number of boxes'"), y se agregó la cita
explícita a §38 pidiendo declarar la confianza del juicio en vez de presentarlo
con la certeza de un chequeo de umbral. Archivo afectado:
`sre-security/references/availability-and-incident-response.md`.

**Inconsistencia 2 — `sre-engineering-mindset` §1b, forma de checklist.**
El formato "Step 1 — ... Step 6 —" en negrita era estructuralmente idéntico a un
checklist pese al propio disclaimer de "tratar esto como preguntas, no como
checklist". Se renombró la sección a "Six Questions for Consequential Automation"
y se reemplazaron los encabezados "Step N — Nombre" por preguntas con nombre
propio en prosa (p. ej. "Does this reduce toil, or just relocate it?"), y se
agregó una aclaración explícita de que el orden es solo para legibilidad, no una
tubería secuencial — que el razonamiento real va y viene entre las seis, igual
que el bucle de decisión SRE (§40) es un bucle y no una línea. Archivo afectado:
`sre-engineering-mindset/SKILL.md` §1b.

Ambas correcciones fueron verificadas por grep contra el archivo real ya escrito
en el proyecto.

---

## Extensión del patrón a los 4 skills restantes de mayor peso

Se aplicó el mismo patrón núcleo-liviano + `references/` a los cuatro skills que
quedaban por encima de las ~24KB, en orden descendente de tamaño.

### sre-release-deployment (42,637 → 5,626 bytes de núcleo)

**Hallazgo no buscado, corregido de paso**: como con `sre-security`, este archivo
tampoco tenía frontmatter YAML. Se agregó al escribir el nuevo núcleo.

Cinco archivos de referencia: `change-contract-and-rollout.md` (§2-16, 493
líneas), `rollback-and-recovery.md` (§17-24, 213 líneas),
`operational-controls-and-concurrency.md` (§25-36, 327 líneas),
`agentic-and-systemic.md` (§37-52, 417 líneas — incluye agent threat model, STPA,
y artefactos de recovery generados), `documentation-postmortems-and-review.md`
(§53-64, 475 líneas — incluye **§59a The Checklist Trap** y la Postmortem
Nutrition Loop). Verificado por grep: las 64 secciones (+1a, 59a) aparecen
exactamente una vez.

### sre-observability (42,113 → 5,400 bytes de núcleo)

Cinco archivos de referencia: `control-loop-and-state.md` (§3-16, 571 líneas —
incluye **§8a los seis términos de telemetría que no son sinónimos**),
`verification-and-signals.md` (§17-30, 345 líneas), `agentic-observability.md`
(§31-42, 344 líneas), `operations-and-generated-artifacts.md` (§43-58, 428
líneas — incluye la Postmortem Nutrition Loop), `review-and-final.md` (§59-67,
286 líneas). Verificado por grep: las 67 secciones (+2a, 8a) aparecen
exactamente una vez.

### sre-incident-review (40,524 → 6,406 bytes de núcleo)

**Hallazgo no buscado, corregido de paso**: tampoco tenía frontmatter YAML — se
agregó. Este skill usa un esquema de dos niveles (encabezados de categoría sin
número + subsecciones numeradas `## N.`), distinto de los demás; se preservó tal
cual.

Cinco archivos de referencia: `evidence-and-reconstruction.md` (§2-7a, 236
líneas — incluye **§7a el hand-off de seguridad, cross-referenciado desde
sre-security §23a**), `causal-and-systemic-analysis.md` (§8-17, 312 líneas),
`agentic-and-observability.md` (§18-29, 393 líneas), `corrective-actions-and-testing.md`
(§30-38, 253 líneas — incluye la Postmortem Nutrition Loop),
`ai-loop-and-review-procedure.md` (§39-46 + Final Principles + Meta-Rule, 398
líneas). Verificado por grep: las 46 secciones (+1a, 7a) aparecen exactamente
una vez.

### sre-documentation (35,736 → 8,857 bytes de núcleo)

Cinco archivos de referencia: `audience-readme-and-scope.md` (§4-10, 265
líneas), `verification-state-and-architecture.md` (§11-20, 263 líneas),
`agentic-security-and-idempotency.md` (§21-29, 260 líneas),
`artifacts-runbooks-incident-and-drift.md` (§30-37, 226 líneas),
`tradeoffs-review-learning-and-ownership.md` (§38-54, 542 líneas — el más
grande del grupo, agrupa deliberadamente varias categorías finales afines en
vez de forzar un sexto archivo). Verificado por grep: las 54 secciones (+3a,
3b) aparecen exactamente una vez.

### sre-testing y sre-slo: decisión de NO fragmentar (llamada de proporcionalidad)

`sre-testing` (21,561 bytes, 30 secciones) y `sre-slo` (23,627 bytes, 16
secciones) son, incluso sin tocar, más pequeños que un solo archivo de
referencia típico de los skills recién divididos (las referencias más grandes
rondan 400-570 líneas). Fragmentarlos en 5 archivos de referencia habría
significado partir un documento ya delgado en fragmentos de un puñado de
secciones cada uno — exactamente el tipo de maquinaria desproporcionada que
`sre-engineering-mindset §1a` advierte no aplicar cuando la señal de riesgo/
tamaño no lo amerita. Se optó por dejar ambos como archivo único, sin
`references/`, aplicando al empaquetado de los propios skills el mismo criterio
de proporcionalidad que los skills exigen para el código que revisan.

**Hallazgo no buscado, corregido de paso**: `sre-testing/SKILL.md` tampoco tenía
frontmatter YAML — se le agregó sin tocar el resto del contenido (verificado por
grep que las 30 secciones + 20a + 23a siguen intactas y sin duplicar).
`sre-slo/SKILL.md` ya tenía frontmatter correcto — sin cambios.

### Estado del hallazgo de frontmatter faltante

En total, de los 9 skills, **4 no tenían frontmatter YAML** (`sre-security`,
`sre-release-deployment`, `sre-incident-review`, `sre-testing`) — todos ya
corregidos en este trabajo. Esto no se había detectado en ninguna batería
adversarial anterior porque las baterías probaron contenido/razonamiento, no la
estructura de carga del skill en sí. Vale la pena que el usuario confirme si
esta ausencia afectaba el descubrimiento/activación real de estos 4 skills en la
práctica — no hay forma de validarlo empíricamente desde esta sesión.

### Pendientes que sobreviven a este lote

- Confirmar con uso real que los cross-references por número de sección siguen
  resolviendo correctamente ahora que 7 de los 9 skills están divididos en
  núcleo + referencias.
- El punto anterior de la retrospectiva sobre validación empírica (correr esto
  contra una sesión real de Claude con una tarea real) sigue sin resolverse.
- Git para todo el proyecto, apuntando al GitHub del usuario, sigue como el
  último paso explícitamente diferido.

---

## Publicación en GitHub

Repositorio: https://github.com/hnacimiento/dev-sre-skills (nombre elegido por
el usuario: `dev-sre-skills`). Licencia: MIT, sin cambios en el titular del
copyright. El usuario confirmó que ya hizo `git push` al repo remoto.

Se prepararon localmente antes del push: `README.md` (con el disclaimer de
"no validado empíricamente" visible al inicio, tabla de los 9 skills,
explicación del patrón núcleo+referencias, y los tres pilares del manifiesto),
`.gitignore` (mínimo: cruft de SO/editor), y `LICENSE` (MIT).

Los archivos `Prompt de prueba de skill.txt` que existían en algunos skills
fueron borrados por el usuario intencionalmente antes de la publicación
(confirmado explícitamente) — no forman parte del repo publicado.

Con esto se cierra el punto pendiente "Git para todo el proyecto, apuntando
al GitHub del usuario", que había quedado explícitamente diferido al final de
todo este esfuerzo.

### Lo que sigue sin resolver, fuera del alcance de este trabajo

- **Validación empírica real**: nada de lo hecho fue probado contra un agente
  real ejecutando una tarea real — todas las baterías adversariales fueron
  autocalificadas por el mismo modelo que escribió los skills.
- **Confirmar en uso real** que el frontmatter agregado a los 4 skills que no
  lo tenían (`sre-security`, `sre-release-deployment`, `sre-incident-review`,
  `sre-testing`) efectivamente mejora su detección/carga — no verificable
  desde esta sesión.

---

## Verificación contra la documentación oficial vigente y corrección de un llamado propio

El usuario compartió un documento de recursos sobre Claude Code Skills y pidió
navegar las URLs oficiales para ver si algo servía para mejorar el proyecto.
Se consultaron `code.claude.com/docs/en/skills` y
`platform.claude.com/docs/en/agents-and-tools/agent-skills/overview`
directamente (no de memoria), con los siguientes hallazgos concretos:

**Frontmatter: `name` no es obligatorio.** La documentación oficial dice
"All fields are optional. Only `description` is recommended". `name` sin
especificar cae al nombre del directorio. Esto corrige una afirmación mía
anterior: catalogar los 4 skills sin `name:` como un "defecto" fue impreciso
— probablemente no impedía su descubrimiento (el directorio ya se llamaba
igual que el `name` que le agregué). Quedan con `name:` explícito de todos
modos, lo cual es inofensivo y consistente con el resto del set, pero el
hallazgo debía corregirse para no sobre-reportar severidad.

**Límite oficial de tamaño: "Keep SKILL.md under 500 lines."** Este es el
hallazgo más importante y accionable de la revisión. No es una recomendación
de tokens aproximada — es una directiva explícita de la documentación oficial
de Claude Code. Se verificaron los 9 núcleos contra este límite:

| Skill | Líneas del núcleo | ¿Cumple? |
|---|---|---|
| sre-engineering-mindset | 304 | Sí |
| sre-bash | 143 | Sí |
| sre-security | 114 | Sí |
| sre-observability | 152 | Sí |
| sre-release-deployment | 166 | Sí |
| sre-incident-review | 159 | Sí |
| sre-documentation | 254 | Sí |
| sre-testing (antes de esta corrección) | 834 | **No — 67% por encima del límite** |
| sre-slo | 480 | Sí (al límite, sin margen) |

**Esto invalida mi decisión anterior de "proporcionalidad" de dejar
`sre-testing` como archivo único.** Ese razonamiento se apoyaba en el
principio interno del propio set (no aplicar más maquinaria de la que el
blast radius justifica), pero la guía oficial de la plataforma es un límite
distinto y no negociable sobre el archivo que se carga siempre — independiente
de cuánto contenido "merezca" tener. Se corrigió: `sre-testing` se dividió con
el mismo patrón núcleo+`references/` ya aplicado a los otros 7 skills.

### sre-testing (834 → 88 líneas de núcleo)

Cuatro archivos de referencia (menos que los 5 típicos, proporcional al
tamaño real del contenido): `contract-and-state-testing.md` (§2-7, 167
líneas), `execution-and-recovery-testing.md` (§8-14, 199 líneas),
`security-observability-and-boundary-testing.md` (§15-20a, 181 líneas),
`operator-agentic-and-final-review.md` (§21-30, 240 líneas — incluye **§23a
testing de interacciones, no solo de componentes**). Verificado por grep: las
30 secciones (+1a, 20a, 23a) aparecen exactamente una vez.

`sre-slo` (480 líneas) queda tal cual — cumple el límite oficial, aunque sin
margen. Si en el futuro crece aunque sea un poco, va a necesitar el mismo
tratamiento.

### Otros hallazgos de la revisión de documentación, sin acción requerida

- Límite de `description` (1,536 caracteres combinados con `when_to_use` en
  el listado): los 9 skills están muy por debajo (483-820 caracteres). Sin
  problema.
- Restricciones del campo `name` (máx. 64 caracteres, minúsculas/números/
  guiones, sin "anthropic"/"claude"): los 9 nombres (`sre-*`) cumplen sin
  cambios.
- La convención de nombrar la carpeta de material adicional `references/` es
  válida — la documentación no exige un nombre fijo (usa `reference.md`,
  `examples.md`, `scripts/` como ejemplos), solo que `SKILL.md` enlace a lo
  que corresponda.
- El estándar Agent Skills (agentskills.io) es efectivamente portable entre
  Claude Code, claude.ai y la API, confirmando que este set no queda atado a
  un solo producto — aunque para subir a claude.ai/API el frontmatter se
  restringe a seis campos (`name`, `description`, `license`, `compatibility`,
  `metadata`, `allowed-tools`); ninguno de los 9 skills usa campos fuera de
  ese conjunto, así que no hay migración pendiente ahí tampoco.

### Pendientes que sobreviven a esta revisión

- Los mismos de siempre: validación empírica real, y confirmar en uso que el
  enrutamiento a `references/` funciona como se espera.
- Nuevo, menor: si `sre-slo` vuelve a crecer, aplicar el mismo patrón antes de
  que supere las 500 líneas.
