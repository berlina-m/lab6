<!--
Sync Impact Report
Version change: Unversioned scaffold -> 1.0.0 (initial constitution)
Modified principle slots:
- PRINCIPLE_1_NAME -> I. Java 21 Baseline
- PRINCIPLE_2_NAME -> II. Maven Build
- PRINCIPLE_3_NAME -> III. Java Swing Interface
- PRINCIPLE_4_NAME -> IV. JUnit 5 Verification
- PRINCIPLE_5_NAME -> V. PlantUML as Diagram Source
Added principles: VI. Object-Oriented Design; VII. GUI and Business Logic Separation;
VIII. Exact Monetary Arithmetic; IX. Customer Requirement Traceability;
X. Human Review of AI Output
Added sections: Architecture and Technology Constraints; Delivery and Review Controls
Removed sections: None
Follow-up TODO: Record the original ratification date.
-->
# SplitSmart Constitution

## Core Principles

### I. Java 21 Baseline
Production Java code MUST target Java 21. Use Java language and runtime features
supported by that baseline, and document any compatibility exception before
adopting it. A common runtime baseline keeps builds and behavior reproducible.

### II. Maven Build
Maven MUST be the canonical build and dependency-management system for Java
modules. Project builds MUST declare required dependencies and plugins, use
repeatable lifecycle goals, and avoid relying on undeclared local machine state.

### III. Java Swing Interface
Desktop graphical interfaces implemented for this project MUST use Java Swing.
Swing event handlers MUST delegate work rather than own domain rules. Swing is a
desktop implementation choice and MUST NOT be represented as satisfying web,
Android, or iOS customer requirements by itself.

### IV. JUnit 5 Verification
Automated Java tests MUST use JUnit 5. Tests MUST cover material business rules,
including expense splitting, approvals, balance changes, and payment settlement;
test results MUST be reproducible through the Maven build.

### V. PlantUML as Diagram Source
Architecture and interaction diagrams maintained for the project MUST use
PlantUML source. Diagrams that describe current design MUST be updated when a
relevant design change is accepted, so the source remains reviewable alongside
the implementation decisions it records.

### VI. Object-Oriented Design
Java code MUST assign behavior to cohesive objects with explicit responsibilities.
Types MUST encapsulate their state and expose operations through meaningful
interfaces; inheritance MUST be used only when substitutability is valid.

### VII. GUI and Business Logic Separation
Business rules MUST reside outside Swing components and event handlers in
testable application or domain components. GUI code MAY format input and present
results, but MUST call application operations for validation and state changes.
This boundary allows domain behavior to be tested without launching the GUI.

### VIII. Exact Monetary Arithmetic
Money values and calculations MUST use `BigDecimal`, never binary floating-point
types. Currency scale and rounding rules MUST be explicit at calculation or
presentation boundaries and covered by tests, preventing silent balance drift.

### IX. Customer Requirement Traceability
Customer requirements MUST remain distinguishable from implementation decisions.
Specifications and plans MUST identify customer-originated outcomes separately
from selected technologies, architecture, and design choices, and trace each
customer requirement to the design and verification that address it. An
implementation decision MUST NOT silently narrow, replace, or claim completion
of a customer requirement.

### X. Human Review of AI Output
AI-generated code, tests, diagrams, and documentation MUST be reviewed and
verified by a human before acceptance. Reviewers MUST check correctness,
requirement alignment, security implications, and consistency with this
constitution; generated output is a proposal, not evidence of correctness.

## Architecture and Technology Constraints

Java 21, Maven, and Swing are implementation constraints for the applicable
Java desktop components. The customer request in `UserRequirement.md` separately
requires a primarily web-based product and applications that run on Android and
iOS. These are customer requirements, not superseded by the implementation
constraints. Specifications and plans MUST show how each target is addressed;
they MUST explicitly surface any unresolved platform or technology conflict for
customer resolution rather than assume that a Swing desktop application meets
web or mobile needs. This constitution does not authorize changing the customer
request.

## Delivery and Review Controls

Before implementation, each feature specification MUST identify its customer
requirements and any implementation decisions that constrain its design.
Architecture and PlantUML diagrams MUST reflect approved decisions. Maven CI or
the documented equivalent MUST run the JUnit 5 suite for changes to Java code.
Review MUST verify requirement traceability, monetary arithmetic, GUI/domain
separation, and human validation of AI-assisted contributions as applicable.

## Governance

This constitution governs project specifications, plans, implementation, and
reviews. Amendments MUST be proposed with a rationale and impact assessment,
approved by the project owner, and recorded in the constitution. Changes that
remove or redefine a principle incompatibly require a MAJOR version increment;
new principles or materially expanded guidance require MINOR; clarifications
and non-semantic corrections require PATCH. Versions use `MAJOR.MINOR.PATCH`.

Feature reviews MUST assess compliance with applicable principles and document
any approved exception, its rationale, and its scope. Unresolved conflict with a
customer requirement MUST be raised for customer decision; it MUST NOT be
resolved by silently weakening that requirement. The constitution takes
precedence over lower-level project guidance until formally amended.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): original date not recorded | **Last Amended**: 2026-10-29
