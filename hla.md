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
  ref{
    color:blue;
    font-size: 14px;
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

1 - This presentation is aimed at providing some (hopefully useful) inputs to the WG...
2 - I am **not** an expert of HLA.


---
### Agenda
- What do we mean by *federated* and *distributed* simulation?
- Some issues of distributed simulation...
- A brief history of HLA...
- The main concepts of HLA
- The implementations of HLA
- HLA and DDS
- HLA today
- The Real Time Infrastructure (RTI)
  
---
### Distributed *vs.* federated simulation

#### Distributed simulation
- Multiple simulation components running on separate physical or virtual machines and working together to execute a unified simulation.

#### Federated simulation
- Multiple **independent** simulation systems (federates) **interoperating** within a common framework called a federation.
- A federation can be seen as a specific case of a distributed simulation where "simulation components" are actually "independent simulators"...

<ref> [ril-04], [hui-16] </ref>

---
### What is a federation

- **Federation**: A named set of **federate** applications and a common federation object model (FOM) that are used as a whole to achieve some specific objective. 
- **Federate**: A member of a federation; a single application that may be or is currently coupled with other software applications under a federation execution data (FED) or federation object model document data (FDD), and a runtime infrastructure (RTI).

---
### What do we expect from a federated simulation?

#### Interoperability
Different independent simulators of sub-systems work together to simulate a larger system:
- they communicate with each other
- they synchronize their operations
- they share some comme view of time
- etc.
#### Composability
Different independent simulators are assembled to simulate a larger system, without needing to re-engineer them...


---
<style>
img[alt~="lcim"] {
  position: absolute;
  top: 0px;
  right:-100px;
  scale: 0.5
}
</style>
### The Levels of Conceptual Interoperability layers Model (LCIM)

![lcim](./imgs/lcim.png)
- Stand-alone systems lack connectivity and cannot interoperate.
- The technical layer enables signal exchange across networks, tackling infrastructure challenges.
- The syntactic layer ensures data is correctly structured and encoded for interpretation.
- The semantic layer provides shared meaning through taxonomies and standardized data definitions.
- The pragmatic and dynamic layers define data usage context, system behavior, and process timing.
- The conceptual layer captures assumptions and harmonizes underlying models to prevent inconsistencies.

<ref> [tol-03], [tol-18] </ref>


---
### The issues of federated simulation - Data

#### Data distribution
- How to wire data from one simulation to another (communication protocols) 
- How to interface simulators that use diverse data formats (syntax)

#### Data alignment
- How to achieve **knowledge alignment** [Tolk et al., 2003] on the shared data (semantic)
- How to adapt shared information to make them consumable by the simulators with different data models,
- How to manage differences in scales (spatial and temporal)

#### Shared entities
- Concepts in the system that are represented at least in two different ways, and on which we may have concurrent access (e.g., the environment)
- Their state is common among the simulators that represent them.
  
<ref> [cre-19], [fuj-18], [zei-19] </ref>

---
### The issues of federated simulation - Time 
#### Synchronization

##### Causality
- Events occurring within simulations must be processed with respect to their timestamps order [fuj_01]

#### Synchronization in time
- How to handle a consistent evolution of the simulations in time with respect to causality ?
- There are two time synchronization approaches [fuj-98]: 
  - conservative: wait until events are safe to process
  - optimistic: allow local causality violations, but detect them and recover using rollback mechanism
  
---
### Parallel and distributed simulation (can be skipped)
#### Objective

> "A synchronization algorithm is required to ensure that the parallel execution of the simulation produces exactly the same results as a sequential execution on a single processor. In some cases approximate results are acceptable, but the bulk of the research in synchronization algorithms has focused on producing exactly the same results. One can show that this can be achieved by ensuring that each LP processes events in timestamp order."

---
### The algorithms of distributed simulation (can be skipped)
#### First generation algorithms
- block the execution of an LP until it can guarantee that an event with a smaller timestamp will not later be received.
- If each FIFO queue contains at least one message, the LP can simply pick the smallest timestamped message, remove it from its queue, and process it. If one or more FIFO queues are empty, the LP must block.
- Problem: deadlock due to waiting loops
- Solution: each LP sends *lookhead* messages $t$ stating that any future message will have a timestamp of at least $t$.
- Problem: if the lookahead is small, there will be a lot of null messages (*lookahead creep*)

---
### The algorithms of distributed simulation (can be skipped)
#### Second generation algorithms
- Algorithms based on global synchronization points.
##### Conservative algorithms 
- Concept of "epochs": "each epoch involves 
  - determining which events can be safely processed without risk of an LP later receiving a smaller timestamped event"
  - processing these safe events
  - delivering the new events

---
### The algorithms of distributed simulation (can be skipped)
#### Optimistic algorithms
- TimeWarp
- Based on rollback: if an LP processes a message timestamped 100 and later receives a message timestamps 50, it rolls back to 50.
- State variables must be restored (state saving or inverse computation)
- Messages may have been sent to other LPs: they must be rolled back (anti-messages)
- Rollback is expensive and may not always be possible (think of IOs). 
  - Concept of Global Virtual Time (GVT) lower bound of the timestamp of any future rollback that might occur.

---
###  HLA
"[...] HLA is based on the premise that no single simulation can satisfy the requirements of all uses and users. An individual simulation
or set of simulations developed for one purpose can be applied to another application under the HLA concept of the federation: a composable set of interacting Simulations" [dah-98]

"[...] HLA supports interfaces to live participants, such as instrumented platforms or live systems. Live participants interact with the simulated world thought something that acts like a simulation (from the point of view of the HLA) that feeds a representation of the live world info the simulated world and that project data from the simulated world to the live system" [dah-98]

<style>
img[alt~="top-right"] {
  position: absolute;
  top: -160px;
  right: -100px;
  scale: 0.3
}
</style>
---
### The history of HLA
![top-right](./imgs/iee1516.png)
- DoD initiative to subsume and unify the 
  - **Distributed Interactive Simulation (DIS)** standard 
    - fixed-format data exchange
  - **Aggregate Level Simulation Protocol (ALP)** standard 
    - time management, ownership transfer
- 1998: HLA 1.3
- 2000: HLA 1.4 (IEEE 1516-2000)
  - XML-based schema, improve Data Distribution Management (DDM) 
- 2010: "HLA evolved" (IEEE 1516-2010)
  - Modular FOM, new web API, Dynamic-link compatible API
- 2025 (?): HLA 4
  - Improved security, scalability, compatibility with cloud and containerized environments

---
### The HLA standards

- IEEE Std 1516-2010 Framework and Rules
- IEEE Std 1516.1-2010 Federate Interface Specification
- IEEE Std 1516.2-2010 Object Model Template Specification

- HLA is required for NATO contracts, see [STANAG 4603](https://nso.nato.int/nso/nsdd/main/standards?APNo=2610&LA=EN)

---
### The typical use of HLA
- Space domain
  - [mor-23] F. Morlang and S. Strassburger, ‘On the Role of HLA-Based Simulation in New Space’, in Proceedings of the Winter Simulation Conference, in WSC ’22. Singapore, Singapore: IEEE Press, 2023, pp. 430–440.
  - SpaceFOM [cru-22]
- Avionics?
- Automotive?
- Other?

---
### Some non typical uses of HLA
  - Combining a simulator of Urban Mobility (SUMO) with an OMNET++ network simulator [nee-24]
![](./imgs/hla-auto.png)

- A federation of SystemC federates [rot-11] ![](./imgs/hla-systemc.jpeg)
  - 1 processing node (Sparc V8) = 1 federate
  - processing note modeled at lossely-timed transaction level
  - Time Advance Request - Time Advance Grant - etc.
![](./imgs/hla-systemc.jpeg)
![](./imgs/hla-systemc-system.png)
![](./imgs/hla-tar-tag.png)

---
### The main components of HLA

- The HLA federation and federate **rules** (basic principles underlying the HLA)
- The **Object Model Template** (OMT)
  - Standard format for describing information
  - FOM and SOM
- The **API**
  - Interface to the RTI
- The **RTI**

---
### HLA federation rules - Federations
1. Federations shall have an HLA FOM, documented in accordance with the HLA OMT.
   - Agreement for information exchange
2. In a federation, all simulation-associated object instance representation shall be in the federates, not in the RTI.
  - Separate federate-specific functionality from general-purpose supporting infrastructure.
  - The supporting infrastructure have no information about the objects being simulated
3. During a federation execution, all exchange of FOM data among joined federates shall occur via the RTI.
  - provide in common the needed basic functionality to permit coherency in data exchange among the joined federates.
4. During a federation execution, joined federates shall interact with the RTI in accordance with the HLA interface specification.
  - independent development and implementation of federate applications
  - independent development of federates and RTI
5. During a federation execution, an instance attribute shall be owned by, at most, one joined federate at any given time.
  - flexibility of federate constitution 

---
### HLA federate rules - Federates
1. Federates shall have an HLA SOM, documented in accordance with the HLA OMT.
2. Federates shall be able to update and/or reflect any instance attributes and send and/or receive interactions, as specified in their SOMs.
3. Federates shall be able to transfer and/or accept ownership of instance attributes dynamically during a federation execution, as specified in their SOMs
4.  Federates shall be able to vary the conditions (e.g., thresholds) under which they provide updates of instance attributes, as specified in their SOMs.
5. Federates shall be able to manage local time in a way that will allow them to coordinate data exchange with other members of a federation.
  - support interoperability among federates that may rely on different internal concepts of time

---
### The Simulation Object Model (SOM)
- Described using the HLA Object Model Template
- A contract for the simulation component: "Here's what I contribute or expect from the simulation environment"
  - objects and interactions the federate can produce or consume
  - the attributes of those objects
  - data types, update rates and other meta data
- Local to a federate
  
---
### The Simulation Object Model (SOM) - object class
Describes object types the federate can create or interact with. indicate whether it publish or subscribe to it.
![alt text](./imgs/som1.png)

---
### The Simulation Object Model (SOM) - interaction class
Describes interactions (events or messages) the federate can send or receive.
![alt text](./imgs/som2.png)

---
### The Simulation Object Model (SOM) - data types 
Defines custom or reused data structures used in attributes and parameters.
![alt text](./imgs/som3.png)

---
### The Simulation Object Model (SOM) - publish/subscribe 
![alt text](./imgs/som4.png)

---
### The Federation Object Model (FOM) 
- Described using the HLA Object Model Template
- Global to the federation
- Collects and aligns all SOMs. 

---
### The API
- Federate Management: APIs manage federate lifecycle—create, join, resign, and destroy federations. Federates interact with the RTI (Runtime Infrastructure) to connect and coordinate.
- Object Management: Federates publish, subscribe, register, and update object instances and their attributes. This supports data sharing between distributed simulations.
- Interaction Management: Supports sending and receiving interactions (events) that are not tied to persistent objects. Useful for messages like commands or status updates.
- Declaration Management: Controls what data federates intend to publish or subscribe to, enabling the RTI to optimize data distribution.
- Ownership Management: APIs allow federates to transfer ownership of object attributes, enabling dynamic control during simulation.
- Time Management: Manages logical time progression and synchronization across federates. Includes services like time advance requests and lookahead.
- Data Distribution Management (DDM): Optimizes data exchange using region-based filtering, ensuring federates receive only relevant updates based on data interest regions.
- Support Services: Includes callbacks, synchronization points, and error handling, enabling coordination and debugging.

---
### The RTI - ambassadors
- Logical interface. No prescription about its implementation 
- Communication is done through the **RTI Ambassador**
- Example in C++
  ![alt text](./imgs/ambass-con.png)


---
### The RTI - role
- Middleware or “simulation bus” that implements the HLA standard services:
  - **Federation management**: Allows simulations to join or leave existing  federations, interrupt, verify and resume execution. 
  - **Declaration management**: Provides the means for simulations to initiate the publication of object attributes and to subscribe to interactions and updates resulting from other simulations. 
  - **Object management**: Allows for simulation to create and delete objects, produce and receive updates of interactions and individual attributes. 
  - **Ownership management**: Allows for the ownership transfer of the attributes object during the execution of the federation and, implicitly, the right to change the value of the attributes. 
  - **Time management**: Coordinates the advancement of logical time along with it’s relationship with real time, during the execution of the federation. 
  - **Data distribution management**: Enables federations to have an efficient data distribution mechanisms with a minimum amount of data exchanged between federations. 
- Federates connects to the RTI via the standardized API

<ref [akr-19] !>



<style>
img[alt~="cen"] {
  position: absolute;
  top: 150px;
  right: 400px;
  scale: 0.5
}
img[alt~="dec"] {
  position: absolute;
  top: -50px;
  right: -100px;
  scale: 0.5
}
</style>
---
### The RTI - centralized or decentralized
![cen](./imgs/rti-centralized.png)
![dec](./imgs/rti-decentralized.png)

<ref> [akr-17] </ref>

---
### The RTI - centralized or decentralized
- Centralized or decentralized?
  - Hybrid
  - Some parts are centralized (federation setting time management,...)
  - Other parts are decentralized (data exchange)
  - Simulation logic is fully distributed
- Examples: 
  - Centralized: **Pitch RTI**, **CERTI**, etc.
  - Decentralized: Portico (before V2.2.0)
    > In Portico's original model, each federate tracked others by monitoring messages, enabling decentralized decisions but causing memory and startup issues as federation size grew. Large federations overwhelmed devices with state-sync traffic, leading to instability. To solve this, a central RTI Server was reintroduced to offload coordination and reduce messaging overhead.

<ref [rot-xx] />

---
### The RTI - examples: the CERTI RTI
- Open source. Developed by ONERA
![alt text](./imgs/certi.png)

<ref [nou-09] />


---
### The RTI - other implementations
![alt text](./imgs/HLA-rtis.png)


---
### Time Management in HLA (1)
- Causality must be preserved: causes precedes consequences for all observers
- a time stamp is assigned to each event 
- events are delivered in time stamp order 
- no federate can receive a message in its past (with a timestamp earlier than its current time)

<ref> [fuj-98] </ref>  

---
### Time Management in HLA (2)
- Time management transparency
  - Local time management mechanism must not be visible to other federates
- Covers *event driven*, *time stepped*, *parallel discrete event simulation*, *wallclock time driven*
- Time Stamp Order delivery
- Advance of federate time is granted by the RTI

---
### Time Management in HLA (3)
- Message order and time stamp
  - Receive Order (RO)
    - Delivered immediately
  - Time Stamp Order (TSO)
    - Delivered to a federate in non decreasing time stamp when the RTI is sure that no other message will be delivered to the federate with a smaller time stamp
- Advancing logical time
  - Time Advance Requests(t) 
    - All RO and TSO with time stamp <= t are delivered to the federate
    - When all RO and TSO have been processed and no other TSO can arrive with time stamp <=t, **Time Advance Grant** is delivered
  - Next Event Request (*see doc*)

<ref> [fuj-98] </ref>  

---
### Time Management in HLA (4)
- Computation of the Lower Bound on Time Stamp (LBTS)
- Optimistic Even Processing

<ref> [fuj-98] </ref>  


---
### HLA and SOA - Properties of SOA
#### SOA
- Service encapsulation
- Loose coupling
- Service contract
- Service abstraction
- Service reusability
- Service composability
- Service autonomy
- Service optimization
- Service discoverability
---
### HLA and SOA - Comparison
<img src="./imgs/hla-soa.png" width=300 />

<ref> [iag-22] </ref>

---
### HLA and DDS
- SISO LSA 
  - replace HLA-RTIs by DDS' [Real-Time Publish Subscribe](https://www.omg.org/spec/DDSI-RTPS/2.2/PDF)
![[SISO LSA](https://fr.slideshare.net/slideshow/siso-lsa-and-omg-dds/17798530)](./imgs/HLA-SISO-LSA-DDS.png)


Why use DDS?
- Communication 
  - HLA cannot control communication. 
  - DDS is centered in controlling the communication
- Scalability and fault tolerance
  - HLA is central-server based: scalability and fault tolerance are difficult
  - DDS has automatic discovery, publish-subscribe, no single point of failure
- 


<ref> [loy-25], [pro-18], [lop-13] </ref>

[1](loyola.com): uses DDS for the communication layer

---
### HLA and DDS - Overlaps
From [lop-13]
<img src="./imgs/dds-hla.png" alt="drawing" width="600"/>

---
### HLA and DDS
<img src="./imgs/hla-vs-dds.png" alt="drawing" width="600"/>

<ref> [adl-19] </ref>

---
### HLA and DDS - Simware RTI

- See [SimWare](https://www.loyola.com/partners/nads/simware-rti.html), and [slides](https://fr.slideshare.net/slideshow/simware-rti-hla-raised-to-the-power-of-dds/15711655)
<img src="./imgs/simware.png" alt="drawing" width="600"/>

---
### HLA *vs.* FMI/FMU
See [awa-13a], [awa-13b]

---
### The Status of HLA
#### HLA4
- HLA4 approved by IEEE in the 2025 Simulation Innovation Workshop (see [‘What’s new in HLA 4’](https://www.mak.com/learn/blog?view=article&id=359:what-s-new-in-hla-4&catid=19:news-blog), [SISO announce](https://www.sisostandards.org/news/695782/HLA-4-Approved-by-IEEE.htm) )
- Among the new features:
  - FOM extensions are simplified by allowing attributes and parameters to be added directly to existing classes, including extensible variant records and enumerations.
  - A relaxed DDM mode enables more flexible region matching, reducing overhead by avoiding strict region overlap requirements.
  - Federate security is enhanced with a credential-based authorization system for RTI access.
  - Directed interactions allow messages to be sent specifically to federates owning certain object instances, improving efficiency.
  - A new Federate Protocol API based on protobufs replaces WSDL, supporting multiple languages and environments.
  - C++ API is now C++11 compatible, with standardized library naming for compiler clarity.
  
<ref> [mol-XX]</ref>

  
---
### References

[awa-13a] M. U. Awais, P. Palensky, A. Elsheikh, E. Widl, and S. Matthias, ‘The high level architecture RTI as a master to the functional mock-up interface components’, in 2013 International Conference on Computing, Networking and Communications (ICNC), Jan. 2013, pp. 315–320. doi: 10.1109/ICCNC.2013.6504102.

[awa-13b] M. U. Awais, P. Palensky, W. Mueller, E. Widl, and A. Elsheikh, ‘Distributed hybrid simulation using the HLA and the Functional Mock-up Interface’, in IECON 2013 - 39th Annual Conference of the IEEE Industrial Electronics Society, Nov. 2013, pp. 7564–7569. doi: 10.1109/IECON.2013.6700393.

[ril-04] G. F. Riley, M. H. Ammar, R. M. Fujimoto, A. Park, K. Perumalla, and D. Xu, ‘A federated approach to distributed network simulation’ ACM Trans. Model. Comput. Simul., vol. 14, no. 2, pp. 116–148, Apr. 2004, doi: 10.1145/985793.985795.

[hui-16] W. Huiskamp and T. van den Berg, ‘Federated Simulations’, in Managing the Complexity of Critical Infrastructures: A Modelling and Simulation Approach, R. Setola, V. Rosato, E. Kyriakides, and E. Rome, Eds., Cham: Springer International Publishing, 2016, pp. 109–137. doi: 10.1007/978-3-319-51043-9_6. 

[tol-03] A. Tolk and J. Mugira, ‘Levels of Conceptual Interoperability’, presented at the 2003 Fall Simulation Interoperability Workshop, Orlando, Florida, Sep. 2003.

[tol-18] A. Tolk, ‘The elusiveness of simulation interoperability - What is different from other interoperability domains?’, in 2018 Winter Simulation Conference (WSC), Dec. 2018, pp. 679–690. doi: 10.1109/WSC.2018.8632363.

[hui-16] W. Huiskamp and T. van den Berg, ‘Federated Simulations’, in Managing the Complexity of Critical Infrastructures: A Modelling and Simulation Approach, R. Setola, V. Rosato, E. Kyriakides, and E. Rome, Eds., Cham: Springer International Publishing, 2016, pp. 109–137. doi: 10.1007/978-3-319-51043-9_6. -->

[cre-19] F. Cremona, M. Lohstroh, D. Broman, E. A. Lee, M. Masin, and S. Tripakis, ‘Hybrid co-simulation: it’s about time’, Softw Syst Model, vol. 18, no. 3, pp. 1655–1679, Jun. 2019, doi: 10.1007/s10270-017-0633-6.

[fuj-15] R. Fujimoto, ‘Paralell and Distributed Simulation’, in Proceedings of the 2015 Winter Simulation Conference, 2015.

[zei-19] B. P. Zeigler, Theory of Modeling and Simulation. Elsevier, 2019. doi: 10.1016/B978-0-12-813370-5.00002-X.

[fuj-98] Fujimoto RM. Time Management in The High Level Architecture. SIMULATION. 1998;71(6):388-400. doi:10.1177/003754979807100604

[akr-19] A. Akram, M. S. Sarfraz, and U. Shoaib, ‘HLA Run Time Infrastructure: A Comparative Study’, Mehran Univ. res. j. eng. technol., vol. 38, no. 4, pp. 961–972, Oct. 2019, doi: 10.22581/muet1982.1904.09.

[rot-xx] T. Roth, M. Burns, and T. Pokorny, ‘Extending Portico HLA to Federations of Federations with Transport Layer Security’.

[nou-09] E. Noulard, J.-Y. Rousselot, and P. Siron, ‘CERTI, an Open Source RTI, why and how’, presented at the Spring Simulation Interoperability Workshop, San Diego USA, 27/03 2009.

[iag-22] E.-L. Iagăru, ‘Comparative Analysis Between High Level Architecture (HLA) and Service Oriented Architecture (SOA) in the Field of Military Modelling and Simulation’, Scientific Bulletin, vol. 27, no. 1, pp. 30–40, Jun. 2022, doi: 10.2478/bsaft-2022-0004.

[dah_98] J. S. Dahmann, R. M. Fujimoto, and R. M. Weatherly, ‘The DoD High Level Architecture: an update’, in 1998 Winter Simulation Conference. Proceedings (Cat. No.98CH36274), Washington, DC, USA: IEEE, 1998, pp. 797–804. doi: 10.1109/WSC.1998.745066.

[pro-18] R. Proctor, ‘Can DDS Help Solve the Distributed Simulation Integration Challenge?’ [Online]. Available: https://www.rti.com/blog/can-dds-help-solve-the-distributed-simulation-integration-challenge

[lop-13] J.-M. Lopez-Rodriguez, ‘Convergence of Distributed Simulatrion Architecturs using DDS’, SlideShare. Accessed: Apr. 15, 2025. [Online]. Available: https://www.slideshare.net/slideshow/siso-lsa-and-omg-dds/17798530

[adl-19] ADLINK, ‘Simulaion Whitepaper’. Accessed: Apr. 15, 2025. [Online]. Available: https://www.omg.org/news/whitepapers/Simulation-Whitepaper-v2.0.pdf

[mak-25] ‘What’s new in HLA 4’. Accessed: Apr. 29, 2025. [Online]. Available: https://www.mak.com/learn/blog?view=article&id=359:what-s-new-in-hla-4&catid=19:news-blog

[mol-xx] B. Möller, M. Karlsson, and F. Antelius, ‘HLA 4 Federate Protocol – Requirements and Solutions’.

[cru-22] Edwin Z. Crues, Dan Dexter, Alberto Falcone, Alfredo Garro & Björn Möller (2022) SpaceFOM - A robust standard for enabling a-priori interoperability of HLA-based space systems simulations, Journal of Simulation, 16:6, 624-644, DOI: 10.1080/17477778.2021.1945962

[wea-96] R. M. Weatherly, A. L. Wilson, B. S. Canova, E. H. Page, A. A. Zabek, and M. C. Fischer, ‘Advanced distributed simulation through the Aggregate Level Simulation Protocol’, in Proceedings of HICSS-29: 29th Hawaii International Conference on System Sciences, Wailea, HI, USA: IEEE, 1996, pp. 407–415 vol.1. doi: 10.1109/HICSS.1996.495488.

[ade-xx] M. Adelantado, J.-B. Chaudron, and A. Oyzel, ‘Using the HLA, Physical Modeling and Google Earth for Simulating Air Transport Systems Environmental Impact’.

[mcc-24] [1] T. McClain, J. Prince, and S. Samani, ‘High Level Architecture (HLA) as a Research Tool for Advanced Air Mobility (AAM)’, NASA Langley, NASA/TM-20240009249, Aug. 2024.


[alv-08] J. R. Alvarado, R. V. Osuna, and R. Tuokko, ‘Distributed Simulation in Manufacturing Using High Level Architecture’, in Micro-Assembly Technologies and Applications, vol. 260, S. Ratchev and S. Koelemeijer, Eds., in IFIP — International Federation for Information Processing, vol. 260. , Boston, MA: Springer US, 2008, pp. 121–126. doi: 10.1007/978-0-387-77405-3_11.

[mor-23] F. Morlang and S. Strassburger, ‘On the Role of HLA-Based Simulation in New Space’, in Proceedings of the Winter Simulation Conference, in WSC ’22. Singapore, Singapore: IEEE Press, 2023, pp. 430–440.

[mol-22] B. Möller, T. Gray, S. Kay, A. Kisdi, K. Buckley, and J. Delfa, ‘Experiences from the SISO SpaceFOM at the European Space Agency’, Pitch Technologues AB, 2021. [Online]. Available: https://pitchtechnologies.com/wp-content/uploads/2021/02/2021-SIW-18-Experiences-from-The-SISO-SpaceFOM-at-ESA.pdf

[jef-85] D. R. Jefferson, ‘Virtual time’, ACM Trans. Program. Lang. Syst., vol. 7, no. 3, pp. 404–425, Jul. 1985, doi: 10.1145/3916.3988.

fuj-00] R. M. Fujimoto, Parallel and distributed simulation systems. in Wiley Series on Parallel and Distributed Compuing. John Wiley and Sons Inc., 2000.


---
### LCIM and HLA
From [Hui]:
<img src="./imgs/lcim_hla.jpg" alt="LCIM-HLA" width="600"/>

<ref> [hui-16]</ref>

