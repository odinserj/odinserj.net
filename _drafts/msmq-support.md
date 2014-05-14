---
layout: page
title: MSMQ Support for HangFire
---

SQL Server job storage implementation does not require you to learn and install additional technologies, such as Redis, for projects to use HangFire. However, it uses polling to get new jobs that *increases latency* between the creation and invocation process (see also [this feature request](https://github.com/odinserj/HangFire/issues/52)).

The MSMQ implementation replaces only the way HangFire enqueues and dequeues jobs. It uses transactional queues to delete jobs only upon successful completion, that allows to process jobs reliably inside ASP.NET applications. MSMQ queues contain **only job identifiers**, other information is still **persisted in the SQL Server database**.

## Advantages of using MSMQ

* **No additional latency**. It uses blocking calls to fetch jobs – they will be processed as soon as possible.
* **Immediate re-queueing of terminated jobs**. If the processing was terminated in the middle, it will be started again immediately after application restart. SQL Server implementation uses InvisibleTimeout to distinguish between long-running and aborted jobs.

## Usage

To use MSMQ queues, you should do the following steps:

1. Create them manually on each host. Don't forget to grant appropriate permissions.
2. Register all MSMQ queues in current `SqlServerStorage` instance.

```csharp
var storage = new SqlServerStorage("<connection string>");
storage.UseMsmqQueues(@".\hangfire-{0}", "critical", "default");
// or storage.UseMsmqQueues(@".\hangfire-{0}") if you are using only "default" queue.

JobStorage.Current = storage;
```

To see the full list of supported paths and queues, check the [MSDN article](http://msdn.microsoft.com/en-us/library/e9d4k4ze.aspx).

## Limitations

* Only transactional MSMQ queues supported for reability reasons inside ASP.NET.
* You can not use both SQL Server Job Queue and MSMQ Job Queue implementations in the same server, see below. This limitation relates to HangFire.Server only. You can still enqueue jobs to whatever queues and see them both from HangFire.Monitor.

The following case will **not work**: the `critical` queue uses MSMQ, and the `default` queue uses SQL Server to store job queue. In this case job fetcher can not make the right decision.

```csharp
var storage = new SqlServerStorage("<connection string>");
storage.UseMsmqQueues(@".\hangfire-{0}", "critical");

JobStorage.Current = storage;

var server = new AspNetBackgroundJobServer("critical", "default");
server.Start();
```

## Transition to MSMQ queues

If you have a fresh installation, just use the `UseMsmqQueues` method. Otherwise, your system may contain unprocessed jobs in SQL Server. Since one HangFire.Server instance can not process job from different queues, you should deploy two instances of HangFire.Server, one listens only MSMQ queues, another – only SQL Server queues. When the latter finish its work (you can see this from HangFire.Monitor – your SQL Server queues will be removed), you can remove it safely.