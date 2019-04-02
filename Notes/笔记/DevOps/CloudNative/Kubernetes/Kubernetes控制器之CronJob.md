# Kubernetes控制器之CronJob

CronJob按照基于时间的时间表创建作业。

一个CronJob就像一行crontab文件。它以给定时间表定期运行，以Cron格式编写。

> 所有CronJob计划：都基于启动作业的主服务器的时区。

## Cron Job 限制

CronJob在其计划的每个执行时间创建一个Job对象。我们之所以这么说，因为在某些情况下可能会创建两个作业，或者不创建任何作业。我们试图使这些罕见，但是别没有完全阻止。因此，Job应该是幂等的。

如果`startingDeadlineSeconds`设置为交大的一个值或未设置，并且`concurrencyPolicy`设置为`Allow`则作业将始终至少运行一次。

对于每个CronJob，CronJob控制器会检查从上次计划时间到现在的持续时间内错过的计划数。如果有超过100个错过的计划，则它不会启动作业并记录错误。

重要的是要注意，如果`startingDeadlineSeconds`设置（非`nil`）字段，则控制器计算从`startingDeadlineSeconds`到现在的值而不是从上一个计划时间到现在为止发生的错误作业的数量。例如，如果`startingDeadlineSeconds`是`200`，则控制器计算在过去200秒内发生的错过的作业数。

如果CronJob未能在其预定时间创建，则将其视为未命中。例如，如果`concurrencyPolicy`设置为`Forbid`并且在先前的计划仍在运行时尝试计划CronJob，那么它将被计为错过。

例如，假设CronJob设置为从开始每隔一分钟安排一个新作业`08:30:00`，并且`startingDeadlineSeconds`未设置其 字段。此字段的默认值为`100`秒。如果CronJob控制器恰好从下降`08:29:00`到`10:21:00`，则作业将不会启动，因为错过其计划的错过作业的数量大于100。

为了进一步说明这个概念，假设CronJob设置为从开始每隔一分钟安排一个新作业`08:30:00`，并将其`startingDeadlineSeconds`设置为200秒。如果CronJob控制器恰好与前一个示例（`08:29:00`to `10:21:00`，）相同的时间段内停止，则作业仍将在10:22:00开始。这发生在控制器现在检查在最后200秒内发生的错过的计划的数量（即，3个错过的计划），而不是从最后的计划时间到现在。

Cronjob只负责创建与其计划相匹配的作业，而Job则负责管理它所代表的Pod。

