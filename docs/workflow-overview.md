---

# **workflow-overview.md**  
### *Technical Documentation for the Feedback Agent (n8n Workflow)*

---

## **Overview**

The **Feedback Agent** is an automated workflow built in **n8n** that processes customer reviews submitted through a QR‑code form.  
It reads new entries from Google Sheets, extracts the relevant fields, classifies the feedback using **Google Gemini**, normalizes the output, updates the sheet, and routes each message to the correct team via Gmail.

This document provides a **technical, node‑by‑node breakdown** of the workflow, including logic, routing, and data transformations.

---

## **High‑Level Architecture**

The workflow follows this sequence:

1. Detect new feedback in Google Sheets  
2. Extract and structure the data  
3. Classify the feedback using Gemini  
4. Normalize the AI output  
5. Update the original row  
6. Route the feedback to the correct team  
7. Send automated email notifications  

---

## **Workflow Diagram (Mermaid)**  
```mermaid
flowchart TD
    A[Google Sheets Trigger - New Row Added]
    B[Extract Data (Set Node)]
    C[AI Categorization (Gemini + LangChain Agent)]
    D[Structured Output Parser]
    E[Normalize Labels (Set Node)]
    F[Update Row in Google Sheets]
    G{Category or Area Unknown?}
    H[Stop Workflow]
    I[Switch: Route by Area]
    J[Send Email - Kitchen Team]
    K[Send Email - Delivery Team]
    L[Send Email - Service Team]
    M[Send Email - Fallback Team]
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
```

---

## **Node‑by‑Node Breakdown**

### **1. Google Sheets Trigger**
**Type:** Google Sheets Trigger  
**Purpose:** Detects new feedback entries added to the sheet.

**Configuration:**
- Event: `rowAdded`
- Polling interval: every minute  
- Captured fields:
  - ID  
  - Timestamp  
  - Name  
  - Contact  
  - Feedback  

This node initiates the workflow whenever a new row appears.

---

### **2. Extract Data (Set Node)**
**Type:** Set  
**Purpose:** Map incoming Google Sheets fields into clean JSON.

**Extracted fields:**
- `Name`
- `Contact`
- `Feedback`
- `Timestamp`

This ensures downstream nodes receive predictable, structured data.

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
- Strict label enforcement  
- No new labels allowed  
- Fallback behavior  
- Validation rules  

This node uses **Google Gemini** to interpret the feedback text and produce structured JSON.

---

### **4. Structured Output Parser**
**Type:** LangChain Structured Output Parser  
**Purpose:** Enforce predictable JSON output.

**Example schema:**
```json
{
  "category": "question",
  "area": "kitchen"
}
```

This ensures the AI output is machine‑readable and safe for routing.

---

### **5. Normalize Labels (Set Node)**
**Purpose:** Convert AI output to lowercase for consistency.

Examples:
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

## **Routing Logic**

### **7. If Node — Check for Unknowns**
**Purpose:** Detect whether category or area is `"unknown"`.

- If **yes** → stop workflow  
- If **no** → continue to Switch node  

This prevents misrouted or incomplete feedback.

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

Each Gmail node sends a formatted message to the team email address stored in an environment variable:

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

To keep the workflow safe for GitHub, use environment variables:

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
└── docs/
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
