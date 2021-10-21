# CronJob Deep Dive

Official Documentation:
```
https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#cronjob-v1beta1-batch
```

## Example of CronJob
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: my-cron
  namespace: default
spec:
  # Run every 5 minutes
  schedule: */5 * * * *
  concurrencyPolicy: Forbid
  backoffLimit: 2
  startingDeadlineSeconds: 600
  successfulJobsHistoryLimit: 0
  failedJobsHistoryLimit: 1
  suspend: false
  jobTemplate:
    spec:
      activeDeadlineSeconds: 300
      template:
        metadata:
          labels:
            app: my-cron
        spec:
          activeDeadlineSeconds: 420
          containers:
          - name: cron
            image: my-repo/cron:latest
```

### Let's explain the main properties:

**spec.concurrencyPolicy:**
   - Forbid: if previous Job is still running, skip execution of a new Job
   - Replace: stop running instance of the job and start a new one
   - Allow: multiple instances of the job can run in parallel
   
**spec.schedule:** cron schedule in cron format

**spec.startingDeadlineSeconds:** optional deadline to start the job if failed to start on schedule (ie. job already running with conncurrency policy set to Forbid)

**spec.suspend:** set as true to disable cron (does not affect already running jobs)

As you can see, spec.jobTemplate.spec is actually the spec of a Job, itself containing the spec of a Pod in it's template.spec property. That means that we can use all capabilities of Jobs and Pods when using CronJobs.

### Among others, the following ones may especially be useful:

**spec.jobTemplate.spec.activeDeadlineSeconds:**
      - Maximum duration of the Job before Kubernetes tries to stop it
      - This deadline is relative to Job creation, Pod may take some time to start, for instance when pulling image is necessary
      - Set this value only if you know what you are doing

**spec.jobTemplate.spec.backoffLimit:**
       - Maximum number of retries before marking this Job as failed when Pod fails (exit code > 0) or Pod deadline exceeded
       - Defaults to 6, set to 0 for no retry possible 
       
**spec.jobTemplate.spec.template.spec.activeDeadlineSeconds:**
        - Maximum duration of the Pod before Kubernetes tries to stop it
        - If Job backoffLimit is > 0, a new Pod will be created until reached

Reference:
>https://michael.bouvy.net/post/deep-dive-kubernetes-cronjob
