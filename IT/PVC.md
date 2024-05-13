### How to restore sitefiles?

[](https://github.com/saritasa-nest/jewexch-kubernetes-aws#how-to-restore-sitefiles)

We use velero for backup PVC.

You can see all backups with command `velero get backups`

We can restore PVC with two ways:

#### Delete old PVC and replace it with restored PVC from backup

[](https://github.com/saritasa-nest/jewexch-kubernetes-aws#delete-old-pvc-and-replace-it-with-restored-pvc-from-backup)

Bases on this tutorial: [https://www.ergton.com/velero-pvc-only-restore.html](https://www.ergton.com/velero-pvc-only-restore.html) We will do an example on staging env.

1. Go to [https://deploy.jewelryexchange.com](https://deploy.jewelryexchange.com) and disable sycing `root-staging-> jewexch-staging-apps-> jewexch-staging-wordpress`
2. Delete deployment `jewexch-staging-wordpress`
3. Go to `jewexch-staging-apps` and delete pvc `jewexch-staging-wordpress-pvc`
4. Now let's restore PVC from backup

```shell
velero get backups
velero restore create --from-backup velero-staging-20230716094831 --restore-volumes=true --include-resources pods,persistentvolumeclaims,persistentvolumes
# you can see velero logs for more details.
# In the end you must see create pvc
➜ k get pvc -n staging
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
jewexch-staging-wordpress-pvc Bound pvc-f1ebf820-0c1a-4be8-a121-00cc5829f3f1 100Gi RWO gp3 111s
jewexch-staging-wordpress-redis-pvc Bound pvc-7828eae0-6392-4ccf-948d-475ab5eb5e37 5Gi RWO gp3 10d
```

5. If you see created PVC - go to ArgoCD and sync `jewexch-staging-wordpress`. Pod was already created with velero, so it will recreate only deployment.
6. Done!

#### Restore any files from backup and keep current (safest way)

[](https://github.com/saritasa-nest/jewexch-kubernetes-aws#restore-any-files-from-backup-and-keep-current-safest-way)

Find recents AWS EBS snapshot by pvc name and click `Create Volume`: See carefully `Availability Zone` and remaining fields.

Now your volume will have an ID. Create file with context below and edit volumeId and env if you do it for production. Apply file with `k apply -f <filename>`

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jewexch-staging-restore-files
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 100Gi
  awsElasticBlockStore:
    fsType: ext4
    # Add necessary volume
    volumeID: vol-02408e0531c145834
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: topology.ebs.csi.aws.com/zone
              operator: In
              values:
                - us-west-2a
  persistentVolumeReclaimPolicy: Delete
  storageClassName: gp3
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jewexch-staging-wordpress-restore-files-pvc
  namespace: staging
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: gp3
  volumeMode: Filesystem
  volumeName: jewexch-staging-restore-files
status:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 100Gi
```

Edit `apps/wordpress/manifests/staging/wordpress.yaml`

Add this to sidecar `devtool` container

```yaml
- mountPath: /bitnami/wordpress2
 name: wordpress-data-restore-files
```

Add this to `extraVolumes`

```yaml
- name: wordpress-data-restore-files
 persistentVolumeClaim:
   claimName: jewexch-staging-wordpress-restore-files-pvc
```

Do argocd sync and exec into `devtool` cotnainer. Now you have current files on `/bitnami/wordpress` and restored files `/bitnami/wordpress2`

```shell
➜ k exec -it jewexch-staging-wordpress-64d4c6784-vk9kd -c devtool -- bash
deploy@jewexch-staging-wordpress-64d4c6784-vk9kd:~$ ls -lah /bitnami/wordpress2
total 24K
drwxrwsr-x 4 root deploy 4.0K Jul 3 18:36 .
drwxr-xr-x 4 root root 41 Jul 17 18:48 ..
drwxrws--- 2 root deploy 16K Jul 3 18:35 lost+found
drwxrwsr-x 13 root deploy 4.0K Jul 12 12:48 wordpress
deploy@jewexch-staging-wordpress-64d4c6784-vk9kd:~$ ls -lah /bitnami/wordpress2/wordpress/
total 888K
drwxrwsr-x 13 root deploy 4.0K Jul 12 12:48 .
drwxrwsr-x 4 root deploy 4.0K Jul 3 18:36 ..
drwxrwsr-x 8 deploy deploy 4.0K Jul 12 12:48 .git
-rw-rw-r-- 1 deploy deploy 797 Jul 12 12:48 .gitignore
-rw-rw-r-- 1 deploy deploy 0 Mar 16 04:20 .htacccess.swn
-rw-rw-r-- 1 deploy deploy 0 Mar 16 04:20
...
```

After you done, remove pv `k delete -f <filename>` and edit `wordpress.yaml`