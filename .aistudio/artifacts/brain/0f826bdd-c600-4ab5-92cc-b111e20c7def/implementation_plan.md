# Enterprise Agentic LLM Platform Workbench: Specialized Standalone UI & Engine Integration

An architectural blueprint and engineering roadmap establishing a **standalone, secure TypeScript Agentic Workbench UI** (`modules/ui-agentic-workbench`). The interface translates the layout and ergonomics of the Perchance chat system into an enterprise agentic workbench, linking directly to the **Lean Java Agentic Core (LJAC)** execution engine and the **AgentForge H-MAP** multi-agent governance platform.

---

## User Review & Critical Decisions

> [!IMPORTANT]
> The following core architectural decisions are synthesized directly from your instructions:
>
> 1. **Complete Module Isolation (UI as a Standalone Module)**:
>    - The frontend is structured as an isolated, self-contained TypeScript module (`modules/ui-agentic-workbench`).
>    - It maintains **zero compile-time dependencies** on the Java runtime or backend object libraries, communicating solely through strictly typed JSON DTOs and Server-Sent Events (SSE) over standard HTTP.
> 2. **Perchance Paradigm Specialized for Agentic Workspaces**:
>    - **No Image Generation**: The image creation affordance is eliminated and replaced with **Workspace Folder & File Linking** (`📁 Workspace / File`).
>    - **Default Chat ("Chloe")**: Configured as the **Built-in Gemini Platform Architect & Diagnostic Helper** (powered by `gemini-3.8-flash` with Google Search grounding or `gemini-4-argon` with High Thinking) for codebase navigation, JDK 21 rules, and architectural guidance.
>    - **Additional "Characters"**: Specialized **AgentForge Subagents** (e.g., Lead Code Architect, AST Slicer, Rule DAG Auditor, Security Prober) equipped with explicit `TaskContracts`, Tier 1 & 2 Rule DAGs (`AGENTS.md`), tool allowlists, and sandboxed working directories.
> 3. **Role Switcher & Interaction Ergonomics**:
>    - Perspective selector above input deck provides four operational streams:
>      - `🤖 Active Agent`: Standard prompt directive dispatched to the selected subagent.
>      - `👤 Operator`: Human feedback / steering injected into the agent's scratchpad.
>      - `⚙️ System Kernel`: Direct injection of an invariant constraint into the system context.
>      - `📁 Workspace Link`: Attachment of file snippets or directory trees into the agent's context window.
>    - Inline turn actions: `✏️` In-place prompt/response edit, `🗑️` Prune turn from history, `🔁` Re-trigger inference / tool execution.
> 4. **Strict TypeScript & Security Discipline**:
>    - 100% strict TypeScript throughout (`strict: true`, no `any`, no plain `.js` files).
>    - XSS-safe DOM rendering (no unchecked `dangerouslySetInnerHTML`), input sanitization, and local-first `localStorage` persistence with complete JSON backup export and import.
> 5. **Planning Mode (Zero Build)**:
>    - In accordance with your explicit instruction, this turn is dedicated strictly to research, system planning, and interface modeling. No source code will be modified until you confirm the plan.

---

## 1. Domain Concept Mapping: Perchance → Enterprise Agentic Workbench

The table below details how every visual and interactive affordance from the Perchance layout is repurposed for the Agentic LLM Platform:

| Perchance Reference Element | Agentic Workbench Translation | Underlying Technical Mechanism (LeanJavallm & AgentForge) |
| :--- | :--- | :--- |
| **`💬 new chat / character`** | **`+ New Session / Spawn Subagent`** | Dispatches a new session bound to an AgentForge `TaskContract` or launches a specialized subagent from the agent catalog. |
| **Thread List & Avatars** | **Active Subagent Tasks & Sessions** | Displays active execution sessions with agent avatar/icon, assigned model, turn counter, contract status (`PENDING`, `RUNNING`, `COMPLETED`), and delete/rename actions. |
| **"Chloe" (Default Character)** | **Built-in Gemini Platform Architect** | Default platform assistant (`gemini-3.8-flash` with Google Search or `gemini-4-argon` with High Thinking) providing zero-dep Java 21, AgentForge, and architectural guidance. |
| **Additional Characters** | **Specialized AgentForge Subagents** | Configurable subagents: Lead Architect, AST Slicer, Rule DAG Auditor, and Code Engineer. Each possesses a charter, tool allowlist, and sandboxed directory (`.agent/workspaces/agents/<id>/`). |
| **`✏️ edit character`** | **`⚙️ Configure Agent Specification`** | Modal to edit: Agent Title, Assigned Model ID, System Charter / Persona Prompt, Active Rule Protocols (`AGENTS.md`), Tool Allowlist, and execution timeouts. |
| **`🏷️ set ur name/pic`** | **`👤 Operator Profile & Workspace`** | Configures Operator handle (e.g. "Lead Engineer"), authorization level, and active workspace root path. |
| **Inline Message Actions (`✏️`, `🗑️`, `🔁`)** | **Turn Rewind, Edit & Tool Re-run** | `✏️` In-place prompt refinement; `🗑️` Prune turn from conversation history; `🔁` Re-trigger inference stream or re-execute agent tool call. |
| **Perspective Selector (`Chloe`, `Anon`, `Narrator`)** | **Stream Role & Injection Selector** | `🤖 Active Agent` (direct prompt), `👤 Operator` (human feedback), `⚙️ System Kernel` (system invariant injection), `📁 Workspace Link` (file/directory snippet link). |
| **Bottom Utility Bar (`settings`, `export`, `import`, `clear`)** | **Platform Governance Toolbar** | `⚙️ Provider Settings` (Cloud models, API keys, Ollama), `💾 Export Session` (complete JSON backup), `📁 Import Session`, `🗑️ Clear Sessions`. |

---

## 2. User Experience & Visual Design

Following the SaaS & Dashboard design guidelines (`references/3_saas_dashboard.md`), the interface adopts a focused, high-contrast, dark-mode developer workbench aesthetic.

### Aesthetic Direction & Visual Tokens
- **Canvas Atmosphere**: Deep slate background (`#0b0f17` / `bg-slate-950`) providing high contrast and minimal eye strain during extended coding sessions.
- **Structural Surfaces**: Single-elevation dark panels (`#131b2e` / `bg-slate-900/80`) bounded by clean hairline borders (`border-slate-800`). Zero nested cards within cards.
- **60-30-10 Color Budget**:
  - `60% Neutral Canvas`: `#0b0f17` deep slate field.
  - `30% Structural Panels`: `#131b2e` sidebar, top header, and message bubbles.
  - `10% Accent Points`: Electric Indigo (`#6366f1` / `indigo-500`) for primary actions (`Send`, `Spawn Agent`), Emerald (`#10b981` / `emerald-500`) for active engine status, and Amber (`#f59e0b` / `amber-500`) for system invariants.
- **Typography**: Clean display sans (`Plus Jakarta Sans` / `Outfit`) for headers and navigation, paired with `Public Sans` for message prose. Monospace figures (`JetBrains Mono`, `tabular-nums`) are enforced across turn counts, timestamps, and tool argument payloads.

### User Workflows

```
1. Zero-State Launch ──> Gemini Platform Architect greets operator in default session
         │
         ├──> [Operator asks architectural question] ──> Real-time streaming response with syntax highlighting
         │
         ├──> [Operator clicks "+ New Session"] ──> Selects "AST Slicer" subagent from AgentForge catalog
         │                                              │
         │                                              └──> Custom workspace isolated to .agent/workspaces/
         │
         ├──> [Operator clicks "⚙️ Configure Agent"] ──> Edits System Charter, assigns model & rule protocols
         │
         ├──> [Operator switches role to "📁 Workspace Link"] ──> Attaches file snippet to prompt
         │
         └──> [Operator clicks "💾 Export"] ──> Downloads offline JSON backup containing all sessions & specs
```

---

## 3. Standalone Architecture & Module Isolation

The frontend is architected as an isolated TypeScript module located at `modules/ui-agentic-workbench/` (or `src/workbench/`), decoupled from the Java runtime.

### Component Layout & Hierarchy

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        STANDALONE WORKBENCH UI (TypeScript)                            │
├──────────────────────────────────┬─────────────────────────────────────────────────────┤
│   AgentThreadSidebar.tsx (280px) │   AgentArena.tsx (Flex 1)                           │
│   ├── [New Session / Spawn Agent]│   ├── ArenaTopBar.tsx                               │
│   ├── SessionList.tsx            │   │   ├── [⚙️ Configure Agent Spec]                 │
│   │   └── AgentSessionCard.tsx   │   │   ├── [👤 Operator Profile]                     │
│   ├── StorageNotice.tsx          │   │   └── [Engine Status: Java 21 Zero-Dep]         │
│   └── PlatformControlBar.tsx     │   ├── AgentMessageStream.tsx                        │
│       ├── SettingsModalTrigger   │   │   ├── AgentMessageBubble.tsx                    │
│       ├── ExportJsonButton       │   │   │   ├── InlineEditForm.tsx                    │
│       ├── ImportJsonButton       │   │   │   └── ToolExecutionBadge.tsx                │
│       └── ClearSessionsButton    │   │   └── StreamingCursor.tsx                       │
│                                  │   └── AgentInputDeck.tsx                            │
│                                  │       ├── StreamRoleSelector.tsx                    │
│                                  │       │   ├── [🤖 Active Agent]                     │
│                                  │       │   ├── [👤 Operator]                         │
│                                  │       │   ├── [⚙️ System Kernel]                    │
│                                  │       │   └── [📁 Workspace Link]                   │
│                                  │       ├── PromptTextArea.tsx                        │
│                                  │       ├── SendButton.tsx                            │
│                                  │       └── OptionsDrawer.tsx (Temp / Max Tokens)     │
└──────────────────────────────────┴─────────────────────────────────────────────────────┘
```

### Typed Data Models (`modules/ui-agentic-workbench/types.ts`)

```typescript
export type AgentRole = 'SYSTEM_ARCHITECT' | 'CODE_ENGINEER' | 'AST_SLICER' | 'RULE_AUDITOR' | 'CUSTOM_AGENT';
export type StreamPerspective = 'ACTIVE_AGENT' | 'OPERATOR' | 'SYSTEM_KERNEL' | 'WORKSPACE_LINK';

export interface AgentDefinition {
  id: string;
  name: string;
  role: AgentRole;
  avatarIcon: string;
  assignedModelId: 'gemini-3.8-flash' | 'gemini-4-argon' | 'claude-3-7' | 'local-ollama';
  systemCharter: string;
  temperature: number;
  assignedRuleProtocols: string[]; // e.g. ['AGENTS.md', '01_architecture_protocols.md']
  enabledTools: string[];          // e.g. ['ast_slicer', 'ast_patcher', 'rule_dag_linter']
  workspacePath: string;           // e.g. '.agent/workspaces/agents/<id>/'
  isDefaultHelper?: boolean;
}

export interface AgentTurn {
  id: string;
  role: 'AGENT' | 'OPERATOR' | 'SYSTEM_KERNEL' | 'TOOL_RESULT';
  senderName: string;
  senderAvatar: string;
  content: string;
  timestamp: number;
  toolCall?: {
    toolName: string;
    argumentsJson: string;
    resultSnippet?: string;
    durationMs?: number;
  };
  isStreaming?: boolean;
  isEditing?: boolean;
}

export interface AgentSession {
  sessionId: string;
  title: string;
  agentId: string;
  turns: AgentTurn[];
  taskContractId?: string;
  linkedWorkspacePath?: string;
  createdAt: number;
  updatedAt: number;
}
```

---

## 4. Technical Architecture: System Flow & Boundary Separation

```
┌────────────────────────────────────────────────────────────────────────┐
│             STANDALONE FRONTEND: modules/ui-agentic-workbench          │
│   • Local-First React Store (`workbenchStorage.ts`)                    │
│   • Multi-role Prompt Deck & Perspective Selector                      │
│   • Zero compile-time coupling to backend classes                      │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   │
                                   │ HTTP REST & SSE Stream (/api/agent/stream)
                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│                   EXPRESS / TYPESCRIPT PROXY (server.ts)               │
│   • Route Dispatcher: /api/agent/stream, /api/agent/contracts          │
│   • Secure Secret Handshake (Zero-Fill Memory Buffer)                  │
│   • LLM Providers: Gemini 3.8 Flash, Gemini 4 Argon, Local Ollama      │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   │
                                   │ IPC / Stdio Process Bridge (No Jackson/Gson)
                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│               LEAN JAVA AGENTIC CORE (Java 21 LTS - java.base)         │
├──────────────────────────────────┬─────────────────────────────────────┤
│   com.leanllm.core               │   com.leanllm.agentforge            │
│   ├── SecureSecret (Wipe char[]) │   ├── TaskContract.java (Record)    │
│   ├── ProcessTreeTerminator      │   ├── ExecutionArtifact.java        │
│   ├── JsonNode & JsonParser      │   ├── McpPathPolicyFilter (Jail)    │
│   ├── ContextCompactor (Sliding) │   ├── RuleDagEngine (AGENTS.md)     │
│   └── AbstractProcessRunner      │   └── TypeScriptTaskRunner (Loom)   │
└──────────────────────────────────┴─────────────────────────────────────┘
```

### Architectural Guarantees & Compatibility Invariants
1. **RFC 8259 Zero-Dependency Parser**: All DTOs exchanged between the TypeScript UI and Java backend are validated using our zero-dependency `com.leanllm.json.JsonParser` (`MAX_DEPTH = 64`, strict Unicode `\uXXXX` handling, no external libraries).
2. **Deterministic Memory Hygiene**: API keys entered in the UI Settings modal are processed on the server via `SecureSecret` zeroing logic (`char[]` overwrite with `\0` upon transport dispatch).
3. **Subprocess Isolation**: When an AgentForge subagent executes a tool script (such as `ast_slicer.ts`), it executes inside an isolated realpath jail enforced by `McpPathPolicyFilter.java`. If the process hangs, `ProcessTreeTerminator.java` terminates the entire OS process hierarchy recursively.
4. **Context Compaction**: The UI implements `ContextCompactor` logic to ensure long conversations never exceed token ceilings while permanently locking the `AGENTS.md` Tier 1 charter at index 0.

---

## 5. Phased Implementation Roadmap (Post-Approval)

When you approve this plan, execution will follow these rigorous phases:

### Phase 1: Standalone TypeScript Workbench UI (`modules/ui-agentic-workbench/` & `src/workbench/`)
- Build `AgentThreadSidebar.tsx`: New session / spawn agent button, session cards, storage disclaimer, and bottom utility toolbar.
- Build `AgentArena.tsx` & `ArenaTopBar.tsx`: Agent specification modal trigger, operator profile modal trigger, and live engine status indicator.
- Build `AgentMessageStream.tsx`: Turn bubbles with avatar, Markdown rendering, syntax highlighting, tool execution cards, and inline edit/delete/re-run actions.
- Build `AgentInputDeck.tsx`: 4-role perspective selector (`🤖 Agent`, `👤 Operator`, `⚙️ System Kernel`, `📁 Workspace Link`), auto-resizing prompt textarea, `Send` button, and options drawer.
- Build `workbenchStorage.ts`: Local-first `localStorage` persistence with complete JSON backup export and import.

### Phase 2: Express Backend Stream Endpoints (`server.ts`)
- Implement `/api/agent/stream` supporting streaming SSE for Gemini 3.8 Flash, Gemini 4 Argon, and local Ollama.
- Implement `/api/agent/contracts` to link AgentForge `TaskContract` creation with active session turns.

### Phase 3: Core Java 21 Zero-Dependency Foundation (`java/`)
- Implement RFC 8259 compliant `JsonNode.java` and `JsonParser.java` under `com.leanllm.json`.
- Implement `TaskContract.java` and `McpPathPolicyFilter.java` under `com.leanllm.agentforge`.
- Implement standalone `JsonParserTest.java` running on standard OpenJDK 21 LTS without JUnit.

### Phase 4: Obsidian Documentation Graph (`docs/`)
- Author `docs/ui/AgenticWorkbench.md` with in-depth logic explaining the Perchance-to-Agentic UI translation.
- Document RFC 8259 compliance, CWE-674 recursion limits, and AgentForge integration hooks in `docs/json/ZeroDepJson.md`.

### Phase 5: Verification & Git Commit
- Run logic unit tests: `npx --yes tsx --test src/utils/*.test.ts`.
- Run compiler (`compile_applet`) and TypeScript linter (`tsc --noEmit`).
- Verify live browser interaction and visual appearance via Playwright CLI.
- Commit all changes and push to `https://github.com/jcham2024-arch/LeanJavallm`.
