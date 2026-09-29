# 🛡️ Process-Level Memory Visibility using Azure Monitor DCR

## 🎯 Objective

Azure Monitor may already collect CPU, memory, disk, network, and system data into the `Perf` table.

However, individual process memory usage is **not collected automatically**.

To identify which process is consuming memory, additional performance counters must be manually added through a DCR.

---

## 🟢 Step 1 — Create a Separate Process DCR

Navigate to:

Azure Portal → Azure Monitor → Data Collection Rules → Create

Create a Windows DCR dedicated to process-level monitoring.

Assign only the servers where deeper visibility is required.

---

## 🟢 Step 2 — Add Custom Performance Counters

Go to:

Data Sources → Performance Counters → Custom

Manually add:

```text
\Process(*)\Working Set - Private
\Process(*)\ID Process
```

These provide:

- Process memory consumption
- Process name
- Process ID

---

## 🟡 Step 3 — Configure Sampling

Use:

```text
300 seconds / 5 minutes
```

instead of 60 seconds for process-level telemetry.

`Process(*)` collects data for every running process, so shorter intervals across many servers can significantly increase Log Analytics ingestion.

Recommended:

```text
Process DCR
→ 5-minute sampling
→ Selected servers only
```

---

## 🟢 Step 4 — Configure Destination

Send the data to:

```text
Log Analytics Workspace
→ Perf
→ Microsoft Sentinel
```

---

## 🔴 Step 5 — Check Existing DCR

Keep the existing performance DCR enabled for:

```text
CPU
Memory
Disk
Network
System
```

Do **not** disable the entire existing DCR.

If it already contains:

```text
\Process(_Total)\Working Set - Private
```

and the new DCR collects:

```text
\Process(*)\Working Set - Private
```

review the `_Total` counter and remove the overlapping counter from the old DCR if it is no longer required.

This helps avoid duplicate telemetry and unnecessary ingestion.

---

## 🟢 Step 6 — Verify Collection

```kusto
Perf
| where TimeGenerated > ago(30m)
| where CounterName == "Working Set - Private"
| where InstanceName !in ("_Total", "Idle")
| summarize
    Samples = count(),
    LastReceived = max(TimeGenerated)
    by Computer, InstanceName
| order by LastReceived desc
```

You should now see individual processes such as:

```text
node
java
w3wp
sqlservr
MsSense
MsMpEng
svchost
```

---

## ✅ Final Design

```text
Existing Performance DCR
→ CPU / Memory / Disk / Network / System
→ Existing sampling

Process DCR
→ Process(*) Working Set - Private
→ Process(*) ID Process
→ 5-minute sampling
→ Selected servers
```

## 💡 Key Takeaway

Having:

```text
AMA + DCR + Perf
```

does not automatically provide process-level visibility.

The required process counters must be **manually added to the DCR**.

This allows Sentinel to move from:

**High memory detected**

to:

**High memory detected → Top consuming processes identified**
