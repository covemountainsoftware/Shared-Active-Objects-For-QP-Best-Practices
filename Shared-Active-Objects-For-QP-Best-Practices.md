# Summary

This document defines a set of standards and best practices for sharing software modules based on the QP framework. 

The goal:  enhance the QP software ecosystem while accelerating firmware development everywhere.

## A. General

### A.1 Shared QP Readme

The module should provide a document indicating the responses to and
adherence to the guidelines herein.

The sole purpose of this document is to aid firmware teams in the
selection of third party shared active objects for use in their
particular project.

### A.2 Source Code

The module should provide all source code required to build the module.
Binary only modules are discouraged.

### A.3 License

The module shall clearly document licensing terms. Examples include:
MIT, GPL, dual-license, etc.

### A.4 Unit testing

The module should include all unit tests used to test and develop the
module.

### A.5 Published Signals

When applicable, the module shall list all published signals required
and note the particular event data structure published with each event.
(See Integration)

### A.6 Subscribed Signals

When applicable, the module shall list all subscribed signals required
and note the particular event data structure expected with each event.
(See Integration)

### A.7 Posted Signals

When applicable, the module shall list all posted signals supported and
note the particular event data structure expected with each event. (See
Integration)

### A.8 QP Timers

The module shall document the number of QP timers required and any
minimum timer tick requirements. (See Integration)

### A.9 Message Pool

The module shall document the module's message pool minimum size and
count requirements necessary to ensure proper operations with this
module. (See Integration)

### A.10 Microcontroller resources

The module should document typical CPU, RAM, and ROM usage for at least
one microcontroller target and one use case.

### A.11 Interfaces

The module should provide abstract interfaces for any hardware or driver
level modules required by the vendor's module. Module's access to a
concrete implementation of the expected interfaces may be through
dependency injection or compile time linker injection.

### A.12 Interface Ownership

The module should document the one or more hardware interfaces this
module expects to own. Examples include UARTs, SPI bus, I2C bus, etc.

### A.13 Example Project

The module should include a concrete example container application
project for at least one microcontroller target. When provided, the
example application shall include working concrete examples of any
interfaces required by the module.

## B. Build System

### B.1 Working Build

Modules shall provide a working build system of some variety with
documentation on how to build and which files are the minimum set of
files necessary to enable the module in a firmware project.

### B.2 Prefer CMake

Modules should use CMake. CMake is the preferred build system for
delivering shared modules.

### B.3 Static library

The CMake configuration should support at least a static library build
mechanism for the shared module based on QP.

### B.4 Build Configuration Options

The module shall document all build configuration options supported,
especially for optional features that may result in non-trivial
increases in consumed system resources.

### B.5 Consistent Prefix

See Integration I.1 Identifier. All build configuration options, CMake
variables, etc., shall make use of a vendor identifier prefix.

### B.6 QP/Spy™

The module shall support a Q_SPY build configuration providing for at
least some minimal inspection of the module's behavior during execution.
The extent of the module's QP/Spy™ usage and integration is module
dependent.

## C. Containing project responsibilities

### C.1 Publish Subscribe header

The containing project will have an internal publish/subscribe
enumeration of all signals published within the QP system. This header
must include the signals required by the vendor.

Additionally, the containing project must create a separate header file
named \<VENDOR\>\_pubsub.h. This header will include the project's
overall header of enumerated published signals. See Integration.

### C.2 Manage Signal Ranges

The containing project may manage signal ranges assigned to active
objects. This is primarily for the benefit of humans consuming logs or
other debug output. Vendor's adhering to these guidelines will require
the containing project to define their internal signal range start and
end values and provide these values when building the vendor's module.
See Integration.

### C.3 Concrete Interfaces

A vendor's module will likely require access to particular hardware or
other driver interfaces. The containing project is responsible for
providing concrete implementations of any abstract interfaces defined by
the vendor. The vendor should provide one or more example
implementations.

## I. Integration

### I.1 Identifier

Module vendor or provider should register their three or more ASCII C
compiler compatible character identifier with this repository's appendix
of vendors.

Reasoning: to prevent C or C++ build conflicts when multiple vendors are
integrated within a single firmware project.

### I.2 Publish/Subscribe Signals

A shared module shall provide one or more include files which assume
they will be included into a greater enumeration of published signal
values.

Reasoning: QP is based on C or C++. The QP published signal list is a
single enumeration of message signal values. All values must be unique,
across the project's collection of modules, active objects, etc. An
independent vendor module must not reserve particular values, rather it
must rely on the containing project to define the exact signal values.

Example file (cms_embedded_cli_required_pubsub.h):

```
//list of signals specific to the embedded-cli service
//include this directly into the greater pub sub enum list

CMS_EMBEDDED_CLI_INACTIVE_SIG,
CMS_EMBEDDED_CLI_ACTIVE_SIG,
```

### I.3 namespace usage (C++ only)

When integrating with QP/C++, vendor shall use a namespace for all
internal and public classes and software. The vendor shall **NOT** use a
namespace for any signal enumeration values intended for consumption and
joining with other signal enumeration types in the greater system.

### I.4 Signal Enumeration Prefix 

All defined signals shall be prefixed with a consistent vendor
identifier of three or more characters.

### I.5 Publish/Subscribe Signals Global Namespace (C++)

Module defined publish and subscribe signals shall be in the global
namespace. Do not use a C++ namespace for pub/sub signals. This
requirement is also imposed on the containing project.

Reasoning: Enable easy merging of publish/subscribe signal lists at
compile time.

### I.6 Posted and Private Signal Range

The containing project shall create a header available in the build's
include path named \<VENDOR\>\_\<MODULE\>\_signal_range.h, which shall
define two constants, \<VENDOR\>\_\<MODULE\>\_SIGNAL_RANGE_START and
\<VENDOR\>\_\<MODULE\>\_SIGNAL_RANGE_END. The vendor's module shall
ensure that all non-published signals (posted signals or internal
private signals) begin at START and are less than END. Internal module
static asserts should be employed to ensure the module's minimum
requirements.

### I.7 Naming

Vendor shall prefix all public enums, symbols, class or structure names,
with a 3 or 4 character vendor identifier.

Reasoning: to reduce the probability of symbol clashes.

### I.8 Static and global variables

The module should avoid global and internal static variables.

Reasoning: static/global variables are an anti-pattern that may add
friction to unit testing.

### I.9 Discovery

All internal active objects should provide a discovery event, which it
subscribes to and responds to, providing the requester a pointer to the
active object in question.

### I.10 Initialization Behavior

A well-formed active object should immediately transition to an idle or
waiting state, performing little to no actual initialization behavior.
The active object should wait for an appropriately defined post or
published signal indicating that the active object should enter its
primary behavior state.

Reason: to ensure the containing project has precise control over the
entire startup/initialization sequence of the greater project.

### I.11 Priority

Vendor should document their expectations and guidelines to the
containing firmware project for setting the priorities of any active
objects provided by the vendor.

