# Validation Results

## Activity Log

Validated a successful diagnostic-setting write event in `AzureActivity`.

## Syslog

Validated test event:

```text
STC-TECH-MONITORING-TEST-002
```

Observed:

```text
Computer: vm-stctech-monitor
Facility: local0
SeverityLevel: info
```

## Performance

Confirmed ingestion for:

```text
Logical Disk / % Used Space
Memory / % Used Memory
Memory / Available MBytes Memory
Network / Total Bytes
Processor / % Processor Time
Processor / % User Time
Processor / % IO Wait Time
```

Confirmed processor instances:

```text
cpu0
cpu1
total
```

## CPU Baseline

```text
Samples: 29
Average CPU: 4.36%
Minimum CPU: 4.07%
Maximum CPU: 4.80%
```

## Controlled CPU Workload

```text
VM vCPUs: 2
Workload duration: ~3 minutes
CPU time consumed: 5m 58.669s
Automatic stop: yes
```
