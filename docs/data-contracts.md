
# Data Contracts — Mechanics Engine

## 1. Purpose

This document defines implementation-independent contracts for the physical data used by Mechanics Engine Version 1.

It specifies required data elements, physical dimensions, spatial dimensionality, invariants, validation rules, unit-handling boundaries, and failure conditions.

These contracts are independent of Python class names, package structure, numerical integration algorithms, and the final unit-library selection.

No production implementation is required to complete this specification.

## 2. Design Principles

1. Physical quantities must retain their physical meaning and dimensional information.
2. Spatial dimensionality must be explicit and consistent within a simulation.
3. Physical properties must be distinguished from time-dependent state.
4. Invalid physical data must be rejected before it can silently corrupt a simulation.
5. Compatible unit conversion must preserve the represented physical quantity.
6. The engine must delegate unit representation, conversion, and dimensional operations to a dedicated unit-handling library.
7. Numerical algorithms must not be selected by this document.
8. All numerical values used in calculations must be finite unless a future, explicitly documented contract permits otherwise.

## 3. Core Data Elements

### 3.1 Particle Identity

Each particle must have an identifier that distinguishes it from every other particle in the same simulation.

Requirements:

- Identifiers must be unique within a simulation.
- An identifier must remain stable throughout the simulation.
- Identifier representation is an implementation decision.
- Particle identity is dimensionless metadata.

### 3.2 Particle Physical Properties

The physical properties of a particle describe characteristics that do not change during a Version 1 simulation.

Required property:

| Property | Meaning | Physical dimension | Invariant |
|---|---|---|---|
| Mass | Constant inertial mass | M | Finite and strictly positive |

Mass must be expressed as a physical quantity compatible with the dimension of mass.

Mass changes are outside the Version 1 contract.

### 3.3 Particle State

The state describes the translational quantities associated with a particle at a particular simulation time.

| Quantity | Meaning | Spatial components | Physical dimension |
|---|---|---:|---|
| Position | Particle location | D | L |
| Velocity | Rate of change of position | D | L T^-1 |
| Acceleration | Rate of change of velocity | D | L T^-2 |
| Resultant force | Net force acting on the particle | D | M L T^-2 |

Here, D is the system's spatial dimension and must be 1, 2, or 3.

Position and velocity are required initial-state quantities.

Acceleration and resultant force must be available as calculated quantities when the force model is defined and evaluated. Whether they are stored or derived is an implementation decision.

### 3.4 Physical System

A physical system contains one or more particles and the force models acting on them.

Required information:

- Spatial dimension.
- Particle identities and physical properties.
- Initial state of every particle.
- Force models and their required parameters.
- Simulation time configuration.
- Unit information required to interpret dimensional quantities.

Every particle and every spatial vector in a system must use the same spatial dimension.

## 4. Spatial Vector Contract

### 4.1 Supported Dimensions

The supported spatial dimensions are:

- 1D: exactly one component.
- 2D: exactly two components.
- 3D: exactly three components.

The dimension is a discrete, dimensionless configuration value.

### 4.2 Vector Requirements

Every position, velocity, acceleration, and force vector must:

- Contain exactly D components.
- Contain only finite component values.
- Represent the appropriate physical quantity.
- Have component units compatible with that quantity's physical dimension.
- Use a consistent spatial coordinate convention within the simulation.

All components of a given vector must have compatible physical dimensions. Compatible units need not be textually identical if the unit library can convert them correctly.

Examples:

- A position vector may be represented in metres or compatible length units.
- A velocity vector may be represented in metres per second or compatible velocity units.
- A force vector may be represented in newtons or compatible force units.

Vectors with different numbers of spatial components must not be combined in spatial operations.

## 5. Initial Conditions

Every simulated particle must have:

- A unique identifier.
- A finite, strictly positive mass.
- An initial position with exactly D components.
- An initial velocity with exactly D components.
- Position and velocity quantities with compatible spatial dimensionality and correct physical dimensions.

Initial position and velocity must be specified at the simulation start time.

Initial acceleration and force need not be independently supplied when they can be determined from the configured force models. A force model that cannot be evaluated at the required initial state must produce a clear failure rather than an undefined numerical result.

## 6. Simulation Time Configuration

A simulation configuration must define:

- Start time.
- End time.
- A positive time step or an explicitly defined temporal-resolution setting.
- Any additional time-sampling settings required by the selected numerical method.

Start time, end time, and time-step quantities must have the physical dimension of time.

Invariants:

- All supplied time values must be finite.
- The end time must not precede the start time.
- A configured time step must be strictly positive.
- Time quantities must use compatible units.
- Conversion between compatible time units must preserve their physical meaning.

The exact relationship between integration steps, output sampling times, and solver-specific settings is deferred to the numerical design tasks.

A zero-duration interval is valid when the start time equals the end time. The solver contract must define how such a simulation is evaluated and recorded.

## 7. Physical Quantity and Unit Requirements

The following physical dimensions apply to core quantities:

| Quantity | SI unit | Physical dimension |
|---|---|---|
| Time | second (s) | T |
| Position | metre (m) | L |
| Velocity | metre per second (m/s) | L T^-1 |
| Acceleration | metre per second squared (m/s²) | L T^-2 |
| Mass | kilogram (kg) | M |
| Force | newton (N) | M L T^-2 |

SI units are the default reference system. Compatible alternative units may be accepted through the unit-handling layer.

The unit-handling layer must support:

- Representation of physical quantities and units.
- Conversion between compatible units.
- Detection and rejection of incompatible dimensions.
- Dimensional validation of inputs and calculated quantities.
- Distinction between dimensional quantities and dimensionless parameters.
- Integration with NumPy where required.
- Compatibility with the Python versions selected for the project.
- Evaluation of maintenance, documentation, licensing, dependency footprint, and numerical-loop suitability.

The selected library and its integration strategy must be documented in a separate decision task.

## 8. Dimensional Consistency

The following relationships must hold:

- Velocity has dimensions of length divided by time.
- Acceleration has dimensions of length divided by time squared.
- Force has dimensions of mass multiplied by acceleration.
- Position differences require compatible length quantities.
- Quantities added or compared physically must have compatible dimensions.
- Vector operations must preserve the expected physical dimension.
- Conversion between compatible units must preserve the represented physical quantity.

Newton's second law must be dimensionally consistent:

$$ F_{net} = m a $$

An operation that violates dimensional consistency must fail explicitly. The engine must not silently discard units or interpret incompatible quantities as equivalent.
### 8.1 Unit Conversion Contract

Unit conversion must preserve the represented physical quantity.

The engine must accept compatible units where supported by the
selected unit-handling library. For example, metres and centimetres
are compatible length units, while metres and seconds are incompatible.

Before numerical operations are performed on vector components,
their units must be converted to a compatible representation, or
the unit library must guarantee that the operation correctly accounts
for the units of each component.

The engine must not treat raw numerical values expressed in different
units as directly interchangeable.

For example, the position vector (1 m, 50 cm) represents the same
physical vector as (1 m, 0.5 m). A numerical representation using
metres must therefore use component values (1, 0.5), not (1, 50).

The unit-handling layer must reject conversions between incompatible
physical dimensions and must preserve the physical meaning of values
during conversion.

The internal unit representation and the point at which conversion
occurs remain implementation decisions to be resolved in a dedicated
design task.

## 9. Dimensionless Data

The following data may remain dimensionless:

- Particle identifiers.
- Spatial dimension selectors.
- Array indices and record counts.
- Boolean configuration flags.
- Dimensionless physical parameters.
- Numerical tolerances, when their specific mathematical meaning and scaling are documented.
- Dimensionless ratios.

A numerical value is not automatically dimensionless merely because it is stored as a number. Its physical meaning must determine whether a unit is required.

## 10. Recorded Simulation Data

At each designated output time, the recorded data must identify:

- Simulation time.
- Particle identifier.
- Particle position.
- Particle velocity.
- Particle acceleration, when evaluated.
- Resultant force, when evaluated.

Every recorded dimensional quantity must retain sufficient unit information to interpret it correctly.

Records must preserve particle identity and chronological meaning.

The output sampling policy, storage format, memory-management strategy, and handling of failed steps are deferred to later design tasks.

## 11. Validation Rules and Failure Conditions

Validation must reject invalid data before numerical evolution begins whenever the invalidity is detectable from the configuration.

| Condition | Required behavior |
|---|---|
| Mass is zero or negative | Reject the particle configuration |
| Mass or a required numeric value is non-finite | Reject the invalid input |
| Unsupported spatial dimension | Reject the configuration |
| Vector component count differs from D | Reject the vector |
| Vector contains a non-finite component | Reject the vector |
| Position has an incompatible physical dimension | Reject the input |
| Velocity has an incompatible physical dimension | Reject the input |
| Acceleration or force has an incompatible dimension | Reject the quantity |
| Time values have incompatible units | Reject the configuration |
| Time step is zero or negative | Reject the configuration |
| End time precedes start time | Reject the configuration |
| Duplicate particle identifier | Reject the system configuration |
| Required initial position or velocity is missing | Reject the particle configuration |
| Force evaluation is undefined or returns invalid data | Report a clear simulation failure |
| Unit conversion is requested between incompatible dimensions | Reject the conversion |
| Dimensional arithmetic is invalid | Report a dimensional-consistency failure |

Failure reporting must identify the invalid quantity or operation and explain the violated contract. The exact exception types and error-message format are implementation decisions.

## 12. Valid Examples

These examples illustrate contract-level validity, not a required Python syntax.

### Example 1: Valid One-Dimensional Particle

- Spatial dimension: 1.
- Particle identifier: P1.
- Mass: 2 kg.
- Initial position: [3 m].
- Initial velocity: [4 m/s].
- Start time: 0 s.
- End time: 5 s.
- Time step: 0.01 s.

All required dimensions are consistent, and the mass and time step are positive and finite.

### Example 2: Valid Two-Dimensional Initial State

- Spatial dimension: 2.
- Particle identifier: P2.
- Mass: 1 kg.
- Initial position: [0 m, 2 m].
- Initial velocity: [5 m/s, 0 m/s].
- Start time: 0 s.
- End time: 2 s.
- Time step: 0.005 s.

Both position and velocity contain exactly two components with the correct physical dimensions.

### Example 3: Valid Three-Dimensional System

- Spatial dimension: 3.
- Particle identifiers: P3 and P4.
- Masses: 1 kg and 3 kg.
- Initial positions: [0 m, 0 m, 0 m] and [1 m, 0 m, 0 m].
- Initial velocities: [0 m/s, 1 m/s, 0 m/s] and [0 m/s, 0 m/s, 0 m/s].
- Start time: 0 s.
- End time: 1 s.
- Time step: 0.001 s.

Both particles use three-dimensional vectors and distinct identifiers. The force model must define any interaction between them.

## 13. Invalid Examples

### Example 1: Invalid Mass

A particle has mass -2 kg.

This violates the strictly positive mass invariant. The configuration must be rejected.

### Example 2: Invalid Vector Dimension

A system is configured for 2D, but a particle's initial position is [1 m, 2 m, 3 m].

The position has three components instead of two. The configuration must be rejected.

### Example 3: Incompatible Units

A velocity vector is supplied with a component expressed in kilograms, or a calculation attempts to add a length to a time.

The physical dimensions are incompatible with the required quantity or operation. The unit-handling layer must reject the operation.

## 14. Boundary Between Engine and Unit Library

The unit library is responsible for:

- Representing dimensional quantities and units.
- Converting compatible units.
- Detecting incompatible dimensions.
- Providing the dimensional information required for validation.

The simulation engine is responsible for:

- Enforcing the data contracts.
- Selecting the required physical dimensions for each quantity.
- Enforcing spatial dimensionality.
- Validating particle and simulation configuration.
- Defining force and state semantics.
- Ensuring that numerical calculations preserve the required physical meaning.
- Reporting contract violations and simulation failures clearly.

The engine must not implement a competing unit system unless a later documented decision demonstrates that the selected library cannot meet the requirements.

The exact adapter, boundary representation, and numerical-loop strategy remain open until the unit-library decision task is completed.

## 15. Open Design Decisions

The following decisions remain unresolved:

1. Which unit library best satisfies the documented requirements?
2. Which representation should be used at the boundary between the unit library and numerical calculations?
3. Should units be retained throughout the numerical loop or converted at a defined internal boundary?
4. How should output sampling relate to integration time steps?
5. Which quantities must be stored versus derived?
6. What exact error types and reporting conventions should be used?
7. How should numerical tolerances be defined and scaled?
8. What behavior is required for a zero-duration simulation?
9. How should undefined or non-finite force evaluations be handled during a simulation?

These decisions must be resolved in their designated tasks and must not be treated as finalized by this document.

## 16. Acceptance Boundary

This document is ready for review when:

- Required data elements and their meanings are defined.
- Physical properties are distinguished from time-dependent state.
- 1D, 2D, and 3D representation rules are explicit.
- Initial conditions and simulation time configuration are specified.
- Core physical dimensions and unit requirements are documented.
- Conversion and dimensional-consistency behavior is defined.
- Invalid inputs and incompatible dimensional operations have defined failure behavior.
- Unit-library requirements and engine/library responsibilities are explicit.
- At least three valid and three invalid examples are provided.
- The contracts are consistent with the approved domain analysis.
- No production implementation is required.

This document defines data contracts only. It does not implement classes, select a unit library, or specify a numerical solver.
