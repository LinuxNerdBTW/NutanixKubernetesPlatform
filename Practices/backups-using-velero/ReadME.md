
## How to take backups using velero ? 
---
1. List available BSL - Backup Storage Location
  ```
  [root@bastion Practices]# velero backup-location get -n kommander
  NAME      PROVIDER   BUCKET/PREFIX   PHASE       LAST VALIDATED                    ACCESS MODE   DEFAULT
  default   aws        dkp-velero      Available   2024-12-23 21:29:49 +0545 +0545   ReadWrite     
  [root@bastion Practices]# 
  ```

I am going to use velero to take backup of following namespace 
  ```
  [root@bastion Practices]# k get pods -n k8s-gtxg6
  NAME                               READY   STATUS    RESTARTS   AGE
  db-6f6779dd68-grhdn                1/1     Running   0          6h39m
  redis-7ff5fd446f-qsqx4             1/1     Running   0          7h2m
  result-d8c4c69b8-2kpw8             1/1     Running   0          7h2m
  vote-69cb46f6fb-8f9sh              1/1     Running   0          7h2m
  wordpress-77b896bcf-vhxrl          1/1     Running   0          5h23m
  wordpress-mysql-5b589464d4-hhdw2   1/1     Running   0          5h23m
  worker-5dd767667f-tjtv5            1/1     Running   0          7h2m
  [root@bastion Practices]# k get pvc -n k8s-gtxg6
  NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     VOLUMEATTRIBUTESCLASS   AGE
  db-data-pvc      Bound    pvc-4eba3fbf-ba33-4bfe-8907-e53e09e437a6   5Gi        RWO            nutanix-volume   <unset>                 6h44m
  mysql-pv-claim   Bound    pvc-8b074f5f-83f6-4cc5-8406-2feffb23ce29   20Gi       RWO            nutanix-volume   <unset>                 5h24m
  redis-data       Bound    pvc-1bbd2d9f-5b24-4484-ba8e-b4e3b7aed6df   1Gi        RWO            nutanix-volume   <unset>                 7h2m
  wp-pv-claim      Bound    pvc-d94261cb-ccef-4e58-bc58-ff3c88763bda   20Gi       RWO            nutanix-volume   <unset>                 5h24m
  [root@bastion Practices]# k get deployments -n k8s-gtxg6
  NAME              READY   UP-TO-DATE   AVAILABLE   AGE
  db                1/1     1            1           6h40m
  redis             1/1     1            1           7h3m
  result            1/1     1            1           7h3m
  vote              1/1     1            1           7h3m
  wordpress         1/1     1            1           5h24m
  wordpress-mysql   1/1     1            1           5h24m
  worker            1/1     1            1           7h3m
  [root@bastion Practices]# 
  ```

2. Time to take backup
  ```
  [root@bastion Practices]# velero backup create myapp-backup --include-namespaces k8s-gtxg6 -n kommander --snapshot-volumes=false
  ```
  Use below command to check backup status: 
  ```
  [root@bastion Practices]# velero backup describe myapp-backup -n kommander
  Name:         myapp-backup
  Namespace:    kommander
  Labels:       velero.io/storage-location=default
  Annotations:  velero.io/resource-timeout=10m0s
                velero.io/source-cluster-k8s-gitversion=v1.30.5
                velero.io/source-cluster-k8s-major-version=1
                velero.io/source-cluster-k8s-minor-version=30

  Phase:  Completed
  ```

3. Using CRDs for listing backups 
  ```
  [root@bastion Practices]# k get backups -n kommander
  NAME           AGE
  myapp-backup   11m
  [root@bastion Practices]# 

  [root@bastion Practices]# k describe backups/myapp-backup -n kommander
  Name:         myapp-backup
  Namespace:    kommander
  Labels:       velero.io/storage-location=default
  Annotations:  velero.io/resource-timeout: 10m0s
                velero.io/source-cluster-k8s-gitversion: v1.30.5
                velero.io/source-cluster-k8s-major-version: 1
                velero.io/source-cluster-k8s-minor-version: 30
  API Version:  velero.io/v1
  Kind:         Backup
  Metadata:
    Creation Timestamp:  2024-12-23T15:41:32Z
    Generation:          6
    Resource Version:    801763
    UID:                 bcf0f762-ae69-4100-b42f-0420784cffe3
  Spec:
    Csi Snapshot Timeout:          10m0s
    Default Volumes To Fs Backup:  false
    Hooks:
    Included Namespaces:
      k8s-gtxg6
    Item Operation Timeout:  4h0m0s
    Metadata:
    Snapshot Move Data:  false
    Snapshot Volumes:    false
    Storage Location:    default
    Ttl:                 720h0m0s
    Volume Snapshot Locations:
      default
  Status:
    Completion Timestamp:  2024-12-23T15:41:41Z
    Expiration:            2025-01-22T15:41:33Z
    Format Version:        1.1.0
    Hook Status:
    Phase:  Completed
    Progress:
      Items Backed Up:  125
      Total Items:      125
    Start Timestamp:    2024-12-23T15:41:33Z
    Version:            1
  Events:               <none>
  [root@bastion Practices]# 
  ```
4. Troubleshooting Velero Backups Common Issues 
  velero logs can be collected using below command 
  ```
  [root@bastion Practices]# velero backup logs myapp-backup -n kommander
  An error occurred: Get "https://192.168.187.202:8085/dkp-velero/backups/myapp-backup/myapp-backup-logs.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=1L8L36CMRT0VGWMFH1CJ%2F20241223%2Fdkp-object-store%2Fs3%2Faws4_request&X-Amz-Date=20241223T155025Z&X-Amz-Expires=600&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=bfb0d3c915724b5b1f117de67e0815f60ad17a55cc1dd82223438db46749cb62": tls: failed to verify certificate: x509: certificate signed by unknown authority
  [root@bastion Practices]# 
  ```

  To troubleshoot velero backups related issues ? 

  list velero pods: 
  ```
  [root@bastion Practices]# k get pods -n kommander | grep velero
  object-bucket-claims-check-dkp-velero-4fdpw                       0/1     Completed   0               6h48m
  velero-backup-storage-location-updater-5d849446c9-46zcj           1/1     Running     0               6h49m
  velero-f487f9885-9w7tm                                            1/1     Running     0               6h50m
  velero-pre-install-shlbw                                          0/1     Completed   0               7h37m

  [root@bastion Practices]# k logs pods/velero-f487f9885-9w7tm -n kommander
  time="2024-12-23T15:51:49Z" level=info msg="BackupStorageLocations is valid, marking as available" backup-storage-location=kommander/default controller=backup-storage-location logSource="pkg/controller/backup_storage_location_controller.go:127"
  time="2024-12-23T15:51:49Z" level=warning msg="There is no existing BackupStorageLocation set as default. Please see `velero backup-location -h` for options." controller=backup-storage-location logSource="pkg/controller/backup_storage_location_controller.go:188"
  time="2024-12-23T15:51:49Z" level=info msg="plugin process exited" backup-storage-location=kommander/default cmd=/plugins/velero-plugin-for-aws controller=backup-storage-location id=8801 logSource="pkg/plugin/clientmgmt/process/logrus_adapter.go:80" plugin=/plugins/velero-plugin-for-aws
  time="2024-12-23T15:52:07Z" level=info msg="plugin process exited" backupLocation=kommander/default cmd=/plugins/velero-plugin-for-aws controller=backup-sync id=8811 logSource="pkg/plugin/clientmgmt/process/logrus_adapter.go:80" plugin=/plugins/velero-plugin-for-aws

  ```

## Creating Backup Schedule 

1. Setting Environment Variables for Backup 

  ```
  k get workspaces or nkp get workspaces 
  export WORKSPACE_NAMESPACE=<workspace_namespace>
  export CLUSTER_NAME=<cluster_name>

  ```

2. Creating Backup Schedule

  ```
  velero create schedule <backup-schedule-name> -n ${WORKSPACE_NAMESPACE} --kubeconfig=${CLUSTER_NAME}.conf --snapshot-volumes=false --schedule="@every 8h"
  velero create schedule <backup-schedule-name> -n ${WORKSPACE_NAMESPACE} --kubeconfig=${CLUSTER_NAME}.conf --snapshot-volumes=false --schedule="@every 8h" --include-namespaces=<namespace_name> # for taking backup of namespaces

  ```

3. Verify using below command

  ```
  k get schedule -n kommander

  NAME             STATUS    SCHEDULE    LASTBACKUP   AGE    PAUSED
  test             Enabled   @every 8h                8s     
  velero-default   Enabled   0 0 * * *   10h          4d1h   

  ```

## Replacing Default Backup Schedule with Custom Schedule 

1. List backup schedules 
  
  ```
  velero get schedules

  ```
2. Delete schedule using below command 
  
  ```
  velero delete schedule velero-default

  ```
3. Finally, replace the default settings with the custom settings using the following command

  ```
  velero create schedule velero-default -n ${WORKSPACE_NAMESPACE} --kubeconfig=${CLUSTER_NAME}.conf --snapshot-volumes=false --schedule="@every 24h"
  
  ```

## Restoring Namespace from backup

1. To create a backup of the sockshop namespace, run the following command:
  
  ```
  velero -n $WORKSPACE_NAMESPACE --kubeconfig roaring-lion.conf backup create sockshop --include-namespaces sockshop
  
  ```
2. Now that the backup is created, run the following command to get a list of all backups.

  ```
  velero -n $WORKSPACE_NAMESPACE --kubeconfig roaring-lion.conf backup get

  ```

3. Then, view the backup details using the following command:

  ```
  velero -n $WORKSPACE_NAMESPACE --kubeconfig roaring-lion.conf backup describe sockshop | more

  ```

## Restoring from a Backup

1. First, delete the sockshop namespace using the following command:

  ```
  kubectl delete namespace sockshop --kubeconfig roaring-lion.conf

  ```
2. Then, verify that the sockshop namespace has been deleted using the following command:

  ```
  kubectl get all -n sockshop --kubeconfig roaring-lion.conf
  
  No resources found in sockshop namespace.

  ```
2. Now that the namespace is deleted, we will run the following command to restore it from the backup created earlier.

  ```
  velero -n $WORKSPACE_NAMESPACE --kubeconfig roaring-lion.conf restore create sockshop --from-backup sockshop

  ```

3. Then, monitor the progress of the restore operation using the command:

  ```
  watch velero -n $WORKSPACE_NAMESPACE --kubeconfig roaring-lion.conf restore get

  ```
4. Review the warnings using the following command:

  ```
  velero -n $WORKSPACE_NAMESPACE --kubeconfig roaring-lion.conf restore logs sockshop --insecure-skip-tls-verify
  
  ```
5. Confirm that the resources in the sockshop namespace have been restored by using the following command

  ```
  kubectl get all -n sockshop --kubeconfig roaring-lion.conf

  ```

6. Then, using the following command retrieve the external IP address assigned to the front-end service to verify if the sockshop namespace is successfully restored.

  ```
  kubectl get service front-end -n sockshop --kubeconfig roaring-lion.conf

  ```

## Generating Diagnostic Data about Backup and Restore Operations

  ```
  velero get schedules
  velero get backups
  velero get restores
  velero get backup-locations
  velero get snapshot-locations

  ```

---