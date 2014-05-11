---
layout: post
---

Recurring jobs and ASP.NET applications have a long story. At the same time, the story is short, because you need to kick them off to external Windows Service to run them in a reliable way (see the [great article](http://haacked.com/archive/2011/10/16/the-dangers-of-implementing-recurring-background-tasks-in-asp-net.aspx/) by Phil Haack).

But Windows Services add additional complexity to your projects, especially on the initial stage of development. And it is sometimes hard enough to make a decision to install them only to send, for example, emails.

Recently I pushed my open-source project [HangFire](http://hangfire.io) – the solution to process background jobs in ASP.NET in a reliable way. Despite it does not support recurring jobs yet (but there are plans to implement it), it can greatly help you to postpone the decision to install Windows Service or completely forget about it.

In short, there are three risks of running recurring jobs inside ASP.NET applications (I changed the last one, comparing to the given article by Phil Haack):

1. Your application domain can be unloaded at any time for a number of reasons, and this can kill your threads in the middle of a job processing (even if you are using the `HostingEnvironment.RegisterObject` statement). No retries are made out of the box.
2. Multiple instances of your application (Web Gardens and Web Farm environments) can lead to multiple execution of the same task.
3. Your application domain may be not running. And your job firings can be missed.

The first risk can be eliminated by HangFire completely, that is why I wrote this article. If the second risk is actual for you, there is a `WebBackgrounder` library. And the third risk is fixed by the developers of ASP.NET (but you should make some changes to your configuration).

So, you can forget about using Windows Services to process jobs in the background in your web applications.

### Unexpected AppDomain unload

Please, read the given article by Phil Haack, or this one, or that one, to fully understand the problem, if you are not familiar with it. I'll provide the way HangFire handle this problem in the next article, and now I'll only show you the way.

The main problem of unexpected AppDomain unload is that your job can be aborted in the middle that leads to unfinished work even if it is atomic (otherwise the problem is bigger – you have inconsistent data state).

If all of your jobs are short-running (less than a second), and their number is small, you can forget about he problem by using the `IRegisteredObject` interface and the `HostingEnvironment.RegisterObject` method.

If you instruct your scheduler library to run the job at application startup (for example, `FluentScheduler` has `ToRunNow` method), then it is not a problem. But you can not use it for every scheduled job – if you want to send weekly email, or process monthly payments, you can not instruct scheduler to start it more often. 

So, you have rare long-running activity that should be started and the fact it can be dropped by ASP.NET in the middle of processing. On the other hand, HangFire is able to automatically restart the aborted task. So, instead of performing the job inside the scheduler, you can instruct the scheduler only to enqueue it.

But enqueueing task itself may be aborted. What to do? Apply the `IRegisteredObject` interface and register the task with `HostingEnvironment.RegisterObject`. And instead of long-running job, your scheduler will process short-running call.

### Multiple application instances

If this section is not for you, you can use even FluentScheduler project.



[**WebBackgrounder**](https://github.com/NuGet/WebBackgrounder)

> WebBackgrounder is a proof-of-concept of a web-farm friendly background task manager meant to just work with a vanilla ASP.NET web application.

It is written by Phil Haack, the author of the article given in the beginning. It is designed specially to run inside ASP.NET applications, so it is more safe to use it.

1. It uses `HostingEnvironment.RegisterObject` itself, you do need to worry about it. But when you use long-running jobs that can be aborted, they will be aborted without automatic retry.
2. Coordination is supported through SQL Server database using `WebBackgrounder.EntityFramework` package.

[**Quartz.NET**](http://quartz-scheduler.net)

It is the most powerful solution. It even can schedule one-off jobs, like the `BackgroundJob.Enqueue` method. But its recommended usage is in Windows Service. Let's describe the given risks when we use it inside ASP.NET application:

1. It does not have default implementation of `IRegisteredObject` interface. You should provide it yourself, otherwise abort percentage can be very large. 
If you have long running jobs, you should retry it manually in case of their abort. Otherwise they will not be retried, and even *misfire* configuration will not work, because it tracks only *fired* time and does not worry about completion result. So, you can fire a job, and if ASP.NET kills it during processing, it will silently die without automatic retry.
2. Quartz.NET has clustering support. http://www.quartz-scheduler.net/documentation/quartz-2.x/tutorial/advanced-enterprise-features.html

So, if you are running weekly or monthly jobs, and do not want to miss them, you can use Quartz.NET to call `BackgroundJob.Enqueue` method instead of job invocation. However, it is overkill solution for this case, and this is a ruddiness towards Quartz.NET to simply call HangFire.

[**FluentScheduler**](https://github.com/jgeurts/FluentScheduler)

> A task scheduler that uses fluent interface to configure schedules. Useful for running cron jobs/automated tasks from your application.

Lets overview our risks:

1. As in Quartz.NET
2. No coordination across application instances supported.

You can use it inside ASP.NET as Quartz.NET, but only if you are not planning to support web gardens or web farm.

Long-running jobs, Short-running jobs
Multiple instance, Single instance

ASP.NET -> always use IRegisteredObject (works by default in WebBackgrounder)

Single Instance? Any storage, including Fluent Scheduler
Multiple Instances? Persistent storage, Fluent scheduler does not work.

Short running? You don't have to install HangFire
Long running? You should install HangFire or Windows Service

Short-running + Single instance solutions:

* Fluent Scheduler
* WebBackgrounder (any storage)
* Quartz.NET (any storage)

Short-running + Multiple Instances

* WebBackgrounder (with SQL Server)
* Quartz.NET + persistent storage

Long-running + Single Instance

* Fluent Scheduler + HangFire
* WebBackgrounder (any storage) + HangFire
* Quartz.NET (any storage)

Long-running + Multiple Instances

* WebBackgrounder (with SQL) + HangFire
* Quartz.NET (with SQL) + HangFire

So, you know each risk and all available solutions. So let's build small helper tables.

### Instance count

Your choice of needed scheduler is based on how many instances of application do you have.

| Instances | FluentScheduler | WebBackgrounder | Quartz.NET  |
| --------- | --------------- | --------------- | ----------- |
| Single    | Yes             | Any edition     | Any edition |
| Multiple  | No              | EntityFramework | Cluster     |

### Background job length

Then, decide how much it will take to process your jobs:

| Job length         | Install HangFire? |
| ------------------ | ----------------- |
| Low (under 1 sec)  | No                |
| High (above 1 sec) | Yes               |

### Background jobs number

| Number | Install HangFire? |
| ------ | ----------------- |
| Low    | No                |
| High   | Yes               |

1. Can be aborted?

Yes - All solutions
No - 

2. Do you use multiple instances?
3. Do you have long-running jobs OR many jobs?

<table>
    <thead>
        <tr>
            <th rowspan="2">Solution</th>
            <th colspan="2">Single Instance</th>
            <th colspan="2">Multiple instances</th>
        </tr>
        <tr>
            <th>Short AND Few</th>
            <th>Long OR Many</th>
            <th>Short AND Few</th>
            <th>Long OR Many</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th>FS</th>
            <td>Yes</td>
            <td>No<sup><a href="#note-1">1</a></sup></td>
            <td colspan="2">No<sup><a href="#note-2">2</a></sup></td>
        </tr>
        <tr>
            <th>FS + HF</th>
            <td>Yes</td>
            <td>Yes</td>
            <td colspan="2">No<sup><a href="#note-2">2</a></sup></td>
        </tr>
        <tr>
            <th>WB</th>
            <td>Yes</td>
            <td>No<sup><a href="#note-1">1</a></sup></td>
            <td colspan="2">No<sup><a href="#note-2">2</a></sup></td>
        </tr>
        <tr>
            <th>WB EF</th>
            <td>Yes</td>
            <td>No<sup><a href="#note-1">1</a></sup></td>
            <td>Yes</td>
            <td>No<sup><a href="#note-1">1</a></sup></td>
        </tr>
        <tr>
            <th>WB + HF</th>
            <td>Yes</td>
            <td>Yes</td>
            <td colspan="2">No<sup><a href="#note-2">2</a></sup></td>
        </tr>
        <tr>
            <th>WB EF + HF</th>
            <td>Yes</td>
            <td>Yes</td>
            <td>Yes</td>
            <td>Yes</td>
        </tr>
        <tr>
            <th>QTZ</th>
            <td>Yes</td>
            <td>No<sup><a href="#note-1">1</a></sup></td>
            <td colspan="2">No<sup><a href="#note-2">2</a></sup></td>
        </tr>
        <tr>
            <th>QTZ + HF</th>
            <td>Yes</td>
            <td>Yes</td>
            <td colspan="2">No<sup><a href="#note-2">2</a></sup></td>
        </tr>
        <tr>
            <th>QTZ CLR</th>
            <td>Yes</td>
            <td>No<sup><a href="#note-1">1</a></sup></td>
            <td>Yes</td>
            <td>No<sup><a href="#note-1">1</a></sup></td>
        </tr>
        <tr>
            <th>QTZ CLR + HF</th>
            <td>Yes</td>
            <td>Yes</td>
            <td>Yes</td>
            <td>Yes</td>
        </tr>
    </tbody>
</table>

**Solutions**

* FS – FluentScheduler
* WB – WebBackgrounder
* HF – HangFire
* WB EF – WebBackgrounder + WebBackgrounder.EntityFramework
* QTZ – Quartz.NET
* QTZ CLR – Quartz.NET Cluster

**Risks**

1. Possible job loss. 
2. Multiple tasks. Uncoordinated task execution.

### Solution combinations

<p id="solutions">
* Fluent Scheduler
* Fluent Scheduler + HangFire
* Web Backgrounder
* Web Backgrounder with EntityFramework
* Web Backgrounder + HangFire
* Web Backgrounder with EntityFramework + HangFire
* Quartz.NET
* Quartz.NET Cluster
* Quartz.NET + HangFire
* Quartz.NET Cluster + HangFire
</p>

**1. Schedule only on intervals?**

* Yes – choose any framework.
* No – remove Web Backgrounder.

**2. Complex scheduling, like cron feature?**

* Yes – Remove Web Backgrounder and Fluent Scheduler (really?).
* No – Choose any framework.

**3. Do your app have or want to have multiple instances?**

* Yes – Remove all except Web Backgrounder with Entity Framework and Quartz.NET Cluster.
* No – Choose any framework.

**4. Do you have Long-Running jobs OR high number?**

* Yes – Remove all except with HangFire
* No – Choose any framework.

### Register your background job in ASP.NET

First of all, you need to implement IRegisteredObject interface and register your jobs through `HostingEnvironment.RegisterObject`. Web Backgrounder performs this work for you. In other frameworks do the following:

```csharp
public class Job : <base interface>, IRegisteredObject, IDisposable
{
    public Job()
    {
        HostingEnvironment.RegisterObject(this);
    }

    public void Dispose()
    {
        HostingEnvironment.UnregisterObject(this);
    }

    /* Other methods */
}
```

### Enable the Always Running feature

ajksdhfkjashf