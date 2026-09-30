<h1 align="center">Formula 1 Championship Simulator <br>in Java</h1>

<p align="center">
  <img src="https://img.shields.io/badge/Java-17%2B-ED8B00?logo=openjdk&logoColor=white">
  <img src="https://img.shields.io/badge/Architecture-MVC%20%7C%20DAO-412991">
  <img src="https://img.shields.io/badge/GUI-Swing%20%2F%20JavaFX-2E8B57">
  <img src="https://img.shields.io/badge/Build-Maven-C71A36?logo=apachemaven&logoColor=white">
  <img src="https://img.shields.io/badge/Testing-JUnit%205-25A162?logo=junit5&logoColor=white">
  <img src="https://img.shields.io/badge/License-MIT-blue">
</p>

---

## The problem

Simulating a **Formula 1 Grand Prix season** requires modeling complex multi-variable dynamic systems where continuous physical state transitions (tire degradation, fuel burn rate, variable track evolution) interact with discrete decision-making events (pit strategy, compound selection, safety car deployments) and stochastic occurrences (driver errors, mechanical failures).

Existing educational and lightweight race simulators often fall into one of two traps: they either oversimplify the domain into pure random-number lotteries, ignoring mechanical and environmental constraints, or they hardcode tight coupling between the user interface and domain logic, preventing custom strategy injection or automated multi-season batch benchmarking.

**The goal of this project was to design and implement an extensible, decoupled, object-oriented Formula 1 simulation engine and management system in Java**, capable of modeling realistic race dynamics, executing configurable pit stop strategies, persisting championship histories, and visualizing telemetry and standings in real time.

### Setting and constraints

| Component | Specification |
|---|---|
| Language & Runtime | **Java 17 LTS** (utilizing OOP paradigms, Streams, and Records) |
| Core Architecture | **Model-View-Controller (MVC)** with decoupled event-driven observer pipelines |
| Simulation Resolution | Discrete tick/lap-based physical state engine with variable stochastic variance |
| Persistence Engine | File-based **JSON DAO / Object Serialization** for teams, drivers, tracks, and season saves |
| UI Framework | Custom desktop dashboard (Swing/JavaFX) featuring real-time timing towers and telemetry |
| Scoring System | Official **FIA Formula 1 Sporting Regulations** (25-18-15-12-10-8-6-4-2-1 + Fastest Lap bonus) |
| Test Coverage | **JUnit 5 & Mockito** unit/integration test suites for domain logic and strategy algorithms |

---

## The approach: object-oriented architecture & stochastic simulation

The software was developed progressively through domain modeling, simulation physics refinement, design pattern integration, and persistence architecture.

### 1. Domain Object Modeling & Design Patterns

The domain model abstracts physical entities and strategy behaviors through strict object-oriented encodings and classic Gang of Four (GoF) design patterns:

- **Strategy Pattern (`PitStopStrategy`, `TireCompound`)**: Isolates tire selection (Soft, Medium, Hard, Intermediate, Wet) and dynamic pit-window logic from the core lap processing engine.
- **Observer Pattern (`RaceEventListener`)**: Decouples the low-level physical simulation thread (`RaceEngine`) from GUI components, publishing real-time telemetry updates, sector times, dynamic overtakes, and pit events.
- **Factory Pattern (`DriverFactory`, `CarFactory`)**: Encapsulates attribute parsing and baseline state construction for vehicles and driver skill profiles.
- **State Pattern (`WeatherState`)**: Manages dynamic track conditions (Dry, Damp, Wet, Torrential) and applies friction/traction penalties dynamically based on track rubbering and precipitation levels.

### 2. Stochastic Lap-Time & Event Engine

Base lap time calculation $T_{\text{lap}}$ avoids pure random assignment by blending car performance metrics, driver capability ratings, fuel mass drain, and tire degradation curves into a deterministic physics baseline, modulated by a Gaussian noise envelope:

$$T_{\text{lap}} = T_{\text{base}} \cdot \left( 1 + \alpha \cdot d_{\text{tire}}^2 + \beta \cdot m_{\text{fuel}} \right) - \gamma \cdot S_{\text{driver}} - \delta \cdot P_{\text{car}} + \epsilon$$

Where:
- $T_{\text{base}}$ is the benchmark lap time for a given circuit under ideal conditions.
- $d_{\text{tire}} \in [0, 1]$ is the current cumulative tire wear ratio, decaying performance quadratically over stint length ($\alpha$).
- $m_{\text{fuel}}$ is the remaining fuel mass in kilograms, reducing weight penalty ($\beta$) linearly per lap.
- $S_{\text{driver}}$ and $P_{\text{car}}$ represent normalized driver skill and chassis overall aerodynamic/power unit efficiency.
- $\epsilon \sim \mathcal{N}(0, \sigma^2)$ introduces Gaussian performance variance modeling minor driver inconsistencies.

**Discrete In-Race Events:**
- **Mechanical Reliability / DNF**: Simulated per lap via a Poisson trial using engine reliability parameters and thermal stress factors.
- **Safety Car (SC) & Virtual Safety Car (VSC)**: Triggered by incidents, compressing field deltas and drastically altering pit window delta costs (reducing pit loss penalty by ~40%).

### 3. Strategy Simulation & Persistence

The engine evaluates pit strategies dynamically during race execution. Drivers monitor wear thresholds $d_{\text{tire}} > \theta_{\text{wear}}$, ambient weather updates, and relative track position to execute undercuts or overcuts. Complete season states, championship standings, driver records, and track profiles are serialized into readable JSON formats via custom Data Access Objects (DAOs).
