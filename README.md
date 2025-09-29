<h1>Cash-Reconciliation-Engine</h1>

This project outlines a two-stage automated system developed using Python and data science principles to drastically improve the efficiency of the firm's cash reconciliation process by minimizing manual intervention and clearing complex breaks.

<br>
<br>

**1. Data Flow and System Architecture**
The reconciliation process compares banking transactions (Bank Side) against the firm's internal investment records Book Side) daily.

Bank Side: Raw Input :Contains transaction details, including payment date, amount, currency, and unstructured text in the Comment field.
Book Side:Contains records of all firm-approved deals, deal IDs, and expected cash movements.


**Stage 1 (JE):** Journal Enrichment Screen: Data Preparation: Automates the tagging of Bank transactions with required Deal IDs.
**Stage 2 (CC):** Cash Control System Reconciliation: Automates the matching of enriched Bank lines against Book lines.

<br>
<br>

**2. Problem Statement (Simplified)**
The manual reconciliation process was inefficient due to two key bottlenecks:

**Journal Enrichment (JE) Problems:**
Manual Tagging: Raw Bank data lacks required identifiers (Deal ID, Product ID) needed for matching. This ID was previously assigned manually by analyzing fuzzy comments.
Noise/Filtering: Non-deal transactions (e.g., tax, legal fees, internal fund transfers, journal entries) flowed unnecessarily into the Cash Control system, creating false breaks.

**Cash Control (CC) Reconciliation Problems:**
Weak Auto-Match: The existing system could not handle non-exact matches, leading to high break volumes for common variances.
Fuzzy/Tolerance: Breakdowns were created solely due to currency-specific bank fees (e.g., $$$30, 24) which should be automatically cleared.
Complex Aggregation: The system failed to handle N:M matching (one large payment covering multiple book entries, or vice versa).
Time Lag: Breaks were created when settlement or currency delays caused payment dates to differ by a few days.

<br>
<br>

**3. Solution and Logic Implemented (Python/SQL)**
We built a Python pipeline consisting of two sequential stages, using pandas for data manipulation and specialized logic for solving each problem.

<br>

**Stage 1: Automated Journal Enrichment (JE)**
The goal of this stage (implemented in stage_1_enrichment.py) is to populate the Deal ID using three passes:

**1. Exclusion Filtering:**
Logic: Uses strict rules based on the Tran Type (DOTH, COTH, JRNL) and keyword lookups (TAX, FFS_, CAPITAL CALL) within the Comment field.
Result: Transactions are marked as Filtered - No Enrichment Required and bypass Stage 2.

**2. Rule-Based Enrichment:**
Logic: Iterates through a high-confidence Deal Knowledge Directory (our initial rule base) to find exact matches for specific deal names or codes in the Comment.
Result: Assigns the correct Deal ID and marks the transaction as Auto-Enriched - Rule Match.

<br>

**Stage 2: Cash Control Matching (CC)**
This stage (implemented in stage_2_matching.py) executes a hierarchy of matching rules on the enriched data to clear breaks by complexity:

**1. Exact Match (Highest Confidence):**
Logic: Checks for Difference ≈ 0.00 AND Payment Date_Bank == Payment Date_Book.

**2. Fuzzy/Tolerance Match (Fees):**
Logic: If the Difference equals the predefined currency-specific fee tolerance (e.g., $$$30 for USD), and the Tran Type is a Funding/Outflow (TRDL), mark as matched.

**3. Time-Lag Match:**
Logic: Checks for Difference ≈ 0.00 AND the dates are within a 3-day window.

**4. N:M (Aggregation) Match:**
Logic: Uses itertools.combinations to search for single Bank amounts that precisely match the sum of two or three unmatched Book amounts (and vice-versa), ensuring common keys (Fund ID, Deal ID, Currency) align.

<br>
<br>

**4. Output and Project Status**
The pipeline processes raw data and outputs a final Cash Control DataFrame (cash_control_data) where every transaction has a definitive status.
