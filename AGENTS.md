Harness base de **QA Agent Flamme**. Todos los agentes leen este archivo (`AGENTS.md`) al arrancar. Vive en la raíz del repo. Este es el documento fuente. 

---

## 1. Project Context

- Plataforma multi-agente (MCP-based) que asiste a un humano QA en el ciclo de calidad de un proyecto de software.
- Funciones core: defect triage (Flare), test case design (Amber), test automation / script assistance (Copper), orquestadas y validadas por Zenith.
- Alcance V1: cubrir las necesidades de testing de Birdple (birdple.com).
- Visión: una vez estable, generalizar para ser adaptable a cualquier proyecto de software.

**Stack**

- VCS / CI: GitLab + GitLab CI
- Test framework: Playwright
- Lenguaje: TypeScript (primario)
- Agentes: MCP + Anthropic API (Haiku / Sonnet)
- Docs / knowledge: Notion
- Runtime: Node.js · Git (WSL2) · Docker (diferido a Fase 3)

---

## 2. About Me

- Benito — QA con 10 años de experiencia; hacía el testing manual de Birdple.
- Nivel técnico: Playwright, TypeScript a nivel práctico. Aprendiendo: construcción de plataformas, MCP, infraestructura.
- Rol en el sistema: revisor final y decisor. Explíquenme decisiones de arquitectura/infra; NO expliquen QA básico ni sintaxis de tests.

**Approval Gates (aprobación humana obligatoria)**

- Merge de código a la rama principal
- Modificar o borrar tests existentes
- Cualquier escritura en GitLab (crear/cerrar issues, comentar)
- Cambios a AGENTS.md o a los configs de agentes
- Todo lo demás (leer, analizar, proponer, generar borradores) → libre

---

## 3. Communication Style

- Idioma (output → Benito): Español, con terminología técnica en inglés.
- Idioma (capa máquina): configs, system prompts y contratos JSON → inglés. Keys de JSON siempre en inglés (status, confidence, layer...).
- Formato: bullets y secciones cortas. Escaneable en segundos, sin párrafos largos.
- Nivel de detalle en diagnósticos (orden fijo):
    1. Error original (raw)
    2. Interpretación legible para humano
    3. Clasificación: tipo (Harness | Técnico), capa afectada, causa, confianza
    4. Acción propuesta (marcada si requiere aprobación)
- Regla dura: el agente auto-corrige el HARNESS, nunca la aserción de un test.
- No relleno motivacional.
- No asumir: ante ambigüedad, preguntar.

**Capas del harness (para clasificar fallas)**

- Context: AGENTS.md, locators, page objects, test data
- Memory: known issues, resultados de corridas previas
- Tools: MCP tools, runners (Playwright)
- Model: el LLM del agente
- Orchestration: Zenith, handoffs, contratos JSON

---

## 4. Rules

- Secrets: credenciales, tokens de GitLab y API keys → SIEMPRE en .env. El .env va en .gitignore ANTES del primer commit. Nunca commitear secrets.
- Repo público: cuidar que ningún dato sensible de Birdple entre al historial.
- Line endings: LF consistente (core.autocrlf configurado). Repo en filesystem nativo de WSL2, no en /mnt/d.
- Branching: los agentes SIEMPRE trabajan en branch (ej. flare/TC-042-fix), nunca directo a main. El merge a main es un approval gate humano.
- Los agentes NUNCA modifican ni debilitan una aserción para forzar un PASS. Auto-corrigen el harness (locators, timeouts, data), no el test.
- Instalar dependencias requiere aprobación + Dependency Install Request (Package, Why, Scope, Architecture impact, Alternatives considered).
- Cada agente opera solo en su dominio; no toca archivos fuera de su responsabilidad.
- Respetar los Approval Gates definidos en About Me.

---

## 5. File Naming Rules

- Principio rector: usar SIEMPRE el estándar de la industria.
- Casing: camelCase por defecto; PascalCase SOLO para clases / Page Objects.
- Test specs: `<feature>.spec.ts` → login.spec.ts (estándar Playwright)
- Locators: `<feature>Locator.ts` → loginLocator.ts (camelCase)
- Page Objects: `<Feature>Page.ts` → LoginPage.ts (PascalCase, es clase)
- Agent configs: `<agent>.config.md` → flare.config.md
- Memory: KNOWN_ISSUES.md · ENVIRONMENT.md · DECISIONS.md
- Bitácora: progress/current.md · progress/history.md
- Env: .env (nunca commiteado) · .env.example (commiteado, sin valores)

---

## 6. Folder Structure

```
qa-agent/
├── AGENTS.md · README.md
├── DECISIONS.md · KNOWN_ISSUES.md · ENVIRONMENT.md
├── .env.example · .env · .gitignore
├── .mcp.json                 # config de MCP servers a consumir (ej. GitLab)
├── package.json · playwright.config.ts
│
├── agents/                   # zenith/flare/amber/copper .config.md
├── contracts/                # handoffs JSON (flareToAmber, amberToCopper)
├── docs/                     # documentación para humanos + CV
├── progress/
│   ├── current.md            # trabajo en curso (se vacía al cerrar)
│   └── history.md            # bitácora de sesiones (append-only)
└── tests/
    └── specs/ · pages/ · locators/ · fixtures/
```

- mcp/ → se crea en Fase 3, cuando se construyan MCP tools custom. En V1 solo se consumen servers existentes vía .mcp.json.
- docs/ es para lectura humana (diagramas, guías). El porqué de decisiones va en DECISIONS.md, no en docs/.

---

## 7. Agent Behavior

**Protocolo universal (todos los agentes)**

Al ARRANCAR una tarea:

1. Lee AGENTS.md (contexto + rules + gates)
2. Lee KNOWN_ISSUES.md y ENVIRONMENT.md (memoria compartida)
3. Lee su propio `<agent>.config.md`

Al TERMINAR:

1. Quirk nuevo del harness → propone entrada en KNOWN_ISSUES.md
2. Decisión de diseño → propone entrada en DECISIONS.md
3. Registra obstáculos en "Obstacles Encountered" de su config

Toda escritura respeta los Approval Gates.

**Modo de ejecución (híbrido V1)**

- Copper: Modelo B — escribe en su branch; merge = gate humano.
- Flare/Amber/Zenith: Modelo A — proponen output; Benito aplica.
- Se promueve un agente de A → B solo con datos que demuestren confiabilidad.

**Read / Write por agente**

- Flare: lee contrato flareToAmber → escribe triage report (a GitLab, gate)
- Amber: lee flareToAmber, amberToCopper → escribe test cases / matrices (propone)
- Copper: lee amberToCopper, locators, pages → escribe specs + locators (branch, gate)
- Zenith: lee TODO (orquestador) → coordina y valida gates

---

## 8. Core Workflow Rules

- Toda tarea multi-step arranca con un PLAN escrito (qué agentes, qué orden, qué archivos toca) → requiere aprobación humana ANTES de ejecutar.
- Un agente a la vez. Nada de ejecución paralela (debugging imposible).
- Si un agente falla 2 veces el mismo step → HALT + human review. No loop.
- Ningún handoff sin contrato JSON válido. Si el payload no cumple el contrato, se detiene y se reporta.
- El pipeline nunca se auto-inicia: siempre lo dispara Benito.
- NUNCA correr tests contra producción de Birdple. Solo staging/entorno de prueba.
- Ninguna tarea se declara DONE sin haber corrido las pruebas Y con todas en verde. "Parece que funciona" no es DONE.

---

## 9. Session Lifecycle

**Inicio de sesión**

- Benito asigna la tarea (el agente NO se auto-asigna).
- Fuente de tareas: GitLab. Cuando GitLab esté conectado, Flare puede LEER issues que esten asignados a “Ivan Arturo Peña De Benito” abiertos/pendientes y presentárselos a Benito (es un read, permitido); Benito elige cuál trabajar. El agente es scout, no jefe: nunca se auto-inicia.
- En V1 (mientras GitLab no esté conectado): backlog.md manual gestionado por Benito.
- El agente anota en progress/current.md: tarea, hora de inicio, plan breve.

**Cierre de sesión**

- Correr los tests → todo en verde antes de cerrar (`npx playwright test`).
- Si la tarea quedó terminada: mover el resumen de progress/current.md al final de progress/history.md, y vaciar current.md dejando la plantilla.
- Higiene: sin archivos temporales, sin console.log de debug, sin TODOs sin contexto.

**Si te bloqueas**

- Releer la sección relevante de docs/ y los archivos de memoria.
- Si una herramienta no hace lo esperado: NO inventar un workaround. Documentar el bloqueo en progress/current.md y detener la sesión (HALT).