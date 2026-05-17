# Kubernetes Job and CronJob Workflow Documentation

## 📌 Overview

This document explains Kubernetes **Job** and **CronJob** resources, their workflows, key parameters, and YAML examples.

---

# 🔁 Job Workflow

```
Job → Pods → Completion
```

## 🧠 What is a Job?
A Job runs a task until completion and ensures it finishes successfully.

---

## ✅ Important Parameters (Job)

```yaml
completions: 1        # Total successful executions required
parallelism: 1        # Number of pods running in parallel
backoffLimit: 3       # Number of retries before marking failed
activeDeadlineSeconds: 100  # Max execution time
```

---

## 🧾 Job YAML Example

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 3
  template:
    spec:
      containers:
      - name: job-container
        image: busybox
        command: ["echo", "Hello Job"]
      restartPolicy: Never
```

---

## 🚀 Job Execution Flow

1. Apply Job
2. Job Controller creates Pod
3. Pod runs task
4. Job completes

---

# 🔁 CronJob Workflow

```
CronJob → Job → Pod
```

## 🧠 What is a CronJob?
A CronJob creates Jobs on a schedule.

---

## ✅ Important Parameters (CronJob)

```yaml
schedule: "*/5 * * * *"   # Cron schedule
concurrencyPolicy: Allow   # Allow, Forbid, Replace
successfulJobsHistoryLimit: 3
failedJobsHistoryLimit: 1
startingDeadlineSeconds: 30
```

---

## 🧾 CronJob YAML Example

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: example-cronjob
spec:
  schedule: "*/1 * * * *"
  concurrencyPolicy: Allow
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cron-container
            image: busybox
            command: ["echo", "Hello CronJob"]
          restartPolicy: Never
```

---

## 🚀 CronJob Execution Flow

1. Apply CronJob
2. Scheduler triggers based on schedule
3. CronJob creates Job
4. Job creates Pod
5. Pod runs and completes
6. Repeats

---

# ⏱️ Cron Format

```
* * * * *
│ │ │ │ │
│ │ │ │ └─ Day of week
│ │ │ └── Month
│ │ └──── Day of month
│ └────── Hour
└──────── Minute
```

---

# ✅ Summary

## Job
- One-time execution
- Ensures completion

## CronJob
- Scheduled execution
- Creates Jobs periodically

---

# 🚀 Best Practices

- Use Jobs for batch tasks
- Use CronJobs for scheduling
- Control retries and history cleanup

---

# 🧪 Practical Verification

- Use `kubectl logs <job-pod-name>` to check the Job execution output:
  - `kubectl logs $(kubectl get pods --selector=job-name=<job-name> -o jsonpath='{.items[0].metadata.name}')`
- Use `kubectl get jobs` to confirm the Job was created:
  - `kubectl get jobs -o wide`
- Use `kubectl get cronjob` and `kubectl get jobs --watch` to verify the CronJob creates Jobs on schedule:
  - `kubectl get cronjob <cronjob-name>`
  - `kubectl get jobs --watch`
