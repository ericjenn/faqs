---
marp: true
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
header: 'FSS WG - HLA'
footer: '05/05/2025'
style: |
  section.centered {
    display: flex;
    flex-direction: column;
    justify-content: center;
    text-align: center;
  }
  img[alt~="center"] {
    display: block;
    margin: 0 auto;
  }
  section {
    font-size: 25px;
  }
  div.twocols {
    margin-top: 35px;
    column-count: 2;
  }
  div.twocols p:first-child,
  div.twocols h1:first-child,
  div.twocols h2:first-child,
  div.twocols ul:first-child,
  div.twocols ul li:first-child,
  div.twocols ul li p:first-child {
    margin-top: 0 !important;
  }
  div.twocols p.break {
    break-before: column;
    margin-top: 0;
  }
---

<!-- _class: centered -->
# The **H**igh **L**evel **A**rchitecture
### Overview

---
### /!\ Warning /!\

I am not an expert of HLA *at all.*
This presentation is aimed at providing some (hopefully useful) inputs to the WG...

---
### Agenda
- A few words about *federated* and *distributed* simulations
- The history of HLA
- The concepts of HLA
- The implementations of HLA
- HLA *vs.* DDS
- HLA today
- The Real Time Infrastructure (RTI)
  
---
### What is a federated simulation?

#### Distributed simulation
- Multiple simulation components running on separate physical or virtual machines and working together to execute a unified simulation.

#### Federated simulation
- Multiple **independent** simulation systems (federates) **interoperating** within a common framework called a federation.
- A federation can be seen as a specific case of a distributed simulation where "simulation components" are actually "independent simulators"...

<!-- 
[ril-04] G. F. Riley, M. H. Ammar, R. M. Fujimoto, A. Park, K. Perumalla, and D. Xu, ‘A federated approach to distributed network simulation’, ACM Trans. Model. Comput. Simul., vol. 14, no. 2, pp. 116–148, Apr. 2004, doi: 10.1145/985793.985795.

[hui-16] W. Huiskamp and T. van den Berg, ‘Federated Simulations’, in Managing the Complexity of Critical Infrastructures: A Modelling and Simulation Approach, R. Setola, V. Rosato, E. Kyriakides, and E. Rome, Eds., Cham: Springer International Publishing, 2016, pp. 109–137. doi: 10.1007/978-3-319-51043-9_6. 
-->

---
### What is a federation

- **federation**: A named set of federate applications and a common federation object model (FOM) that are used as a whole to achieve some specific objective. 
- **federate**: A member of a federation; a single application that may be or is currently coupled with other software applications under a federation execution data (FED) or federation object model document data (FDD), and a runtime infrastructure (RTI).

---
### What do we expect from a federated simulation?

#### Composability

#### Interoperability

---
### The Levels of Conceptual Interoperability layers Model (LCIM)

[1] A. Tolk, ‘Levels of Conceptual Interoperability’, 2003.

---
### LCIM and HLA
From [Hui]:
<img src="./imgs/lcim_hla.jpg" alt="LCIM-HLA" width="600"/>

<!-- [1] W. Huiskamp and T. van den Berg, ‘Federated Simulations’, in Managing the Complexity of Critical Infrastructures: A Modelling and Simulation Approach, R. Setola, V. Rosato, E. Kyriakides, and E. Rome, Eds., Cham: Springer International Publishing, 2016, pp. 109–137. doi: 10.1007/978-3-319-51043-9_6. -->
>

---
### The issues of federated simulation (part 1)

#### Data distribution
- How to wire data from one simulation to another (communication protocols) 
- How to interface simulators that use diverse data formats (syntax)

#### Data alignment
- How to achieve knowledge alignment [Tolk et al., 2003] on the shared data (semantic)
- How to adapt shared informations to make them consumable by the simulators with different data models,
- How to manage differences in scales (spatial and temporal)


<!--
[1] F. Cremona, M. Lohstroh, D. Broman, E. A. Lee, M. Masin, and S. Tripakis, ‘Hybrid co-simulation: it’s about time’, Softw Syst Model, vol. 18, no. 3, pp. 1655–1679, Jun. 2019, doi: 10.1007/s10270-017-0633-6.

[1] R. Fujimoto, ‘PARALLEL AND DISTRIBUTED SIMULATION’.

[1] B. P. Zeigler, Theory of Modeling and Simulation. Elsevier, 2019. doi: 10.1016/B978-0-12-813370-5.00002-X.

-->

---
### The issues of federated simulation (part 2)
#### Synchronization

##### Causality

- Events occurring within simulations must be processed with
respect to their timestamps order [Fujimoto, 2001]

#### Synchronization in time
- How to handle a consistent evolution of the simulations in time with respect to the casualty principal ?
- There are two time synchronization approaches [Fujimoto, 1998]: 
  - conservative: wait until events are safe to process
  - optimistic: allow local causality violations, but detect them and recover using rollback mechanism
  
#### Shared entities
- Concepts in the system that are represented at least in two different ways, and on which we may have concurrent access (e.g., the environment)
- Their state is common among the simulators that represent them.
  

---
### Parallel and distributed simulation
#### Objective

> "A synchronization algorithm is required to ensure that the parallel execution of
the simulation produces exactly the same results as a sequential execution on a single processor. In some cases approximate results are acceptable, but the bulk of the research in synchronization algorithms has focused on producing exactly the same results. One can show that this can be achieved by ensuring that
each LP processes events in timestamp order."

---
### The algorithms of distributed simulation
#### First generation algorithms
- block the execution of an LP until it can guarantee that an event with a smaller timestamp will not later be received.
- If each FIFO queue contains at least one message, the LP can simply pick the smallest timestamped message, remove it from its queue, and process it. If one or more FIFO queues are empty, the LP must block.
- Problem: deadlock due to waiting loops
- Solution: each LP sends *lookhead* messages $t$ stating that any future message will have a timestamp of at least $t$.
- Problem: if the lookahead is small, there will be a lot of null messages (*lookahead creep*)

---
### The algorithms of distributed simulation
#### Second generation algorithms
- Algorithms based on global synchronization points.
##### Conservative algorithms 
- Concept of "epochs": "each epoch involves 
  - determining which events can be safely processed without risk of an LP later receiving a smaller timestamped event"
  - processing these safe events
  - delivering the new events

---
### The algorithms of distributed simulation
#### Optimistic algorithms
- TimeWarp
- Based on rollback: if an LP processes a message timestamped 100 and later receives a message timestamps 50, it rolls back to 50.
- State variables must be restored (state saving or inverse computation)
- Messages may have been sent to other LPs: they must be rolled back (anti-messages)
- Rollback is expensive and may not always be possible (think of IOs). 
  - Concept of Global Virtual Time (GVT) lower bound of the timestamp of any future rollback that might occur.
  
---
### The history of HLA

- DoD initiative to subsume and unify the 
  - **Distributed Interactive Simulation (DIS)** standard 
    - fixed-format data exchange
  - **Aggregate Level Simulation Protocol (ALP)** standard 
    - time management, ownership transfer
- 1998: HLA 1.3
- 2000: HLA 1.4 (IEEE 1516-2000)
  - XML-based schema, imprive Data Distrivution Management (DDM) 
- 2010: "HLA evolved" (IEEE 1516-2010)
  - Modular FOM, new web A¨PI, Dynamic-link compatible API
- 2025 (?): HLA 4
  - Improved security, scalability, compatibility with cloud and containerized environments


---
### How do HLA answer the question of federated simulation


---
### Key concepts of HLA

- Federate and federation
- Runtime Infrastructure (RTI)
  - Middleware or “simulation bus” that implements the HLA standard services
  - Federates connects to the RTI via the standardized API
- Federation Object Model (FOM) and Simulation Object Model (SOM)


---
### The main components of HLA

- The HLA rules
  - Basic principles underlyting the HLA
- The Object Model Template (OMT)
  - Standard format for describing information
  - FOM and SOM
- The API
  - Interface to the RTI
- The RTI

---
### The FOM 

- Object Model Template


---
### The HLA services
- **Federation management**: Allows simulations to join or leave existing  federations, interrupt, verify and resume execution. 
- **Declaration management**: Provides the means for simulations to initiate the publication of object attributes and to subscribe to interactions and updates resulting from other simulations. 
- **Object management**: Allows for simulation to create and delete objects, produce and receive updates of interactions and individual attributes. 
- **Ownership management**: Allows for the ownership transfer of the attributes object during the execution of the federation and, implicitly, the right to change the value of the attributes. 
- **Time management**: Coordinates the advancement of logical time along with it’s relationship with real time, during the execution of the federation. 
- **Data distribution management**: Enables federations to have an efficient data distribution mechanisms with a minimum amount of data exchanged between federations. 

<!-- 
[1] E.-L. Iagăru, ‘Comparative Analysis Between High Level Architecture (HLA) and Service Oriented Architecture (SOA) in the Field of Military Modelling and Simulation’, Scientific Bulletin, vol. 27, no. 1, pp. 30–40, Jun. 2022, doi: 10.2478/bsaft-2022-0004.
-->

---
### The interface specification

---
### The RTI
- Middleware or “simulation bus” that implements the HLA standard services
- Federates connects to the RTI via the standardized API




[1] A. Akram, M. S. Sarfraz, and U. Shoaib, ‘HLA Run Time Infrastructure: A Comparative Study’, Mehran Univ. res. j. eng. technol., vol. 38, no. 4, pp. 961–972, Oct. 2019, doi: 10.22581/muet1982.1904.09.


---
### HLA RTI and DDS

[1] R. Proctor, ‘Can DDS Help Solve the Distributed Simulation Integration Challenge?’.

[SimWare RTI](loyola.com): uses DDS for the communication labyer

---
### HLA *vs.* FMI/FMU

---
### The Status of HLA
#### HLA4

[1] B. Möller, M. Karlsson, and F. Antelius, ‘HLA 4 Federate Protocol – Requirements and Solutions’.

---
### The Drawbacks of HLA
