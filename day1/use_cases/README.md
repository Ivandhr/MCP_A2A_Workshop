# Session 4: Practical Industrial Use Cases

## Status: ✅ COMPLETE

**Completion Date:** December 17, 2025  
**Part of:** Day 1 - Advanced MCP Workshop

This session successfully applied the multi-server MCP architecture to real industrial scenarios. We built a domain-specific MES MCP server for Press 103 and demonstrated its capabilities by generating a React dashboard with AI-powered recommendations.

### Session Achievements

✅ Built complete MES MCP server for Press 103  
✅ Implemented 5 domain-specific manufacturing tools  
✅ Generated React dashboard from natural language prompts  
✅ Integrated Claude API for AI-powered recommendations  
✅ Demonstrated cross-server data queries  
✅ Showed write capabilities (agent observations to UNS)  
✅ Proved domain-specific servers outperform generic servers

---

## Overview

This session applies the multi-server MCP architecture built in Sessions 2 and 3 to a real industrial scenario. We build a **domain-specific MES MCP server** for Press 103 that combines MQTT and MySQL into unified MES-objective tools, then use it to generate a React dashboard with AI-powered recommendations.

## Goals

- Apply multi-server MCP architecture to real industrial scenarios
- Demonstrate the "domain-specific agent server" pattern
- Build a React MES dashboard from natural language prompts
- Show how dashboards can pull from multiple data sources via MCP
- Integrate Claude API into artifacts for AI-powered analysis

## Prerequisites

- Completed Session 2 (MQTT MCP Server)
- Completed Session 3 (MySQL MCP Server)
- Understanding of both data sources and their schemas

---

## What We Built

### 1. MES MCP Server for Press 103

**Location:** `day1/mes_server/`

A domain-specific MCP server scoped to a single asset (Press 103) that demonstrates the "single-asset agent" pattern. Unlike generic data access servers, this server exposes **MES-domain tools** that map directly to manufacturing operations.

#### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     MES Agent (Claude)                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   MES MCP Server (Press 103)                 │
│                                                              │
│  ┌─────────────────┐              ┌──────────────────────┐  │
│  │  MQTT Client    │              │   MySQL Client       │  │
│  │  (UNS Cache)    │              │   (MES_Lite)         │  │
│  └────────┬────────┘              └──────────┬───────────┘  │
│           │                                  │              │
│           ▼                                  ▼              │
│  Enterprise/Dallas/Press/Press 103/#    mes_lite.* tables   │
└─────────────────────────────────────────────────────────────┘
```

#### Tools Implemented

| Tool | MES Objective | Data Source |
|------|---------------|-------------|
| `get_equipment_status` | Is Press 103 running? Current state? Speed? | UNS |
| `get_active_work_order` | What are we making? Progress to target? | UNS + MySQL |
| `get_oee_summary` | How is Press 103 performing? A/P/Q breakdown | UNS |
| `get_downtime_summary` | Why has it been down? Top reasons? | MySQL + UNS |
| `log_observation` | Record agent notes to UNS | UNS (write) |

#### Key Design Decisions

- **Standalone server** — Has its own MQTT + MySQL connections (not dependent on Sessions 2/3 servers)
- **Scoped to Press 103** — `LineID = 1` in mes_lite, `Enterprise/Dallas/Press/Press 103/` in UNS
- **MES-domain tools** — Not generic query/read, but manufacturing-specific actions
- **File-based MQTT cache** — Same pattern as Session 2 for instant responses

### 2. React MES Dashboard

A two-tab dashboard generated from natural language:

**Tab 1: Real-Time**
- Equipment status banner (running/stopped)
- Current state alert with downtime reason
- OEE gauges (Overall, Availability, Performance, Quality)
- Work order progress bar
- Key metric cards (runtime, downtime, rates)
- Time breakdown visualization

**Tab 2: Recommendations**
- "Generate Recommendations" button
- Calls Claude API to analyze Press 103 data
- Returns structured recommendations:
  - Summary assessment
  - Critical issues
  - Prioritized action items (HIGH/MEDIUM/LOW)
  - Quick wins
  - Estimated OEE improvement

---

## Press 103 Data Context

### Real Data Used

| Metric | Value | Source |
|--------|-------|--------|
| Work Order | 12237611 | UNS |
| Progress | 8,576 / 57,218 lf (53.5%) | UNS |
| OEE | 0.0% (stopped) | UNS |
| Availability | 13.04% | UNS |
| Quality | 100% | UNS |
| Runtime | 1,673 min (~28 hrs) | UNS |
| Unplanned Downtime | 6,450 min (~107 hrs) | UNS |
| Standard Rate | 34,500 lf/hr | UNS |
| Actual Rate | 3,801 lf/hr (11% of standard) | UNS |

### Top Downtime Reasons (Historical)

| Reason | Events | Total Minutes |
|--------|--------|---------------|
| Washdown/Parts Change | 1,212 | 20,255 |
| Color Match | 1,251 | 7,223 |
| Running Setup | 2,423 | 7,146 |
| Waiting on Mounting | 114 | 5,787 |
| Scheduled Down - Planned | 14 | 4,025 |

---

## Session Flow (20 minutes)

### Part 1: Domain Server Concept (3 min)

**Key Points:**
- Why build a domain-specific server vs. using generic MQTT + MySQL?
- The tools map to **MES objectives**, not data sources
- Single-asset pattern — server scoped to Press 103
- Combines real-time (UNS) and historical (MySQL) in unified tools

### Part 2: Walk Through MES Server Code (7 min)

**Demonstrate:**
- README.md specification (data mappings, queries, tool definitions)
- How it builds on Session 2 MQTT patterns (caching, connection handling)
- Added MySQL connection pooling
- Tool implementations that combine both sources

**File:** `day1/mes_server/src/mes_mcp_server.py`

### Part 3: Live Demo — Query Press 103 (5 min)

**Prompts to Try:**

```
"What is Press 103 doing right now?"
```
→ Calls `get_equipment_status`

```
"How is Press 103 performing? Give me the OEE breakdown."
```
→ Calls `get_oee_summary`

```
"What have been the main causes of downtime on Press 103?"
```
→ Calls `get_downtime_summary`

```
"Analyze Press 103 and log an observation about what you find."
```
→ Calls multiple tools, then `log_observation` to write to UNS

### Part 4: Generate Dashboard (5 min)

**Prompt:**

```
"Build me a React MES dashboard for Press 103 with two tabs. 
One tab shows real-time status (OEE gauges, work order progress, 
time breakdown). The second tab has an AI recommendations feature 
that analyzes the data and suggests improvements."
```

**Demonstrate:**
- Claude generates complete React component
- Dashboard renders in artifact panel
- Real-time tab shows current Press 103 data
- Recommendations tab calls Claude API for analysis

---

## Why Domain-Specific Servers?

### Generic Servers (Sessions 2 & 3)

```
User: "What's the OEE for Press 103?"

Claude must:
1. Figure out which UNS topics contain OEE data
2. Search through mes_lite schema for relevant tables
3. Know that LineID=1 is Press 103
4. Combine data from multiple queries
5. Calculate relationships
```

### Domain Server (Session 4)

```
User: "What's the OEE for Press 103?"

Claude:
1. Calls get_oee_summary()
2. Done.
```

### Benefits

| Aspect | Generic Servers | Domain Server |
|--------|-----------------|---------------|
| Token usage | High (exploration) | Low (direct) |
| Accuracy | Variable | Consistent |
| Speed | Multiple round-trips | Single call |
| Agent complexity | Must understand data model | Just call the right tool |
| Error handling | Distributed | Centralized |

---

## Claude API in Artifacts

The Recommendations tab demonstrates calling Claude from within an artifact:

```javascript
const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1000,
    messages: [{
      role: "user",
      content: `Analyze this Press 103 data and provide recommendations...
      
      ${JSON.stringify(pressData)}
      
      Respond with JSON: { summary, criticalIssues, recommendations, quickWins }`
    }]
  })
});
```

**Key Pattern:** Request structured JSON output for reliable parsing in the UI.

---

## Files Created

```
day1/mes_server/
├── README.md           # Full specification for Cursor
├── CURSOR_PROMPT.md    # Condensed prompt to build the server
├── requirements.txt    # Dependencies
└── src/
    └── mes_mcp_server.py   # MCP server implementation
```

---

## Claude Desktop Configuration

Three servers configured after Day 1:

```json
{
  "mcpServers": {
    "mqtt-uns": {
      "command": ".../day1/mqtt_server/venv/bin/python",
      "args": [".../day1/mqtt_server/src/mqtt_mcp_server.py"]
    },
    "mysql-mes": {
      "command": ".../day1/mysql_server/venv/bin/python",
      "args": [".../day1/mysql_server/src/mysql_mcp_server.py"]
    },
    "mes-press103": {
      "command": ".../day1/mes_server/venv/bin/python",
      "args": [".../day1/mes_server/src/mes_mcp_server.py"]
    }
  }
}
```

---

## Extending the Pattern

### Multi-Asset

```python
# Instead of hardcoded Press 103:
ASSET_CONFIG = {
    "line_id": os.getenv("MES_LINE_ID", "1"),
    "uns_base": os.getenv("MES_UNS_BASE", "Enterprise/Dallas/Press/Press 103")
}
```

Deploy multiple instances with different `.env` files.

### Additional MES Objectives

| Tool Idea | Objective |
|-----------|-----------|
| `get_schedule_status` | What's planned for this shift? |
| `compare_to_standard` | How does current run compare to historical? |
| `get_quality_alerts` | Any quality holds or out-of-spec conditions? |
| `start_work_order` | Begin production on a scheduled order |
| `assign_downtime_reason` | Categorize current downtime event |

### Agent Specialization (Day 2 Preview)

- **Production Agent** — Uses MES server to monitor and report
- **Quality Agent** — Focused on quality tools and escalation
- **Maintenance Agent** — Tracks equipment health, schedules PMs
- **Coordination Agent** — Orchestrates communication between specialized agents

---

## Day 1 Wrap-Up ✅

### What We Built

| Session | Deliverable | Status |
|---------|-------------|--------|
| Session 2 | MQTT MCP Server — Generic UNS access | ✅ Complete |
| Session 3 | MySQL MCP Server — Generic database access | ✅ Complete |
| Session 4 | MES MCP Server — Domain-specific Press 103 agent tools | ✅ Complete |
| Session 4 | React Dashboard — Real-time + AI recommendations | ✅ Complete |

### Key Takeaways

1. **MCP bridges AI and industrial data** — Natural language to OT systems ✅
2. **Multi-server architecture enables rich queries** — Combine sources seamlessly ✅
3. **Domain servers beat generic servers for agents** — Right abstraction level ✅
4. **Artifacts can call Claude API** — AI-powered dashboards ✅
5. **Iterate quickly from prompt to visualization** — Minutes, not days ✅

### Success Metrics

- **3 MCP servers** built and deployed
- **12 tools** implemented across all servers
- **4 database schemas** accessible to Claude
- **1 React dashboard** generated from natural language
- **Real-time + historical data** combined seamlessly
- **Write capabilities** demonstrated (agent observations)

**Day 1 Complete!** All code tested and pushed to repository.

---

## Preview — Day 2 (Agent2Agent)

**Status:** 🚧 Coming Soon

Building on Day 1's foundation, Day 2 will extend to collaborative intelligence:

- **Agent Specialization:** Separate agents for Production, Quality, and Maintenance
- **Agent Coordination:** Agents that communicate and coordinate with each other
- **Workflow Orchestration:** Multi-agent workflows spanning different domains
- **A2A Protocol:** Standardized agent-to-agent communication
- **Collaborative Intelligence:** Multiple AI agents working together on complex industrial problems

**Prerequisites:** All Day 1 servers (MQTT, MySQL, MES) must be working

The MES server built in Session 4 serves as the foundation for specialized agents in Day 2. The `log_observation` tool demonstrates the write pattern that enables agent-to-agent communication.

---

## Code Publishing ✅

**Status:** All Day 1 code has been pushed to GitHub and is ready for use.

**Repository:** github.com/iiot-university/MCP_A2A_Workshop

### Quick Start for Students

```bash
# Clone the repository
git clone https://github.com/iiot-university/MCP_A2A_Workshop.git
cd MCP_A2A_Workshop

# Set up credentials
cp .env.example .env
# Edit .env with your credentials

# Set up each server (example for MQTT server)
cd day1/mqtt_server
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt

# Configure Claude Desktop (see individual README files)
```

### What's Included

- ✅ Complete implementation of all 3 MCP servers
- ✅ Step-by-step guides for each session
- ✅ Database schema documentation
- ✅ Requirements files with exact dependencies
- ✅ Configuration templates (.env.example)
- ✅ Cursor prompts for rebuilding servers

---

## Resources

- [MCP Documentation](https://modelcontextprotocol.io)
- [React Documentation](https://react.dev)
- [Tailwind CSS](https://tailwindcss.com)
- [Claude API Documentation](https://docs.anthropic.com)
- [ISA-95 Standard](https://www.isa.org/isa95)
