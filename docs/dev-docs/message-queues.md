# Message Queues

Message queues used in GRACC

---

In AMQP, there is a difference between a _queue_ and an _exchange_.  Messages delivered on a _queue_ are read by a single subscriber; messages delivered on an _exchange_ will be delivered to all subscribers (implying they may be buffered for some time at the broker if a given client goes missing).

We would like collectors to serve multiple databases (hence the use of an _exchange_) while queues are used for messages sent to a database agent.

Well known message queues and exchanges used:

* `/gracc.<collector>.raw` - An exchange which listens to raw records to insert into the collector.  This is the interface that probes would send raw records.
* `/grace.<db>.summary` - A queue that listens for summary records to insert into a specific `<db>`.  This is used to replicate summary records from other collectors or db's.
* `/grace.<db>.raw` - Raw record queue for a database instance.
* `/grace.<db>.requests` - The [Ad Agent](agent-arch.md) listens to this queue for requests for raw and summary replications.

Here, `<db>` is the instance name of a given database install while `<collector>` is the instance name of an existing Gratia collector.

There are currently three defined message schemas in GRACC: raw records, summary records, and replay requests:

Raw Records
-----------

These are JSON-formatted documents; the key-value pairs are derived from the OGF *UsageRecord* format.  For ease of compatibility with the prior Gratia system, we include an `njobs` attribute if a given record represents more than one job.

The [Raw Records](raw-records.md) page has more details and the mapping from XML UsageRecord.

```
{
    "RecordId": "osg-gw-7.t2.ucsd.edu:35741.2",
    "CreateTime": "2016-05-27T22:46:46Z",
    "GlobalJobId": "condor.osg-gw-7.t2.ucsd.edu#185777.0#1464388242",
    "LocalJobId": "185777",
    "LocalUserId": "cmsuser",
    "GlobalUsername": "cmsuser@t2.ucsd.edu",
    "DN": "/DC=ch/DC=cern/OU=Organic Units/OU=Users/CN=sciaba/CN=430796/CN=Andrea Sciaba",
    "VOName": "/cms/Role=production/Capability=NULL",
    "ReportableVOName": "cms",
    "JobName": "osg-gw-7.t2.ucsd.edu#185777.0#1464388242",
    "MachineName": "osg-gw-7.t2.ucsd.edu",
    "SubmitHost": "osg-gw-7.t2.ucsd.edu",
    "Status": "0",
    "Status_description": "Condor Exit Status",
    "WallDuration": 617,
    "WallDuration_description": "Was entered in seconds"
    "TimeDuration": {
        "CommittedSuspensionTime": 0,
        "CommittedTime": 617,
        "CumulativeSuspensionTime": 0,
        "LocalSysCpu": 0,
        "LocalUserCpu": 0,
        "RemoteSysCpu": 18,
        "RemoteUserCpu": 0
    },
    "CpuDuration": 18,
    "CpuDuration_system": 18,
    "CpuDuration_system_description": "Was entered in seconds",
    "CpuDuration_user": 0,
    "CpuDuration_user_description": "Was entered in seconds",
    "EndTime": "2016-05-27T22:44:08Z",
    "StartTime": "2016-05-27T22:33:51Z",
    "Host": "cabinet-1-1-1.t2.ucsd.edu",
    "Queue": "5",
    "Queue_description": "Condor's JobUniverse field",
    "NodeCount": "1",
    "NodeCount_metric": "max",
    "Processors": "1",
    "Processors_metric": "max",
    "Resource": {
        "AccountingGroup": "group_cmsprod.cmsuser",
        "CondorMyType": "Job",
        "ExitBySignal": "false",
        "ExitCode": "0",
        "ResourceType": "Batch",
        "condor-JobStatus": "4"
    },
    "Network": "0",
    "Network_metric": "total",
    "Network_phaseUnit": 617,
    "Network_storageUnit": "b",
    "ProbeName": "condor:osg-gw-7.t2.ucsd.edu",
    "SiteName": "UCSDT2-D",
    "Grid": "OSG",
    "Njobs": "1",
}
```

!!! note
    We consider these to be "base" keys: additional ones may be given (for example, if the record is derived from a HTCondor ClassAd).


Summary Records
---------------

The summary record represents a grouping of multiple similar raw records.  In GRACC, we often group jobs run on the same date, by the same user, on the same resource.
```
TODO: copy JSON document here
```

Replay Requests
---------------

The replay request indicates that a remote listener agent attached to an ElasticSearch database should load and re-send some amount of data.

*Keys*:

* `time_range`: A Lucene-formatted time range containing the data that should be resent.
* `kind`: What type of records should be resent (valid values are curently `raw` or `summary`).
* `destination`: A queue on the same broker where records should be sent.  Should be a string value.
* `filter`: A ElasticSearch-formatted query filter (JSON value).  Only records matching this filter should be sent.

Example
```
{
  "time_range": "now-3d",
  "kind": "raw",
  "destination": "/grace.osg.raw",
  "filter": {
    "query": {
      "query_string": {
        "query": "vo=cms"
      }
    }
  }
}
```
