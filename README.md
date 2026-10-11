# Azure Monitoring & Observability Capstone

A hands-on Azure monitoring lab built manually in the Azure Portal to understand how Azure Monitor, Log Analytics, Azure Monitor Agent (AMA), Data Collection Rules (DCR), Syslog, performance counters, and KQL work together.

> This repository documents a **manual Portal-first implementation**. The exported ARM template is included as deployment evidence/reference; it is not presented as the primary deployment method or as a complete reproducible IaC solution.

## Objectives

- Build a Linux monitoring pipeline in Azure.
- Send subscription Activity Log data to Log Analytics.
- Collect Linux Syslog through AMA and a DCR.
- Collect guest OS performance counters.
- Query collected telemetry with KQL.
- Generate a controlled CPU workload for monitoring validation.
- Troubleshoot ingestion delay and query-scope issues.
- Clean up temporary Azure resources after validation.

## Architecture

```text
Azure Subscription
      |
      +-- Activity Log
      |       |
      |       +-- Diagnostic Setting
      |               |
      |               v
      |       Log Analytics Workspace
      |
      +-- Ubuntu Linux VM
              |
              +-- Azure Monitor Agent (AMA)
                      |
                      +-- Data Collection Rule (DCR)
                              |
                              +-- Syslog ----------> Syslog table
                              |
                              +-- Performance -----> Perf table
                                                   |
                                                   v
                                                  KQL
```

## Core Lab Resources

| Resource | Configuration |
|---|---|
| Resource Group | `rg-stctech-monitor-lab-uaenorth` |
| Region | UAE North |
| Linux VM | `vm-stctech-monitor` |
| OS | Ubuntu Server 24.04 LTS |
| VM size | `Standard_FX2mds_v2` |
| Log Analytics Workspace | `law-stctech-monitor-lab` |
| Data Collection Rule | `dcr-stctech-linux-monitor` |
| Action Group | `ag-stctech-monitor` |
| VNet | `vnet-uaenorth-1` |
| Subnet | `snet-uaenorth-1` |

The VM used Trusted Launch, Secure Boot, vTPM, a system-assigned managed identity, Standard SSD storage, and no public IP.

## Monitoring Pipelines Validated

### 1. Azure Activity Log

Subscription Activity Log data was routed to the Log Analytics workspace through a diagnostic setting.

A successful `MICROSOFT.INSIGHTS/DIAGNOSTICSETTINGS/WRITE` event was observed in the `AzureActivity` table.

### 2. Linux Syslog

A controlled Syslog message was generated on the Linux VM:

```text
STC-TECH-MONITORING-TEST-002
```

It was confirmed locally in Linux and later found in Log Analytics with:

- Computer: `vm-stctech-monitor`
- Facility: `local0`
- Severity: `info`

This validated the end-to-end path:

```text
Linux -> AMA -> DCR -> Log Analytics -> Syslog -> KQL
```

### 3. Guest Performance Counters

The DCR collected selected CPU, memory, disk, network, and system counters.

Examples:

- `Processor(*)\% Processor Time`
- `Processor(*)\% User Time`
- `Processor(*)\% IO Wait Time`
- `Memory(*)\% Used Memory`
- `Memory(*)\Available MBytes Memory`
- `Logical Disk(*)\% Used Space`
- `Network(*)\Total Bytes`
- `System(*)\Load1`

Performance data was successfully ingested into the `Perf` table.

Observed processor instances included:

```text
cpu0
cpu1
total
```

### 4. CPU Baseline and Controlled Workload

Before the workload test, the observed CPU baseline for `cpu0` was:

| Measurement | Value |
|---|---:|
| Samples | 29 |
| Average | 4.36% |
| Minimum | 4.07% |
| Maximum | 4.80% |

The VM confirmed two online vCPUs.

A time-limited systemd workload ran two `yes` processes for approximately three minutes. The service automatically terminated at its configured runtime limit.

Observed service result:

```text
Duration: ~3 minutes
CPU consumed: 5m 58.669s
Result: timeout (expected due to RuntimeMaxSec)
```

The purpose was to create a controlled monitoring incident without leaving runaway CPU processes.

## KQL

Reusable queries are stored under [`kql/`](kql/):

- `activity-log.kql`
- `syslog.kql`
- `performance.kql`

## Troubleshooting Performed

This lab included real troubleshooting rather than only deployment:

- Syslog initially returned no KQL results even though the Linux event existed locally.
- The pipeline was checked from source to destination.
- Later ingestion confirmed the event in the `Syslog` table.
- A short KQL time window initially returned no performance results.
- Expanding the query window confirmed continued ingestion.
- Processor instances `cpu0`, `cpu1`, and `total` were later confirmed.
- A KQL query was accidentally executed through VM Run Command and failed because KQL is not a Linux shell language; the query was then correctly executed in Log Analytics.

See [`docs/troubleshooting.md`](docs/troubleshooting.md).

## ARM Export

A sanitized Azure Portal export is included at:

```text
arm-export/template.json
```

The export confirms major resources and configuration such as the VM, VNet, Log Analytics workspace, DCR, performance counters, Syslog configuration, action group, and auto-shutdown schedule.

The exported template is **not a complete reconstruction of every runtime association**. In particular, the Portal export used for this repository did not include the AMA extension or DCR association even though AMA provisioning and end-to-end collection were validated during the lab.

Sensitive/account-specific values such as the exported SSH public key and notification email were redacted before publishing.

## Scope / Not Claimed as Completed

The following were explored or discussed but are **not claimed as completed and validated** in this version:

- Production alert rule implementation
- Alert notification end-to-end validation
- Azure Workbook dashboard
- Terraform deployment of this monitoring environment

Keeping these items out of the completed scope makes the repository reflect only work that was actually validated.

## Cleanup

After validation, the temporary lab resources were deleted to avoid unnecessary Azure charges.

## Skills Demonstrated

`Azure Monitor` · `Log Analytics` · `AMA` · `DCR` · `Linux Syslog` · `Performance Counters` · `KQL` · `Activity Log` · `Azure VM` · `Troubleshooting` · `Cost Control`
