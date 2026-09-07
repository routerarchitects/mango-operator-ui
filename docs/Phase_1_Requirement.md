# Mango Cloud MDU UI --- Phase 1 Requirements

**Document status:** Ready for Product and Architecture Approval
**Version:** 1.2\
**Date:** 5 September 2026

------------------------------------------------------------------------

## 1. Purpose

This document defines the product and functional requirements for Phase
1 of the Mango Cloud MDU operator interface.

Phase 1 provides the operational experience required to monitor
Properties, inspect and manage Devices, manage hierarchical
Configuration, and administer Users and Policies.

The interface must be organized around operator workflows rather than
exposing backend service modules directly.

Detailed API contracts, service integration, data models, configuration
resolution, authorization implementation, and deployment mechanics will
be defined separately.

------------------------------------------------------------------------

## 2. Phase 1 Objectives

Phase 1 must enable authorized users to:

1.  Identify Properties and Devices that need attention.
2.  Navigate consistently from Property to Venue to Device.
3.  Search and inspect the managed Device fleet.
4.  Review Device health, telemetry, configuration, and diagnostic
    information.
5.  Create and manage reusable Configuration Profiles and
    Configurations.
6.  Assign Configuration at Property, Venue, or Device scope.
7.  Understand the effective Configuration applied to a Device.
8.  View and manage Users.
9.  View and manage Policies and their assignments.
10. View the scope associated with a Policy assignment.
11. Manage Operators where authorized.

------------------------------------------------------------------------

## 3. Scope

### 3.1 In Scope

-   Global operational Dashboard
-   Property and Venue hierarchy
-   Property overview
-   Venue overview
-   Fleet-level Device management
-   Individual Device operations and diagnostics
-   Configuration Profiles
-   Configuration creation and editing
-   Hierarchical Configuration assignment
-   Effective Configuration and configuration provenance
-   User management
-   User-to-Policy assignment
-   Policy management
-   Policy permissions
-   Assignment Scope for Management Role Assignments
-   Operator administration

### 3.2 Deferred to Phase 2

The following are explicitly outside Phase 1:

-   Global audit logs
-   Firmware management and firmware upgrade workflows
-   Client inventory and client troubleshooting
-   Subscriber management and onboarding
-   MPSK/PPSK subscriber workflows
-   Billing and service-class management
-   Dedicated issue assignment, acknowledgement, and resolution workflow

Aggregate client KPIs, client lists, client trends, client association
tables, and subscriber-related workflows must not be included in Phase
1.

Device health indicators may identify conditions that need attention,
but Phase 1 does not introduce a persistent issue-management workflow.

------------------------------------------------------------------------

## 4. Terminology

| UI Term | Meaning |
|---|---|
| **Operator** | A business/provisioning object used to bind subscribers. Creating an Operator automatically creates its corresponding Operator Entity. An Operator is not a loginable User. |
| **Property** | A top-level MDU site or managed customer location. Backend terminology may represent this as an Entity. |
| **Venue** | A building, floor, common area, or other child scope within a Property. |
| **Device** | A managed AP, switch, gateway, or other supported network device. |
| **Configuration Profile** | A reusable section-level Configuration component. When referenced by a Configuration section, the Configuration retains a live reference to the Profile. Profile-provided values are read-only within that Configuration section. |
| **Configuration** | The actual named configuration assembled from supported settings and, where applicable, Configuration Profile values. |
| **Assignment** | Association of a Configuration with a Property, Venue, or Device. |
| **Effective Configuration** | The resulting configuration applicable to a Device after applicable assignments and supported overrides/resolution. |
| **User** | A person with access to Mango Cloud. |
| **User Role** | A management classification associated with a User, such as Admin, CSR, NOC, or Installer. User Role is distinct from a Management Role Assignment. |
| **Management Role Assignment** | A scope-specific assignment object connecting a User to an Entity, one or more Venues, and a Policy. It determines which Policy applies to that User within that scope. |
| **Policy** | A set of resource/action permissions that can be associated with a User through a Management Role Assignment. |
| **Assignment Scope** | The Property and/or Venue associated with a Management Role Assignment. A Policy defines resource/action permissions but does not independently contain an operational scope. |

The conceptual relationship is:

```text
User
 ├── User Role: Admin / CSR / NOC / Installer / ...
 │
 └── Management Role Assignment(s)
      ├── Assignment Scope
      │    ├── Property
      │    └── Venue(s)
      └── Policy
           └── Resource/action permissions
```

The authorization relationship is:

```text
Management Role Assignment
    = User + Assignment Scope + Policy

Backend authorization service
    → evaluates the resulting permissions
```
User Role is an administrative classification only. MDU must not derive or calculate permissions from User Role.

MDU must display and manage these concepts where supported by the backend, but must not implement its own authorization algorithm.

The UI must use **Property** instead of **Entity** for normal user-facing experiences. Backend terminology may appear in technical or diagnostic contexts where required.

------------------------------------------------------------------------

## 5. Information Architecture

The Phase 1 sidebar should use the following structure:

``` text
OPERATIONS
  Dashboard
  Properties
  Devices

NETWORK
  Configuration
    Configuration Profiles
    Configurations

ADMINISTRATION
  Users & Access
    Users
    Policies
  Operators
```

### Requirements

-   **IA-001:** Selecting **Properties** must open the Property/Venue
    hierarchy.
-   **IA-002:** The application must preserve the selected authorized Property and Venue context while the user moves between related pages. Operator context may be preserved only for users authorized to access Operator administration.
-   **IA-003:** Pages below the Property level must provide breadcrumbs.
-   **IA-004:** Empty, loading, permission-denied, partial-data, and
    error states must be defined for every primary page.
-   **IA-005:** Navigation and available actions must reflect the user's
    authorized access as evaluated by the platform.
-   **IA-006:** MDU must not derive operational authorization or resource visibility from a particular User Role.

------------------------------------------------------------------------

## 6. Global Requirements

-   **GLB-001:** Users must only be able to view and act on resources
    for which they have authorization.
-   **GLB-002:** The UI must not expose unauthorized resource
    information.
-   **GLB-003:** Destructive actions must require confirmation and
    clearly identify the target.
-   **GLB-004:** Mutating actions must show progress and a final success
    or failure state.
-   **GLB-005:** Tables must support pagination, sorting, search, and
    filtering where applicable.
-   **GLB-006:** Status labels must not rely on color alone.
-   **GLB-007:** User-facing errors must describe what failed and
    whether the user can retry.
-   **GLB-008:** Duplicate submissions must be prevented while a
    mutation is in progress.
-   **GLB-009:** Pages displaying operational data must provide an
    appropriate refresh or automatic update mechanism.
-   **GLB-010:** The UI must clearly distinguish current, stale,
    unavailable, and failed data.
-   **GLB-011:** Configuration Profile, Configuration, User, and Policy editors must warn the user before navigating away when unsaved changes exist.

------------------------------------------------------------------------

# 7. Dashboard Requirements

## 7.1 Purpose

The Dashboard is the cross-Property operational starting point.

It must answer:

**What needs attention now, and where should the operator go next?**

## 7.2 Functional Requirements

-   **DASH-001:** The Dashboard must support appropriate scope and
    time-range filters.
-   **DASH-002:** It must display:
    -   Total Properties
    -   Properties Needing Attention
    -   Devices Offline
    -   Devices Needing Attention
-   **DASH-003:** It must display Property Health using:
    -   Healthy
    -   Warning
    -   Critical
    -   Unknown
-   **DASH-004:** It must display Device Availability using:
    -   Online
    -   Offline
    -   Unknown
-   **DASH-005:** It must display availability trends when historical
    telemetry is available.
-   **DASH-006:** The Dashboard must provide:
    - an **All Properties** view for users authorized to view Properties; and
    - an **Authorized Venues** view for users whose highest authorized scope is a Venue.

    The applicable view must show, at minimum:
    - Scope name
    - Health status
    - Total Devices
    - Online Devices
    - Offline Devices
    - Devices Needing Attention
    - Last status update
-   **DASH-007:** Users must only see Properties and Venues available
    within their authorized scope.
-   **DASH-008:** Users whose authorized scope contains Venues but does not include the parent Property must receive an **Authorized Venues** view instead of an unauthorized Property view.

-   **DASH-009:** The Dashboard must not expose the name, identifier, health, counts, or other information of a parent Property that the user is not authorized to view.

-   **DASH-010:** Selecting an authorized Venue must open that Venue directly.

-   **DASH-011:** Property-level KPIs must not be calculated, inferred, or displayed for a user who does not have access to that Property.

-   **DASH-012:** Breadcrumbs and scope selectors must not expose unauthorized hierarchy information.
-   **DASH-013:** Selecting a Property or Venue must open its overview.
-   **DASH-014:** Selecting a Device count or attention indicator must
    open the relevant Device view with the appropriate scope/filter
    applied.
-   **DASH-015:** Health information shown on the Dashboard must be
    consistent with Property, Venue, and Device views.

## 7.3 Health and Availability

**Availability** represents whether the platform can currently communicate with or obtain state from a Device.

**Health** represents the operational condition of the Device based on available health indicators.

Health and Availability are separate concepts. An Online Device may have Warning or Critical Health.

| Availability | Meaning |
|---|---|
| Online | The Device is currently reachable or reporting available state. |
| Offline | The Device is not currently reachable or reporting. |
| Unknown | Sufficient current information is unavailable. |

| Health | Meaning |
|---|---|
| Healthy | No monitored condition currently requires attention. |
| Warning | One or more conditions require review. |
| Critical | Conditions indicate significant impairment. |
| Unknown | Current or sufficient health data is unavailable. |

Exact health calculations and thresholds are defined separately.


------------------------------------------------------------------------

# 8. Property and Venue Requirements

## 8.1 Property Selection and Hierarchy

-   **PROP-001:** The Properties module must show only Properties and
    Venues available to the current user.
-   **PROP-002:** The current scope must be clearly visible.
-   **PROP-003:** Users must be able to move between available
    Properties.
-   **PROP-004:** The hierarchy must support access beginning at a Venue
    where applicable.

## 8.2 Property Overview

* **PROP-005:** The Property header must show the Property name, health state, breadcrumb, and scope selector where applicable.

* **PROP-006:** The Property summary must show:

  * Total Devices
  * Online Devices
  * Offline Devices
  * Devices Needing Attention
  * Total Venues
  * Impacted Venues

* **PROP-007:** The Property page must provide Property health and Device availability information.

* **PROP-008:** The Property page must provide a Venues section displaying the Venues belonging to the Property in a table or equivalent list. The Venue view must provide:

  * Venue name
  * Health
  * Total Devices
  * Online Devices
  * Offline Devices
  * Last update

* **PROP-009:** Selecting a Venue from the Property page must open the Venue overview.

* **PROP-010:** The Property view must provide access to Devices requiring attention.

* **PROP-011:** The Venues section must allow users to identify the health and device availability of each Venue and navigate to the selected Venue.

* **PROP-012:** The Property page must clearly represent the relationship between the Property and its Venues, with the Property acting as the parent scope and Venues as child scopes.


## 8.3 Venue Overview

-   **VEN-001:** The Venue header must identify the Venue and show only the authorized hierarchy context. The parent Property must be shown only when the user is authorized to view that Property.
-   **VEN-002:** The Venue summary must show:
    -   Total Devices
    -   Online Devices
    -   Offline Devices
    -   Devices Needing Attention
-   **VEN-003:** The page must provide Device status and available
    health information.
-   **VEN-004:** The page must show Devices requiring attention.
-   **VEN-005:** The page must show all Devices within the Venue with
    appropriate search and filtering.
-   **VEN-006:** Selecting a Device must open the individual Device
    page.

------------------------------------------------------------------------

# 9. Device Management Requirements

## 9.1 Fleet View

-   **DEV-001:** The Devices page must show Devices available within the
    user's authorized scope.
-   **DEV-002:** Fleet information must provide visibility into:
    -   Total Devices
    -   Online
    -   Offline
    -   Warning
    -   Critical
    -   High Memory
    -   Certificate Warning
-   **DEV-003:** Client counts and client association information must
    not be displayed in Phase 1.
-   **DEV-004:** Fleet views must provide Device status, health, type,
    vendor, uptime, memory, and operational-result information.
-   **DEV-005:** Users must be able to search by Device name, serial
    number, MAC address, Property, or Venue.
-   **DEV-006:** Users must be able to filter by available Device and
    scope attributes.
-   **DEV-007:** The Device table must provide, at minimum:
    -   Name
    -   Serial number
    -   Device type
    -   Property
    -   Venue
    -   Status
    -   Health
    -   Model
    -   Uptime/last seen
    -   Certificate state
-   **DEV-008:** Selecting a Device must open the individual Device
    page.

## 9.2 Individual Device Page

The Device page must provide the following tabs:

1.  Overview
2.  Radios
3.  Configuration
4.  Activity
5.  Diagnostics

-   **DEV-009:** The Device header must show key identity, connection,
    health, scope, and last-seen information.
-   **DEV-010:** Primary actions are Blink, Event Queue, Factory Reset Device, Firmware Upgrade,Re-enroll, Telemetry, Script, Trace, Export Device Data.
-   **DEV-011:** Destructive Device actions must require confirmation.
-   **DEV-012:** Firmware upgrade actions are excluded from Phase 1.
-   **DEV-013:** Device deletion must be exposed .

-   **DEV-014:** Device deletion must require explicit confirmation identifying the target Device.

-   **DEV-015:** The UI must display the final backend result of the deletion request and must not imply success before the backend confirms it.

### Overview

-   **DEV-016:** The Overview must provide key Device identity,
    connectivity, operational health, and provisioning information.
-   **DEV-017:** The latest health state and current attention
    conditions must be visible.
-   **DEV-018:** Missing or stale information must be explicitly
    identified.

### Radios

-   **DEV-019:** The Radios tab must show supported and active radios.
-   **DEV-020:** Radio information must include operational
    information such as enabled state, channel, width, TX power,
    utilization, and temperature.
-   **DEV-021:** Client association information is deferred to Phase 2.

### Configuration

-   **DEV-022:** The Configuration tab must distinguish:
    -   Assigned Configuration
    -   Device Overrides
    -   Effective Configuration
-   **DEV-023:** Users must be able to understand where effective values
    originate.
-   **DEV-024:** Authorized users must be able to modify supported
    Device-specific overrides.
-   **DEV-025:** The UI must preview relevant changes before saving or
    applying an override.
-   **DEV-026:** Saving a configuration assignment or override must not
    be presented as equivalent to applying it to a live Device.

### Activity

-   **DEV-027:** The Activity tab must show relevant Device operational
    activity.
-   **DEV-028:** The Activity tab must include command history, configuration
    activity, and reboot history.
-   **DEV-029:** Activity is separate from the future global audit-log
    workflow.
-   **DEV-030:** Operational activities must show available status,
    time, and result information.



------------------------------------------------------------------------

# 10. Configuration Requirements

## 10.1 Configuration Model

Phase 1 supports configuration at multiple levels:

``` text
Property
   ↓
Venue
   ↓
Device
   ↓
Device Override
```

The resulting Device configuration is the **Effective Configuration**.

The detailed configuration-resolution mechanism is defined separately.

### Requirements

-   **CFG-001:** Users must be able to understand which configuration
    applies to a Device.
-   **CFG-002:** The UI must identify inherited and Device-specific
    values.
-   **CFG-003:** Users must be able to preview relevant configuration
    changes before applying them.
-   **CFG-004:** Invalid or incomplete configurations must provide a
    clear validation error.

## 10.2 Configuration Profiles

Configuration Profiles are reusable section-level Configuration components.

A supported Configuration section may reference one Configuration Profile at a time.

When a Configuration Profile is selected for a Configuration section, the Configuration retains a live reference to that Profile. Profile-provided values are resolved from the current Profile definition and are read-only within the Configuration.

Profile-provided values cannot be individually overridden within the Configuration.

When a Profile is selected, the user may:

- View Profile
- Replace Profile
- Remove Profile and configure manually

Removing the Profile reference allows that Configuration section to be configured manually.

Complete Configuration Templates are not included in Phase 1. Configuration Profiles are reusable section-level components and must not be presented as complete Configurations.

- **CFG-005:** Reusable section-level configuration components must be presented as **Configuration Profiles**.

- **CFG-006:** Users must be able to create, view, edit, and duplicate Configuration Profiles where authorized.

- **CFG-007:** A Configuration Profile must include:
  - Name
  - Description
  - Applicable Device context
  - Configuration section
  - Supported reusable values

- **CFG-008:** A supported Configuration section may reference only one Configuration Profile at a time.

- **CFG-009:** When a Configuration Profile is selected for a Configuration section, the Configuration must retain a live reference to that Profile.

- **CFG-010:** Profile-provided values must be read-only within the Configuration and must not support field-level overrides.

- **CFG-011:** The UI must identify Profile-provided values using a source label such as `From Profile: <Profile Name>`.

- **CFG-012:** When a Profile is referenced, the available Profile actions must include:
  - View Profile
  - Replace Profile
  - Remove Profile and configure manually

- **CFG-013:** Updating a Configuration Profile must affect the resolved values of every Configuration that references that Profile.

- **CFG-014:** Before a Profile change is saved, the UI must show the Configurations and Devices that may be affected by the change.

- **CFG-015:** Saving a Configuration Profile must not automatically deploy configuration changes to Devices.

- **CFG-016:** Configuration previews must resolve the latest saved Profile values.

- **CFG-017:** An in-use Configuration Profile must not be deleted until its references are removed or replaced.

- **CFG-018:** Deployment records must identify the Configuration Profile revision used when the downstream platform provides revision information.

## 10.3 Configuration Profiles List
- **CFG-019:** The Configuration Profiles list must display:
  - Profile name
  - Description
  - Device context
  - Configuration section
  - Validation state
  - Number of referencing Configurations
  - Last modified

- **CFG-020:** The Configuration Profiles list must support search and filtering.

- **CFG-021:** Available Profile actions must include:
  - Create
  - View
  - Edit
  - Duplicate

- **CFG-022:** Profile deletion must be dependency-safe and must not allow deletion while unresolved Configuration references remain.

## 10.4 Configurations List
- **CFG-023:** The Configurations list must display:
  - Configuration name
  - Description
  - Applicable Device context
  - Validation state
  - Profiles used
  - Number of assignments
  - Latest deployment state
  - Last modified

- **CFG-024:** The Configurations list must support search and filtering.

- **CFG-025:** Available Configuration actions must include:
  - Create
  - View
  - Edit
  - Duplicate

## 10.5 Configuration Creation and Editing

-   **CFG-026:** Users must be able to create and edit named
    Configurations.
-   **CFG-027:** Configurations must provide understandable groups of
    settings.
-   **CFG-028:** A supported Configuration section must allow the user to either reference a compatible Configuration Profile or configure the section manually.
-   **CFG-029:** Before saving a Configuration, the UI must validate required fields, supported data types and ranges, Profile references, and applicable cross-field constraints.
-   **CFG-030:** Saved Configurations must have identifiable names and
    modification information.

## 10.6 Assignment

-   **CFG-031:** Configuration assignment must support:
    -   Property
    -   Venue
    -   Device
-   **CFG-032:** Users must only be able to assign Configuration within
    their authorized scope.
-   **CFG-033:** Before saving a Configuration assignment, the UI must show the target scope and the Devices affected by the assignment.
-   **CFG-034:** Assignment must represent the intended configuration
    relationship and must not imply that the live Device has already
    received the change.


## 10.7 Deployment

Configuration assignment and live deployment are separate concepts.

-   **CFG-035:** The UI must provide a separate action to apply or deploy a Configuration to its assigned Devices.

-   **CFG-036:** Before deployment, the UI must show the deployment target, affected Devices, and relevant impact information, including Devices that are currently offline.

-   **CFG-037:** Deployment results must show the per-Device deployment status and clearly identify Devices that did not complete successfully.

-   **CFG-038:** Phase 1 must support the OWGW command states relevant to Configuration deployment:
    - Pending
    - Executing
    - Executed
    - Completed
    - Failed
    - Timed Out
    - Expired

-   **CFG-039:** Partial deployment results must summarize Devices by deployment outcome and identify unsuccessful or incomplete Devices.

-   **CFG-040:** The UI must allow failed Devices to be retried without redeploying successfully completed Devices unless the user explicitly requests a full redeployment.

-   **CFG-041:** Duplicate deployment submissions for the same Configuration and target must be prevented while an equivalent deployment is already in progress.

-   **CFG-042:** Deployment history must be available for the relevant Configuration or Device.

-   **CFG-043:** Deployment history is operational Configuration history and must remain separate from the future Phase 2 global audit log.

## 10.8 Effective Configuration

-   **CFG-044:** The UI must display the Effective Configuration for an
    individual Device.
-   **CFG-045:** The UI must show the source of inherited, Profile-provided, assignment-provided, and Device-specific values.
-   **CFG-046:** Inherited, Profile-provided, assignment-provided, and Device-specific values must be visually distinguishable without relying only on color.
-   **CFG-047:** Users must be able to compare proposed changes with the
    current Effective Configuration.
-   **CFG-048:** A structured human-readable view must be the primary
    experience. Raw configuration data may be available as an advanced
    view.

------------------------------------------------------------------------

# 11. Users & Access Requirements

Users & Access is divided into **Users** and **Policies**.

MDU does not define or implement its own authorization algorithm.

The platform/downstream services remain authoritative for authentication, authorization, policy evaluation, Management Role Assignments, and access scope.

## 11.1 Users

-   **USR-001:** Users must be accessible under **Users & Access**.
-   **USR-002:** The Users table must provide relevant user information such as:
    -   Name
    -   Email/login
    -   Status
    -   User Role
    -   Last login
    -   Assigned Policy information
-   **USR-003:** Authorized administrators must be able to view User details.
-   **USR-004:** User details must provide relevant profile information.
-   **USR-005:** User details must show the User Role where provided by the platform.
-   **USR-006:** User details must show the Management Role Assignments associated with the User.
-   **USR-007:** Each Management Role Assignment must show:
    -   User
    -   Entity/Property
    -   Venue(s)
    -   Policy
-   **USR-008:** User details must show the scope associated with applicable Policy assignments.
-   **USR-009:** Authorized administrators must be able to create and update Users through the authoritative backend.
-   **USR-010:** User creation/update UX must support the profile information, User Role, and Management Role Assignment information supported by the backend.
-   **USR-011:** Authorized administrators must be able to add, edit, or remove Management Role Assignments .
-   **USR-012:** User lifecycle actions such as suspend/reactivate must be provided.
-   **USR-013:** Authentication-specific operations must use the configured identity service.
-   **USR-014:** MDU must not use a User Role to independently determine authorization or implement role-based permission logic.
-   **USR-015:** User Role must be presented as an administrative classification and must not be presented as the source of operational resource permissions.

-   **USR-016:** Operational authorization must be represented through Management Role Assignments containing Assignment Scope and Policy.
-   **USR-017:** MDU must call the authoritative backend APIs for User, Management Role Assignment, scope, and Policy operations.

## 11.1.1 Management Role Assignments

A Management Role Assignment is the scope-specific object used to connect a User, scope, and Policy.

-   **MRA-001:** The UI must represent Management Role Assignments separately from the User Role classification.
-   **MRA-002:** A Management Role Assignment must represent the applicable User, Entity/Property, Venue(s), and Policy.
-   **MRA-003:** The UI must show the Policy attached to each Management Role Assignment.
-   **MRA-004:** The UI must show the Entity/Property and Venue scope attached to each Management Role Assignment.
-   **MRA-005:** MDU must not independently calculate whether a Management Role Assignment grants an action.
-   **MRA-006:** Creation, update, validation, duplicate-scope handling, and authorization of Management Role Assignments are handled by the authoritative backend service.

## 11.2 Policy List

-   **POL-001:** Users & Access must provide a Policy management view.
-   **POL-002:** The Policy list must show available Policies.
-   **POL-003:** Only Root-authorized users will create, edit, or delete custom Policies. Other authorized users may view and assign existing Policies but must not modify Policy definitions.
-   **POL-004:** The Policy list should provide relevant information
    such as:
    -   Policy name
    -   Description
    -   Number of Users
    -   Number of assignments/scopes
-   **POL-005:** Selecting a Policy must open its Policy overview.

## 11.3 Policy Overview

-   **POL-006:** The Policy overview must show the Policy name and
    description.
-   **POL-007:** The overview must show Users associated with the
    Policy.
-   **POL-008:** The overview must show the scope associated with Policy
    assignments.
-   **POL-009:** The overview must provide access to the Policy's
    permissions.

## 11.4 Policy Permissions

-   **POL-010:** The UI must provide a clear view of the permissions
    associated with a Policy.
-   **POL-011:** Policy permissions must be presented using the resource/action model defined by the authoritative Policy service.
-   **POL-012:** The Policy editor must expose the Phase 1 resources and actions defined by the authoritative Policy model.
-   **POL-013:** Permission information must represent the Policy as
    provided by the downstream authorization service.
-   **POL-014:** MDU must not redefine the underlying authorization or
    permission-evaluation model.

## 11.5 Policy Creation and Editing

-   **POL-015:** Only Root-authorized users may create, edit, or delete custom Policies.
-   **POL-016:** Other authorized users may view and assign existing Policies within their permitted Assignment Scope but must not modify Policy definitions.
-   **POL-017:** Policy creation must provide:
    -   Name
    -   Description
    -   Permissions
-   **POL-018:** The Policy editor must provide a clear way to select
    supported resource permissions.
-   **POL-019:** The UI must validate required Policy information before
    saving.
-   **POL-020:** Editing a Policy must clearly indicate that existing
    Users or assignments may be affected.
-   **POL-021:** Built-in Policies must be protected from unsupported
    modification or deletion.
-   **POL-022:** A Policy that is still assigned to Users or scopes must
    not be deleted without an appropriate dependency workflow.

## 11.6 Management Role Assignment and Assignment Scope

-   **POL-023:** A Policy must be associated with a User through a Management Role Assignment.
-   **POL-024:** The UI must show which Users are using a Policy.
-   **POL-025:** The UI must show where each Management Role Assignment applies.
-   **POL-026:** Policy and scope information must be retrieved from and
    remain consistent with the authoritative downstream service.
-   **POL-027:** MDU must not independently redefine Policy precedence
    or authorization behavior.

------------------------------------------------------------------------

# 12. Operator Administration Requirements

Operator administration is a separate administrative workflow from normal Property operations.

An Operator is not a loginable User. It is a business/provisioning object used to bind subscribers. Creating an Operator automatically creates the corresponding Operator Entity.

Users may be associated with the Operator Entity through the existing Management Role Assignment mechanism. Authorization is evaluated against the applicable Entity/Property and Venue scope. The Operator object itself is not an independent operational access scope.

All Operator lifecycle, Entity relationship, subscriber relationship, scope, and authorization behavior is handled by the backend service.
-   **OPR-001:** The automatically created Operator Entity is an internal root scope and must not appear as an MDU Property unless the authoritative backend explicitly classifies it as an operational Property.
-   **OPR-002:** The Operators module must only be available to users authorized to manage Operators.
-   **OPR-003:** Internal Operator Entity identifiers must not appear in:
      - Property selectors
      - Property breadcrumbs
      - Dashboard Property tables
      - Property counts
      - Normal Property/Venue navigation
-   **OPR-004:** The Operators table must provide relevant Operator information such as:
    -   Name
    -   Description
    -   Registration information
    -   Status
    -   Modified information
- **OPR-005:** Users authorized for Operator administration must be able to create and edit Operators through the authoritative backend.

- **OPR-006:** Firmware, Subscribers, and Service Classes workflows are not part of Phase 1.

- **OPR-007:** Operator details must show associated operational Properties and Devices only when those relationships are returned by the authoritative backend.

- **OPR-008:** Operator deletion must require confirmation. The backend service is responsible for dependency validation and cleanup.

- **OPR-009:** MDU must use the authoritative backend APIs for Operator operations and must not duplicate Operator-to-Entity or subscriber relationships.

# 13. Non-functional Requirements

## 13.1 Performance

-   **NFR-001:** Primary pages should provide usable content within the
    agreed performance target under normal operating conditions.
-   **NFR-002:** Search and filtering should provide results within the
    agreed performance target.
-   **NFR-003:** Long-running commands and deployments must not block
    the browser session.
-   **NFR-004:** Fleet-scale tables must support appropriate pagination
    or virtualization.

Specific measurable performance targets will be defined in the technical specification and agreed test plan.

## 13.2 Security

-   **NFR-005:** Authorization must be enforced by backend services; UI
    visibility is not a security boundary.
-   **NFR-006:** Sensitive configuration values and credentials must be
    protected and masked by default.
-   **NFR-007:** Secrets must not be exposed through URLs, browser logs,
    analytics, or user-facing errors.
-   **NFR-008:** Authentication and identity operations must use the
    configured identity service.

## 13.3 Reliability and Data Quality

-   **NFR-009:** Operational information must provide an indication of
    freshness where appropriate.
-   **NFR-010:** Stale, partial, and unavailable information must be
    distinguishable from healthy or zero values.
-   **NFR-011:** The UI must accurately represent the result of
    operations even when a Device becomes unavailable during the
    operation.
-   **NFR-012:** Configuration preview and applied configuration must
    use consistent configuration semantics.

## 13.4 Accessibility

-   **NFR-013:** The application should follow the agreed accessibility
    standard for keyboard navigation, labels, focus, contrast, and
    non-color status indicators.
-   **NFR-014:** Charts and visual status information must have an
    accessible alternative.

------------------------------------------------------------------------

# 14. Dependencies

The MDU UI consumes capabilities from the underlying Mango Cloud
services.

  Capability                      Downstream Service
  ------------------------------- ---------------------------------------
  Authentication / identity       OWSEC
  Properties / Venues             OWPROV
  Inventory / Device ownership    OWPROV
  Users                           OWPROV / configured identity services
  Policies / access information   OWPROV
  Device operations               OWGW
  Device telemetry                Operational/analytics services
  Configuration                   OWPROV + OWGW

MDU should consume these capabilities without duplicating their
underlying data models or authorization logic.

------------------------------------------------------------------------

# 15. Phase 1 Acceptance Criteria

Phase 1 is ready for product acceptance when:

1. An authorized user can navigate Dashboard → Property → Venue → Device using Mango Cloud terminology.
2. Dashboard, Property, Venue, and Device health and availability information is consistent for the same underlying data.
3. An authorized user can search for and inspect Devices through the defined Device views.
4. Users only see resources and actions available to them through the platform authorization model.
5. An authorized user can create a Configuration Profile and use it within a Configuration.

6. Profile-provided values are resolved through the live Profile reference, are displayed as read-only within the Configuration, and cannot be individually overridden.

7. A Configuration retains a live reference to a selected Configuration Profile.

8. Updating a Configuration Profile changes the resolved Profile values used by every referencing Configuration.

9. Before a Profile change is saved, the UI displays the affected Configurations and Devices.

10. Saving a Profile change does not automatically deploy the resulting Configuration changes to Devices.

11. An in-use Profile cannot be deleted until all Profile references are removed or replaced.

12. An authorized user can assign a Configuration to a Property, Venue, or Device.

13. Configuration assignment and live deployment are represented as separate actions.

14. A Device page shows its Effective Configuration and the source of relevant values, including Profile-provided values where applicable.

15. Per-Device Configuration deployment results clearly display the relevant OWGW command state: Pending, Executing, Executed, Completed, Failed, Timed Out, or Expired.

16. Authorized administrators can view Users, User Roles, Management Role Assignments, and assigned Policies.

17. Authorized administrators can view the scope associated with Management Role Assignments.

18. Authorized administrators can view Policy details, Users associated with a Policy, Assignment Scope, and Policy permissions.

19. A Root-authorized user can create and manage custom Policies. Other authorized administrators can view and assign existing Policies within their permitted Assignment Scope.

20. Authorized administrators can manage Operators where permitted.

21. An authorized user can initiate Device deletion through an explicit confirmation, and the UI displays the final backend operation result.

22. Audit Logs, Firmware, Clients, Subscribers, MPSK/PPSK, Billing, and Service Classes are absent from Phase 1 navigation and workflows.

23. Loading, empty, error, stale-data, partial-success, and permission-denied states are implemented for primary workflows.

24. A user authorized only for one or more Venues can open those Venues and their authorized Devices without seeing unauthorized parent Property names, identifiers, metrics, breadcrumbs, or hierarchy information.

25. Deployment history is available for the relevant Configuration or Device and is clearly separated from the future Phase 2 global audit log.

26. Profile-provided values are identified with their Profile source, such as `From Profile: <Profile Name>`, and provide View, Replace, and Remove Profile actions.

# 16. Open Product Decisions

The following items require explicit product or architecture approval before implementation freeze:

1. Property, Venue, and Device health formulas and thresholds.

2. Phase 1 Device Capability Matrix, including supported Device types and capability-dependent UI actions and data fields.

3. Exact Policy resources and actions exposed in the Phase 1 Policy editor.

4. Exact Policy assignment and scope behavior supported by the authoritative backend.

5. Configuration assignment and deployment behavior.

6. Configuration rollback behavior after failed or partial deployment.

7. Identity-provider behavior for invitation, password setup, MFA, and email validation.

8. Exact diagnostic information exposed through the UI.

9. What happens when more than one Configuration is assigned at the same scope.

10. What happens when a Configuration assignment is replaced.

11. What happens when an assignment is removed.

12. How Property, Venue, and Device assignments are resolved when they define the same field.

13. What happens when an assigned Configuration becomes invalid because a referenced Configuration Profile changes.

14. Numerical performance targets for initial page usability, search and filter response, pagination, Dashboard refresh, and long-running operation feedback.

15. Supported browser versions, minimum desktop viewport, and tablet-layout requirements for Installer workflows.


Device deletion behavior, Operator-to-Entity behavior, authorization calculation, and underlying cleanup are backend responsibilities and are not Phase 1 MDU UI decisions.

# 17. Phase 2 Extension Points

Phase 1 should preserve navigation and data-model extension points for:

-   Clients and client troubleshooting
-   Subscribers
-   Units
-   SSID/MPSK/PPSK access
-   Subscriber Devices
-   Firmware catalog and rollout
-   Global audit trail and change history
-   Persistent issues and issue management
-   Billing
-   Service classes

These capabilities must not introduce empty or inactive Phase 1 pages or
navigation items.