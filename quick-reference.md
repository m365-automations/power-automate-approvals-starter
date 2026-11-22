# Quick Reference — Your First Approval Flow

Keep this open while you build. Full explanations are in
`01-build-your-first-approval-flow.md`.

## The flow in 6 steps

| # | Action | Connector | Key settings |
|---|--------|-----------|--------------|
| 1 | When an item is created | SharePoint | Site: your site. List: Approval Requests |
| 2 | Start and wait for an approval | Approvals | Type: Approve/Reject – First to respond. Assigned to: approver email |
| 3 | Condition | Built-in | Outcome (from step 2) is equal to `Approve` |
| 4a | Update item — If yes | SharePoint | Id: ID from trigger. Status: `Approved` |
| 4b | Send an email (V2) — If yes | Office 365 Outlook | To: Created By Email. Subject: `Approved: ` + Request title |
| 5a | Update item — If no | SharePoint | Id: ID from trigger. Status: `Rejected` |
| 5b | Send an email (V2) — If no | Office 365 Outlook | To: Created By Email. Subject: `Not approved: ` + Request title |

## SharePoint list: Approval Requests

| Column | Type | Notes |
|--------|------|-------|
| Request title | Single line of text | Renamed from Title |
| Details | Multiple lines of text | What the requester needs |
| Status | Choice: New / Approved / Rejected / Timed out | Default: `New`. Values are case-sensitive |

No Requester column needed — use **Created By** from the trigger.

## Approval action settings

| Field | Value |
|-------|-------|
| Approval type | Approve/Reject – First to respond |
| Title | `Approval requested: ` + Request title |
| Assigned to | Approver email (your own while testing) |
| Details | Requested by, what was requested. Basic Markdown works |
| Timeout (Settings → …) | `P7D` recommended. Blank = waits up to 28 days |

## Limits — the numbers that matter

| Limit | Value | Notes |
|-------|-------|-------|
| Approval wait | **28 days max** | Microsoft's hard limit. Then the action ends TimedOut and the run shows Failed |
| Flow run duration | 30 days max | The run cap — not the approval limit |
| Timeout format | ISO 8601 | `P7D` = 7 days, `P2D` = 2 days, `PT12H` = 12 hours |
| Approver licence | Exchange Online + Teams | Approvers do **not** need a Power Automate licence |
| Flow licence | Microsoft 365 | Standard connectors only (SharePoint, Approvals, Outlook) — no premium licence needed |

## Test checklist — do this before any real approver

- [ ] Assigned to is **your own email**
- [ ] Test 1: submit item → approve in Teams → Status becomes `Approved`, approval email arrives
- [ ] Test 2: submit item → reject → Status becomes `Rejected`, "Not approved" email arrives
- [ ] Flow is **On** (top-right toggle)
- [ ] Only then: change Assigned to to the real approver

## Troubleshooting in one line each

- **Approver saw nothing:** check Junk/Deleted Items + Defender quarantine; the request is also in the Teams Approvals app regardless of email.
- **Update item failed:** Status value must match the Choice column exactly (`Approved`, not `approved`); Id must be the ID from the trigger.
- **Run shows Failed / TimedOut:** nobody answered within the Timeout (or 28 days). Set Status to `Timed out` by hand; set a `P7D` Timeout; don't re-run the old run expecting the old approval to wake up.
- **Flow never fires:** check the flow is On and the trigger's List Name is Approval Requests.

## Want more?

- **Approvals Flow Pack ($49)** — 5 ready flows, multi-stage approvals, audit log, timeout escalation, Teams templates: https://yossimeisner.gumroad.com/l/wrkleo
- **Complete Library ($149)** — all six packs: https://yossimeisner.gumroad.com/l/npzrgk
- **Store:** https://yossimeisner.gumroad.com
