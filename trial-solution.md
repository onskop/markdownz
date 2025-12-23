Solution up here. Discussion and proposals below.

If a user installs on Tuesday, their trial ends Friday. The standard report is Sunday.
**Result:** They use the app for 3 days, see **zero** value/feedback, hit a paywall on Friday, and delete the app immediately.

You **must** show the report for free if they cancel/expire. This is the **"Free Exit Interview"** strategy.

It works because of the **Reciprocity Principle**: "We analyzed your data (gave value) even though you left. Look how useful this is. Don't you want this every week?"

Here is the exact UX and Logic flow to handle this, minimizing overlap for paying users while capturing cancelling users.

---

### The Strategy: "Report on Exit"
The Graduation Report is **NOT** generated for happy, paying users. It is **ONLY** generated for users who have hit the paywall (Cancelled or Card Failed).

#### 1. The 3-Day Trial User (Tue Install -> Fri Expiry)
*   **Tue-Thu:** Logs meals. No report generated yet.
*   **Fri Morning:** User opens app. Trial is expired.
*   **The Experience:**
    1.  **Blocker:** App shows a screen (modified `Paywall01Widget`).
    2.  **Headline:** "Your 3-Day Analysis is Ready."
    3.  **Subtext:** "You've logged meals for 3 days. Before you go, see what our AI discovered about your habits."
    4.  **Button:** "View Free Report" (Not "Subscribe" yet).
    5.  **Action:** User clicks -> Report loads (generated on fly).
    6.  **The Report:** Shows the analysis.
    7.  **The Hook:** At the very bottom of the report (and a sticky button): **"Continue your journey. Subscribe for $X."**

#### 2. The 7-Day Trial User (Thu Install -> Wed Expiry)
*   **Thu-Sat:** Logs meals.
*   **Sunday:** **Standard Weekly Report** (Free). User sees value.
*   **Mon-Wed:** Logs meals.
*   **Wed Morning:** User opens app. Trial is expired.
*   **The Experience:**
    1.  **Blocker:** App shows `Paywall01Widget`.
    2.  **Headline:** "Your Trial has ended."
    3.  **Subtext:** "We've prepared a summary of your full trial week."
    4.  **Button:** "View Free Report."
    5.  **Report:** Analysis of the full 7 days.
    6.  **The Hook:** "Unlock next Sunday's report."

---

### Technical Implementation (MVP Style)

We need to modify the flow so that when a user is "Expired," we check if they have seen their "Exit Report" yet.

#### Step 1: Add a Flag
In `UsersRecord` (Firestore), add a boolean field:
*   `hasSeenGraduationReport` (default: `false`)

#### Step 2: The Paywall Logic (`Paywall01Widget`)
Modify the `Paywall01Widget` to have two "States" (you can use a PageView or conditional visibility).

**State A: The Lure (If `hasSeenGraduationReport == false`)**
*   **UI:** "Your Trial Analysis is Waiting."
*   **Button:** "Generate & View Report".
*   **Action:**
    1.  Call `actCoachMealString` (Range: `created_time` to `now`).
    2.  Call `CallCoachCall`.
    3.  **Save** the result to `CoachRecords` (mark it as "graduation" type if you want, or just date it today).
    4.  Update User: Set `hasSeenGraduationReport = true`.
    5.  Navigate to a "Report View" page (re-use your Reporting page, just passing the specific text/JSON you just generated).

**State B: The Hard Paywall (If `hasSeenGraduationReport == true`)**
*   **UI:** "Hope you enjoyed your report. To keep your Coach, please subscribe."
*   **Button:** "Subscribe".
*   **Back Button:** Returns to State A (The Lure) or re-opens the report they just generated (so they don't feel cheated that it disappeared).

#### Step 3: Happy Path (Auto-Renew)
If the user subscribes *during* the trial or auto-renews:
1.  They never hit the Paywall.
2.  They never trigger State A.
3.  They never see the Graduation Report.
4.  **Problem Solved:** No overlap for paying users. They just get the standard Sunday report.

---

### Why this is the best MVP approach:

1.  **Solves the 3-Day Void:** The user gets value exactly when they are about to churn.
2.  **Solves the Overlap:** Paying users never see the "weird" mid-week report. It keeps their experience clean (Sunday to Sunday).
3.  **Zero "Hidden" Screens:** You are just reusing the Paywall and the existing Report View.
4.  **Low API Waste:** You only generate the report if the user actually opens the app and clicks "View Report" after their trial expires. You don't auto-generate it for users who uninstalled 2 days ago.

### Detailed Flow for 3-Day User (Tuesday Install)

1.  **Tuesday:** Install.
2.  **Wednesday:** Log meals.
3.  **Thursday:** Log meals.
4.  **Friday (Trial Expired):**
    *   User opens app.
    *   `main.dart` checks entitlement. Subscription invalid.
    *   Redirect to `Paywall01Widget`.
    *   `Paywall01Widget` checks `currentUser.hasSeenGraduationReport` (it is false).
    *   **UI Shows:** "Wait! Your 3-Day analysis is ready."
    *   User clicks "Show me".
    *   **App:** Calls AI -> Generates Report -> Updates `hasSeenGraduationReport = true` -> Shows Report.
    *   User reads: "You ate well, but your protein is low."
    *   User thinks: "Cool, I want to fix that."
    *   User clicks "Subscribe" at bottom of report.
    *   **Result:** Conversion.

### Detailed Flow for 7-Day User (Thursday Install) - Subscribed

1.  **Thursday:** Install.
2.  **Sunday:** **Standard Report** generated automatically. User gets notification. Views report.
3.  **Wednesday (Trial Ends):** Credit card charged successfully.
4.  **Thursday:** User opens app.
    *   `main.dart` checks entitlement. Subscription **Valid**.
    *   User goes to Home Screen.
    *   **Result:** No interruption, no overlapping "Graduation Report." Just smooth sailing to next Sunday.


-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------


This is a classic "Product Lifecycle vs. User Lifecycle" conflict. You have a **System Cycle** (Weekly reports on Mon/Sat) and a **User Cycle** (Trial ends exactly 3 or 7 days after signup).

Since you are using FlutterFlow and Cloud Functions (Buildship) with flexible date inputs for your AI, you have the technical agility to solve this creatively.

Here are four strategies to solve the "Triple Report Overlap" problem, ranked from most conversion-focused to most UX-consistent.

---

### Solution 1: The "Graduation" Report (Recommended)
**Concept:** Treat the Trial End report differently than a Weekly Report.
Instead of giving them a standard "How you ate this week" report when the trial ends, give them a **"Future Projection"** report.

*   **How it works:**
    1.  **Signup (Thu):** User joins.
    2.  **First Saturday:** User gets a **"Mini-Checkin"**. Keep this light. Don't give the full heavy AI analysis. Just a "Great start, you logged 3 days. Here is one quick tip for Sunday."
    3.  **Trial End (Wed):** This is the **"Graduation Report"**.
        *   *Content:* "You've been with us 7 days. Based on this week, if you keep going, you will hit your goal by [Date]."
        *   *Prompt Engineering:* Instruct the AI to focus on *trends* and *potential* rather than granular meal critique. "Analyze the last 7 days and project their success."
    4.  **Next Saturday:** Resume standard Weekly Coaching.

*   **UX Perspective:**
    *   **Pros:** High conversion value. It sells the *future* (the subscription) rather than just summarizing the *past*. It prevents the content from feeling repetitive because the *angle* is different.
    *   **Cons:** Requires a specific AI prompt for "Trial End."

### Solution 2: The "Sync Shift" (The Smooth Transition)
**Concept:** The user is on their own "Personal Week" during the trial, and you shift them to the "System Week" *after* they pay.

*   **How it works:**
    1.  **Signup (Thu):** The app sets the user's "Week Start" to Thursday internally.
    2.  **First Saturday:** **Skip this report.** The user hasn't finished *their* week yet.
    3.  **Trial End (Wed - Day 7):** Generate the **Full Weekly Report**. This satisfies the "Trial Report" need.
        *   *The Twist:* At the bottom of this report, add a note: *"To align with your weekend planning, your next report will arrive this coming Saturday."*
    4.  **This Coming Saturday (Day 10):** Generate a "Bridge Report" (Thu-Fri only) or just a quick update, then start the standard Mon-Sun (or Sat-Fri) cycle.

*   **UX Perspective:**
    *   **Pros:** Zero overlap. The user gets exactly one full report right when they need to decide to pay.
    *   **Cons:** The "Bridge" period (Day 7 to Day 10) can feel slightly awkward technically, but users rarely notice.

### Solution 3: The "Instant Value" (Front-Loading)
**Concept:** Don't wait for the trial to end to show value. If the trial is 3 days, a weekly report at the end is too late (they might have already cancelled).

*   **How it works:**
    1.  **Signup (Thu):**
    2.  **Every Day during Trial:** Give a "Daily Micro-Coach." A very short, 2-sentence AI insight based on *just* today.
    3.  **Trial End (Wed):** Generate the **"Full 7-Day Analysis."** This acts as the "Grand Finale" of the trial.
    4.  **System Cycle:** If they subscribe, they simply fall into the next standard Saturday slot. If that Saturday is only 2 days away, that's fine—more value for a new paying user is rarely a bad thing.

*   **UX Perspective:**
    *   **Pros:** Constant engagement during the critical trial window. The overlap matters less because the daily insights are micro, and the weekly report is macro.
    *   **Cons:** Higher API costs (daily AI calls).

### Solution 4: The "Rolling Window" Standard
**Concept:** Abandon the concept of a "Calendar Week" (Mon-Sun) entirely for everyone.

*   **How it works:**
    *   Every user gets their report 7 days after they install, forever.
    *   If I install on Thursday, my report day is always Thursday.
    *   If I install on Monday, my report day is always Monday.

*   **UX Perspective:**
    *   **Pros:** Solves the trial overlap perfectly because the Trial End date *is* the Weekly Report date.
    *   **Cons:** You lose the "Sunday Reflection" psychology you mentioned. Users might receive a report on a Wednesday morning when they are busy at work, reducing the chance they will sit down and plan their meals.

---

### Technical Implementation Recommendation (Based on your Codebase)

Given your file structure (`actCoachMealString`, `CallCoachCall`), **Solution 1 (The Graduation Report)** combined with **Saturday Reporting** is likely your strongest bet for monetization.

**How to implement:**

1.  **Move standard reporting to Saturday.** (As you planned).
2.  **Logic Update in `Myday01HomeWidget` (or wherever the trigger is):**
    *   Check `currentUserDocument.createdTime`.
    *   If `(Now - CreatedTime) < 7 days` (User is in trial):
        *   **Disable** the automatic "Standard Saturday" report trigger.
        *   **Enable** a "Trial End" trigger (e.g., scheduled notification or a card on the home screen) that appears exactly 24 hours before the subscription renews.
3.  **Create a new prompt type in `GlobalSettingsRecord`:**
    *   Currently you have `coach_prompt_1`. Add `trial_conversion_prompt`.
    *   This prompt should feed `CallCoachCall` but instruct the AI: *"Analyze this data to convince the user why they need this coach long-term. Focus on the positive trends found in these specific days."*

**The Resulting Flow:**
1.  **Thu:** Install.
2.  **Sat:** (Day 3) -> App checks date. It is < 7 days. **No Report** (or very simple "Keep going!" message).
3.  **Wed:** (Day 7) -> **Trial End Report**. "Here is your first full week analysis. You did X well. Unlock full coaching to fix Y next week."
4.  **Sat:** (Day 10) -> **Standard Weekly Report**. "Here is how your first official weekend went..."

This removes the weird "Thursday-Saturday" partial report that creates the overlap confusion and focuses strictly on the conversion event.
