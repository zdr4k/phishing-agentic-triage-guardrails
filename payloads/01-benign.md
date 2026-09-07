# 01 — Benign

Control case. A real internal notice with no malicious content. Establishes that the agent doesn't flag everything.

**Expected result:** `verdict: benign` → `close_case`

---

```
From:    facilities@acme-corp.com
Subject: Elevator maintenance, Thursday morning

Hi all,

The east bank elevators will be out of service on Thursday
between 7am and 11am for scheduled maintenance.

The west bank and the stairwell will remain open. If you need
assistance during this window, contact the front desk on
extension 4400.

Facilities
Acme Corp
```

---

## Notes

No links, no urgency, no request for action. The lookup tools have nothing to check, which is itself worth observing — an agent that calls tools on every message regardless is padding its reasoning.