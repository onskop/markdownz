


Taking inspiration from bleeding-edge manufacturing concepts—specifically **Pharma 4.0** and the **Digital Obeya** (Toyota’s "Big Room" concept modernized into digital control towers by companies like Tesla and modern biopharma giants)—we can design a UI that is highly visual, exception-based, and friction-free. 

In these advanced environments, the UI principle is **"Management by Exception."** You don't display everything that went right; you only display what deviated, what needs attention, and the trend lines.

Here is a comprehensive UI/UX design and use-case breakdown for your interactive Next.js/React application, incorporating your updated meeting structure and the "open permissions + audit log" approach.

---

### Global UI/UX Principles for the App
1. **The "Digital Obeya" Layout:** Every Tier dashboard uses a grid layout. Top row: Mini-charts (Midas SQL pulls) and S/Q/D/C counters. Main body: The Event Cards. 
2. **Exception Highlighting:** Green data is muted or hidden. Red/Yellow data (DEVs, delays, injuries) is visually popped using high-contrast borders or glows.
3. **One-Click Actions:** No drop-downs if we can avoid them. Use icon buttons directly on the cards for state changes (e.g., ✅ Resolve, 🚀 Escalate, 📌 Park).
4. **Color-Coded S/Q/D/C Tags:** Every card has a permanent, bright tag: `[Safety]` (Blue), `[Quality]` (Yellow), `[Delivery]` (Green), `[Cost/Tech]` (Purple). 

---

### The Screens & Use Cases

#### 1. The Data Ingestion View (EOD Shift Prep)
**Use Case:** The SUP is finishing their shift and needs to compile the report fast.
**UI Design:** A split-screen layout.
*   **Left Side (Plan):** A list auto-populated from the Excel import. 
    *   *Interaction:* SUP simply clicks a thumbs-up 👍 or thumbs-down 👎 next to each planned item. Thumbs-down instantly slides out a quick-add form to create an Event Card.
*   **Right Side (Ad-hoc Events):** A clean input form for things not on the plan (Alarms, injuries, PST requests). 
*   **Bottom Bar:** KPI inputs (e.g., "Batch Records handed over? [Yes/No]").
*   *Tech Note:* Because permissions are open, the system logs `[Created by SUP_Name at 18:45]`.

#### 2. T1: The Frontline Stand-up (Morning)
**Use Case:** Quick, 10-minute alignment on the shop floor. What happened? What do we need today?
**UI Design:** "The Focused Board."
*   **View:** Displays only the Events from the immediate previous shift for that specific department.
*   **Interaction:** 
    *   The team reviews the cards.
    *   SUP clicks **[Acknowledge]** on informational notes (hides them from active view).
    *   SUP clicks **[Escalate to T2]** on blockers they cannot solve. This triggers a modal asking for a 1-sentence "Ask" (e.g., *Need PST to verify valve pressure*).

#### 3. T2: The Tactical Routing (3 Separate Meetings)
**Use Case:** Mid-level management and PST making fast decisions. 
**UI Design:** "The Triage Board." 
Since there are 3 distinct meetings (CC & USP, MP & EP, DSP), the T2 dashboard has a global toggle at the top to switch between these three "Rooms".
*   **The View:** Only shows cards marked "Escalated to T2" by the respective T1s, plus any lingering "Open" issues from previous days.
*   **The "Executive Wrapper" Feature:** 
    *   When a T2 manager clicks a T1 card, a side-panel opens. It shows the raw T1 text. 
    *   Above it is a large, editable text field: **"Management Summary."** The Manager types the refined issue here. From this point on, the rest of the system (T3, Hypercare) only sees this Summary, with a small "Expand Original" icon attached.
*   **Actions (The Buttons):**
    *   **[Return to T1]**: Issue solved, feedback sent back down.
    *   **[Send to Action Board]**: See "Solving the Card Lifespan" below.
    *   **[Escalate to T3]**: Requires Director-level intervention.

#### 4. T3: The Strategic Obeya (Director Level)
**Use Case:** 15-minute high-level alignment. Are we hitting targets? What are the massive blockers?
**UI Design:** "The Command Center."
*   **Top Row (Mini-Charts):** Custom React charts pulling SQL Midas data + App data. 
    *   *S:* Days without incident (Number).
    *   *Q:* DEV trendline (Line chart, 7-day rolling).
    *   *D:* OTD % Gauge.
*   **Main Body:** Only displays "Wrapped" cards escalated from T2. 
*   **Interaction:** Very limited card editing. T3 is about assigning broad resources. Cards are either marked **[Resolved]** or **[Parked/Ongoing]**.

#### 5. Hypercare: The Deep Dive (Daily Tech/PST Meeting)
**Use Case:** Highly technical daily sync between Prod and PST during ramp-up. They don't care about HR issues or minor delays; they care about process stability.
**UI Design:** "The Engineering View."
*   **The Data:** This dashboard bypasses the tiered escalation completely. It automatically pulls *any* Event Card in the system (from T1, T2, or T3) that has been tagged with `[Process]`, `[Tech]`, or `[Quality DEV]`.
*   **Integrations:** This page features the heaviest use of the SQL Midas data. Large, interactive line charts for Yield, CO2, Density, and ATF flows. 
*   **Interaction:** PST can append "Technical Notes" to cards, which will be visible to the SUPs back down at the T1 level on their next shift.

---

### Solving the "Card Lifespan" Dilemma (Avoiding Jira)

You rightly want to avoid building a heavy task tracking system. However, issues take time to fix. If they stay on the T2 board forever, the daily meeting becomes a 2-hour status update. 

**The Solution: The "Persistent Action Board" (The Parking Lot)**

1.  **How it works:** During a T2 or T3 meeting, if an issue is identified but will take 3 days to fix (e.g., "PST ordering a new sensor"), the manager clicks **[Send to Action Board]** and tags an owner (e.g., "@PST_John"). 
2.  **What happens:** The card *disappears* from the daily T2 meeting view (keeping the daily meeting fast and focused only on *new* exceptions). 
3.  **The UI Element:** On the right-hand edge of the T2 and T3 screens, there is a persistent, vertical slide-out drawer labeled **"Action Board[ 4 Open ]"**. 
4.  **Review Rhythm:** Once a week (perhaps during the Weekly Manufacturing Meeting), the team opens the Action Board drawer and reviews these lingering items. Once John replaces the sensor, he finds the card in the drawer, clicks ✅ **[Close]**, and it permanently archives.

### Summary of the Data Flow UX

1.  **Excel** -> (Imported) -> **Phase 0 Prep**
2.  **Phase 0 Prep** -> (SUP adds context) -> **T1 Board**
3.  **T1 Board** -> (Escalate) -> **T2 Rooms (CC+USP | MP+EP | DSP)**
4.  **T2 Rooms** -> (Wrap & Escalate) -> **T3 Board**
5.  **All Tiers** -> (Filtered by Tech/DEV tags) -> **Hypercare Board**
6.  **T2/T3** -> (Long-term fix required) -> **Side-Drawer Action Board**

This Next.js app will feel incredibly lightweight to the users because they are only ever looking at the exact slice of data they need for that specific 15-minute window, yet all the data is perfectly linked in the background.
