# Lecture 11
https://github.com/southbridgeio/slurm-school-k8s/blob/master/7.Advanced_abstraction/advanced_abstractions.pdf

## Jobs
https://kubernetes.io/docs/concepts/workloads/controllers/job/

_backoffLimit_ - the number of retries before considering a Job as failed
```
backoffLimit: 10
```
_activeDeadlineSeconds_ - applies to the duration of the job, no matter how many Pods are created
```
activeDeadlineSeconds: 60
```

Restart policy is applicable to Pods and not to a Job.
```
      restartPolicy: Never
```

```
kubectl describe jobs.batch hello
. . .
# Events:
#   Type     Reason            Age                From            Message
#   ----     ------            ----               ----            -------
#   Normal   SuccessfulCreate  13m                job-controller  Created pod: hello-6mk4q
#   Normal   SuccessfulDelete  12m                job-controller  Deleted pod: hello-6mk4q
#   Warning  DeadlineExceeded  12m (x2 over 12m)  job-controller  Job was active longer than specified deadline
```

## CronJob
https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

You can use a CronJob to run Jobs on a time-based schedule.

### concurrencyPolicy
- Allow (default): The cron job allows concurrently running jobs
- Forbid: The cron job does not allow concurrent runs; if it is time for a new job run and the previous job run hasn't finished yet, the cron job _skips_ the new job run
- Replace: If it is time for a new job run and the previous job run hasn't finished yet, the cron job _replaces_ the currently running job run with a new job run

### successfulJobsHistoryLimit
This field specifies how many completed jobs should be kept.

### failedJobsHistoryLimit
This field specifies how many failed jobs should be kept.

### Example

```
kubectl get cronjobs.batch rnd-sleep 
. . . 
# NAME        SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
# rnd-sleep   */1 * * * *   False     1        51s             34m
```

```
kubectl describe cronjobs.batch rnd-sleep
. . .
# Successful Job History Limit:  4      
# Failed Job History Limit:      3   
# . . .
# Active Jobs:         rnd-sleep-1593554700
# Events:
#   Type    Reason            Age                   From                Message
#   ----    ------            ----                  ----                -------
#   Normal  SuccessfulCreate  58m                   cronjob-controller  Created job rnd-sleep-1593551220
#   Normal  SawCompletedJob   58m                   cronjob-controller  Saw completed job: rnd-sleep-1593551100, status: Failed
#   Normal  SawCompletedJob   57m                   cronjob-controller  Saw completed job: rnd-sleep-1593551220, status: Complete
#   Normal  SuccessfulCreate  57m                   cronjob-controller  Created job rnd-sleep-1593551340
#   Normal  SawCompletedJob   55m                   cronjob-controller  Saw completed job: rnd-sleep-1593551340, status: Complete
#   Normal  SuccessfulCreate  55m                   cronjob-controller  Created job rnd-sleep-1593551400
#   Normal  SuccessfulDelete  54m                   cronjob-controller  Deleted job rnd-sleep-1593550740
#   Normal  SawCompletedJob   54m                   cronjob-controller  Saw completed job: rnd-sleep-1593551400, status: Complete
#   Normal  SuccessfulCreate  54m                   cronjob-controller  Created job rnd-sleep-1593551460
#   Normal  SawCompletedJob   54m                   cronjob-controller  Saw completed job: rnd-sleep-1593551460, status: Complete
#   Normal  SuccessfulDelete  54m                   cronjob-controller  Deleted job rnd-sleep-1593550980
#   Normal  SuccessfulDelete  52m                   cronjob-controller  Deleted job rnd-sleep-1593551220
#   Normal  SuccessfulDelete  50m                   cronjob-controller  Deleted job rnd-sleep-1593551340
#   Normal  SuccessfulCreate  47m (x4 over 54m)     cronjob-controller  (combined from similar events): Created job rnd-sleep-1593551880
#   Normal  SawCompletedJob   3m52s (x24 over 52m)  cronjob-controller  (combined from similar events): Saw completed job: rnd-sleep-1593554460, status: Complete
```

```
kubectl get jobs.batch
. . .
# NAME                   COMPLETIONS   DURATION   AGE
# rnd-sleep-1593553500   1/1           20s        21m       <---- Completed Job #1
# rnd-sleep-1593553920   1/1           86s        14m       <---- Completed Job #2
# rnd-sleep-1593553980   0/1           12m        12m       <---- Failed Job #1
# rnd-sleep-1593554160   0/1           9m42s      9m42s     <---- Failed Job #2
# rnd-sleep-1593554340   1/1           82s        6m31s     <---- Completed Job #3
# rnd-sleep-1593554460   1/1           78s        5m1s      <---- Completed Job #4 
# rnd-sleep-1593554520   0/1           3m40s      3m40s     <---- Failed Job #3
#     ^^^^^^
# 3 failed and 4 completed
# rnd-sleep-1593554700   0/1           49s        49s       <---- Active Job
```