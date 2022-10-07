---
id: backup-and-restore
title: "Backup & Restore"
sidebar_label: "Backup & Restore"
---

You can use backup feature of Camunda Platform 8 Self Managed to regularly backup the state of all of its components (Zeebe, Operate, Tasklist, and Optimize) without any downtime. In case of failures that lead to data loss, you can recover the cluster from a backup.

A backup of the Camunda Platform 8 consists of backup of Zeebe, Operate, TaskList, Optimize and the backup of exported zeebe records in ElasticSearch. Since the data of these applications are dependent on each other, it is important that the backup is consistent across all components. Hence you must take the backup of a Camunda Platform 8 cluster as a whole. The backups of individual components which are taken independently may not form a consistent recovery point. To ensure a consistent backup, follow the process described below.

### Configure Backup Store

To take backups, you must first configure backup storage. Operate, TaskList and Optimize use ElasticSearch as the backend storage. They use the snapshot feature of ElasticSearch for backing up their state. So you must configure a [snapshot repository](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html) in ElasticSearch. Zeebe stores its backup to an external storage. It must be configured before the cluster is started. Refer [Zeebe backup configuration](/self-managed/backup-restore/zeebe-backup-and-restore.md/#configuration).

### Backup Process

The backup of each component and the backup of a Camunda Platform 8 cluster is identified by an id. That means, a backup `x` of Camunda Platform 8 consists of backup `x` of Zeebe, backup `x` of Optimize, backup `x` of Operate, and backup `x` of Tasklist. The backup id must be an integer and greater than the previous backups. We recommend using the timestamp as the backup id.

To backup a Camunda Platform 8 cluster, execute the following steps in the order specified.

1. Trigger Backup `x` of Optimize (See [How to take Optimize backup](/self-managed/backup-restore/backup-and-restore.md))
2. Trigger Backup `x` of Operate (See [How to take Operate backup](/self-managed/backup-restore/backup-and-restore.md))
3. Trigger Backup `x` of Tasklist (See [How to take Tasklist backup](/self-managed/backup-restore/backup-and-restore.md))
4. Wait until Backup `x` of Optimize is complete (See [How to monitor Optimize backup](/self-managed/backup-restore/backup-and-restore.md))
5. Wait until Backup `x` of Operate is complete (See [How to monitor Operate backup](/self-managed/backup-restore/backup-and-restore.md))
6. Wait until Backup `x` of TaskList is complete (See [How to monitor Tasklist backup](/self-managed/backup-restore/backup-and-restore.md))
7. Pause exporting in Zeebe (See [Zeebe management api](/self-managed/zeebe-deployment/opearations/))
8. Take Backup `x` of exported zeebe records in ElasticSearch using ElasticSearch Snapshots API.

```
PUT /_snapshot/my_repository/camunda_zeebe_records_backup_x
{
   "indices": "zeebe-record*",
   "feature_states": ["none"]
}
```

By default, the indices are prefixed with `zeebe-record`. If you have configured a different prefix when configuring elasticsearch exporter in Zeebe, use that instead.

9. Take Backup `x` of Zeebe (See [How to take Zeebe backup](self-managed/backup-restore/zeebe-backup-and-restore.md))
10. Wait until Backup `x` of exported zeebe records is complete.
11. Wait until Backup `x` of Zeebe is completed (See [How to monitor Zeebe backup](self-managed/backup-restore/zeebe-backup-and-restore.md))
12. Resume exporting in Zeebe (See [Zeebe management api](/self-managed/zeebe-deployment/opearations/))

If any of the above steps failed, you may have to restart with a new backup id.

### Restore

To restore Camunda Platform 8 cluster from a backup, all components must be restored from their backup corresponding to the same backup id.

1. Restore the state of Operate, Tasklist and Optimize
2. Restore `zeebe-records*` indices
3. Restore Zeebe
4. Start Zeebe, Operate, Tasklist and Optimize
