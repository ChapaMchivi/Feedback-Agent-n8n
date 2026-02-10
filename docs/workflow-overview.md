### *Technical Documentation for the Feedback Agent (n8n Workflow)*

## **Overview**

The **Feedback Agent** is an automated workflow built in **n8n** that processes customer reviews submitted through a QR‑code form.  
It reads new entries from Google Sheets, extracts the relevant fields, classifies the feedback using **Google Gemini**, normalizes the output, updates the sheet, and routes each message to the correct team via Gmail.

This document provides a **technical, node‑by‑node breakdown** of the workflow, including logic, routing, and data transformations.

---

## **High‑Level Architecture**

---

# **Color‑Coded Mermaid Diagram**

```mermaid
flowchart TD

    %% ============================
    %% NODES
    %% ============================

    A[Google Sheets Trigger<br/>New Row Added]:::trigger
    B[Extract Data<br/>(Set Node)]:::process
    C[AI Categorization<br/>(Gemini + LangChain Agent)]:::ai
    D[Structured Output Parser]:::ai
    E[Normalize Labels<br/>(Set Node)]:::process
    F[Update Row in Google Sheets]:::update

    G{Category or Area Unknown?}:::decision
    H[Stop Workflow]:::stop
    I[Switch: Route by Area]:::decision

    J[Send Email<br/>Kitchen Team]:::email
    K[Send Email<br/>Delivery Team]:::email
    L[Send Email<br/>Service Team]:::email
    M[Send Email<br/>Fallback Team]:::email

    %% ============================
    %% FLOWS
    %% ============================

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G

    G -- Yes --> H
    G -- No --> I

    I -- kitchen --> J
    I -- delivery --> K
    I -- service --> L
    I -- other/unknown --> M

    %% ============================
    %% STYLES
    %% ============================

    classDef trigger fill:#FDE68A,stroke:#CA8A04,stroke-width:2px,color:#000
    classDef process fill:#BFDBFE,stroke:#1D4ED8,stroke-width:2px,color:#000
    classDef ai fill:#FBCFE8,stroke:#DB2777,stroke-width:2px,color:#000
    classDef update fill:#BBF7D0,stroke:#15803D,stroke-width:2px,color:#000
    classDef decision fill:#FECACA,stroke:#DC2626,stroke-width:2px,color:#000
    classDef email fill:#DDD6FE,stroke:#6D28D9,stroke-width:2px,color:#000
    classDef stop fill:#E5E7EB,stroke:#374151,stroke-width:2px,color:#000

```

# What the colors represent

| Stage | Color | Meaning |
|-------|--------|---------|
| Trigger | Yellow | Workflow start |
| Processing nodes | Blue | Data extraction + normalization |
| AI nodes | Pink | Gemini + LangChain logic |
| Sheet update | Green | Writes back to Google Sheets |
| Decision logic | Red | Routing conditions |
| Email notifications | Purple | Team‑specific outputs |
| Stop node | Gray | Workflow termination |


---

If you want, I can also create:

- a **dark‑mode version**  
- a **compact horizontal layout**  
- a **Mermaid sequence diagram**  
- a **diagram that matches your brand colors**  

Just tell me the vibe you want.
```

---

## **Node‑by‑Node Breakdown**

### **1. Google Sheets Trigger**
**Type:** Google Sheets Trigger  
**Purpose:** Detects new feedback entries added to the sheet.

**Key configuration:**
- Event: `rowAdded`
- Polling: every minute
- Inputs captured:
  - ID  
  - Timestamp  
  - Name  
  - Contact  
  - Feedback  

This node initiates the workflow whenever a new row appears.

---

### **2. Extract Data (Set Node)**
**Type:** Set  
**Purpose:** Cleanly map incoming Google Sheets fields into structured JSON.

**Fields extracted:**
- `Name`
- `Contact`
- `Feedback`
- `Timestamp`

This ensures downstream nodes receive clean, predictable data.

---

### **3. AI Categorization (Gemini + LangChain Agent)**
**Type:** LangChain Agent  
**Purpose:** Classify the feedback into:

- **Category:**  
  - question  
  - problem  
  - suggestion  
  - other  
  - unknown  

- **Area:**  
  - kitchen  
  - delivery  
  - service  
  - other  
  - unknown  

**Prompt includes:**
- Validation rules  
- Fallback behavior  
- Strict label enforcement  
- No new labels allowed  

This node uses **Google Gemini** to interpret the feedback text and produce structured JSON.

---

### **4. Structured Output Parser**
**Type:** LangChain Structured Output Parser  
**Purpose:** Enforce predictable JSON output.

**Schema example:**
```json
{
  "category": "question",
  "area": "kitchen"
}
```

This guarantees the AI output is machine‑readable and safe for routing.

---

### **5. Normalize Labels (Set Node)**
**Purpose:** Convert AI output to lowercase for consistency.

**Example:**
- `"Kitchen"` → `"kitchen"`
- `"Unknown"` → `"unknown"`

This prevents routing errors caused by inconsistent capitalization.

---

### **6. Update Feedback Row (Google Sheets)**
**Purpose:** Write the AI‑generated category and area back into the original row.

**Columns updated:**
- `Category`
- `Area`

**Matching column:** `ID`

This ensures the sheet becomes the single source of truth.

---

## 🔀 **Routing Logic**

### **7. If Node — Check for Unknowns**
**Purpose:** Detect whether category or area is `"unknown"`.

If either is unknown → skip routing and end workflow.  
If both are valid → continue to Switch node.

---

### **8. Switch Node — Route by Area**
**Purpose:** Determine which team should receive the feedback.

**Routes:**
- `kitchen` → Kitchen Gmail node  
- `delivery` → Delivery Gmail node  
- `service` → Service Gmail node  
- Anything else → Fallback Gmail node  

This ensures each team receives only the feedback relevant to them.

---

## **Email Notification Nodes**

Each Gmail node sends a formatted message to the team email address stored in an environment variable (recommended):

```
={{ $env.FEEDBACK_EMAIL }}
```

**Email includes:**
- Feedback text  
- Category  
- Area  
- Timestamp  

This creates a clean, automated alert system for internal teams.

---

## **Security & Environment Variables**

To keep the workflow safe for GitHub:

Use environment variables for sensitive values:

```
FEEDBACK_EMAIL
GEMINI_API_KEY
GOOGLE_SHEETS_DOC_ID
GOOGLE_SHEETS_TAB_ID
```

This prevents secrets from appearing in your workflow JSON.

---

## **Testing the Workflow**

Recommended testing steps:

1. Add a new row to the Google Sheet  
2. Confirm the trigger fires  
3. Check the AI classification output  
4. Verify the sheet updates with category + area  
5. Confirm the correct team receives an email  
6. Test edge cases:
   - Empty feedback  
   - Very short feedback  
   - Ambiguous feedback  
   - Non‑English feedback  

---

## **Repository Structure**

```
Feedback-Agent-n8n/
├── assets/
│   ├── Authentication in n8n.png
│   └── feedback-agent.png
├── workflows/
│   └── feedback agent.json
├── README.md
└── workflow-overview.md
```

---

## **Future Enhancements**

- Add sentiment analysis  
- Add Slack or Teams notifications  
- Add a dashboard for analytics  
- Add multi‑language support  
- Add responsible team assignment logic  

---
