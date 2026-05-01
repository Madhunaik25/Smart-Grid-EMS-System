# Smart Grid Energy Management System (EMS) ## 1. Project Overview This project simulates
a basic Smart Grid Energy Management System (EMS) designed to monitor total power
consumption across three primary nodes (House, Factory, and Industry). The system utilizes a
tiered threshold strategy to maintain grid stability. When power usage exceeds predefined safety
limits, the system issues warnings and systematically sheds lower-priority loads to prevent the
total power from exceeding the absolute substation limit. ## 2. System Architecture & Nodes
The grid consists of three consumption nodes. Each load acts purely as a real-time power
(wattage) adder. ### Node 1: House * TV (Priority 3) * Fan (Priority 2) * Heavy Load
(Essential/Un-sheddable) ### Node 2: Factory * Motor 1 (Essential) * Motor 2 (Essential) *
Motor 3 (Peak-Restricted) * Light (Priority 2) ### Node 3: Industry * Machine (Essential) * Heavy
Load (Essential) ## 3. Priority Classifications To maintain grid integrity during high-demand
events, specific loads are classified into shed-able priority tiers. * **Priority 3 (P3) - First Tier
Shedding:** * House: TV * **Priority 2 (P2) - Second Tier Shedding:** * House: Fan * Factory:
Light *(Note: All other listed loads are considered Priority 1 / Essential and are never
automatically shed by the EMS).* ## 4. System Parameters & Thresholds The system monitors
the sum of all active loads (TotPwr) against static hardware limits and calculated safety
thresholds. * **Substation Limit (Sublim):** 2500 W (The absolute maximum capacity of the
grid). * **Warning Threshold (SafeLim_1):** 2125 W (Calculated as 0.85 × Sublim). * **Critical
Threshold (SafeLim_2):** 2375 W (Calculated as 0.95 × Sublim). ## 5. Core Control Logic The
EMS Function Block executes the following logical routines continuously during every scan
cycle: ### A. Total Power Calculation The system calculates real-time power draw by summing
the active wattage of every node currently turned on: `TotPwr = House Loads + Factory Loads +
Industry Loads` ### B. Time-of-Use (ToU) Peak Management The system monitors the current
time of day to restrict high-draw equipment during expensive or high-demand periods. *
**Rule:** If the system is operating within a designated Peak Hour window, Factory Motor 3 is
disabled and cannot be turned on, regardless of total grid capacity. ### C. Tiered Load
Shedding Sequence The EMS evaluates TotPwr against the safety limits in a cascading
sequence to ensure the grid does not exceed 2500 W. 1. **Level 1 Check (Warning State):** *
**Condition:** If TotPwr > SafeLim_1 (2125 W). * **Action:** The system sets a Warning Flag
(e.g., triggering an HMI alarm). No loads are disconnected. 2. **Level 2 Check (P3 Shedding):**
* **Condition:** If TotPwr > SafeLim_2 (2375 W). * **Action:** The system sheds Priority 3 loads
immediately. The House TV is turned OFF. 3. **Level 3 Check (P2 Shedding):** * **Condition:**
Following the shedding of P3, the system recalculates TotPwr. If the remaining TotPwr is still >
SafeLim_2 (2375 W). * **Action:** The system sheds Priority 2 loads. The House Fan and
Factory Light are turned OFF to bring the system back to a safe state. ## 6. Sequence of
Operations (Example) 1. Operator turns on House Heavy Load, Factory Motors 1 & 2, and
Industry Machine. Total power is 2000 W. System is in Normal State. 2. Operator turns on
Factory Motor 3. Total power jumps to 2200 W. System triggers Warning State (Exceeds
SafeLim_1). 3. Operator turns on Industry Heavy Load. Total power jumps to 2450 W. System
triggers Critical State (Exceeds SafeLim_2). 4. System automatically forces House TV (P3) OFF.
Total power drops to 2400 W. 5. System evaluates power again. 2400 W is still above
SafeLim_2. 6. System automatically forces House Fan (P2) and Factory Light (P2) OFF. Total
power drops to 2300 W. 7. System stabilizes under SafeLim_2 but remains in Warning State
