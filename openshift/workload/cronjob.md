A CronJob creates [[job]] on a repeating schedule in [[openshift]].

One CronJob object is like one line of a crontab (cron table) file. It runs a job periodically on a given schedule, written in Cron format.

>[!caution]
All CronJob `schedule`: times are based on the timezone of the `kube-controller-manager`.
If your control plane runs the `kube-controller-manager` in Pods or bare containers, the timezone set for the `kube-controller-manager` container determines the timezone that the cron job controller uses.

When creating the manifest for a CronJob resource, make sure the name you provide is a valid DNS subdomain name. The name must be no longer than 52 characters. This is because the CronJob controller will automatically append 11 characters to the job name provided and there is a constraint that the maximum length of a Job name is no more than 63 characters.

CronJobs are meant for performing regular scheduled actions such as **backups, report generation, and so on**. Each of those tasks should be configured to recur indefinitely (for example: once a day / week / month); you can define the point in time within that interval when the job should start.

### Cron schedule syntax
```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of the month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
│ │ │ │ │                                   7 is also Sunday on some systems)
│ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
│ │ │ │ │
* * * * *
```

| Entry                  | Description                                                | Equivalent to |
| ---------------------- | ---------------------------------------------------------- | ------------- |
| @yearly (or @annually) | Run once a year at midnight of 1 January                   | 0 0 1 1 \*    |
| @monthly               | Run once a month at midnight of the first day of the month | 0 0 1 \* \*   |
| @weekly                | Run once a week at midnight on Sunday morning              | 0 0 \* \* 0   |
| @daily (or @midnight)  | Run once a day at midnight                                 | 0 0 \* \* \*  |
| @hourly                | Run once an hour at the beginning of the hour              | 0 \* \* \* \* |

For example, the line below states that the task must be started every Friday at midnight, as well as on the 13th of each month at midnight:

```
0 0 13 * 5
```
You can also use web tools like [crontab.guru](https://crontab.guru/)