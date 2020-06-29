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