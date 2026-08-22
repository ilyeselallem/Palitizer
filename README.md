🏭 Automated Palletizing Cell: Siemens S7-1200 Control Architecture



(https://makeagif.com/i/WkTqAY)

##  Executive Summary
This project demonstrates the design, simulation, and commissioning of a fault-tolerant automated palletizing cell. Moving beyond open-loop timer sequences, this control architecture utilizes closed-loop sensor verification, modular state-machine programming in SCL, and parallel diagnostic watchdogs to prevent physical hardware failure from material jams or mechanical coasting.

##  System Architecture
The system integrates a virtual **Siemens S7-1200 (CPU 1214C)** PLC with a **WinCC Basic HMI**, controlling a physical electromechanical simulation driven by **Factory I/O**.

### PLC Software Structure
The logic is highly modular, separating state management from physical output mapping:
* **`OB1 [Main]`**: Cyclic execution and routine calling.
* **`FB1_ModeManager`**: Global safety gatekeeper. Evaluates E-Stops and selector switches before granting run permissives.
* **`FB3_Elevator_Stacker`**: The core SCL state machine handling row/layer math and actuator sequences.
* **`DB_Machine`**: A centralized Global Data Block utilizing custom `UDT_SysStatus` and `UDT_SysCommand` structs to ensure clean tag exchange between the PLC and HMI.

##  Key Engineering Features

### 1. Closed-Loop SCL State Machine
The core sequence operates via a mathematically isolated `CASE` statement. Movement profiles rely on Set/Reset (SR) latches and physical sensor verification (e.g., photoelectric limit switches) rather than theoretical clock cycles, eliminating race conditions.

### 2. Parallel Watchdog Diagnostics
The system protects its own hardware. Parallel `TON` instructions execute alongside the state machine. If an actuator (such as the pneumatic pusher) fails to reach its physical limit switch within 3.0 seconds, the watchdog immediately drops the permissives, forces a safe state, and generates a specific integer fault code.

### 3. SCADA / HMI Integration
The WinCC HMI dashboard translates raw data into actionable operator insights:
* **Slice-Access Alarms**: Fault integers map directly to a `Word` via slice access (`%X1`), triggering discrete HMI alarms.
* **Secured Recovery**: A dedicated, password-protected Manual Overrides screen allows operators to safely jog individual actuators to clear physical jams.

## 📂 Repository Structure
* `/TIA_Portal_Archive/` - Contains the `.zap16` compressed project file.
* `/Documentation/` - Contains the exported PDF of the Ladder Logic and SCL blocks.
* `/IO_Mapping/` - Contains the `.xlsx` export of the physical PLC tags.
* `/Factory_IO_Scene/` - Contains the `.factoryio` environment file to run the simulation.

##  How to Run the Simulation
1. Clone this repository.
2. Unarchive the `.zap16` file using Siemens TIA Portal V16.
3. Start **PLCSIM** and download the hardware/software configuration.
4. Open the `.factoryio` scene in Factory I/O.
5. Go to *File > Drivers* in Factory I/O, select *Siemens S7-PLCSIM*, and click Connect.
6. Switch the physical panel to AUTO and press START.
