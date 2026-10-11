# Architecture

## Monitoring Design

The lab used two primary telemetry paths.

### Azure control-plane telemetry

```text
Azure Subscription
    |
    v
Activity Log
    |
    v
Diagnostic Setting
    |
    v
Log Analytics Workspace
    |
    v
AzureActivity
```

### Linux guest telemetry

```text
Ubuntu VM
    |
    v
Azure Monitor Agent
    |
    v
Data Collection Rule
    |
    +--------------------+
    |                    |
    v                    v
Syslog              Performance Counters
    |                    |
    v                    v
Syslog table          Perf table
    |                    |
    +---------+----------+
              |
              v
             KQL
```

## Network Design

The VM was deployed without a public IP. The lab VNet used `172.16.0.0/16` with subnet `172.16.0.0/24`.

Outbound HTTPS connectivity to an Azure Monitor endpoint was tested from the VM before adding additional network components. The connection succeeded, so a NAT Gateway was not added merely for the lab.

## Design Principle

The project intentionally separates:

- **Platform/control-plane telemetry** from Azure resources and the subscription.
- **Guest OS telemetry** collected from Linux through AMA and DCR.
- **Metrics** from **logs**, because Azure Monitor metrics and Log Analytics logs use different monitoring paths and query experiences.
