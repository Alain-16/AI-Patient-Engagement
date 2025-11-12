# 🧠 AI Patient Engagement Copilot

A WhatsApp-based assistant designed to **streamline the hospital experience** for patients and staff.  
It tracks the patient’s journey, sends real-time updates, simplifies billing, delivers results, and provides guided hospital navigation—all through automated WhatsApp messages.

---

## 📖 1. Overview

The **AI Patient Engagement Copilot** reduces confusion, desk traffic, and manual effort while enhancing transparency and patient satisfaction.

**Tech Stack:**  
OpenMRS (Bahmni) · Apache ActiveMQ · n8n (Docker) · OpenAI (ChatGPT) · WhatsApp Cloud API

---

## 🏗️ 2. How It Works (High-Level Architecture)

### 🔄 Data Flow

1. **OpenMRS** publishes clinical events (e.g., encounter created).  
2. **ActiveMQ** receives these events as messages.  
3. **n8n** listens to the relevant ActiveMQ topics and pulls corresponding Atomfeed entries.  
4. **Atomfeed** provides resource URLs, which n8n uses to fetch full data from the **OpenMRS REST API**.  
5. **OpenAI Chat model** processes and summarizes this data.  
6. **WhatsApp Cloud API** sends the final message to the patient.

---

## 💡 3. Feature Set

### ✅ Registration & Welcome
- WhatsApp number collected during Bahmni registration.  
- Instant welcome message with consultation fee and hospital info.  
- Optional hospital map, services list, or FAQs.

### ⚖️ Consultation
- Triggered when doctor places lab/imaging/pharmacy orders.  
- Sends:
  - Order summary with cost breakdown.  
  - Mobile money payment reference.  
  - Confirmation and PDF receipt.

### 🔬 Diagnostics & Results
- Notifies patient when results are validated.  
- Includes location and optional PDF lab report.

### 🏥 Admission / Inpatient
- Admission alerts with ward, daily charges, and required deposit.  
- Daily updates for family (invoices + charges).

### 🌿 Pharmacy
- Notification when prescription is ready.  
- Includes co-pay amount and payment reference.

### 🚪 Discharge
- Final WhatsApp summary including:
  - Total bill, insurance, and payment details.  
  - Combined medical + financial PDF summary.  
  - Feedback and thank-you message.

---

## ⚙️ 4. Core Components

| Component | Purpose |
|------------|----------|
| **OpenMRS + Event Module** | Generates real-time events (Encounters, Orders, etc.) |
| **ActiveMQ** | Message broker handling `CREATED`/`UPDATED` topics |
| **n8n (Docker)** | Workflow orchestrator to fetch, process, and dispatch messages |
| **OpenAI Chat API** | Summarizes clinical data into human-friendly messages |
| **WhatsApp Cloud API** | Sends final messages to patients via WhatsApp |

---

## 🔗 5. Detailed Message Workflow

### 5.1 Trigger
- Subscribes to:
  - `topic://CREATED:org.openmrs.Encounter`
  - `topic://CREATED:org.openmrs.Order`

### 5.2 Event Parsing
- Extracts fields:
  - `eventType`, `entity`, `uuid`, `dest`
- Determines message type (consultation, lab result, discharge, etc.)

### 5.3 Atomfeed URL Mapping

| Entity | Atomfeed URL |
|---------|---------------|
| Encounter | `/ws/atomfeed/encounter/recent` |
| Order | `/ws/atomfeed/order/recent` |

### 5.4 Atomfeed → REST Mapping
- Finds Atomfeed entry by `UUID`.
- Extracts REST URL for detailed resource.

### 5.5 Fetch Full Resource
```http
GET /ws/rest/v1/encounter/{uuid}?includeAll=true
```
- Retrieves full JSON object for processing.

---

## 🧠 6. Message Generation (AI Layer)

- Uses **OpenAI Chat model** to summarize JSON data.
- Prompt example:
  ```text
  You are a medical assistant creating a WhatsApp message for a patient.
  Here is the visit data in JSON format:
  {{ JSON.stringify($json, null, 2) }}
  ```
- Output:
  > "Hi Hilda, your CBC test results are ready. Please proceed to Room 12. Your co-pay of RWF 2,000 has been received. Thank you for visiting!"

---

## 💬 7. WhatsApp Integration

**Platform:** WhatsApp Cloud API  
**Node:** “Send Message” (n8n)

**Requirements:**
- Meta sender number  
- Patient’s phone number (E.164 format)  
- Message body from AI node output  

> ⚠️ Note: Meta test numbers require a join code.  
> For production, use approved message templates and a live WABA account.

---

## 🧩 8. Configuration & Customization

Modify key parameters in your n8n environment:

| Variable | Description |
|-----------|-------------|
| `ACTIVEMQ_HOST` | Broker host for ActiveMQ |
| `ATOMFEED_BASE_URL` | Base Atomfeed endpoint |
| `OPENMRS_BASE_URL` | REST API base URL |
| `AI_PROMPT_TEMPLATE` | Message prompt per event type |
| `SUPPORTED_ENTITIES` | e.g., Encounter, Order, ProgramEnrollment |

💡 *To support a new event type (e.g., DischargeSummary), add its feed/REST mapping and extend the AI prompt.*

---

## 🧭 9. Observability & Error Handling

**Common Failures:**
- Atomfeed entry missing for UUID  
- REST API returns `404` or `500`  
- AI response generation failure  
- WhatsApp API rejection  

**Check Logs:**
- n8n → *Execution History*  
- ActiveMQ → *Web Console (topics, queues)*  
- OpenMRS → *Application logs*

---

## 🔒 10. Security & Privacy

- Patient data flows: `OpenMRS → n8n → AI → WhatsApp`.  
- Follow **PHI/PII** privacy standards per region.  
- Store credentials securely in **n8n Credential Manager**.  
- Avoid persisting full message payloads unless anonymized.

---

## 🧭 11. Limitations & Roadmap

**Current Gaps**
- Limited to Encounter, Order, Patient events  
- Meta test number only  
- No deduplication or checkpointing  

**Planned Enhancements**
- Live billing & balances  
- Full visit summary (diagnosis + billing)  
- Multilingual support (EN, FR, RW)  
- Patient feedback & satisfaction analytics  
- AI-based triage and appointment booking

---

## 🌍 12. Why It Matters

This copilot doesn’t just send alerts — it **transforms patient engagement**.  
By automating communication, it reduces hospital congestion, simplifies billing, and improves care coordination.  
The result? **Happier patients, less workload, and smarter healthcare delivery.**

---

### 👨‍💻 Author
**Alain Solomon Mugisha**  
Software Engineer
[LinkedIn](www.linkedin.com/in/alainrw)
