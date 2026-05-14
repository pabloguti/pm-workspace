# Subdivisión de bundles — Plugins atómicos

**Fecha:** 2026-05-12
**Complementa:** `output/20260512-modularizacion-servicios-pm-workspace.md`

---

## Modelo de capas

```
Layer 0 — Foundation (compartido, dep obligatoria)
   └─ claude-code-foundations (rules + protocols + identity baseline)

Layer 1 — Plugins atómicos (40-60 unidades, cherry-pickeables)
   └─ Una skill ± uno o dos agentes muy acoplados

Layer 2 — Distros (combos predefinidos opcionales)
   └─ "SDD Python Pack", "Banking Compliance", "Document Inbox", ...

Layer 3 — Workspace completo (Savia interno)
   └─ pm-workspace = distro Layer 3 con identidad Savia
```

Un proyecto que solo necesita ingesta de PDFs instala 1 plugin (~5 MB) + foundations. No se carga el ecosistema entero.

---

## Layer 0 — Foundation (extraer primero, obligatorio para el resto)

| Plugin | Contenido | Por qué compartido |
|---|---|---|
| `cc-foundations` | `radical-honesty.md`, `autonomous-safety.md`, `agent-notes-protocol.md`, `context-placement-confirmation.md`, helpers de output (`output/YYYYMMDD-tipo.ext`) | Todos los demás plugins referencian estos contratos |
| `cc-judge-protocol` | Contrato JSON de salida de jueces, score aggregation, veto rules | Court + Tribunal + cualquier multi-judge futuro |
| `cc-digest-pipeline` | Framework de 4 fases (extract → structure → enrich → output) | Todos los `*-digest` lo usan |
| `cc-orchestration-base` | Patrón Plan/ExitPlan + DAG scheduling primitives | SDD core + court + tribunal |

Sin esto cada plugin replicaría los mismos contratos. **Sacar antes que cualquier otro bundle.**

---

## Subdivisión bundle por bundle

### 1. `sdd-core` → 4 sub-bundles + 12 language packs

**Esto es el cambio más importante.** Hoy un proyecto Python carga developers de Java, Ruby, COBOL... que no usa. Solución: language packs separados.

| Sub-bundle | Componentes | Cuándo se instala |
|---|---|---|
| `sdd-engine` | `spec-driven-development` skill, `sdd-spec-writer`, `dev-orchestrator`, `architect`, `business-analyst`, `fix-assigner` | Siempre que quieras SDD |
| `sdd-extensions` | `dag-scheduling`, `tdd-vertical-slices`, `feasibility-probe`, `code-comprehension-report`, `context-optimized-dev`, `smart-routing` | Opcional, suma capacidades |
| `sdd-quality-gates` | `coherence-validator`, hooks de scope-guard | Recomendado, no obligatorio |
| `sdd-codemaps` | `agent-code-map`, `human-code-map`, `codebase-map` | Si trabajas con repos legacy |

**Language packs** (1 por idioma, cherry-pick):
- `sdd-lang-dotnet` · `sdd-lang-python` · `sdd-lang-typescript` · `sdd-lang-java` · `sdd-lang-go` · `sdd-lang-rust` · `sdd-lang-php` · `sdd-lang-ruby` · `sdd-lang-cobol`
- `sdd-lang-frontend` (Angular + React)
- `sdd-lang-mobile` (Swift + Kotlin + Flutter)
- `sdd-lang-iac` (Terraform)

Cada language pack = 1 agente + sus reglas (`.claude/rules/languages/{lang}.md`) + tests específicos. Tamaño mínimo, instalación opt-in.

---

### 2. `code-review-court` → 3 sub-bundles

| Sub-bundle | Componentes | Para qué |
|---|---|---|
| `court-core` | `court-orchestrator` + `correctness-judge` + `spec-judge` + skill `court-review` | Court mínimo viable |
| `court-judges-extra` | `architecture-judge`, `cognitive-judge`, `security-judge` | Cherry-pick por preocupación |
| `court-external-judge` | `pr-agent-judge` (qodo-ai wrapper) | Opt-in 5º juez externo |
| `code-reviewer-standalone` | Agente `code-reviewer` sin court | Para revisión simple sin tribunal |

Un team que solo quiere "correctness + security" instala `court-core` + un juez de `court-judges-extra`. Sin pagar overhead de los demás.

---

### 3. `truth-tribunal` → 3 sub-bundles

| Sub-bundle | Componentes | Para qué |
|---|---|---|
| `tribunal-core` | `truth-tribunal-orchestrator` + `coherence-judge` + `factuality-judge` + `hallucination-judge` + `completeness-judge` | Tribunal mínimo para validar reports |
| `tribunal-domain` | `calibration-judge`, `compliance-judge`, `source-traceability-judge` | Verticales con citas obligadas o PII |
| `validators-light` | `coherence-validator`, `reflection-validator` (sin tribunal completo) | Validación ligera embebida en otros flujos |

---

### 4. `security-pipeline` → 5 sub-bundles

| Sub-bundle | Componentes | Para qué |
|---|---|---|
| `sec-redteam` | `security-attacker`, `nuclei-scanning` skill | Solo ataque |
| `sec-blueteam` | `security-defender` | Solo remediación |
| `sec-auditor` | `security-auditor` | Independiente, audita Red/Blue |
| `sec-pentester` | `pentester` (pipeline Shannon 5 fases) | Pentest dinámico contra running systems |
| `sec-precommit` | `security-guardian` + `commit-guardian` | Gate antes de commit |

Cada uno es útil aislado. Un equipo que solo quiere "no commitear secrets" instala `sec-precommit` (1 agente + 1 hook).

---

### 5. `document-digest` → 6 sub-bundles + framework

| Sub-bundle | Componente único | Tamaño |
|---|---|---|
| `digest-pdf` | `pdf-digest` | Pequeño |
| `digest-excel` | `excel-digest` | Pequeño |
| `digest-pptx` | `pptx-digest` | Pequeño |
| `digest-word` | `word-digest` | Pequeño |
| `digest-visual` | `visual-digest` (OCR pizarras/diagramas) | Mediano (deps OCR) |
| `digest-meeting` | `meeting-digest` + `meeting-risk-analyst` + `meeting-confidentiality-judge` | Mediano (3 agentes) |
| `digest-voice` | skill `voice-inbox` (audio→texto) | Pequeño |

Todos dependen de `cc-digest-pipeline` (foundation). Un proyecto que solo recibe PDFs instala `digest-pdf` (~50 KB de prompts).

---

### 6. `architecture-intelligence` → 4 sub-bundles

| Sub-bundle | Componentes | Para qué |
|---|---|---|
| `arch-codemaps` | `agent-code-map`, `human-code-map`, `codebase-map` | Mapas estructurales persistentes |
| `arch-ast-tools` | `ast-comprehension`, `ast-quality-gate` | Análisis estructural sin leer ficheros enteros |
| `arch-diagrams` | `diagram-generation`, `diagram-import`, `diagram-architect` | Mermaid + ingest de diagramas existentes |
| `arch-drift` | `drift-auditor`, skill `architecture-intelligence` | Detección de drift docs↔código |

---

### 7. `testing-arsenal` → 5 sub-bundles

| Sub-bundle | Componentes | Para qué |
|---|---|---|
| `test-architect-pack` | skill + agente `test-architect` | Diseño de suites (16 lenguajes × 14 tipos) |
| `test-runners` | `test-runner`, `frontend-test-runner` | Ejecución post-commit |
| `test-e2e-web` | `web-e2e-tester` | Tests E2E web autónomos |
| `test-e2e-android` | `android-autonomous-debugger` + skill | E2E Android vía ADB |
| `test-mutation` | skill `mutation-audit` | Calidad real de tests |
| `test-visual-qa` | `visual-qa-agent` | Screenshots + wireframe comparison |

---

### 8. `agile-pm-azure` → 5 sub-bundles

Aquí la subdivisión es **crítica** porque hoy es monolítico y muy acoplado a Savia.

| Sub-bundle | Componentes | Para qué |
|---|---|---|
| `azdo-operator` | `azure-devops-operator` + skill `azure-devops-queries` | Solo operaciones WIQL/work items |
| `azdo-pipelines` | skill `azure-pipelines` | CI/CD via MCP |
| `pm-sprints` | `sprint-management`, `capacity-planning` | Cadencia ágil agnóstica del tracker |
| `pm-backlog` | `pbi-decomposition`, `backlog-git-tracker`, `rules-traceability` | Grooming + traceability |
| `pm-team` | `team-coordination`, `team-onboarding`, `smart-calendar` | Coord. equipo |

Idea: `pm-sprints` y `pm-backlog` son agnósticos del tracker; `azdo-operator` es el adaptador Azure. Un equipo en Jira instala `pm-sprints` + un adaptador `jira-operator` futuro.

---

### 9. `governance-compliance` → 4 sub-bundles

| Sub-bundle | Componentes | Para qué |
|---|---|---|
| `gov-regulatory` | skill `regulatory-compliance` (framework agnóstico, multi-sector) | Compliance genérico |
| `gov-legal-es` | skill + agente `legal-compliance` (legalize-es) | Vertical España |
| `gov-rbac` | skill `rbac-management` | Roles y permisos |
| `gov-enterprise` | skill `governance-enterprise` (audit trail, decision registry) | Trazabilidad enterprise |

`gov-legal-es` se vende como opt-in regional; no obligas a un proyecto francés a cargar legalize-es.

---

### 10. `observability-reporting` → 5 sub-bundles

| Sub-bundle | Componentes | Para qué |
|---|---|---|
| `obs-flow-metrics` | flow-metrics, dora, board-flow (skills + comandos) | DORA + Savia Flow agnostic |
| `obs-cost` | `cost-management`, `time-tracking-report` | Tracking horas/costes |
| `obs-exec-reports` | `executive-reporting`, `enterprise-analytics` | Informes ejecutivos |
| `obs-evals` | `evaluations-framework`, `skill-evaluation` | Evals de outputs LLM |
| `obs-dx` | `developer-experience` (SPACE + Core4) | Salud del equipo dev |

---

### 11. `memory-context` → 4 sub-bundles

| Sub-bundle | Componentes | Para qué |
|---|---|---|
| `mem-core` | `memory-agent` + memory-store scripts | Capa de persistencia mínima |
| `mem-rag` | `reranker`, `topic-cluster` (BERTopic) | Recuperación de calidad |
| `mem-knowledge-graph` | skill `knowledge-graph` | Grafo de entidades |
| `ctx-engineering` | `context-rot-strategy`, `context-caching`, `context-task-classifier`, `prompt-optimizer` | Optimización del propio contexto Claude |

---

### 12. `meta-workspace-tools` → 4 sub-bundles

| Sub-bundle | Componentes | Para qué |
|---|---|---|
| `ws-doctor` | `workspace-integrity`, `drift-auditor`, scripts de health | Diagnóstico de la propia instalación |
| `ws-model-upgrade` | skill + agente `model-upgrade-audit` | Detectar workarounds obsoletos al subir de modelo |
| `ws-mcp-tools` | `mcp-recommend`, `mcp-browse` | Descubrimiento de MCP servers |
| `ws-commit-guard` | `commit-guardian` (sin sec, solo workspace rules) | Verifica reglas pre-commit |

---

### 13. `sovereignty-shield` → 7 sub-bundles (el más fragmentable)

Este es el bundle donde la subdivisión más se nota — cada pieza vale por sí sola y resuelve un caso diferente.

| Sub-bundle | Componentes | Caso de uso aislado |
|---|---|---|
| `shield-pii-filter` | `savia-shield` (renombrado `cc-shield`) + scripts de filtrado | Proyecto que sube logs a repo público y debe filtrar PII/IPs |
| `shield-dual-inference` | `savia-dual` (renombrado `dual-llm`) + proxy 127.0.0.1:8787 | Empresas con red inestable o que quieren cloud+local transparente |
| `shield-emergency-llm` | `emergency-mode` (renombrado `llm-failover`) | Cualquier setup que sufre outages de Anthropic |
| `shield-vault` | `personal-vault` (renombrado `local-vault`) + comandos vault-* | Almacenamiento soberano de preferencias/cache de un humano |
| `shield-memory-backup` | `memvid-backup` (renombrado `mem-backup`) | Backup portable de memoria con integrity SHA256 |
| `shield-sovereignty-audit` | `sovereignty-auditor` | One-shot: ¿cuánto lock-in tengo? Plan de mitigación |
| `shield-confidentiality` | `confidentiality-auditor` + `security-guardian` + `meeting-confidentiality-judge` | Gate de fugas pre-PR y post-meeting |
| `shield-wellbeing` (opt-in) | `wellbeing-guardian` | Sistema proactivo de burnout-radar |

Un banco quiere `shield-pii-filter` + `shield-confidentiality`. Un nómada digital quiere `shield-dual-inference` + `shield-emergency-llm`. No tienen por qué ser el mismo paquete.

---

## Distros recomendadas (Layer 2)

Combos predefinidos para casos de uso típicos. Cada uno = manifest YAML que declara dependencias entre plugins Layer 1.

| Distro | Componentes incluidos | Cliente típico |
|---|---|---|
| `distro-startup-python` | foundations + sdd-engine + sdd-lang-python + court-core + digest-pdf + sec-precommit | Startup pequeña con un solo lenguaje |
| `distro-fullstack-ts` | foundations + sdd-engine + sdd-lang-typescript + sdd-lang-frontend + test-runners + test-e2e-web + court-core | Equipo fullstack TS |
| `distro-banking` | foundations + sdd-engine + sdd-lang-java + court-core + gov-regulatory + gov-rbac + shield-pii-filter + shield-confidentiality + sec-pentester | Sector bancario regulado |
| `distro-document-inbox` | foundations + digest-pdf + digest-excel + digest-word + digest-meeting + digest-voice + mem-core | Asistente de ingesta de docs |
| `distro-research-lab` | foundations + tribunal-core + tribunal-domain + mem-rag + mem-knowledge-graph + ctx-engineering + obs-evals | Equipo de investigación con citas obligadas |
| `distro-sovereignty` | foundations + shield-* (todos) | Cliente que prioriza soberanía sobre features |
| `distro-pm-azure-classic` | foundations + azdo-operator + pm-sprints + pm-backlog + pm-team + obs-flow-metrics | Equivalente actual de pm-workspace básico |
| `distro-savia-internal` | TODO + identidad Savia + savia-school + savia-hub | Lo que es hoy pm-workspace |

---

## Reglas de packaging

Para que esto funcione necesitas un manifest estándar por plugin:

```yaml
# plugin.yaml
name: digest-pdf
version: 1.0.0
layer: 1
depends_on:
  foundations: ">=1.0"
  cc-digest-pipeline: ">=1.0"
provides:
  agents: [pdf-digest]
  skills: [pdf-digest]
  commands: [pdf-digest, pdf-extract]
  hooks: []
conflicts_with: []
optional_extensions:
  - shield-confidentiality  # añade gate de PII al pipeline
```

Y un instalador que resuelva DAG (similar a apt/npm). Ya tienes `install.sh` — habría que parametrizarlo por plugin.

---

## Métricas para decidir granularidad

No subdividir por subdividir. Una pieza merece ser plugin atómico si cumple **≥2 de 4**:

1. **Tamaño relevante**: ≥1 agente o ≥1 skill con `context_cost: medium/high`.
2. **Uso aislado real**: hay al menos un caso de uso donde se usa sin las otras del bundle.
3. **Dependencia controlada**: depende solo de foundations + ≤2 plugins.
4. **Riesgo de imposición**: si se instala obligatoriamente carga coste a proyectos que no lo necesitan.

Ejemplo aplicado:
- `pdf-digest` → cumple los 4. Plugin atómico. ✅
- `coherence-judge` → cumple 3 y 4, no 1 (es solo un juez). Mejor dentro de `tribunal-core`. ❌ atómico.
- `sdd-lang-cobol` → cumple 1, 2, 4. Plugin atómico. ✅
- `caveman` skill → cumple 4 pero no 1, 2, 3 (es identidad Savia). No extraer. ❌

---

## Recomendación de implementación (radical honesty)

Subdividir los 13 bundles en ~50 plugins atómicos suena bien sobre el papel pero es trampa: te genera 50 manifest YAMLs, 50 ciclos de release, 50 README, 50 versiones que mantener.

**Empieza fino:**
1. Saca `cc-foundations` primero (1 plugin Layer 0).
2. Saca `digest-pdf` como **proof of concept** del modelo atómico — 1 agente, 1 skill, 1 comando. Si esto funciona limpio, replicas el patrón.
3. Saca **2-3 distros** simultáneamente (`distro-document-inbox`, `distro-sovereignty`) — empaquetando los atómicos como manifest. La distro es lo que un cliente instala; los atómicos son la cocina.
4. Mide adopción real antes de subdividir el resto. Si nadie pide `sdd-lang-ruby` por separado, no inviertas en sacarlo del bundle.

**Lo que no haría:**
- Subdividir `sdd-core` por idioma desde el día 1. Mantener `sdd-engine` + un único pack de developers al principio. Separar idiomas cuando haya señal (un cliente Python que se queja del peso de COBOL).
- Sacar `agile-pm-azure` en sub-bundles antes que el resto. Es el más acoplado y no es la pieza con mejor ROI externo.

La granularidad correcta no es la máxima — es la que reduce fricción de adopción sin explotar el coste de mantenimiento.
