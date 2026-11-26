
# 🤖 Automated PCB Testing Robotic Cell

![Python](https://img.shields.io/badge/Python-3.10-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Flexiv](https://img.shields.io/badge/Flexiv-Rizon_4-333333?style=for-the-badge)
![Robotics](https://img.shields.io/badge/Industrial_Robotics-0078D4?style=for-the-badge)
![Fusion 360](https://img.shields.io/badge/Fusion_360-FF6F00?style=for-the-badge&logo=autodesk&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

---

## 📖 Project Overview

Development and integration of an automated robotic cell for printed circuit board (PCB) testing using a Flexiv Rizon 4 collaborative robotic arm.

The project involved developing a centralised Python-based controller application responsible for coordinating robotic movements, managing communication with multiple PCB testing machines and automating the complete PCB handling and testing workflow.

The system integrates industrial robotics, asynchronous machine communication, custom electromechanical mechanisms and a graphical operator interface.

Mechanical modifications and supporting automation mechanisms were also designed to enable the integration of existing testing equipment into the robotic cell.

---

## 🎬 System Demonstration

<p align="center">
  <img width="46%" src="https://github.com/user-attachments/assets/b314935a-708a-4745-8a0d-cc8f9a267198" />
  &nbsp;&nbsp;&nbsp;
  <img width="46%" src="https://github.com/user-attachments/assets/47f00add-5d25-4169-848b-7f75ca7c57d4" />
</p>

<p align="center">
  <img width="65%" src="https://github.com/user-attachments/assets/8801377f-8390-4bd5-81b0-054fcda74666" />
</p>



https://github.com/user-attachments/assets/8801377f-8390-4bd5-81b0-054fcda74666


---

## ✨ Key Features

- Centralised robotic cell controller.
- Automated PCB pickup, scanning, loading and unloading.
- Concurrent management of two PCB testing machines.
- Asynchronous machine communication.
- Automated PASS/FAIL sorting.
- Configurable robotic motion sequences.
- Multithreaded application architecture.
- Robot status and fault monitoring.
- Graphical human-machine interface (HMI).
- Custom mechanical fixtures and machine interface mechanisms.
- Electromechanical PCB tray-feeding system.
- Operator-controlled fault recovery.

---

## ⚙️ System Architecture

```mermaid
flowchart TD
    H[Operator HMI] <-->|Commands and Status| S[Cell Sequencer]
    H <-->|Backend Events| R[Robot Backend]
    S -->|Motion Plan Requests| R
    R -->|Motion Status and Faults| S
    R <-->|Flexiv RDK| C[Flexiv Robot Controller]
    C --> A[Flexiv Rizon 4]
    A --> G[Gripper]
    S <-->|Serial Interface 1| M1[PCB Tester 1]
    S <-->|Serial Interface 2| M2[PCB Tester 2]
    G --> T[PCB Material Handling]
    T --> M1
    T --> M2
    T --> P[PASS / FAIL Locations]
```

The system architecture separates operator interaction, robotic motion execution, machine communication and process sequencing into individual software modules.

This modular approach enables independent monitoring of testing machines while coordinating the shared robotic arm.

---

## 🦾 Robotic System Integration

The robotic cell utilises a Flexiv Rizon 4 collaborative robotic arm for PCB manipulation and interaction with existing testing equipment.

Robot communication is implemented through the Flexiv Robotics Development Kit (RDK), enabling the controller application to execute predefined robotic motion plans and monitor robot operation.

### Robot Control Functionality

- Robot connection and initialisation.
- Servo enabling and operational status verification.
- Named motion plan execution.
- Configurable velocity scaling.
- Motion completion monitoring.
- Pause, resume and software stop.
- Gripper initialisation and tool configuration.
- Digital output control.
- Fault detection and operator-requested fault clearing.

### Motion Execution

The controller executes predefined robotic motion plans through the Flexiv RDK.

Rather than defining individual robot trajectories within the application, the central controller coordinates smaller motion sequences, referred to as micro-plans, stored within the robot controller.

This separates the high-level process logic from the low-level robotic motion implementation.

---

## 💻 Central Controller Application

The controller application was developed in Python using a modular software architecture.

The application consists of four primary components:

| Component | Responsibility |
|---|---|
| HMI | Operator interaction and system monitoring |
| Sequencer | Workflow coordination and task scheduling |
| Robot Backend | Robot communication and motion execution |
| Configuration | Machine-specific motion plans and communication settings |

### Software Architecture

```mermaid
flowchart LR
    UI[HMI Layer]
    SEQ[Sequencer]
    BE[Robot Backend]
    CFG[Configuration]
    Q[Thread-Safe Event Queue]
    M1[Tester 1 Listener]
    M2[Tester 2 Listener]
    R[Flexiv RDK]

    UI --> SEQ
    UI --> BE
    CFG --> SEQ
    SEQ --> BE
    BE --> R
    BE --> Q
    Q --> UI
    SEQ --> M1
    SEQ --> M2
    M1 -->|Test Events| SEQ
    M2 -->|Test Events| SEQ
```

The software uses background threads, thread-safe event queues and synchronisation events to coordinate robotic operations and machine communication without blocking the graphical interface.

---

## 🧠 Robotic Cell Sequencing

[View sequencer source code](LegrandPCB/backend/sequencer.py)

The sequencer is responsible for coordinating the complete PCB testing workflow.

Individual machine states are maintained independently, allowing the robotic arm to service available testing machines while other machines are performing tests.

### Machine State Tracking

The sequencer tracks:

- Machine communication availability.
- Current PCB tray slot.
- PCB scanning status.
- Machine loading status.
- Active testing status.
- Test completion events.
- PASS/FAIL results.
- Completed PCB quantities.

### Automated Testing Workflow

```mermaid
flowchart TD
    A[Select Available Testing Machine] --> B[Pick PCB from Tray]
    B --> C[Scan PCB]
    C --> D{Scan Successful?}
    D -->|No| C
    D -->|Yes| E[Insert PCB into Tester]
    E --> F[Close Testing Machine]
    F --> G[PCB Test in Progress]
    G --> H[Receive Test Result]
    H --> I[Open Testing Machine]
    I --> J{Test Verdict}
    J -->|PASS| K[Place in PASS Location]
    J -->|FAIL| L[Place in FAIL Location]
    K --> M{Remaining PCBs?}
    L --> M
    M -->|Yes| A
    M -->|No| N[Sequence Complete]
```

Each robotic operation is executed as a predefined micro-plan, allowing the controller to monitor completion before progressing to the next dependent operation.

---

## 🔄 Dual-Machine Task Coordination

A key engineering objective was to minimise unnecessary robot idle time while PCB testing machines were operating.

The controller independently tracks the state of both machines and schedules robotic handling operations according to their availability.

While one machine is performing a PCB test, the robot can perform handling operations associated with the second machine.

```mermaid
sequenceDiagram
    participant R as Robotic Arm
    participant M1 as Tester 1
    participant M2 as Tester 2

    R->>M1: Load PCB
    M1-->>M1: Perform Test
    R->>M2: Load PCB
    M2-->>M2: Perform Test
    M1-->>R: Test Result
    R->>M1: Unload PCB
    R->>R: Sort PASS / FAIL
    R->>M1: Load Next PCB
    M2-->>R: Test Result
    R->>M2: Unload PCB
    R->>R: Sort PASS / FAIL
```

This architecture allows testing operations to overlap while maintaining sequential access to the shared robotic arm.

---

## 🔌 Machine Communication

Communication with the PCB testing equipment is implemented using serial interfaces through the PySerial library.

Each testing machine has an independent communication listener operating in a background thread.

### Communication Configuration

| Parameter | Configuration |
|---|---|
| Communication interface | Serial |
| Python library | PySerial |
| Baud rate | 9600 |
| Independent communication channels | 2 |
| Serial read timeout | 0.2 seconds |
| Communication model | Asynchronous event-driven listeners |

### Machine Events

| Event | Function |
|---|---|
| SUCCESS | Confirms successful PCB scanning |
| PASS | Indicates a successful PCB test |
| FAIL | Indicates an unsuccessful PCB test |

Incoming machine messages are processed and converted into internal events used by the sequencer.

Communication availability is monitored independently for each machine, allowing the controller to determine whether it is available for operation.

Connectivity checks are designed to avoid transmitting additional probe messages during an active PCB test.

---

## 🖥️ Human-Machine Interface (HMI)

A custom graphical user interface was developed using Python and Tkinter to provide centralised control and monitoring of the robotic cell.

### Operator Functionality

- Robot connection and disconnection.
- Selection of configured machine setups.
- Start, pause, resume and stop operations.
- Robot velocity adjustment.
- Robot and gripper enabling.
- Fault clearing.
- Individual testing machine status monitoring.
- Process progress monitoring.
- Application and robot status logging.

The HMI receives asynchronous backend events through a thread-safe queue, allowing the interface to remain responsive during robotic operations.

The interface processes queued events approximately every 50 milliseconds.

<p align="center">
  <img width="48%" src="https://github.com/user-attachments/assets/acaee83c-1137-4506-bf53-7fe8415fe9ed" />
</p>

---

## 🛡️ Fault Monitoring & Operational Control

The robot backend incorporates continuous monitoring of robot operating conditions.

[View robot backend source code](LegrandPCB/backend/robot_backend.py)

### Implemented Functionality

- Background robot monitoring at approximately 10 Hz.
- Robot fault detection.
- Motion execution status monitoring.
- Robot operational status verification.
- Motion plan timeout handling.
- Pause-aware motion completion monitoring.
- Digital status indicator control.
- Controlled software stop.
- Operator-requested fault clearing and robot enabling.

### Configured Timeouts

| Parameter | Configuration |
|---|---|
| Robot motion plan timeout | 120 seconds |
| Recovery timeout parameter | 45 seconds |
| PCB test timeout parameter | 900 seconds |
| PCB scan confirmation wait | 2 seconds |

The recovery and test timeout parameters are defined within the sequencer configuration; their enforcement depends on the implemented workflow.

Automatic recovery movements are not performed by the current recovery function. Fault clearing and robot enabling remain operator-controlled.

Application-level monitoring and software stopping functions are not substitutes for safety-rated industrial robot safety systems.

---

## ⚙️ Mechanical Design & Equipment Integration

In addition to the controller application, custom mechanical assemblies and machine interface mechanisms were designed to support automated PCB handling.

Existing testing equipment required mechanical adaptation to enable reliable interaction with the collaborative robotic arm.

### Engineering Activities

- Mechanical design using Autodesk Fusion 360.
- Robotic cell layout development.
- Custom fixture design.
- Machine interface redesign.
- Electromechanical mechanism development.
- PCB positioning and presentation.
- Rapid prototyping.
- Physical integration and functional testing.


<p align="center">
  <img width="45%" src="https://github.com/user-attachments/assets/87e8e526-d43f-4f46-8088-7491818d277e" />
  &nbsp;&nbsp;&nbsp;
  <img width="45%" src="https://github.com/user-attachments/assets/9331d728-75bd-4d1b-97d3-334a988ce33a" />
</p>
<img width="576" height="808" alt="ezgif com-cut" src="https://github.com/user-attachments/assets/f23b2757-07dc-4807-bba0-2bd9d71632b7" />


---

## 📦 Electromechanical PCB Tray-Feeding System

A supporting electromechanical tray-feeding mechanism was developed to present PCB trays to the robotic arm and support automated material handling.

The mechanism was designed around a vertical tray presentation system, enabling trays to be positioned at an accessible robotic pickup location and removed following processing.

### Engineering Considerations

- Vertical load handling.
- Actuator and transmission selection.
- Lead screw-driven linear motion.
- Resistance to back-driving.
- Tray alignment and positioning.
- Mechanical stability.
- Manufacturing and assembly constraints.
- Integration with robotic pickup operations.

<p align="center">
  <img width="45%" src="https://github.com/user-attachments/assets/6f4b7fdc-f010-4261-8297-5cbcc678080b" />
  &nbsp;&nbsp;&nbsp;
  <img width="45%" src="https://github.com/user-attachments/assets/6e921961-0f1e-43ce-a921-50f5a2c06127" />
</p>


### CAD Design Files

| File | Description |
|---|---|
| [ElevatorAssembly.f3z](ElevatorAssembly.f3z) | PCB tray elevator assembly |
| [Legrand Set up model v1.f3d](Legrand%20Set%20up%20model%20v1.f3d) | Robotic cell layout and assembly model |
| [MachineOpeningMech.f3d](MachineOpeningMech.f3d) | Testing machine opening mechanism |

---

## 🔬 Testing & Validation

Functional testing was performed on individual robotic operations and machine interfaces to support integration of the automated testing workflow.

### Demonstration Videos

- [Machine 1 Interface Test](Machine1InterfaceTest.mp4)
- [Machine 5 Test](Machine5TestVideo.mp4)
- [Machine 5 Additional Test](Machine5Test2.mp4)

### Functional Validation Areas

- PCB pickup and placement.
- Robotic interaction with testing equipment.
- Machine opening and closing mechanisms.
- PCB scanning workflow.
- Serial communication.
- Robot motion plan execution.
- Mechanical alignment and integration.

The repository contains demonstration footage, mechanical design files and controller application source code.

Quantitative cycle-time, throughput and reliability measurements are not currently documented.

---

## 📁 Repository Structure

```text
PCB-Testing-COBOT/
│
├── LegrandPCB/
│   ├── main.py
│   ├── run.sh
│   │
│   ├── backend/
│   │   ├── events.py
│   │   ├── robot_backend.py
│   │   └── sequencer.py
│   │
│   ├── config/
│   │   ├── settings.py
│   │   └── sequencer_profile.py
│   │
│   └── ui/
│       └── app.py
│
├── ElevatorAssembly.f3z
├── Legrand Set up model v1.f3d
├── MachineOpeningMech.f3d
│
├── Machine1InterfaceTest.mp4
├── Machine5Test2.mp4
├── Machine5TestVideo.mp4
│
└── README.md
```

### Software Components

| File | Description |
|---|---|
| main.py | Application entry point |
| robot_backend.py | Flexiv robot communication and motion management |
| sequencer.py | Dual-machine workflow coordination |
| events.py | Internal backend event definitions |
| settings.py | Robot and serial communication configuration |
| sequencer_profile.py | Machine-specific motion plans and tray configurations |
| app.py | Tkinter graphical operator interface |

---

## 🚀 Getting Started

### Development Environment

| Dependency | Version |
|---|---|
| Python | 3.10.12 |
| Tk / Tkinter | 8.6 |
| Flexiv RDK | 1.7.0 |
| PySerial | 3.5 |

### Clone Repository

```bash
git clone https://github.com/davidrcm1/PCB-Testing-COBOT.git

cd PCB-Testing-COBOT
```

### Install Dependencies

```bash
sudo apt install python3-tk

python3 -m venv .venv

source .venv/bin/activate

pip install pyserial==3.5 flexivrdk==1.7.0
```

Flexiv RDK installation may require a compatible operating system, Python version and vendor-supported SDK distribution.

### Configure Hardware

Update the configuration files:

- LegrandPCB/config/settings.py
- LegrandPCB/config/sequencer_profile.py

Configuration includes:

- Robot serial number.
- Network interface.
- Testing machine serial ports.
- Serial baud rate.
- Robot motion plan names.
- PCB tray slot configurations.

### Launch Application

```bash
python3 -m LegrandPCB.main
```

The application requires access to a compatible Flexiv robot controller, configured motion plans and testing machine interfaces for full operation.

Physical robot operation must only be performed within an appropriately commissioned and risk-assessed robotic cell.

---

## 🔬 Potential Future Development

- Dynamic task scheduling across additional testing machines.
- Automated production statistics and throughput monitoring.
- Persistent event and fault logging.
- Automated PCB traceability and result recording.
- Machine communication simulation.
- Unit and integration testing.
- Cycle-time optimisation.
- Expanded material handling automation.

---

## 🛠️ Technologies & Engineering Skills

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![Flexiv RDK](https://img.shields.io/badge/Flexiv_RDK-333333?style=flat-square)
![Robotics](https://img.shields.io/badge/Industrial_Robotics-0078D4?style=flat-square)
![Automation](https://img.shields.io/badge/Industrial_Automation-00897B?style=flat-square)
![Multithreading](https://img.shields.io/badge/Multithreading-3776AB?style=flat-square)
![Serial](https://img.shields.io/badge/Serial_Communication-555555?style=flat-square)
![Tkinter](https://img.shields.io/badge/Tkinter-3776AB?style=flat-square)
![System Integration](https://img.shields.io/badge/System_Integration-0078D4?style=flat-square)
![CAD](https://img.shields.io/badge/CAD-FF6F00?style=flat-square)
![Electromechanical Design](https://img.shields.io/badge/Electromechanical_Design-555555?style=flat-square)
![Fusion 360](https://img.shields.io/badge/Fusion_360-FF6F00?style=flat-square)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black)

---

## 👨‍💻 Project Contributor

**David Ceballos**

Mechatronics Engineer | Robotics & Autonomous Systems

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/davidrcm1)
