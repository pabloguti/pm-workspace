# Coste de extracción — Primera ola

**Fecha:** 2026-05-12
**Alcance:** Primera ola de extracción según `output/20260512-subdivision-plugins-atomicos.md`
**Unidad:** Horas ingeniero IA-assisted (humano dirige + Claude Code implementa).

---

## Supuestos

- 1 FTE = ~32h/semana de trabajo efectivo (no 40h — descontando reuniones, contexto, revisión humana).
- IA-assisted reduce ~40% el tiempo de boilerplate y extracción, pero no el de diseño ni el de validación humana (Regla #8: Code Review E1 SIEMPRE humano).
- Coste hora ingeniero senior España 2026: 45-65 €/h (interno) · 80-120 €/h (consultoría).
- No incluye: marketing, marketplace fees, licensing review legal.

---

## Coste por componente

### Phase 1 — Infra base (bloqueante)

| Item | Esfuerzo (h) | Notas |
|---|---|---|
| Diseño schema `plugin.yaml` + validador JSON Schema | 4-8 | Decisión arquitectónica, no delegable |
| Empaquetado de `cc-foundations` (rules + protocols) | 6-10 | Extracción mecánica, IA-assisted alto |
| `cc-judge-protocol` (contrato JSON jueces + scoring) | 8-12 | Reutiliza patrón Court actual |
| `cc-digest-pipeline` (framework 4 fases) | 8-12 | Reutiliza pdf/excel/word actuales |
| `cc-orchestration-base` (DAG primitives) | 12-20 | Más nuevo, requiere diseño |
| Install/uninstall con resolución de dependencias | 16-28 | **El item más arriesgado** — si simplificas a "instala todo o nada" baja a 4-6h |
| Tests BATS de contratos foundation | 10-16 | Crítico para no regresionar |
| README + ejemplos + CHANGELOG por plugin | 6-10 | Repetir patrón por plugin |
| Setup CI/CD si decides multi-repo | 8-16 | **Opcional** — si monorepo con workspaces, 0h |
| **Subtotal Phase 1** | **78-132h** | **2.5-4 semanas 1 FTE** |

Sin Phase 1, las siguientes phases no funcionan. **No se puede saltar.**

---

### Phase 2 — Proof of concept (`digest-pdf` aislado)

| Item | Esfuerzo (h) | Notas |
|---|---|---|
| Extracción agente `pdf-digest` + skill + comando | 4-6 | Mecánica |
| Stripping de referencias Savia-specific | 2-4 | Buscar `pm-config`, `savia.md`, hooks Savia |
| Manifest `plugin.yaml` + dependencias declaradas | 1-2 | Boilerplate |
| Test E2E en clean room (instalación desde cero) | 4-8 | Validar que arranca sin pm-workspace |
| Doc + ejemplo de uso | 2-4 | README mínimo |
| **Subtotal Phase 2** | **13-24h** | **0.5-1 semana 1 FTE** |

Si Phase 2 sale en menos del rango bajo, el patrón es replicable y las siguientes phases bajan ~30%. Si sale más, hay un problema de diseño y conviene replantear Phase 1.

---

### Phase 3 — `distro-document-inbox`

Requiere extraer 5 atómicos adicionales + manifest de distro.

| Atómico | Esfuerzo (h) | Riesgo |
|---|---|---|
| `digest-excel` | 12-18 | Bajo — patrón pdf |
| `digest-word` | 10-16 | Bajo — patrón pdf |
| `digest-meeting` (3 agentes: digest + risk-analyst + confidentiality-judge) | 18-28 | **Medio** — 3 agentes acoplados |
| `digest-voice` (audio→texto) | 14-22 | Medio — dependencia externa whisper |
| `mem-core` (memory-agent + scripts memory-store) | 16-26 | Medio — refactor scripts |
| Manifest `distro-document-inbox` + test integración | 8-14 | Bajo |
| **Subtotal Phase 3** | **78-124h** | **2.5-4 semanas 1 FTE** |

---

### Phase 4 — `distro-sovereignty` (el más caro)

| Atómico | Esfuerzo (h) | Riesgo |
|---|---|---|
| `shield-pii-filter` (gitleaks-like + hooks) | 18-28 | Medio — patrones PII |
| `shield-dual-inference` (proxy 127.0.0.1:8787 + failover) | 28-44 | **Alto** — Ollama integration, retry, telemetría |
| `shield-emergency-llm` (switch a LocalAI) | 16-24 | Medio — reusa parte de dual |
| `shield-vault` (vault + comandos vault-*) | 18-28 | Medio — cifrado en reposo |
| `shield-memory-backup` (memvid + SHA256) | 10-16 | Bajo |
| `shield-sovereignty-audit` (one-shot) | 8-12 | Bajo |
| `shield-confidentiality` (3 agentes: confidentiality-auditor + security-guardian + meeting-confidentiality-judge) | 16-24 | Medio |
| Renombrado Savia → genérico + white-labeling | 8-14 | **Doloroso pero crítico** — afecta a todos los anteriores |
| Manifest `distro-sovereignty` + tests | 10-16 | Bajo |
| **Subtotal Phase 4** | **132-206h** | **4-6.5 semanas 1 FTE** |

---

## Totales Primera Ola

| Phase | Min (h) | Max (h) | Min (€ interno @ 50€/h) | Max (€ consultoría @ 100€/h) |
|---|---|---|---|---|
| 1. Foundations | 78 | 132 | 3.900 € | 13.200 € |
| 2. `digest-pdf` PoC | 13 | 24 | 650 € | 2.400 € |
| 3. `distro-document-inbox` | 78 | 124 | 3.900 € | 12.400 € |
| 4. `distro-sovereignty` | 132 | 206 | 6.600 € | 20.600 € |
| **TOTAL** | **301h** | **486h** | **15.050 €** | **48.600 €** |

**Duración calendario:** 9-15 semanas 1 FTE dedicado, o 18-30 semanas a 50% del tiempo.

---

## Coste de mantenimiento ongoing

Tras la extracción, mantener vivos los plugins cuesta:

| Concepto | Esfuerzo (h/mes) |
|---|---|
| Bug fixes (12 atómicos × ~0.5h/mes) | 6 |
| Updates por nuevo modelo Claude (cada ~3 meses) | 8-16/release |
| Documentación + changelog | 4 |
| Soporte issues GitHub si se publican | 8-20 (depende de adopción) |
| Coordinación de versiones entre plugins | 4-8 |
| **Total** | **~30-50h/mes** = ~1 día/semana 1 FTE |

Si nadie usa los plugins fuera de Savia, el "soporte issues" se va a 0 pero el resto se mantiene. **Coste mínimo realista: ~15-20h/mes** = medio día/semana indefinidamente.

---

## Lo que NO está en estos números

1. **Marketplace strategy** — ¿GitHub releases? ¿Anthropic Plugin Marketplace? ¿npm? Decisión + setup: 16-40h una vez.
2. **Licensing legal review** — pm-workspace tiene LICENSE-ENTERPRISE.md. Separar OSS vs comercial por plugin: 8-16h consultoría legal.
3. **Marketing / landing pages** — si quieres venderlos: 40-80h por plugin destacado.
4. **Versionado SemVer entre 12+ plugins** — tooling tipo Lerna/Changesets: 8-16h setup + ongoing.
5. **Internacionalización** — hoy mucho contenido en ES. Si se vende fuera, traducir: 16-30h por plugin.

Estos pueden sumar **otras 100-250h** según ambición comercial.

---

## Riesgos que pueden duplicar el estimado

| Riesgo | Probabilidad | Impacto |
|---|---|---|
| Diseño del schema de plugin se queda corto y hay que reescribir | Media | +40-60h |
| `shield-dual-inference` requiere más Ollama/proxy work del previsto | Alta | +20-40h |
| Tests E2E en clean room revelan dependencias ocultas a pm-workspace | Alta | +30-60h |
| Hooks de `.claude/settings.json` no se desacoplan limpiamente | Media | +20-40h |
| Renombrado Savia→genérico rompe referencias internas en pm-workspace | Media | +15-30h |
| Decides multi-repo en lugar de monorepo y CI/CD explota | Baja | +40-80h |

**Margen razonable:** sumar 25-40% al estimate alto. Con margen, **600-700h** = ~5 meses 1 FTE.

---

## Beneficio esperado (para ponderar)

| Beneficio | Cuantificación |
|---|---|
| Reducción de coste de contexto en proyectos diana (~80%) | Ahorro tokens/proyecto: indirecto |
| Adopción externa de plugins (si se publican) | Incierto — depende de demanda no validada |
| Reuso interno entre proyectos Savia | Real — sala-reservas no necesita SDD completo |
| Onboarding más fácil a pm-workspace | Marginal — el problema es la complejidad, no el tamaño |
| Posicionamiento comercial "AI Sovereignty Shield" | Potencial alto — pero requiere marketing (no en el coste) |

---

## Recomendación de gating (radical honesty)

**No invertir las 300-500h sin validar antes.** Antes de Phase 1 completa, hacer un **spike de 20-30h** con tres preguntas:

1. ¿Hay al menos **3 personas fuera de Savia** que han pedido alguno de estos plugins? Si no → empieza por uso interno y deja de gastar en marketplace strategy.
2. ¿`digest-pdf` puede sacarse en **<24h** como atómico mínimo viable, sin foundations completas (versión "all-in-one zip")? Si sí → valida el patrón de extracción antes de comprometerte con la infra Layer 0.
3. ¿El equipo tiene **continuidad de 5 meses** para sostener este esfuerzo? Si no → no empieces, queda peor un extractivo a medias que un monolito que funciona.

**Plan mínimo barato (60-80h, 2-3 semanas):**
- Extraer `digest-pdf` y `shield-pii-filter` como zips standalone sin marketplace.
- Probarlos manualmente en sala-reservas y proyecto-alpha.
- Medir: ¿se usan? ¿alguien los pide?
- Decisión: si SÍ → invertir las 300-500h restantes. Si NO → pararse y revertir el spike.

Coste del spike: **3.000-8.000 €**. Coste de equivocarse con la primera ola completa: **15.000-50.000 € + 5 meses + deuda de mantenimiento perpetuo**.

El gating es la inversión correcta.
