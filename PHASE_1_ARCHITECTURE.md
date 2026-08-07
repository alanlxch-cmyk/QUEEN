# QUEEN - PHASE 1: Core Infrastructure Architecture

## Objective
Build the stable, minimal, completely decoupled nucleus of the system.

---

## PHASE 1 Components

### 1. **Meta Kernel**
- Purpose: Coordination and decision-making only
- Responsibilities:
  - Decide what to do next
  - Distribute tasks to specialized components
  - Monitor system state
  - Reorganize priorities
  - NO specific logic, only orchestration

```
Input: Objective
↓
Meta Kernel decides WHO should handle it
↓
Sends event to appropriate component
↓
Component responds via Event Bus
```

---

### 2. **Router (AI Model Router)**
- Purpose: Decide which AI model to use
- Considers:
  - Cost
  - Speed
  - Context window
  - Quality
  - Availability
  - Specific capabilities needed

Models available:
- OpenAI GPT
- Anthropic Claude
- Google Gemini
- Groq
- DeepSeek
- Local models
- Future models (plugin-based)

---

### 3. **Capability Manager**
- Purpose: Agents never know about specific tools
- Agents request: "I need to generate a video"
- Capability Manager finds: "Use SkyReels API or FFmpeg"
- This decouples agents from tools

```
Agent: "I need vision capability"
↓
Capability Manager checks registry
↓
Returns available options (Claude Vision, GPT-4V, Gemini Vision)
↓
Agent picks one or Capability Manager picks best
```

---

### 4. **Plugin Manager**
- Purpose: Lifecycle management of ALL plugins
- Responsibilities:
  - Install plugins
  - Update plugins
  - Remove plugins
  - Validate plugins
  - No plugin requires core modification

Every plugin implements the same interface:
```python
class IPlugin:
    def initialize(config)
    def validate()
    def execute(task)
    def health_check()
    def shutdown()
```

---

### 5. **Event Bus**
- Purpose: Internal message passing
- No direct component-to-component communication
- All communication via events

```
Component A → Event Bus → Component B
Component C → Event Bus → Component D
```

Event types:
- `component.registered`
- `task.assigned`
- `task.completed`
- `error.occurred`
- `capability.requested`
- `memory.store`
- `knowledge.query`

---

### 6. **Memory Manager**
- Purpose: Store and retrieve experience
- Not conversation history - EXPERIENCE

Stores:
- Past objectives
- Plans executed
- Tools used
- Models used
- Errors encountered
- Corrections made
- Time spent
- Cost spent
- Results
- Lessons learned

```
After each task:
↓
Memory Manager stores complete record
↓
Can be retrieved by future agents
↓
Prevents solving same problem twice
```

---

### 7. **Knowledge Manager**
- Purpose: Index and search local documentation
- Structure:

```
knowledge/
  engineering/
    mechanical/
    civil/
    electrical/
  programming/
    python/
    cpp/
    javascript/
  standards/
    ASME/
    ISO/
    AWS/
  science/
  medicine/
  finance/
```

Query priority:
1. Memory (past experience)
2. Local documentation (knowledge/)
3. Internal repositories
4. GitHub
5. Web
6. AI Model

---

### 8. **Logger**
- Purpose: Centralized logging
- All components log to Logger, not stdout
- Structured logging with:
  - Timestamp
  - Component name
  - Log level
  - Message
  - Context data

---

### 9. **Configuration Manager**
- Purpose: Centralized configuration
- Format: JSON/YAML
- No hardcoded values in code
- Configuration per component:

```json
{
  "metakernel": {
    "max_retries": 3,
    "timeout": 300
  },
  "router": {
    "preferred_model": "gpt-4",
    "fallback_models": ["claude-3", "gemini"]
  },
  "plugin_manager": {
    "auto_update": false,
    "sandbox_mode": true
  }
}
```

---

## Interaction Flow

### Example: User gives objective

```
User: "Optimize this Python code"

↓ Meta Kernel receives objective

↓ Meta Kernel publishes event: "task.received"

↓ Components listening:
   - Memory Manager: Check if similar task exists
   - Knowledge Manager: Find Python optimization standards
   - Plugin Manager: Verify required plugins available

↓ Meta Kernel decides:
   - Need Programming Agent
   - Need Python analyzer tool
   - Use fast model (Claude) for speed

↓ Capability Manager checks:
   - Programming capability available? YES (multiple agents)
   - Python analysis capability? YES (AST analyzer plugin)
   - Fast model available? YES (Claude via Router)

↓ Meta Kernel creates plan and publishes: "task.assigned"

↓ Programming Agent starts work
   - Requests Python analysis capability
   - Requests code generation capability
   - Requests model router for fast Claude

↓ Each request goes through Event Bus
↓ Each response comes back through Event Bus

↓ Agent completes task
↓ Publishes: "task.completed"

↓ Memory Manager stores:
   - Objective: "Optimize Python code"
   - Process: Steps taken
   - Tools: What was used
   - Model: Claude-3
   - Time: 45 seconds
   - Cost: $0.05
   - Result: Optimized code
   - Lessons: What worked

↓ User receives result
```

---

## Key Design Principles

### 1. **Zero Direct Dependencies**
✗ Bad:
```python
class Agent:
    def __init__(self):
        self.tool = SolidWorksTool()  # Direct dependency
```

✓ Good:
```python
class Agent:
    def request_capability(self, capability_name):
        # Capability Manager finds the tool
        return event_bus.request("capability.execute", capability_name)
```

### 2. **Everything Communicates via Events**
- No function calls between components
- No imports between separate organs
- Only event publishing and subscribing

### 3. **Components are Stateless**
- No component remembers state between calls
- State goes to Memory Manager
- Components are pure functions

### 4. **Plugin Interchangeability**
- Today: Claude API for text
- Tomorrow: Gemini API
- Next week: Local LLaMA
- System never breaks

### 5. **Configuration Over Code**
- All settings in config files
- No rebuilding to change behavior
- Plugins configured via JSON

---

## File Structure (PHASE 1)

```
QUEEN/
├── queen/
│   ├── __init__.py
│   └── core/
│       ├── __init__.py
│       ├── interfaces.py           # IComponent, IPlugin, Event
│       ├── metakernel.py           # Meta Kernel
│       ├── event_bus.py            # Event Bus
│       ├── router.py               # AI Model Router
│       ├── capability_manager.py   # Capability lookup
│       ├── plugin_manager.py       # Plugin lifecycle
│       ├── memory_manager.py       # Experience storage
│       ├── knowledge_manager.py    # Documentation index
│       ├── logger.py               # Centralized logging
│       └── config_manager.py       # Configuration
├── config/
│   ├── default.json                # Default configuration
│   └── plugins.json                # Plugin registry
├── registries/
│   ├── tools_registry.json
│   ├── apis_registry.json
│   ├── models_registry.json
│   └── plugins_registry.json
├── knowledge/                      # Empty (for PHASE 3)
├── memory/                         # Empty (for PHASE 7)
├── tests/
│   ├── test_interfaces.py
│   ├── test_event_bus.py
│   ├── test_metakernel.py
│   └── test_desoupling.py
├── requirements.txt
└── README.md
```

---

## PHASE 1 Success Criteria

✅ All 9 components created
✅ All communicate via Event Bus only
✅ Zero direct dependencies between components
✅ IComponent interface enforced
✅ All components configurable via JSON
✅ Comprehensive logging
✅ Unit tests verify desoupling
✅ Can swap any component for mock without breaking others

---

## Next Phases Preview

**PHASE 2**: Plugin system fully operational
**PHASE 3**: Knowledge system indexing documents
**PHASE 4**: Agents requesting capabilities
**PHASE 5**: GitHub integration
**PHASE 6**: Specialized agents
**PHASE 7**: Autonomous mode
**PHASE 8**: Self-improvement through plugins
