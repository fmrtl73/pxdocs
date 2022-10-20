---
title: 3. Synchronize your clusters or Schedule migrations
linkTitle: 3. Synchronize your clusters
weight: 300
keywords: Asynchronous DR, disaster recovery, kubernetes, k8s, cloud, backup, restore, snapshot, migration
description: Find out how to synchronize your cluster by scheduling periodic migrations between them
aliases:
    - /portworx-install-with-kubernetes/disaster-recovery/px-metro/3-schedule-migrations/
   
---
To keep the Kubernetes resources between your paired source and destination cluster in sync, you need to periodically migrate them.

For reference,

* **Source Cluster** is the Kubernetes cluster where your applications are running
* **Destination Cluster** is the Kubernetes cluster where the applications will be failed over, in case of a disaster in the source cluster.

## Create a schedule policy

To schedule a migration, you must create a schedule policy.

1. Paste the following content into a file called `testpolicy.yaml`:

    ```text
    apiVersion: stork.libopenstorage.org/v1alpha1
    kind: SchedulePolicy
    metadata:
      name: testpolicy
      namespace: mysql
    policy:
      interval:
        intervalMinutes: 1
      daily:
        time: "10:14PM"
      weekly:
        day: "Thursday"
        time: "10:13PM"
      monthly:
        date: 14
        time: "8:05PM"
    ```

    For details about how you can configure a schedule policy, see the [Schedule Policy](/reference/crd/schedule-policy/) reference page.

2. Apply your spec by entering the following command:

    ```text
    kubectl apply -f testpolicy.yaml
    ```

3. Display your schedule policy. Enter the `storkctl get` command passing it the name of your policy:

    ```text
    storkctl get schedulepolicy
    ```

    ```output
    NAME           INTERVAL-MINUTES   DAILY     WEEKLY             MONTHLY
    testpolicy     1                  10:14PM   Thursday@10:13PM   14@8:05PM
    ```

## Schedule a migration

Once a policy has been created, you can use it to schedule a migration. The MigrationSchedule object is namespaced.

{{<info>}}
**NOTE:** If your cluster has a DR license applied to it, you can only perform migrations in DR mode; this includes operations involving the `pxctl cluster migrate` command.
{{</info>}}

Continuing our previous example with `testpolicy`, here is how to create a `MigrationSchedule` object that schedules a migration:

```text
apiVersion: stork.libopenstorage.org/v1alpha1
kind: MigrationSchedule
metadata:
  name: mysqlmigrationschedule
  namespace: migrationnamespace
spec:
  template:
    spec:
      # This should be the name of the cluster pair created above
      clusterPair: remotecluster
      # If set to false this will migrate only the Portworx volumes. No PVCs, apps, etc will be migrated
      includeResources: true
      # If set to false, the deployments and stateful set replicas will be set to 0 on the destination.
      # If set to true, the deployments and stateful sets will start running once the migration is done
      # There will be an annotation with "stork.openstorage.org/migrationReplicas" on the destinationto store the replica count from the source.
      startApplications: false
       # If set to false, the volumes will not be migrated
      includeVolumes: false
      # List of namespaces to migrate
      namespaces:
      - migrationnamespace
  schedulePolicyName: testpolicy
```

A few things to note:

* The option `includeVolumes` is set to false because the volumes are already present on the destination cluster as there is a single storage fabric
* The option `startApplications` is set to false so that the applications do not start when the resources are migrated. This is because the applications on the destination cluster should start only when the application fails over.


If the policy name is missing or invalid there will be events logged against the schedule object. Success and failures of the migrations created by the schedule will also result in events being logged against the object. These events can be seen by running a `kubectl describe` on the object

The output of `kubectl describe` will also show the status of the migrations that were triggered for each of the policies along with the start and finish times. The statuses will be maintained for the last successful migration and any Failed or InProgress migrations for each policy type.

Let's now run `kubectl describe` and see how the output would look like:

```text
kubectl describe migrationschedules.stork.libopenstorage.org -n mysql
```

```output
Name:         mysqlmigrationschedule

Namespace:    migrationnamespace
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"stork.libopenstorage.org/v1alpha1","kind":"MigrationSchedule","metadata":{"annotations":{},"name":"mysqlmigrationschedule",...
API Version:  stork.libopenstorage.org/v1alpha1
Kind:         MigrationSchedule
Metadata:
  Creation Timestamp:  2019-02-14T04:53:58Z
  Generation:          1
  Resource Version:    30206628
  Self Link:           /apis/stork.libopenstorage.org/v1alpha1/namespaces/mysql/migrationschedules/mysqlmigrationschedule
  UID:                 8a245c1d-3014-11e9-8d3e-0214683e8447
Spec:
  Schedule Policy Name:  daily
  Template:
    Spec:
      Cluster Pair:       remotecluster
      Include Resources:  true
      Include Volumes:    false
      Namespaces:
        mysql
      Post Exec Rule:
      Pre Exec Rule:
      Selectors:           <nil>
      Start Applications:  false
Status:
  Items:
    Daily:
      Creation Timestamp:  2019-02-14T22:16:51Z
      Finish Timestamp:    2019-02-14T22:19:51Z
      Name:                mysqlmigrationschedule-daily-2019-02-14-221651
      Status:              Successful
    Interval:
      Creation Timestamp:  2019-02-16T00:40:52Z
      Finish Timestamp:    2019-02-16T00:41:52Z
      Name:                mysqlmigrationschedule-interval-2019-02-16-004052
      Status:              Successful
      Creation Timestamp:  2019-02-16T00:41:52Z
      Finish Timestamp:    <nil>
      Name:                mysqlmigrationschedule-interval-2019-02-16-004152
      Status:              InProgress
    Monthly:
      Creation Timestamp:  2019-02-14T20:05:41Z
      Finish Timestamp:    2019-02-14T20:07:41Z
      Name:                mysqlmigrationschedule-monthly-2019-02-14-200541
      Status:              Successful
    Weekly:
      Creation Timestamp:  2019-02-14T22:13:51Z
      Finish Timestamp:    2019-02-14T22:16:51Z
      Name:                mysqlmigrationschedule-weekly-2019-02-14-221351
      Status:              Successful
Events:
  Type    Reason      Age                    From   Message
  ----    ------      ----                   ----   -------
  Normal  Successful  4m55s (x53 over 164m)  stork  (combined from similar events): Scheduled migration (mysqlmigrationschedule-interval-2019-02-16-003652) completed successfully
```

Each migration is associated with a Migrations object. To get the most important information, type:

```text
kubectl get migration -n migrationnamespace
```

```output
NAME AGE
mysqlmigrationschedule-daily-2019-02-14-221651 1d
mysqlmigrationschedule-interval-2019-02-16-004052 5m
mysqlmigrationschedule-interval-2019-02-16-004152 4m
mysqlmigrationschedule-monthly-2019-02-14-200541 1d
mysqlmigrationschedule-weekly-2019-02-14-221351 1d
```

Once the MigrationSchedule object is deleted, all the associated Migration objects should also be deleted as well.
