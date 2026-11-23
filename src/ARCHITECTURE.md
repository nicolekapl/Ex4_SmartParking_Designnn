# System Architecture & Object-Oriented Design (`Ex4_SmartParking_Design`)

**Course:** 61756 Software Engineering Methods (Academic Year 2026-2027)  
**Institution:** ORT Braude College of Engineering | Department of Software Engineering  
**Domain:** Smart Campus Parking, LPR, and EV Network (Continued from Exercise 1)  
**System Story:** [`../SYSTEM_STORY.md`](../SYSTEM_STORY.md)

---

## 1. Engineering Team Details

| Role | Full Name | Student ID | Braude Academic Email |
| :--- | :--- | :--- | :--- |
| **Student A (Role A)** | `[Enter Student A Full Name]` | `[Enter ID]` | `[studentA@e.braude.ac.il]` |
| **Student B (Role B)** | `[Enter Student B Full Name]` | `[Enter ID]` | `[studentB@e.braude.ac.il]` |

* **Group Number:** `G[XX]`
* **GitHub Repository URL:** `https://github.com/[Username]/Ex4_SmartParking_Design`

---

## 2. Step 1: Business Process Modeling — Activity Diagram (`Visual Paradigm`)

* **PlantUML Source:** [`diagrams/activity_appeal.puml`](diagrams/activity_appeal.puml)
* **Visual Paradigm Model:** [`SystemModel_Ex4.vpp`](SystemModel_Ex4.vpp)
* **Partitions / Swimlanes:** Separates external **Actors** (`Security Guard`, `Student / User`, `Traffic Administrator`) from the internal **System Controller** (`EnforcementControl`), using an `initial node`, `fork`/`join` (AND), `decision` (XOR) with explicit `[guards]`, `merge` (OR), and `activity final nodes`.

![Activity Diagram - Enforcement and Digital Appeal](diagrams/activity_appeal.png)

---

## 3. Step 2: Structural Modeling — ECB Class Diagram (`PlantUML` -> `Visual Paradigm`)

Following the **ECB (Entity-Control-Boundary)** architectural pattern, UI/external system interfaces are strictly decoupled from passive domain entities through specialized business controllers:

| ECB Stereotype | Classes in System | Architectural Responsibility |
| :--- | :--- | :--- |
| **`<<boundary>>`** | `StudentDashboardUI`, `GuardTerminalUI`, `AdminAppealUI`, `LprGateBoundary`, `PaymentGatewayBoundary`, `SmsGatewayBoundary` | Interfaces with external Actors and external hardware, payment, and SMS systems. Communicates **exclusively** with `<<control>>` classes. |
| **`<<control>>`** | `ReservationControl`, `EvBillingControl`, `EnforcementControl` | Specialized controllers orchestrating business rules, grace-period timers, optimistic concurrency checks, and end-to-end workflows. |
| **`<<entity>>`** | `User` (`Student`, `Faculty`, `SecurityGuard`, `TrafficAdministrator`), `GuestPass`, `Vehicle`, `ParkingZone`, `ParkingSpot`, `Reservation`, `PaymentAuthorization`, `ChargingSession`, `Violation`, `Appeal`, plus 4 UML `enum`s (`UserRole`, `SpotStatus`, `ReservationStatus`, `AppealDecision`) | Passive domain data classes storing persistent business state, associations, and domain-specific helper methods. |

* **PlantUML Source:** [`diagrams/class_diagram_ecb.puml`](diagrams/class_diagram_ecb.puml)
* **Visual Paradigm Model:** [`SystemModel_Ex4.vpp`](SystemModel_Ex4.vpp)

![ECB Class Diagram](diagrams/class_diagram_ecb.png)

---

## 4. Step 3: Behavioral Modeling — ECB Sequence Diagrams (`PlantUML` -> `Visual Paradigm`)

### 4.1 Sequence Diagram 1: Spot Reservation, LPR Gate Entry & No-Show Destruction
* **PlantUML Source:** [`diagrams/sequence_reservation_ecb.puml`](diagrams/sequence_reservation_ecb.puml)
* **Visual Paradigm Model:** [`SystemModel_Ex4.vpp`](SystemModel_Ex4.vpp)

![Sequence Diagram 1 - Reservation and LPR](diagrams/sequence_reservation_ecb.png)

### 4.2 Sequence Diagram 2: EV Charging 30-Minute Idle Penalty (`loop` & `<<create>>`)
* **PlantUML Source:** [`diagrams/sequence_idle_penalty_ecb.puml`](diagrams/sequence_idle_penalty_ecb.puml)
* **Visual Paradigm Model:** [`SystemModel_Ex4.vpp`](SystemModel_Ex4.vpp)

![Sequence Diagram 2 - EV Idle Penalty](diagrams/sequence_idle_penalty_ecb.png)

---

## 5. Independent AI Prompt & Architectural Design Critique (Part 4)

### 5.1 Verbatim AI Critique Prompt
```text
Act as a Senior Software Architect reviewing our Object-Oriented Design for the Smart Campus Parking, LPR, and EV Network system (ECB Class Diagram and UML 2.5 Sequence Diagrams). Critically evaluate our design for:
1. Concurrency and race conditions when two users attempt to reserve the same parking spot simultaneously.
2. Edge cases and loopholes in the EV charging 30-minute idle grace period and 5 NIS/hour penalty enforcement.
3. Strict adherence to ECB (Entity-Control-Boundary) separation of concerns across all boundaries, controllers, and entities.
Provide concrete architectural improvements we should apply to our classes, attributes, methods, and sequence flows.
```

### 5.2 Applied Architectural Improvements
* **Improvement 1 Applied (Optimistic Concurrency / Race Condition Prevention):** Added `version: int` and `tryReserve(expectedVersion: int): boolean` on `ParkingSpot` (alongside the nested `alt acquired == true/false` branch in Sequence Diagram 1) so simultaneous reservation requests cannot double-book the same spot.
* **Improvement 2 Applied (EV Physical Occupancy Detection):** Separated `isPluggedIn: boolean` from `isPhysicallyOccupied: boolean` on `ChargingSession` so unplugging the charging cable at minute 29 without vacating the bay still triggers the 30-minute Charger Idle Penalty (`5.0 NIS/hr`).
* **Improvement 3 Applied (Strict ECB Isolation):** Verified that all `<<boundary>>` components (`StudentDashboardUI`, `GuardTerminalUI`, `AdminAppealUI`, `LprGateBoundary`, `PaymentGatewayBoundary`, `SmsGatewayBoundary`) interact strictly with `<<control>>` classes (`ReservationControl`, `EvBillingControl`, `EnforcementControl`) and never directly manipulate `<<entity>>` state.

---

## 6. License
Distributed under the **ORT Braude College Academic License** (61756 Software Engineering Methods, 2026-2027).