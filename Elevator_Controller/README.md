# Synthesizable Verilog Elevator Controller



A commercial-grade, fully synthesizable 4-story elevator controller written in Verilog. This project goes beyond basic state machine design by implementing physical safety protocols (Phase 1 Fire Recall, Overload sensing) and advanced request routing (the LOOK Algorithm) to mirror real-world elevator physics and hardware constraints.



## 🚀 Key Features



**The LOOK Algorithm (Direction Priority):** Eliminates "up-bias" by maintaining current directional sweeps. The elevator will finish serving all requests in its current direction before reversing, mirroring real-world mechanical efficiency.

**Phase 1 Fire Recall (Emergency Override):** A non-blocking emergency state that immediately overrides passenger requests, forces the elevator safely to the ground floor, and traps the doors open for first responders.

**Overload Sensor Integration:** Integrates load-cell safety logic. If an overload is detected while the doors are open, the FSM safely holds the doors open and triggers an alarm, ignoring standard timer-close requests until the weight is cleared.

**Robust Queue Memory:** The scheduler utilizes bitwise masking and single-assignment logic to latch transient button presses without creating multiple-driver synthesis errors.

**Synthesizable FSM Architecture:** Designed to eliminate unintentional latches using master default assignments, preventing timing failures on physical FPGA/ASIC hardware.



## 🏗️ Architecture



The system is decoupled into three primary hardware modules and wrapped in a top-level motherboard:



1.  **`elevator.v` (The FSM Brain):** A 7-state Moore/Mealy machine that evaluates inputs and physically drives the motor outputs (`move\_up`, `move\_down`, `door\_open`, `door\_close`, `alarm`).

2.  **`scheduler.v` (The Queue Memory):** Latches transient button presses from the physical world into a memory vector and combinationally calculates `req\_above`, `req\_below`, and `req\_here` for the FSM.

3.  **`elevator\_system.v` (The Top-Level Wrapper):** Acts as the physical motherboard, instantiating the Scheduler and FSM and routing internal wire traces between them.

4. **`tb\_elevator\_system.v` (Self-Checking Testbench):** Simulates a full physical environment, including passenger calls, sensor triggers, overloading, and fire emergencies.



## 📊 State Machine Overview



The controller operates across 7 distinct states:

* `IDLE` (0): Resting, awaiting calls. Determines initial direction via LOOK logic.

* `MOVING\_UP` (1): Hoist motor engaged upwards.

* `MOVING\_DOWN` (2): Hoist motor engaged downwards.

* `DOOR\_OPENING` (3): Door motor engaged to push doors apart.

* `DOOR\_CLOSING` (4): Door motor engaged to pull doors together.

* `DOOR\_OPEN` (5): Resting state awaiting `timer\_done` or `overload` triggers.

* `EMERGENCY\_STATE` (6): Locked safety state for Fire Recall.



## 💻 Simulation \& Testing



The project includes a robust testbench (`tb\_elevator\_system.v`) that verifies the FSM against complex edge cases.



### Testbench Scenarios Verified:

**Scenario 1:** Normal operation (Call to Floor 2, door cycling, and timer delays).

**Scenario 2:** Overload detection (Doors refuse to close and alarm sounds until weight is removed).

**Scenario 3:** Fire Alarm (Immediate directional reversal, ground floor return, and permanent door-open trap until emergency release).

