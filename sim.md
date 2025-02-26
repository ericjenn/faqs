# [Coupling simulations](https://www.emse.fr/~boissier/enseignement/maop17-autumn/pdf/coupling-simulation.pdf?utm_source=chatgpt.com)

## Challenges 

### Data distribution
- How to wire data from one simulation to another (communication protocols) 
- How to interface simulators that use diverse data formats (syntax)

### Data alignment
- How to achieve knowledge alignment [Tolk et al., 2003] on the shared data (semantic)
- How to adapt shared informations to make them consumable by the simulators with different data models,
manage differences in scales (spatial and temporal)

### Synchronization

#### Causality

Events occurring within simulations must be processed with
respect to their timestamps order [Fujimoto, 2001]

#### Synchronization in time
- How to handle a consistent evolution of the simulations in time with respect to the casualty principal ?
- There are two time synchronization approaches [Fujimoto, 1998]: 
  - conservative: wait until events are safe to process
  - optimistic: allow local causality violations, but detect them and recover using rollback mechanism
  
#### Shared entities
- Concepts in the system that are represented at least in two different ways, and on which we may have concurrent access (e.g., the environment)
- Their state is common among the simulators that represent them.
  
# Parallel and distributed simulation

## Objective

> "A synchronization algorithm is required to ensure that the parallel execution of
the simulation produces exactly the same results as a sequential execution on a single processor. In some
cases approximate results are acceptable, but the bulk of the research in synchronization algorithms has
focused on producing exactly the same results. One can show that this can be achieved by ensuring that
each LP processes events in timestamp order."

## First generation algorithms
- block the execution of an LP until it can
guarantee that an event with a smaller timestamp will not later be received.
- If each FIFO queue contains at least one message, the LP can simply pick the smallest
timestamped message, remove it from its queue, and process it. If one or more FIFO queues are empty,
the LP must block.
    - Problem: deadlock due to waiting loops
      - Solution: each LP sends *lookhead* messages $t$ stating that any future message will have a timestamp of at least $t$.
      - Problem: if the lookahead is small, there will be a lot of null messages (*lookahead creep*)

## Second generation algorithms
- Algorithms based on global synchronization points.
### Conservative algorithms 
- Concept of "epochs": "each epoch involves 
  - determining which events can be safely processed without risk of an LP later receiving a smaller timestamped event"
  - processing these safe events
  - delivering the new events
### Optimistic algorithms
- TimeWarp
- Based on rollback: if an LP processes a message timestamped 100 and later receives a message timestamps 50, it rolls back to 50.
- State variables must be restored (state saving or inverse computation)
- Messages may have been sent to other LPs: they must be rolled back (anti-messages)
- Rollback is expensive and may not always be possible (think of IOs). 
  - Concept of Global Virtual Time (GVT) lower bound of the timestamp of any future rollback that might occur.
  

# Real-Time Simulation in Real-Time Systems: Current Status, Research Challenges and A Way Forward

## Real-time simulation



- data distribution management (Morse and Zyda 2001).
  