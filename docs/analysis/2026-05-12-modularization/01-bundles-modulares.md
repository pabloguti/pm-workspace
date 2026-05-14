# Modularización de servicios — pm-workspace

**Fecha:** 2026-05-12
**Rama:** `claude/analyze-modularize-services-imOhD`
**Alcance:** Skills (91) + Agentes (65) de `.claude/` candidatos a empaquetarse como plugins independientes.

---

## Resumen ejecutivo

pm-workspace v2.24 declara 45 skills / 33 agentes en `plugin.json`, pero el filesystem tiene **91 skills y 65 agentes**. El monolito hace dos cosas que conviene separar: (a) **el motor SDD + Court + Tribunal** (reutilizable en cualquier empresa) y (b) **operaciones PM Savia-specific** (Azure DevOps, Jira, sprints, Savia Hub).

Propongo **13 bundles extraíbles**. Los de mayor ROI:
- **SDD Core** (bundle 1) — pipeline reutilizable, 12 developers × 16 lenguajes
- **Code Review Court** (bundle 2) y **Truth Tribunal** (bundle 3) — multi-juez, ortogonales
- **Sovereignty Shield** (bundle 13) — la pieza más diferenciadora: filtrado PII, failover local, vault, emergencia, soberanía. Ver §13.

**Lo que NO se debería sacar:** `personal-vault`, `savia-dual`, `savia-school`, `savia-hub-sync`, `savia-flow-practice`, `emergency-mode`, `company-messaging` — están atados a la identidad/infra Savia y romperían si se aíslan.

---

## Bundles propuestos

### 1. `sdd-core` — Spec-Driven Development (alta prioridad)

**Qué es:** El pipeline ejecutable spec → developer → test → review. Es el núcleo reutilizable.

| Tipo | Componentes |
|---|---|
| Skills | `spec-driven-development`, `context-optimized-dev`, `tdd-vertical-slices`, `dag-scheduling`, `code-comprehension-report`, `feasibility-probe`, `smart-routing` |
| Agentes orquestación | `sdd-spec-writer`, `dev-orchestrator`, `architect`, `business-analyst`, `fix-assigner` |
| Agentes developers (16 lenguajes) | `dotnet-developer`, `python-developer`, `typescript-developer`, `java-developer`, `go-developer`, `rust-developer`, `php-developer`, `ruby-developer`, `frontend-developer`, `mobile-developer`, `cobol-developer`, `terraform-developer` |

**Por qué sacarlo:** Cualquier empresa con Claude Code + un backlog puede usarlo. Es lo más vendible y lo que más independencia tiene del resto.
**Dependencias internas a romper:** algunos developers referencian `agent-notes-protocol.md` y `agent-teams-sdd.md` — empaquetar ambos en `docs/` del plugin.

---

### 2. `code-review-court` — Tribunal de revisión de código (alta prioridad)

**Qué es:** Quality gate multi-juez para PRs. Output `.review.crc` estructurado.

| Tipo | Componentes |
|---|---|
| Skill | `court-review`, `pr-agent-judge` |
| Orquestador | `court-orchestrator` |
| Jueces | `architecture-judge`, `cognitive-judge`, `correctness-judge`, `security-judge`, `spec-judge`, `pr-agent-judge` (5º opt-in vía qodo-ai) |
| Agente de fixes | `code-reviewer`, `fix-assigner` |

**Por qué sacarlo:** Pieza independiente, sin estado compartido con SDD salvo entrada (spec). Puede invocarse standalone sobre cualquier branch o PR.

---

### 3. `truth-tribunal` — Validación de informes/specs (alta prioridad)

**Qué es:** 7 jueces que verifican confiabilidad de outputs (specs, reports, código).

| Tipo | Componentes |
|---|---|
| Orquestador | `truth-tribunal-orchestrator` |
| Jueces | `calibration-judge`, `coherence-judge`, `completeness-judge`, `compliance-judge`, `factuality-judge`, `hallucination-judge`, `source-traceability-judge` |
| Validadores complementarios | `coherence-validator`, `reflection-validator` |
| Skill | `verification-lattice`, `consensus-validation`, `reflection-validation` |

**Por qué sacarlo:** Útil para cualquier dominio (legal, médico, financiero) donde haga falta evaluar veracidad de outputs LLM. Hoy vive mezclado con SDD pero es ortogonal.

---

### 4. `security-pipeline` — Adversarial Red/Blue/Audit

| Tipo | Componentes |
|---|---|
| Skills | `adversarial-security`, `pentesting`, `nuclei-scanning`, `security-auto-remediation` (en `.claude/skills/` ver hooks) |
| Agentes | `security-attacker`, `security-defender`, `security-auditor`, `security-guardian`, `pentester`, `confidentiality-auditor` |

**Por qué sacarlo:** Pipeline auto-contenido (3 fases + auditor). Aplica OWASP/CWE genérico. Lo único Savia-specific es `confidentiality-auditor`, que descubre datos dinámicamente.

---

### 5. `document-digest` — Ingesta de documentos

| Tipo | Componentes |
|---|---|
| Agentes | `excel-digest`, `pdf-digest`, `pptx-digest`, `word-digest`, `visual-digest`, `meeting-digest`, `meeting-risk-analyst`, `meeting-confidentiality-judge` |
| Skill | `voice-inbox` (audio→texto) |

**Por qué sacarlo:** Pipeline puro de ingestión (4 fases). Salida: contexto vivo. Cualquier asistente que reciba ficheros del cliente lo necesita.

---

### 6. `architecture-intelligence` — Análisis de arquitectura y deuda

| Tipo | Componentes |
|---|---|
| Skills | `architecture-intelligence`, `agent-code-map`, `human-code-map`, `codebase-map`, `ast-comprehension`, `ast-quality-gate`, `diagram-generation`, `diagram-import` |
| Agentes | `diagram-architect`, `drift-auditor` |

**Por qué sacarlo:** Indexación + diagramas + fitness functions. Reutilizable en cualquier monorepo. Hoy mezclado con SDD pero no lo necesita.

---

### 7. `testing-arsenal` — Test architect + runners + visual QA

| Tipo | Componentes |
|---|---|
| Skill | `test-architect`, `mutation-audit` |
| Agentes | `test-architect`, `test-engineer`, `test-runner`, `frontend-test-runner`, `web-e2e-tester`, `visual-qa-agent`, `android-autonomous-debugger` |

**Por qué sacarlo:** Independiente de SDD. Cubre 16 lenguajes × 14 tipos de test. Modular si SDD se separa.

---

### 8. `agile-pm-azure` — Operaciones PM (Savia-flavor pero parametrizable)

| Tipo | Componentes |
|---|---|
| Skills | `azure-devops-queries`, `azure-pipelines`, `sprint-management`, `capacity-planning`, `pbi-decomposition`, `backlog-git-tracker`, `smart-calendar`, `team-coordination`, `team-onboarding`, `rules-traceability` |
| Agente | `azure-devops-operator` |

**Por qué sacarlo:** Es el core "PM con Azure DevOps". Sale como producto independiente si se parametriza el PAT path y las constantes Savia.
**Atención:** depende de `pm-config.md` — extraer plantilla genérica.

---

### 9. `governance-compliance` — Legal, RGPD, regulatorio

| Tipo | Componentes |
|---|---|
| Skills | `legal-compliance`, `regulatory-compliance`, `governance-enterprise`, `rbac-management`, `aepd-compliance` (comando), `compliance-matrix` |
| Agentes | `legal-compliance`, `compliance-judge` (también vive en truth-tribunal) |

**Por qué sacarlo:** Mercado vertical claro (regulated industries). Hoy escondido entre quality skills.
**Cuidado:** `legalize-es` (legislación española consolidada) es contenido externo — referencia, no embeber.

---

### 10. `observability-reporting` — Métricas y reporting ejecutivo

| Tipo | Componentes |
|---|---|
| Skills | `enterprise-analytics`, `executive-reporting`, `cost-management`, `evaluations-framework`, `skill-evaluation`, `time-tracking-report`, `developer-experience` |
| Hooks/scripts | flow-metrics, board-flow, kpi-* (en `.claude/commands/`) |

**Por qué sacarlo:** SPACE/DORA/Core4 son frameworks abiertos. Empaquetar como "AI-powered SPACE dashboard" tiene sentido comercial.

---

### 11. `memory-context` — Capa de memoria + RAG

| Tipo | Componentes |
|---|---|
| Skills | `memvid-backup`, `reranker`, `topic-cluster`, `knowledge-graph`, `context-caching`, `context-rot-strategy`, `context-task-classifier`, `prompt-optimizer` |
| Agente | `memory-agent` |

**Por qué sacarlo:** Componente transversal. Aplicable a cualquier asistente Claude que necesite memoria persistente, no solo PM.

---

### 12. `meta-workspace-tools` — Auditoría de la propia instalación

| Tipo | Componentes |
|---|---|
| Skills | `workspace-integrity`, `model-upgrade-audit`, `doc-quality-feedback`, `skill-evaluation`, `scaling-operations`, `tier3-probes`, `mcp-recommend` |
| Agentes | `model-upgrade-auditor`, `commit-guardian` |

**Por qué sacarlo:** Útil para cualquiera que mantenga un workspace Claude Code grande. Vendible como "Claude Code workspace doctor".

---

### 13. `sovereignty-shield` — Defensa, soberanía y resiliencia (NUEVO)

**Qué es:** El "escudo" transversal. Funcionalidades defensivas — filtrado de datos sensibles, soberanía de inferencia, vault personal, modo emergencia, auditoría de lock-in, salud del operador. Hoy disperso entre skills, comandos y scripts; tiene sentido empaquetarlo como un solo plugin "Shield".

| Tipo | Componentes | Función |
|---|---|---|
| Skill | `savia-shield` (cmd + scripts/savia-shield-*.sh) | Filtrado de credenciales, IPs y PII antes de que lleguen a ficheros públicos. Toggle on/off |
| Skill | `sovereignty-auditor` | Diagnostica lock-in de IA, portabilidad de datos, score de soberanía |
| Skill | `savia-dual` | Failover transparente Anthropic ↔ gemma4 local (proxy 127.0.0.1:8787) |
| Skill | `emergency-mode` | Switchover a LocalAI cuando Anthropic está caído |
| Skill | `personal-vault` | Vault N3 del usuario — perfil, preferencias, memoria, cache |
| Skill | `memvid-backup` | Backup portable de memoria externa con integrity SHA256 |
| Skill | `wellbeing-guardian` | Sistema proactivo de bienestar individual (señales burnout) |
| Skill | `confidentiality-check` (cmd) + `confidentiality-auditor` (agente) | Auditoría pre-PR de fugas de datos |
| Agente | `security-guardian` | Quality gate de seguridad pre-commit |
| Agente | `meeting-confidentiality-judge` | Valida que datos confidenciales no se filtren tras digestión de reuniones |
| Comandos | `vault-init`, `vault-export`, `vault-restore`, `vault-status`, `vault-sync`, `sovereignty-audit`, `emergency-plan`, `aepd-compliance` | API operativa |

**Por qué sacarlo:** Es la pieza más diferenciadora frente a otros plugins Claude Code. "AI Sovereignty Shield" tiene un mercado claro:
- Empresas con datos sensibles (banca, salud, legal)
- Equipos en geografías con disponibilidad inestable (failover local)
- Compradores que valoran portabilidad y anti-lock-in
- Compliance RGPD/AEPD (filtrado pre-output)

**Cohesión interna:** Todas las piezas comparten un patrón: *defensa en profundidad sobre el flujo IA*. Entrada → filtrado (savia-shield) → inferencia con failover (savia-dual / emergency-mode) → output auditado (confidentiality-auditor, security-guardian) → almacenamiento soberano (personal-vault, memvid-backup) → auditoría continua (sovereignty-auditor, wellbeing-guardian).

**Dependencias a romper antes de extraer:**
1. **Nombre `savia-*`** → renombrar a genérico: `shield`, `dual-inference`, `vault`, `emergency-llm`. Mantener compatibilidad vía alias.
2. **`pm-config.md` constantes** → exponer como variables de entorno del plugin (`SHIELD_ENABLED`, `DUAL_PROXY_PORT`, `VAULT_PATH`, `LOCAL_LLM_MODEL`).
3. **`legalize-es` (AEPD)** → opcional, vertical España; exponer hook para otras jurisdicciones.
4. **Hooks `.claude/settings.json`** → empaquetar los 4-5 hooks que requiere (pre-commit gitleaks, post-tool shield filter, session-start sovereignty check).
5. **Identidad Savia (femenino, búho)** → desacoplar; el plugin debe ser white-label.

**Prioridad de extracción:** ALTA. Es el bundle con mayor diferencia competitiva. Probablemente el primero que debería sacarse junto con `truth-tribunal` y `document-digest`.

---

## NO modularizar (anclado a Savia, no recuperable sin reescritura)

| Componente | Razón |
|---|---|
| `savia-school` | Vertical educativo Savia con cifrado de rúbricas — requiere reescritura completa para genérico |
| `savia-hub-sync` | Orquestación del repo SaviaHub privado — infra específica |
| `savia-flow-practice` | Práctica del método propio Savia Flow — IP de Savia |
| `company-messaging` | Sistema branch-based v3 acoplado a SaviaHub |
| `client-profile-manager` | CRUD perfiles cliente SaviaHub |
| `voice-inbox` (parcial) | Pipeline audio→Savia inbox — el OCR es genérico (va a `document-digest`), la inbox no |
| `caveman`, `grill-me`, `zoom-out` | Modos de personalidad Savia (perfil savia.md) |

> **Nota importante:** En la primera versión de este informe había clasificado `savia-dual`, `emergency-mode`, `personal-vault`, `memvid-backup`, `wellbeing-guardian` como NO modularizables. **Corrijo:** son extraíbles dentro del bundle `sovereignty-shield` (bundle 13) si se renombran y desacoplan del namespace Savia. Mantenerlos en core era pérdida de oportunidad.

---

## Dependencias cruzadas a romper antes de extraer

1. **`agent-notes-protocol.md`** + **`agent-teams-sdd.md`** — referenciados por developers + Court + Tribunal. Cada plugin que los necesite debe llevar su copia en `docs/`.
2. **`pm-config.md`** — constantes (PATs, paths, autonomous reviewer). Cada plugin define las suyas.
3. **`AGENTS.md`** — auto-regenerado. Cada plugin lo regenera para su scope.
4. **Rules `docs/rules/domain/`** — `radical-honesty`, `autonomous-safety`, `context-placement-confirmation` son transversales: candidates a un plugin `claude-code-policies` separado, o convertirse en submódulo git.
5. **Hooks** (`.claude/settings.json`) — 61 hooks/65 registros. Cada bundle declara los suyos en su propio `settings.json` parcial.

---

## Plan de extracción sugerido (orden de menor riesgo)

1. **`document-digest`** — pipeline puro, sin estado, sin dependencias. *Sprint 1.*
2. **`truth-tribunal`** — ya tiene orquestador propio. *Sprint 1.*
3. **`code-review-court`** — análogo, opt-in del 5º juez. *Sprint 2.*
4. **`security-pipeline`** — Red/Blue/Audit. *Sprint 2.*
5. **`testing-arsenal`** — depende sólo de proyectos diana. *Sprint 3.*
6. **`memory-context`** — transversal pero auto-contenido. *Sprint 3.*
7. **`sovereignty-shield`** — alta diferenciación comercial; sacar pronto tras validar patrón. *Sprint 2-3.*
8. **`sdd-core`** — el más grande; sacarlo después de validar que 1-7 conviven. *Sprint 4-5.*
9. **`architecture-intelligence`** — ortogonal a SDD. *Sprint 4.*
10. **`governance-compliance`** — vertical claro. *Sprint 5.*
11. **`observability-reporting`** — depende de datos PM. *Sprint 6.*
12. **`agile-pm-azure`** — el más acoplado a Savia; sacar último o mantener interno. *Sprint 6-7.*
13. **`meta-workspace-tools`** — útil internamente, puede quedarse en core. *Opcional.*

---

## Métricas para decidir

Antes de sacar cada bundle, medir:
- **Imports externos**: ¿cuántos paths fuera del bundle referencia? (`grep` sobre paths absolutos)
- **Hooks asociados**: ¿qué hooks de `.claude/settings.json` necesita?
- **Tests existentes**: ¿hay tests en `tests/`? (mover juntos)
- **Comandos en `.claude/commands/` que invocan el bundle**: mover también.
- **Documentación en `docs/`**: candidates a viajar con el bundle.

---

## Recomendación final (radical honesty)

Sacar los 13 bundles a la vez es un error de scope. **Empieza por `document-digest` + `truth-tribunal`** — son los más limpios, validan el patrón de extracción, y te dan un par de plugins publicables en marketplace en 1-2 sprints.

Después, antes del SDD, sacar **`sovereignty-shield`**. Es el bundle con mayor narrativa diferenciada (datos seguros + soberanía + resiliencia) y el que más interés genera fuera de Savia hoy — empresas reguladas y equipos que han sufrido outages de Anthropic preguntan por esto, no por "otra herramienta de PM".

El bundle más grande es **`sdd-core`** (12 developers × 16 lenguajes), pero también el más arriesgado. Sácalo cuando los tres primeros (digest + tribunal + shield) hayan estabilizado el patrón de packaging.

**Lo que NO conviene:** intentar modularizar `agile-pm-azure` antes que el resto. Es el bundle más acoplado al stack Savia (PAT paths, Savia Flow, Savia Hub) y desviar esfuerzo ahí retrasa los plugins comercialmente valiosos.
