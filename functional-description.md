# Manager Application – Functional Description

1. Introduction and Background
2. The Starting Point – Identified Challenges
3. Goals and Objectives
4. The Proposed Solution
5. Scope and Design Principles
6. Functional Overview
   6.1 Documents to Assign
   6.2 Documents with Deadline
   6.3 Documents to Handle
   6.4 Documents to Approve
7. Current Status and Next Steps

# Manager Application – Functional Description

## 1. Introduction and Background

Two weeks ago, workshops were conducted with Trøndelag County Municipality to gather
insights directly from managers and end-users. The purpose of these workshops was
to understand how managers interact with Public 360 today, and which challenges
they experience in their daily work.

Following these workshops, the input was consolidated and used to define a clear
development plan for the first version of the application.

Due to a short development timeline, the work was carried out in close collaboration
with the customer, with a clear distinction between:
- **Must-have features** – core functionality required to deliver immediate value.
- **Nice-to-have features** – enhancements that can be introduced in later iterations.

This approach ensures fast value realization while maintaining a clear roadmap for
future development.

## 2. The Starting Point – Identified Challenges

Many managers experience Public 360 as complex and difficult to use.
They often have limited familiarity with:
- Case processing workflows
- Archival and documentation requirements
- The overall structure and navigation of the system

As a result, managers frequently rely on administrative staff to perform tasks on
their behalf. This leads to inefficiencies, delays in document handling, and an
increased administrative burden across the organization.

## 3. Goals and Objectives

The primary goal of the application is to provide managers with a simplified,
intuitive, and focused tool that enables them to perform their most essential tasks
quickly and efficiently.

At the same time, the solution must ensure full compliance with archival, legal,
and documentation requirements.

Key objectives include:
- Reducing time spent navigating complex systems
- Lowering the administrative burden for managers
- Improving adoption among managers who rarely use Public 360
- Ensuring that all actions are performed in accordance with applicable regulations

## 4. The Proposed Solution

To address these challenges, a lightweight and user-friendly application has been
designed specifically for managers.

The solution provides:
- A simplified interface tailored to managerial tasks
- Support for executing critical tasks outside Public 360
- Reduced complexity by focusing only on the most relevant actions
- Improved usability without compromising legal or archival compliance

All functionality is powered by the SIF API, using Bearer Token authentication.
This ensures that the correct permissions and role matrix are applied to every
request, in alignment with Public 360.

## 5. Scope and Design Principles

The application is designed according to the following principles:
- **Task-oriented design** – functionality is structured around real managerial tasks
- **Readability and clarity** – minimal clicks and clear work lists
- **No data duplication** – all data is retrieved from Public 360 via SIF
- **Security by design** – permissions are enforced by SIF and the underlying platform
- **Incremental delivery** – the solution can be extended in future iterations

## 6. Functional Overview

### 6.1 Documents to Assign

This work list presents documents that require assignment to a case handler.

Managers can:
- View relevant document metadata
- Open documents for review
- Forward or assign documents to the appropriate case handler

The purpose of this functionality is to ensure that incoming documents are routed
efficiently, without delays caused by unclear ownership or unfamiliar workflows.
