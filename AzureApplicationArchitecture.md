# Azure Application Architecture Guide

## Introduction
- Applications are decomposed into smaller, decentralized services.
- Services communicate through APIs or by using asynchronous messaging or eventing.
- Applications scale horizontally, adding new instances as demand requires.
- Application state is distributed.
- Operations are done in parallel and asynchronously.
- Applications must be resilient when failures occur.
- Deployments must be automated and predictable.
- Monitoring and telemetry are critical for gaining insight into the system.

### Traditional on-premises
- Monolithic
- Designed for predictable salability
- Relational database
- Synchronized processing
- Design to avoid failures : Mean time between failures (MTBF) - It is a measure of how reliable a hardware product or componenet is.
- Occasional large updates
- Manual management
- Snowflake servers

### Modern cloud
- Decomposed
- Designed for elastic scale
- Polyglot presistence (mix of storage technologies)
- Asynchronous processing
- Design of failure : Mean time to repair (MTTR) - It is a basic measure of the maintainability of repairable items.It represents the average time required to repair a failed component or device.
- Frequent small updates
- Automated self-management
- Immutable infrastructure

## Workflow to a well architectured cloud application
![Workflow for a well architectured cloud application](../images/workflow_well_architectured_cloud_application.png)


