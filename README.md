## Overview

A comprehensive internal chatbot system with Role-Based Access Control (RBAC) for the REDACTED platform. The system uses specialized sub-agents for different domains (deals, engineering, files, projects, SQL queries) with session-based memory to reduce hallucination.

---

## Main Workflow

### `RBAC Internal Chatbot - v3.json`

**Purpose:** Core chatbot system with multi-agent routing and RBAC

**Key Features:**

- **Chat Trigger:** Public endpoint that receives chat messages
- **Session Memory:** Separate memory per session ID to reduce hallucination
- **Memory Buffer Window:** Maintains context window (30 messages)
- **Chat Retriever:** Retrieves chat history with custom session key
- **HTTP Integration:** Backend API calls to `https://your_api/api/v1/project/get-projects`
- **Memory Manager:** Controls memory context with tool call capture
- **AI Agent:** Routes queries to appropriate sub-agents
- **OpenAI Model:** Uses `o4-mini` for response generation

**Technical Notes:**
- Solves hallucination by using separate memory modules per session
- Uses memory buffer window to capture both user messages and tool call context
- Loads previous session from memory for continuity

---

## Sub-Agent Workflows

### 1. Deal Agent
**File:** `RBAC Internal Chatbot - STM as Cache - Deal Agent.json`

**Purpose:** Handle deal-related, business, and commercial questions

**Scope:**
- Deal status, stage, and progress
- Pricing, margins, commissions (permission-based)
- Commercial terms, timelines, scope
- Partner/customer deal visibility
- Deal-level summaries and comparisons
- Employee details

**Restrictions:**
- Must NOT answer engineering/technical design questions
- Must NOT reveal internal system fields or IDs (deal_id, project_id, stage_id, company_id)
- Must enforce RBAC rules strictly
- Must deny access if permission is missing

**Response Rules:**
- Answer clearly if information is allowed
- Deny with exact denial message if access is restricted
- Never reveal system IDs
- Professional, business-appropriate tone
- Do NOT mention RBAC or internal logic explicitly
- Partners can see commission details; customers cannot

**Model:** GPT-4.1-mini

---

### 2. Engineering Agent
**File:** `RBAC Internal Chatbot - STM as Cache - Engineering Agent.json`

**Purpose:** Handle engineering and technical design questions

**Features:**
- Uses GPT-4.1 for complex reasoning
- Tool: `Think` for engineering context analysis
- Max tokens: 30,000 for detailed technical responses

**Model:** GPT-4.1 (primary), GPT-4.1-mini (initial processing)

---

### 3. Files Agent
**File:** `RBAC Internal Chatbot - STM as Cache - Files Agent.json`

**Purpose:** File management and retrieval queries

**Capabilities:**
- File search and lookup
- File metadata queries
- Access control based on user permissions

---

### 4. PP Files Agent
**File:** `RBAC Internal Chatbot - STM as Cache - PP Files Agent (1).json`

**Purpose:** Project Plan (PP) files management

**Capabilities:**
- Project plan document handling
- Related file retrieval
- Permission-based access

---

### 5. Project Agent
**File:** `RBAC Internal Chatbot - STM as Cache - Project Agent (1).json`

**Purpose:** Project management queries

**Capabilities:**
- Project status and details
- Project-level summaries
- Project comparison queries

---

### 6. RAG Workflow for File Upload
**File:** `RBAC Internal Chatbot - STM as Cache - RAG Workflow for file upload.json`

**Purpose:** RAG (Retrieval Augmented Generation) integration for file uploads

**Capabilities:**
- Document ingestion and processing
- Vector store integration for semantic search
- File content indexing

---

### 7. SQL Agent
**File:** `RBAC Internal Chatbot - STM as Cache - SQL Agent.json`

**Purpose:** SQL and database query handling

**Capabilities:**
- Execute SQL queries (permission-controlled)
- Database schema queries
- Data retrieval and reporting

---

### 8. MongoDB Integration
**File:** `mcp for mongodb.json`

**Purpose:** MongoDB integration via MCP (Model Context Protocol)

**Capabilities:**
- MongoDB database operations
- NoSQL query handling
- Document-based data access

---

## Architecture Summary

```
┌─────────────────────────────────────────────────────────────┐
│                    Main Chatbot (v3)                        │
├─────────────────────────────────────────────────────────────┤
│  Chat Trigger → Session Memory → AI Agent Router            │
│                          ↓                                   │
│         ┌──────────────────┬──────────────────┐             │
│         ↓                  ↓                  ↓             │
│   Deal Agent      Engineering Agent      Files Agent        │
│   Project Agent   SQL Agent              RAG Agent          │
│   PP Files Agent  MongoDB (MCP)                              │
└─────────────────────────────────────────────────────────────┘
```

## Common Stack

| Component | Technology |
|-----------|------------|
| **Orchestration** | n8n |
| **LLM** | OpenAI (GPT-4.1, GPT-4.1-mini, o4-mini) |
| **Memory** | postgres, Session-based |
| **Database** | Supabase, MongoDB |
| **Trigger** | Chat Webhook |

## RBAC Implementation

The system enforces role-based access control through:

1. **Session-based memory** - Each user session has isolated context
2. **Permission filtering** - Sub-agents receive `allowedProjects` list
3. **Field-level restrictions** - Internal IDs are stripped from responses
4. **Scope validation** - Agents refuse out-of-scope queries with denial message

---
