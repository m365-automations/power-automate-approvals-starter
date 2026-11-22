# Build Your First Approval Flow

**What you will build:** an automated cloud flow that watches a SharePoint
list, sends each new request to one approver in Teams, and writes the decision
back to the list.

**Time:** about 30–45 minutes, including testing.

**You will need:** a Microsoft 365 licence that includes Power Automate, a
SharePoint site where you can create a list, and one approver (use yourself
for testing — see the test-first box below).

---

> **TEST FIRST — read this before you touch anything**
>
> Build the whole flow with **yourself** as the approver. Submit a test
> request, approve it, then submit a second one and reject it. Only change
> **Assigned to** to a real approver after both tests behave the way you
> expect. Never point a first draft at your manager and hope for the best.

---

## How it works

```text
Staff member adds an item to the SharePoint list
        |
        v
Flow starts automatically ("When an item is created")
        |
        v
"Start and wait for an approval" — approver gets a card in Teams + an email
        |
        v
Approver clicks Approve or Reject
        |
        v
Flow writes the decision back to the list (Status = Approved / Rejected)
and emails the requester the outcome
```

## Part 1 — Create the SharePoint list

1. Go to your SharePoint site → **New** → **List** → **Blank list**. Name it
   `Approval Requests`.
2. The list already has a **Title** column. Rename it to **Request title**
   (List settings → click **Title** → rename). Keep it as a single line of
   text.
3. Add a **Details** column: **+ Add column** → **Multiple lines of text**.
   This is where the requester explains what they need.
4. Add a **Status** column: **+ Add column** → **Choice**. Type these choices
   exactly, one per line:
   - `New`
   - `Approved`
   - `Rejected`
   - `Timed out`
5. Set the **Default value** of Status to `New`.
6. Save the list. You do not need a Requester column — the list already
   records who created each item (**Created By**), and the flow uses that.

Your list is ready. Do not add items to it yet — the flow will be the thing
that reacts to them.

## Part 2 — Create the flow

1. Go to [make.powerautomate.com](https://make.powerautomate.com) → **Create**
   → **Automated cloud flow**.
2. Name it `Approvals – Starter`.
3. In the trigger search box, type `When an item is created` and pick the
   **SharePoint** trigger **When an item is created**. Click **Create**.
4. In the trigger card:
   - **Site Address:** pick your SharePoint site from the dropdown.
   - **List Name:** pick **Approval Requests**.
5. Leave everything else alone. Save the flow (top right) before going on —
   saving early means a lost connection doesn't cost you the whole build.

## Part 3 — Start and wait for an approval

Add the action **Start and wait for an approval** (Approvals connector).
Fill it in:

- **Approval type:** `Approve/Reject – First to respond`
- **Title:** type `Approval requested: ` then click the field, go to
  **Dynamic content**, and select **Request title** (from the trigger). The
  title is what the approver sees first — make it identify the request.
- **Assigned to:** type the approver's email address. **For your first build,
  type your own email address.** You will change this after testing.
- **Details:** paste the following, inserting dynamic content where shown.
  Details supports basic Markdown, so `**bold**` and line breaks work.

```text
Requested by: [Created By DisplayName — from the trigger]

What was requested:
[Details — from the trigger]

Link: open the Approval Requests list and find this item by its title.
```

- **Item link (optional):** if the action shows an **Item link** field, paste
  the link to the SharePoint item here so the approver can open it in one
  click. To get the link, open the item in SharePoint and copy the address
  from the browser. If your action does not show this field, skip it — the
  approval works without it.

Save the flow again.

### What the approver sees

The approver gets two copies of the same request, and answering either one
completes the approval:

- **Email:** from Microsoft Power Automate, with the title, the details you
  wrote, and **Approve** / **Reject** buttons.
- **Teams:** an Adaptive Card in the **Approvals app** in Teams, with the same
  title and details, Approve and Reject buttons, and a comments box.

What the card shows is what you put in **Title** and **Details** — nothing
else. Keep the title specific ("Approval requested: New laptop for Sara")
and put the facts in Details, not in a separate document the approver has to
open.

What the card does **not** do: you cannot redesign this built-in card. If you
ever want a fully custom card layout, that is a different action — the Teams
action **Post adaptive card and wait for a response** — and it is beyond this
starter. For one approver and one decision, the built-in card is the right
tool.

## Part 4 — Branch on the outcome

The flow is waiting on the approver. Now tell it what to do with the answer.

Add a **Condition** control, directly under the approval action:

- **Left box:** click the field, go to **Dynamic content**, and select
  **Outcome** (from Start and wait for an approval).
- **Middle dropdown:** **is equal to**
- **Right box:** type `Approve`

### If yes (approved)

Add these two actions inside the **If yes** branch:

1. **Update item** (SharePoint connector).
   - **Site Address:** your SharePoint site.
   - **List Name:** **Approval Requests**.
   - **Id:** click the field, go to **Dynamic content**, and select **ID**
     (from the trigger). This tells the flow which item to update.
   - **Status:** select `Approved` from the dropdown. Type it exactly if
     you type it by hand — Choice columns are case-sensitive, and `approved`
     is not the same as `Approved`.
   - Leave Title and Details alone — the flow only changes what you fill in.
2. **Send an email (V2)** (Office 365 Outlook connector).
   - **To:** click the field, go to **Dynamic content**, and select
     **Created By Email** (from the trigger).
   - **Subject:** type `Approved: ` then insert **Request title**.
   - **Body:** write something like this, inserting dynamic content where
     shown:

```text
Your request "[Request title]" was approved.

Approver comments:
[Responses comments — from Start and wait for an approval]
```

> Power Automate may wrap the email step in an **Apply to each** when you
> insert **Responses comments**. That is normal: the approval's responses are
> a list, one entry per approver, and with a single approver the loop runs
> once. Leave it as it is.

### If no (rejected)

Add the same two actions inside the **If no** branch, with these changes:

1. **Update item** — identical, except **Status:** `Rejected`.
2. **Send an email (V2)** — identical, except the subject starts
   `Not approved: ` and the body says the request was not approved, followed
   by the approver's comments.

Save the flow. Your flow now has four actions after the trigger: the
approval, the Condition, and two actions in each branch.

## Part 5 — Save and test

1. Click **Save**, then **Test** → **Manually** → **Save & Test**.
2. Go to the **Approval Requests** list and add an item:
   - **Request title:** `TEST — please approve`
   - **Details:** `Testing the starter flow. Please approve this one.`
   - Leave **Status** as `New`.
3. Check that:
   - [ ] You get the approval in Teams (Approvals app) and by email.
   - [ ] Clicking **Approve** sends the requester (you) an "Approved" email.
   - [ ] The list item's Status changes to `Approved`.
4. Add a second test item (`TEST — please reject`) and click **Reject**.
   Check that:
   - [ ] You get a "Not approved" email.
   - [ ] The list item's Status changes to `Rejected`.
5. Only now, change **Assigned to** in the approval action to the real
   approver's email, save, and tell them the first real request is coming.
6. Check the flow is **On** (top-right toggle on the flow details page). A
   flow that is off never fires, and it fails silently — no error, no email,
   nothing in the run history.

If a test step fails, go to the flow's **Run history** (28-day history on the
flow details page), open the failed run, and read which action has the red
exclamation mark. The troubleshooting section below covers the three failures
you are most likely to see.

## The 28-day limit — the one number to remember

An approval in Power Automate can wait for **28 days at most**. That is
Microsoft's hard limit for approvals — not 30 days.

What happens at 28 days:

- The approval action ends with status **TimedOut**, and the flow run is
  marked **Failed** unless you handle the timeout.
- The approval itself stays in the Approvals center with nobody waiting on
  it. It does not disappear, and it does not approve itself.
- Separately, a single cloud flow run cannot last more than **30 days** in
  total. That is the run cap, not the approval limit — do not mix the two
  numbers up.

What to do about it:

- **Set a Timeout on the approval action.** Click the **…** on **Start and
  wait for an approval** → **Settings** → **Timeout**. Enter `P7D` (7 days,
  ISO 8601 format: `P` for period, `7D` for 7 days). Seven days is a sane
  default for a staff request — long enough for a holiday, short enough that
  a forgotten request doesn't sit for a month. Other values that work the
  same way: `P2D` for 2 days, `PT12H` for 12 hours.
- **Check the run history weekly** while you are learning the flow. A timed-out
  run shows up as Failed — that is the signal to chase the approver, not a
  sign the flow is broken.
- **Do not rely on the 28-day default.** An approval that waits four weeks
  is a process problem, not a flow problem. Agree a business deadline with
  the approver (two business days is a good starting point for most
  requests) and set the Timeout to match it.
- If you want the flow to do something automatic on timeout — email the
  requester, escalate to a second approver — that needs a parallel branch
  configured to run after **Has timed out**. That pattern is in the full
  Approvals Flow Pack (Flow 3, purchase approval), not in this starter.

## Troubleshooting — the 3 most common failures

### 1. The approver never saw the approval

**What it looks like:** the flow run sits waiting, the approver says nothing
arrived.

**Most likely causes, in order:**

1. **The email went to junk.** Approval emails come from Microsoft Power
   Automate and are often caught by spam filters or Defender quarantine,
   especially in the first weeks. Ask the approver to check **Junk Email**
   and **Deleted Items** in Outlook, and check quarantine in the Microsoft
   365 Defender portal (**Review** → **Quarantine**) and release the message
   if it is there.
2. **The approver is looking in the wrong place.** The approval waits in the
   **Approvals app** in Teams whether or not the email arrived. Open Teams
   → **Approvals** — the request is there.
3. **The flow is off, or the trigger points at the wrong list.** Check the
   flow is **On** (top-right toggle on the flow details page) and that the
   trigger's **List Name** is **Approval Requests**, not another list.

**Prevention:** add the sender to Safe Senders in Outlook, and if it keeps
happening, ask your email admin for a mail flow rule that bypasses spam
filtering for approval notifications.

### 2. "Update item" fails — Status never changes

**What it looks like:** the approver answers, the email goes out, but the
list item stays on `New` — and the run history shows **Update item** failed.

**Most likely causes, in order:**

1. **The Status value doesn't match the Choice column exactly.** Choice
   columns are case-sensitive: `Approved` works, `approved` fails. Open the
   list settings, check the exact choice text, and make sure the flow writes
   exactly that.
2. **The Id is wrong.** The **Update item** step must use the **ID** dynamic
   content from the trigger. If you typed a number by hand, or picked an ID
   from a different action, the flow updates the wrong item or fails.
3. **A required column is empty.** If you later add a required column to the
   list, Update item fails until the flow supplies it. Either fill it in the
   flow or make the column optional.

**Fix:** correct the value, re-run the failed flow from run history
(**Resubmit**), and confirm the Status changes.

### 3. The approval timed out and the run shows Failed

**What it looks like:** the run history shows a Failed run, and the approval
action has status **TimedOut**.

**What it means:** nobody answered within the Timeout (or within 28 days if
you set no Timeout). This is expected behaviour, not a broken flow — but a
run that shows as Failed is easy to miss, and the approval is now sitting in
the Approvals center with nobody waiting on it.

**Fix:**

1. Decide what should have happened: chase the approver, or treat the
   request as declined.
2. Update the list item's Status by hand to `Timed out` so the request
   doesn't look like it is still waiting.
3. Set a Timeout on the approval action (`P7D`, see "The 28-day limit"
   above) so unanswered requests surface after a week instead of a month.
4. If the approver still wants to decide, delete the stale approval in the
   Approvals center and ask the requester to submit the item again — do not
   just re-run the old flow run and expect the old approval to wake up.

## What to do next

- Run the starter for a week with real requests and one approver. Watch the
  run history. Fix whatever the troubleshooting section didn't cover by
  reading the failed action's error message — it usually names the field.
- When you want a second approver, an audit log, or timeout escalation, that
  is the full **Approvals Flow Pack ($49)**:
  https://yossimeisner.gumroad.com/l/wrkleo
- When you want the same treatment for onboarding, reports, helpdesk, and
  PowerShell, the **Complete Library ($149)** covers all six packs:
  https://yossimeisner.gumroad.com/l/npzrgk
