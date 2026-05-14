# Coste bajo modelo DLC (on-demand)

**Fecha:** 2026-05-12
**Reemplaza:** `output/20260512-coste-extraccion-primera-ola.md` (modelo big-bang) para el caso DLC.
**Paradigma:** Base mínima + DLCs que se descargan/activan cuando un proyecto los necesita.

---

## Diferencia con el modelo big-bang

| Aspecto | Big-bang (informe anterior) | DLC (este informe) |
|---|---|---|
| Pago inicial | 300-500h sacando todo | 60-120h solo la infra + 1 DLC piloto |
| Pago por DLC adicional | 0 (ya está sacado) | 8-30h cada vez que un proyecto lo necesita |
| Riesgo de "sacar algo que nadie pide" | Alto | Cero — solo extraes bajo demanda real |
| Dep resolver | Necesario (DAG completo) | Simple (cada DLC depende solo de foundations) |
| Versioning hell | Real entre 12+ plugins | Mínimo — cada DLC versiona aislado |
| Time-to-first-value | 9-15 semanas | 2-3 semanas |

El modelo DLC convierte un proyecto de 5 meses en una **infraestructura de 2-3 semanas + extracciones puntuales** según necesidad.

---

## Coste de la infraestructura DLC (one-time)

Esta es la parte que pagas una sola vez, antes del primer DLC útil.

| Item | Esfuerzo (h) | Notas |
|---|---|---|
| Schema `dlc.yaml` (más simple que plugin.yaml general) | 3-5 | Solo necesita: nombre, versión, foundations_required, files_to_install, hooks_to_register, settings_patches |
| Catálogo DLC (un repo Git con índice JSON) | 4-8 | Lista de DLCs disponibles + metadata + URL del tarball |
| Comando `/dlc install <name>` | 8-14 | Descarga + descomprime + valida hash + registra |
| Comando `/dlc enable/disable <name>` | 4-8 | Toggle sin desinstalar (patch settings.local.json) |
| Comando `/dlc uninstall <name>` | 4-8 | Limpieza completa, revierte settings patches |
| Comando `/dlc list / status` | 2-4 | Inventario y estado |
| `cc-foundations` empaquetado (reglas + protocolos base) | 6-10 | Lo mismo que en big-bang |
| Tests E2E del flujo install/enable/disable | 8-14 | Crítico |
| Documentación del modelo + un README ejemplo | 4-6 | |
| **TOTAL infra DLC** | **43-77h** | **1.5-2.5 semanas 1 FTE** |

Frente a las 78-132h del modelo big-bang Phase 1. **Ahorro: ~35-55h.**

La simplificación clave: un DLC no necesita conocer otros DLCs. Solo depende de foundations. No hay grafo, no hay SemVer cruzado, no hay resolución de conflictos.

---

## Coste por DLC (incremental, on-demand)

Cuando un proyecto Savia (interno) o externo pide un DLC concreto, lo extraes. Estimación por categoría:

| Tipo de DLC | Esfuerzo (h) | Ejemplos |
|---|---|---|
| Atómico simple (1 skill o 1 agente) | 8-14 | `digest-pdf`, `digest-excel`, `shield-memory-backup`, `shield-sovereignty-audit` |
| Atómico medio (1 skill + 1-2 agentes) | 14-24 | `digest-meeting`, `shield-confidentiality`, `code-reviewer-standalone` |
| Atómico complejo (proxy, hooks, infra externa) | 24-44 | `shield-dual-inference`, `shield-pii-filter`, `sec-pentester` |
| Bundle pequeño (3-5 piezas cohesionadas) | 30-60 | `tribunal-core`, `court-core`, `mem-rag` |
| Bundle grande (SDD engine, sovereignty completo) | 80-150 | `sdd-engine`, `distro-sovereignty` |

**Punto importante:** un DLC se extrae **solo si** un proyecto real lo necesita. Si nadie pide `sdd-lang-cobol`, nunca se extrae. Coste evitado: 8-14h × DLCs no demandados.

---

## Escenarios de gasto realista

Asumiendo que la infra DLC ya está hecha (~60h):

### Escenario A — Frugal (solo lo que Savia necesita internamente)

Si los proyectos `sala-reservas`, `savia-web`, `savia-monitor`, `savia-mobile-android` solo necesitan:
- `digest-pdf` (alguien sube un PDF a un proyecto)
- `shield-pii-filter` (proyecto cliente con datos sensibles)
- `code-reviewer-standalone` (revisión sin court completo)

Coste = infra (60h) + 3 DLCs simples (~36h) = **~96h ≈ 3 semanas 1 FTE**

### Escenario B — Cliente externo pide compliance

Cliente regulado pide algo concreto: filtrado PII + audit trail.

DLCs a extraer: `shield-pii-filter` + `shield-confidentiality` + `gov-regulatory` + `gov-rbac`

Coste = infra (60h) + 4 DLCs (60-100h) = **~120-160h ≈ 4-5 semanas**

Pero ese cliente paga, por lo que el coste se amortiza directo. Big-bang habría exigido sacar 13 bundles antes de ver el primer euro.

### Escenario C — SDD completo bajo demanda

Tras 6 meses, varios proyectos quieren SDD. Extraes progresivamente:
- Mes 1: `sdd-engine` (60h)
- Mes 2: `sdd-lang-typescript` + `sdd-lang-python` (24h)
- Mes 4: `sdd-extensions` (40h)
- Mes 6: `sdd-quality-gates` (30h)

Coste acumulado = infra (60h) + SDD progresivo (154h) = **~214h en 6 meses** (no en bloque)

Comparado con big-bang (132-206h Phase 4 + foundations + tests) en 5-6 semanas continuas, el modelo DLC distribuye el coste y reduce riesgo de quemar al equipo.

---

## Riesgos específicos del modelo DLC

| Riesgo | Probabilidad | Mitigación |
|---|---|---|
| Drift entre DLC instalado y pm-workspace upstream | Alta | Hook `/dlc check-updates` en SessionStart |
| Settings.local.json se ensucia con patches | Media | Cada DLC declara qué settings patcha, revertible |
| Hooks de varios DLCs entran en conflicto | Media | Manifest declara orden y categoría de hook |
| Foundations cambia y rompe DLCs viejos | Alta | SemVer estricto en foundations; DLCs declaran rango compatible |
| Usuarios olvidan qué tienen instalado | Baja | `/dlc list` + telemetría local |

Ninguno bloquea, todos son tooling adicional (~20-40h ongoing).

---

## Mantenimiento ongoing bajo DLC

Mucho menor que big-bang:

| Concepto | h/mes |
|---|---|
| Mantener infra DLC (install/update/disable) | 4-6 |
| Bugs por DLC instalado (~0.3h/mes por DLC activo) | 2-6 (según cuántos DLCs Savia mantenga) |
| Updates por nuevo modelo Claude | 4-8 / release (escala con n DLCs) |
| **Total realista (5 DLCs activos)** | **~12-20h/mes** | medio día/semana |

Frente a las 30-50h/mes del big-bang completo.

---

## Comparativa final

| Métrica | Big-bang | DLC on-demand |
|---|---|---|
| Inversión inicial | 300-500h (~5 meses) | 60h infra + 8-30h por DLC |
| Time-to-first-value | 9-15 semanas | 2-3 semanas |
| Riesgo de sacar algo no demandado | Alto | Nulo |
| Mantenimiento ongoing | 30-50h/mes | 12-20h/mes |
| Coste total año 1 (5 DLCs activos) | ~700h | ~250h |
| Vendibilidad externa | Lista al final | Lista por DLC, gradual |

**El modelo DLC reduce el coste año 1 ~65%.** No por ser más eficiente intrínsecamente, sino por evitar extraer plugins que nadie pide.

---

## Recomendación (radical honesty)

El modelo DLC es **claramente superior** para tu caso porque:

1. No tienes demanda externa validada para los 13 bundles → big-bang especula con esfuerzo no recuperable.
2. Savia interno solo usa una fracción de los plugins → DLC modela tu uso real.
3. Si en el futuro aparece un cliente que necesita el bundle X, extraes X bajo demanda con margen.
4. La infra DLC es pequeña (60h) — la pagas una vez y aceleras todo lo demás.

**Plan concreto:**
- **Semana 1-2:** Infra DLC + `cc-foundations` empaquetado (~60h).
- **Semana 3:** `digest-pdf` como primer DLC piloto (~12h). Probarlo en `sala-reservas` o `proyecto-alpha`.
- **Semana 4:** Decisión gate. Si el patrón funciona limpio → publicar catálogo DLC y abrir a más extracciones bajo demanda. Si no → ajustar infra antes de seguir.

**Coste total compromiso inicial: ~72h ≈ 2.5 semanas 1 FTE ≈ 3.600-5.800 €.**

A partir de ahí pagas solo lo que un proyecto real pide. Es la decisión correcta.
