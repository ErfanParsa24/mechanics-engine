
# Domain Analysis — Mechanics Engine

## 1. Purpose

Mechanics Engine is a Python framework for modeling and numerically simulating classical Newtonian mechanics.

Version 1 focuses on point-particle systems in one, two, and three spatial dimensions. The engine will accept initial conditions and force models, evolve the system over a specified time interval, and record simulation results for later analysis.

The domain model must remain independent of specific Python class names, package structure, numerical algorithms, and unit-library choices.

## 2. Version 1 Scope

Version 1 is intended to support:

- One or more point particles.
- One-dimensional, two-dimensional, and three-dimensional motion.
- Constant and strictly positive particle masses.
- Initial positions and velocities.
- Forces that may depend on time, particle position, particle velocity, and the state of other particles.
- Interactions between particles.
- Numerical evolution over a finite time interval.
- Recording time-dependent simulation results.
- Explicit physical quantities and units.
- Unit conversion and dimensional-consistency validation.

The same physical problem should be expressible in any supported spatial dimension when its mathematical formulation is appropriate for that dimension.

## 3. Physical Entities and Concepts

### 3.1 Point Particle

A point particle is an idealized physical entity whose mass is represented as a scalar and whose translational state is described by position and velocity.

The model assumes that the particle's spatial extent and rotational state are not represented.

Each particle has:

- A constant, strictly positive mass.
- An initial position.
- An initial velocity.
- An acceleration determined by the resultant force and its mass.

Particle identity must remain distinguishable when multiple particles are present.

### 3.2 Physical System

A physical system contains one or more particles and the force models acting on them.

A system has a consistent spatial dimension: 1D, 2D, or 3D.

The system may include external forces and interactions between particles. The exact representation and composition of force models will be specified in a later design task.

### 3.3 State

The instantaneous mechanical state describes the information required to determine the system's subsequent evolution under the selected force models.

At minimum, the translational state includes the position and velocity of every particle.

Acceleration and resultant force are also expected to be available as calculated physical quantities. Whether they are stored as independent state variables or derived values is a later design decision.

### 3.4 Force Model

A force model specifies how a force acting on a particle is determined.

Depending on the physical problem, a force may depend on:

- Time.
- The position of the affected particle.
- Its velocity.
- The positions and velocities of other particles.
- Other explicitly defined physical parameters.

The domain must permit multiple force contributions to act on the same particle. Their resultant determines the particle's acceleration according to Newton's second law.

The mathematical formulation and software interface of force models remain to be defined.

### 3.5 Simulation

A simulation evolves a physical system from its initial conditions over a specified time interval and records selected quantities at designated simulation times.

Numerical integration approximates continuous physical evolution. Accuracy, stability, error estimation, and solver selection belong to separate design and validation tasks.

## 4. Spatial Dimensions

Version 1 supports three spatial configurations:

- **1D:** Position, velocity, acceleration, and force each have one spatial component.
- **2D:** These quantities each have two spatial components.
- **3D:** These quantities each have three spatial components.

All particles and spatial vector quantities within a single system must use the same spatial dimension.

Spatial dimension is distinct from physical dimension. For example, a three-dimensional position is a spatial vector with three components, each having the physical dimension of length.

## 5. Initial-Value Problem Inputs

A simulation problem must specify, directly or through well-defined defaults:

- Spatial dimension.
- Simulation start time and end time.
- Initial position of each particle.
- Initial velocity of each particle.
- Constant mass of each particle.
- Force models and their required parameters.
- Relevant external conditions and particle interactions.
- Unit information for all dimensional input quantities.
- Numerical evolution settings, such as requested temporal resolution, where required by the selected solver.

The start and end times must be finite, and the end time must not precede the start time.

Particle masses must be finite, strictly positive physical quantities with the dimension of mass.

Initial positions and velocities must have the spatial dimension selected for the system and the appropriate physical dimensions.

The precise configuration format and numerical time-stepping contract will be defined in subsequent tasks.

## 6. Expected Simulation Outputs

The simulation must provide sufficient information to reconstruct and analyze the system's evolution.

Expected recorded outputs include:

- Simulation time.
- Particle positions.
- Particle velocities.
- Particle accelerations.
- Resultant force acting on each particle.

Outputs must preserve particle identity and the units associated with their physical quantities.

The sampling frequency, storage format, memory-management strategy, and optional derived quantities will be specified separately.

Potential future analysis quantities include kinetic energy, momentum, work, and other observables derived from the recorded state.

## 7. Physical Assumptions

Version 1 adopts the following initial assumptions:

1. Classical Newtonian mechanics applies.
2. Particles are idealized as point particles.
3. Each particle has constant, finite, strictly positive mass.
4. Motion is represented in a consistent spatial dimension of 1D, 2D, or 3D.
5. Newton's second law governs translational acceleration:
   F_net = m a.
6. Force models provide physically meaningful force values for the states in which they are evaluated.
7. Numerical results approximate the continuous equations of motion and may contain numerical error.
8. SI units are the default reference system.
9. Compatible alternative units may be accepted through a dedicated unit-handling layer.
10. Physical quantities must be dimensionally consistent regardless of the units used to express them.

These assumptions must be reviewed before they are treated as final requirements.

## 8. Physical Quantities and Dimensions

The initial domain includes the following physical quantities:

| Quantity | Symbol | SI unit | Physical dimension |
|---|---|---|---|
| Time | t | second (s) | T |
| Position | x | metre (m) | L |
| Velocity | v | metre per second (m/s) | L T^-1 |
| Acceleration | a | metre per second squared (m/s^2) | L T^-2 |
| Mass | m | kilogram (kg) | M |
| Force | F | newton (N) | M L T^-2 |

Position, velocity, acceleration, and force are spatial vectors in the selected spatial dimension. Mass and time are scalar quantities.

Additional quantities may be required by particular force models. Their units and dimensions must be documented as those models are introduced.

## 9. Unit-Handling Requirements

Physical units are part of the domain contract and must not be postponed until after the initial engine design.

The unit-handling layer must support the following requirements:

- Represent physical quantities together with their units.
- Convert compatible quantities between supported units.
- Reject operations that combine incompatible physical dimensions.
- Validate the dimensions of inputs, force values, and calculated quantities.
- Preserve dimensional consistency across numerical calculations.
- Distinguish dimensionless parameters from dimensional quantities.
- Define consistent boundaries between user-supplied units, internal calculations, and reported outputs.
- Support the SI system as the default reference system.

Examples of required dimensional relationships include:

- Velocity = length / time.
- Acceleration = length / time^2.
- Force = mass * acceleration.
- Position differences require compatible length units.
- Quantities added together must have compatible dimensions.

The specific unit library, internal representation, conversion policy, and performance strategy remain undecided.

## 10. Representative Use Cases

### Use Case 1: One-Dimensional Constant Acceleration

A single particle moves along one axis under a constant force.

Inputs include its mass, initial position, initial velocity, force, and simulation interval.

Expected outputs include position, velocity, acceleration, and resultant force as functions of time.

This use case validates the basic single-particle, one-dimensional domain.

### Use Case 2: Two-Dimensional Projectile Motion

A particle moves in two dimensions under a uniform gravitational acceleration, with no air resistance.

Inputs include mass, initial position, initial velocity, gravitational acceleration, and simulation interval.

Expected outputs include the particle's two-dimensional trajectory, velocity, acceleration, and resultant force.

This use case validates multidimensional motion and a force model that does not depend on particle velocity.

### Use Case 3: Three-Dimensional Motion with Multiple Particles

Two or more particles move in three dimensions and exert forces on one another.

The precise interaction law is supplied by the selected physical model.

Expected outputs include the state of each particle and the resultant force acting on each particle over time.

This use case validates multiple-particle identity, interaction support, and consistent three-dimensional state representation.

### Use Case 4: One-Dimensional Spring Oscillator

A particle is attached to an ideal linear spring and moves along one axis.

The spring force depends on displacement from equilibrium and follows a specified linear force law.

Expected outputs include position, velocity, acceleration, and force over time.

This use case validates state-dependent forces and provides a foundation for later numerical and physical validation.

## 11. Initial Non-Goals

The following capabilities are excluded from the initial version unless a later scope review changes these boundaries:

- Rigid-body rotation and rotational dynamics.
- Relativistic mechanics.
- Quantum mechanics.
- Fluid dynamics.
- General constraint solving.
- Complex collision detection and contact resolution.
- Advanced symbolic mechanics.

Simple particle interactions remain in scope. Excluding complex collision handling does not, by itself, define the behavior of particles that reach the same position; this requires a separate decision.

## 12. Unresolved Design Questions

The following questions require further analysis:

1. What unit library best satisfies dimensional safety, conversion, documentation, maintenance, and numerical-performance requirements?
2. What internal unit representation should be used during repeated numerical calculations?
3. How should the engine define force-model interfaces and compose multiple force contributions?
4. Which interaction models should be included in the first release?
5. What behavior is expected when particles overlap or collide?
6. How should simulation times, output sampling, and solver time steps relate to one another?
7. Which quantities must be recorded at every step, and which may be calculated later?
8. How should invalid physical inputs, incompatible units, and undefined force evaluations be reported?
9. Which numerical accuracy and stability requirements must be established before solver selection?
10. Should the initial release support discontinuous forces or event-driven changes in the physical model?

These questions must be resolved in the appropriate design tasks before their answers are treated as implementation requirements.

## 13. Version 1 Acceptance Boundary

The domain analysis is ready for review when:

- The purpose and scope of version 1 are clear.
- Supported spatial dimensions and physical assumptions are documented.
- Required inputs and expected outputs are identified.
- Core physical concepts are defined independently of implementation details.
- Physical quantities and dimensional relationships are identified.
- Unit-handling requirements are documented.
- At least three representative use cases are described.
- Non-goals and unresolved questions are recorded.
- The document is reviewed for consistency before implementation begins.

This document defines the physical problem domain. It does not implement software, select a numerical integration algorithm, prescribe a final package structure, or select a unit library.