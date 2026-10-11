# Troubleshooting Notes

## 1. Syslog query initially returned no data

A test event was generated:

```bash
logger -p local0.info "STC-TECH-MONITORING-TEST-002"
```

The event was verified locally:

```text
STC-TECH-MONITORING-TEST-002
```

This established that Linux was generating the event correctly.

The same event later appeared in Log Analytics:

```text
Computer: vm-stctech-monitor
Facility: local0
SeverityLevel: info
SyslogMessage: STC-TECH-MONITORING-TEST-002
```

### Lesson

A temporary empty query result does not automatically mean the DCR or agent is broken. Validate each stage of the telemetry path and account for ingestion delay.

---

## 2. Performance query with a short time window returned no data

A 10-minute query initially returned no results even though performance collection had already been observed.

The query window was expanded to one hour, which returned processor data.

A later check showed fresh records for:

```text
cpu0
cpu1
total
```

### Lesson

Before changing a working DCR, first verify the latest ingested timestamp and broaden the query window.

---

## 3. KQL accidentally executed as a Linux shell command

A `Perf | where ...` KQL query was accidentally pasted into Azure VM Run Command.

Linux returned:

```text
Perf: not found
Syntax error: "|" unexpected
```

### Root cause

`Perf` is a Log Analytics table. KQL must be executed in the Log Analytics query environment, not Bash.

### Resolution

The query was moved to:

```text
Log Analytics Workspace -> Logs
```

---

## 4. Controlled CPU workload ended with Result=timeout

The CPU test used a transient systemd service with a runtime limit.

The service ended with:

```text
Result: timeout
```

This was expected behavior: the runtime limit intentionally terminated the workload after approximately three minutes.

The service reported approximately `5m 58.669s` of CPU time across a two-vCPU VM.

### Lesson

For monitoring labs, load-generation commands should have an automatic termination mechanism so a forgotten process cannot continue consuming compute.
