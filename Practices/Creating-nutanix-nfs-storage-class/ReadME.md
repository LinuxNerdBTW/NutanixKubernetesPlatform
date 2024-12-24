Using Nutanix Files as Storage Class on Nutanix Kubernetes Platform: 
---


1. Prerequisites
    - Make sure CSI Driver is deployed
    - Kubernetes version 1.20 or later
    - Prism Central version PC 2024.1 or later
    - API User and Password from Nutanix Files Dashboard
        - Navigate to Nutanix Files dashboard -> Configurations -> Manage Roles -> API Access 

2. Creating a Secret (Nutanix Files)
    Secrets are objects that contain sensitive data such as a key, token, or password.
    Copy and paste the following code into the file and then save the file. 
    
    ```
    apiVersion: v1
    kind: Secret
    metadata:
        name: pe-files-secret
        namespace: kube-system
    stringData:
    # Provide Nutanix Prism Element credentials which is a default UI credential separated by colon in "key:".
    # Provide Nutanix File Server credentials which is a REST API user created on File server UI separated by colon in "files-key:".
        key: "10.21.34.192:9440:admin:Password1"
        files-key: "fileserver01.sample.com:csi:password1"
    
    ```
3. Run kubectl to create the secret with your file. 
    
    ```
    kubectl create -f file_name.yaml 
    
    ```
4. Verify the secret name and information 
    
    ```
    kubectl get secret/pe-files-secret -n kube-system -o yaml
    
    ```

5. Creating a Storage Class (Nutanix Files)
    Log on as the cluster admin to create a storage class for Nutanix Files.
    1. Copy and paste the following code into the file and save the file.
        
        ```
        kind: StorageClass
        apiVersion: storage.k8s.io/v1
        metadata:
            name: acs-afs
            annotations:
                storageclass.kubernetes.io/is-default-class: "false"
        provisioner: csi.nutanix.com
        parameters:
            nfsServer: nfs-server # eg. nfs server ip / fqdn
            nfsPath: afs-nfs-share-path # eg. /fsfornai
            storageType: NutanixFiles
        reclaimPolicy: Delete or Retain
        volumeBindingMode: Immediate
        mountOptions:
        -option1
        -option2

        ```
    2. Run kubectl to create the file with your file name.
        
        ```
        kubectl create -f file_name.yaml
        
        ```
    3. Verify the storage class

        ```
        kubectl get storageclass

        ```
6. Creating a Storage Class for Dynamic NFS Shares [ Create this if you are using NFS storage for storing NAI LLM Modules ]
    You can use a storage class to dynamically provision Nutanix Files shares, or you can use existing Files shares for storage. Dynamically provisioned shares have one-to-one mapping with persistent volumes (PVs): each PV has a dedicated share for storage. The size of the PV and the share is enforced by the size specified in the persistent volume claim (PVC).
    1. Copy and paste the following code into the file and save the file:
        Replace the secret-name with secret name that we have created on step 2 and replace secret-namespace with kube-system

        ```
        allowVolumeExpansion: true
        kind: StorageClass
        apiVersion: storage.k8s.io/v1
        metadata:
            name: test-sc
        provisioner: csi.nutanix.com
        parameters:
            csi.storage.k8s.io/node-publish-secret-name: secret-name
            csi.storage.k8s.io/node-publish-secret-namespace: secret-namespace
            csi.storage.k8s.io/controller-expand-secret-name: secret-name
            csi.storage.k8s.io/controller-expand-secret-namespace: secret-namespace
            dynamicProv: ENABLED
            nfsServerName: fs-name
            csi.storage.k8s.io/provisioner-secret-name: secret-name
            csi.storage.k8s.io/provisioner-secret-namespace: secret-namespace
            storageType: NutanixFiles
            squashType: root-squash
        reclaimPolicy: Delete
        volumeBindingMode: Immediate
        mountOptions:
        -option1

        ```
    2. Run kubectl to create the storage class 
        
        ```
        kubectl create -f file_name.yaml

        ```
    3. Verify the storage class

        ```
        kubectl get storageclass

        ```








More information can be found on below documentation:
https://portal.nutanix.com/page/documents/details?targetId=CSI-Volume-Driver-v3_2:csi-csi-plugin-overview-c.html


---