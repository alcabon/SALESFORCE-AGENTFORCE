Based on the projected Salesforce release calendar for **Summer '26**, you must plan your Full Copy sandbox refresh to complete **before the Sandbox Preview cutoff date** in early May 2026.

Here is the breakdown of the critical dates and the specific window for your refresh.

### **Target Period for Summer '26 Preview Refresh**

To ensure your Full Copy sandbox is on the Summer '26 Preview release, you should initiate your refresh during the following window:

* **Refresh Initiation Window:** **Mid-April 2026 — Late April 2026**
* **Critical Deadline (Cutoff):** **May 2, 2026** (Projected)*
* *Note: Your refresh must be **100% completed** (not just started) by this date.*



### **Detailed Timeline for Summer '26**

| Milestone | Projected Date | Action Required |
| --- | --- | --- |
| **Pre-Release Notes** | ~April 26, 2026 | Review upcoming features. |
| **Sandbox Preview Cutoff** | **~May 2, 2026** | **DEADLINE:** Your sandbox must be fully refreshed and active on a "Preview Instance" by this date. |
| **Sandbox Preview Starts** | ~May 3–4, 2026 | Preview sandboxes are upgraded to Summer '26. |
| **Production Release** | ~June 10–14, 2026 | Production orgs are upgraded to Summer '26. |

---

### **Strategic Recommendation for Full Copy Sandboxes**

Because Full Copy sandboxes take time to copy (often 24–48+ hours depending on data volume) and have a **29-day refresh interval**, timing is critical.

1. **The "Safety" Buffer:** Do not wait until May 1st to click refresh. If the queue is busy and the copy takes 2 days, you might miss the cutoff.
2. **The 29-Day Lock:** Ensure you haven't refreshed your Full Copy sandbox in the 29 days leading up to mid-April. If you refresh on April 10th, you will be locked out until May 9th, causing you to **miss the preview entirely**.
* *Best Practice:* Stop refreshing your Full Copy sandbox after **April 1, 2026**, to ensure it is unlocked and ready for the strategic refresh in late April.



### **How to Verify Your Status**

Before refreshing, check if your Sandbox is *already* on a preview instance. Salesforce routes instances to "Preview" or "Non-Preview" locations.

1. Visit the **Salesforce Sandbox Preview Guide** (usually updated 1-2 months before release).
2. Enter your current Sandbox Instance ID (e.g., CS12, CS88).
3. **If it says "Upgrade":** Do **NOT** refresh. You are already set to get Summer '26 automatically.
4. **If it says "Non-Preview":** You **MUST** refresh before the cutoff date to move it to a Preview instance.

### **Summary Checklist**

* [ ] **Now:** Identify your Full Copy Sandbox instance (e.g., CSxx).
* [ ] **April 1, 2026:** distinct "No Refresh" zone starts (to avoid the 29-day lockout).
* [ ] **April 20–25, 2026:** Initiate your Full Copy refresh.
* [ ] **May 2, 2026:** Verify the refresh is complete.

----

To answer your question directly: **No, you do not *necessarily* need to redo it in April**, but refreshing now (February) is a gamble if your only goal is the Summer '26 preview.

You are playing "Instance Roulette."

Here is the logic you need to decide whether to refresh now or wait.

### 1. The "Instance Roulette" Logic

When you refresh your Full Copy sandbox today (February), Salesforce will assign it to a specific server instance (e.g., `CS45`, `CS128`, `CS87`).

* **The Gamble:** We do not know yet if that specific instance (`CSxx`) is scheduled to be a **Preview Instance** or a **Non-Preview Instance** for Summer '26. Salesforce typically confirms this list in April.
* **Scenario A (Lucky):** You refresh now, and land on `CS12`, which turns out to be a "Preview Instance."
* *Result:* In May, your sandbox automatically upgrades to Summer '26. **No refresh needed in April.** (However, your data will be 3 months old).


* **Scenario B (Unlucky):** You refresh now, and land on `CS88`, which turns out to be a "Non-Preview Instance."
* *Result:* In May, your sandbox will stay on the old release. To get Summer '26, you **MUST refresh again** during the specific April window to move it to a Preview instance.



### 2. The "Safety Zone" Strategy

If you want to be **100% certain** you get the Preview *and* have relatively fresh data, the best strategy is:

1. **Do NOT refresh now.** (Unless you urgently need fresh production data today).
2. **Wait until mid-April.**
3. **Check the "Salesforce Sandbox Preview Guide"** (released in April). It will explicitly tell you: *"Refresh **before** May X to get Preview"*.
4. **Refresh then.** This guarantees you are routed to a Preview Instance with data that is only weeks old, rather than months old.

### Summary: "Can I refresh now?"

| If you refresh NOW (Feb)... | Will you need to refresh again in April? |
| --- | --- |
| **...and land on a Preview Instance** | **No.** (But your data is old). |
| **...and land on a Non-Preview Instance** | **Yes.** (You must refresh again to "move" to a preview server). |

**The Risk of Refreshing Now:**
The biggest risk is the **29-day lockout**.

* If you refresh **April 10th**, you are locked until **May 9th**.
* If the "Preview Cutoff Date" is **May 2nd**, you will be **locked out** and physically unable to refresh in time to get the preview.

**My Advice:**
If you can afford to wait, **wait until April 20th**. It removes the element of chance and prevents you from getting locked out of the window by accident.

Would you like me to explain how to check if your specific sandbox instance (e.g., CS45) is currently slated for preview or non-preview?
