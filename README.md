# A Hands-On Guide to Kubernetes Volumes 🛠️
![10](https://github.com/user-attachments/assets/46ba9266-5e37-4599-9fd9-b2a4d030ae5a)

# Understanding PV, PVC & SC in Kubernetes: With Practical Examples

Kubernetes volumes are crucial for managing data within your Kubernetes cluster. They enable stateful applications and data sharing between pods, providing persistent storage beyond the lifecycle of a single container. In this guide, we’ll explore the key concepts of Kubernetes volumes, including Persistent Volumes (PV), Persistent Volume Claims (PVC), Storage Classes (SC), attaching volumes to pods, Static Provisioning, and Dynamic Provisioning.

# Kubernetes Persistent Storage
This repository provides examples and explanations of different Kubernetes storage options. Here’s a simple guide to understand how different types of persistent storage work in Kubernetes.

# Storage Types:

1.	**EmptyDir**: An initially empty directory created on the node’s filesystem. It is usually used for scratch space or temporary storage and lasts as long as the pod is running. Data is deleted when the pod is terminated.

2.	**HostPath**: Maps a file or directory from the host node’s filesystem into the pod. It is often used to expose host resources (like Docker or logs) to the containers. However, it ties the pod to a specific node and can reduce portability.

3. **Persistent Volume (PV)** : PVs are managed by the cluster and can be dynamically provisioned using storage classes.

4.	**PersistentVolumeClaim (PVC)**: A request for storage by a pod. PVCs are used to claim resources defined by PersistentVolumes (PVs) and are used with dynamically or statically provisioned storage. This type is ideal for stateful applications where data needs to persist beyond the lifecycle of the pod.
  
5.	**AWS EBS (Elastic Block Store)**: An Amazon Web Services-specific volume type. It provides durable, block-level storage that can be used with single pods in the AWS environment. The volume is tied to a single Availability Zone.

6.	**Storage Classes (SC)**: Storage Classes define the types of storage available in the cluster and how they are provisioned. They enable dynamic provisioning of PVs based on predefined templates or policies.


###########################################################################################################

# Architecture flow Diagram:


![architecture](https://github.com/user-attachments/assets/49a4340e-5507-426d-bc2d-74544b271dbd)






# Lets see the Practicals of all one by one:

# 1. EmptyDir:

Lets create a volume:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: troubleshooting
          image: rajpractise/troubleshootingtools:v1
          volumeMounts:
            - name: myemptydir
              mountPath: /etc/myemptydir
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: /tmp/myemptydir
              name: myemptydir
      volumes:
        - name: myemptydir
          emptyDir: {}
```


![emptydir1](https://github.com/user-attachments/assets/1c772409-602a-4996-98e3-2ee98d19908c)


# Lets get into one of the containers i.e. troubleshooting and create some files:

![emptydir2-troubleshoot-cont](https://github.com/user-attachments/assets/b8fd8d72-7061-4a02-86be-f65b7ce5f500)



# Now lets get into the other container i.e nginx container:

![emptydir3-nginx-cont](https://github.com/user-attachments/assets/93c292bd-42f0-49cb-a670-8932895631c7)


We can see the same files here which we created from the other conatiner due to volume mount.

############################################################################################################

# 2. HostPath:

# lets create a volume and mount it.

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: containerd-deamonset
  labels:
    app: containerd-deamonset
spec:
  selector:
    matchLabels:
      app: containerd-deamonset
  template:
    metadata:
      labels:
        app: containerd-deamonset
    spec:
      containers:
      - name: troubleshooting
        image: rajpractise/troubleshootingtools:v1 
        volumeMounts:
          - name: containerdsock
            mountPath: "/run/containerd/containerd.sock"
      volumes:
        - name: containerdsock
          hostPath:
            path: /run/containerd/containerd.sock
```



# lets get into the container and install some packages:


![hostpath1](https://github.com/user-attachments/assets/9d94e92c-6235-4e1d-9de0-3b403d65824a)



![hostpath2](https://github.com/user-attachments/assets/483be5f9-d778-497b-83ec-1c0edbd1db84)


Generally we use volume of type hostpath when we want to get some info from any path on the host's.


##############################################################################################################################################


# 3. AWS EBS (Elastic Block Store):

Lets create a volume of type ebs and mount it.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
      name: mongodb
  template:
    metadata:
      labels:
        app: mongodb
        name: mongodb
    spec:
      containers:
        - image: mongo
          name: mongodb
          imagePullPolicy: Always
          volumeMounts:
            - name: mongodb-data
              mountPath: /data/db
      volumes:
        - name: mongodb-data
          awsElasticBlockStore:
            volumeID: vol-0b6df5d4b684b35d9
            fsType: ext4
      nodeSelector:
        kubernetes.io/hostname: i-00eebdfcde16dcdc2
```


# Lets get into the mongo container:


![ebsvol1](https://github.com/user-attachments/assets/82461abd-465a-4594-843f-cc11aabfda3e)




# Now add some data into the mongo db:

![ebsvol2-add-data](https://github.com/user-attachments/assets/eafe9b23-4a7a-4447-812f-c0e5ee3fdcc0)



# Data got added.


![ebsvol3-data-added](https://github.com/user-attachments/assets/da8b6654-9354-4fdc-8e8e-f5522f7e7363)





# Now lets delete the pod and recreate it.

![ebsvol4-delete-deployment](https://github.com/user-attachments/assets/bf8fb0f4-eac3-47ec-aaec-f45d7a5a2ed9)




# after creating the pod again, we can still see the data we stored previously in mongo db:


![ebsvol4-created-pod-again-tosee-persist-data-using-ebs](https://github.com/user-attachments/assets/a5f55d10-fa77-48b5-a56c-4c0e8e446809)




![ebsvol5-created-pod-again-tosee-persist-data-using-ebs](https://github.com/user-attachments/assets/52a7f289-7e39-4d13-9136-c7eab9ca55ba)



###################################################################################################################



# Provisioning of PersistentVolumes:

There are two ways PVs may be provisioned: **statically or dynamically**.

**1. Static**
In static provisioning, the cluster administrator creates PVs with details of available storage. These PVs are pre-defined in the Kubernetes API and are ready for use by cluster users.

**2. Dynamic**
In dynamic provisioning, when no static PV matches a user’s PersistentVolumeClaim (PVC), the cluster can automatically provision a volume. This is based on StorageClasses: the PVC must specify a storage class, and the administrator must have set up and configured that class for dynamic provisioning. Claims that request an empty string for the class effectively opt-out of dynamic provisioning.


Let’s walk through both examples of how static and dynamic provisioning work in Kubernetes.


# Static Provisioning:

**Step 1: PersistentVolume (PV)**


# First create **ebs volumes** on aws using aws cli:

```bash
# aws ec2 create-volume --volume-type gp2 --size 2 --availability-zone us-east-1a --tag-specifications 'ResourceType=volume,Tags=[{Key=KubernetesCluster,Value=cloudopswithswapnil.in},{Key=Name,Value=aws-pv1}]' && aws ec2 create-volume --volume-type gp2 --size 4 --availability-zone us-east-1a --tag-specifications 'ResourceType=volume,Tags=[{Key=KubernetesCluster,Value=cloudopswithswapnil.in},{Key=Name,Value=aws-pv2}]' && aws ec2 create-volume --volume-type gp2 --size 6 --availability-zone us-east-1a --tag-specifications 'ResourceType=volume,Tags=[{Key=KubernetesCluster,Value=cloudopswithswapnil.in},{Key=Name,Value=aws-pv3}]' && aws ec2 create-volume --volume-type gp2 --size 8 --availability-zone us-east-1a --tag-specifications 'ResourceType=volume,Tags=[{Key=KubernetesCluster,Value=cloudopswithswapnil.in},{Key=Name,Value=aws-pv4}]' && aws ec2 create-volume --volume-type gp2 --size 10 --availability-zone us-east-1a --tag-specifications 'ResourceType=volume,Tags=[{Key=KubernetesCluster,Value=cloudopswithswapnil.in},{Key=Name,Value=aws-pv5}]'
```


# volumes got created:


![pv1-volumescreated](https://github.com/user-attachments/assets/9c7372fa-2838-4926-94b1-cc62a362f8a7)





# Create PersistentVolume (PV) manifest file using the below content.

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aws-pv1
  labels:
    type: aws-pv1
spec:
  storageClassName: gp2
  persistentVolumeReclaimPolicy: Delete
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: vol-0d7fb4a82baebe5f8
    fsType: ext4

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aws-pv2
  labels:
    type: aws-pv2
spec:
  storageClassName: gp2
  persistentVolumeReclaimPolicy: Delete
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: vol-066858df80bc16346
    fsType: ext4

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aws-pv3
  labels:
    type: aws-pv3
spec:
  storageClassName: gp2
  persistentVolumeReclaimPolicy: Delete
  capacity:
    storage: 6Gi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: vol-0848ad77be4251e75
    fsType: ext4

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aws-pv4
  labels:
    type: aws-pv4
spec:
  storageClassName: gp2
  persistentVolumeReclaimPolicy: Delete
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: vol-00e1fd8eb84fc6ab3
    fsType: ext4

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aws-pv5
  labels:
    type: aws-pv5
spec:
  storageClassName: gp2
  persistentVolumeReclaimPolicy: Delete
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: vol-02a4a6a38b500fcda
    fsType: ext4
```


![pv2-pvvols-created](https://github.com/user-attachments/assets/5e42a604-0958-4f4f-be0a-50530b1b370b)




**Step 2: PersistentVolumeClaims (PVCs)**:

Create PersistentVolumeClaim (PV) manifest file using the below content.

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim1
spec:
  storageClassName: gp2
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim2
spec:
  storageClassName: gp2
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim3
spec:
  storageClassName: gp2
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim4
spec:
  storageClassName: gp2
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim5
spec:
  storageClassName: gp2
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 6Gi
```


When a PVC is created, Kubernetes binds it to an available PV that matches its requirements.

# We can see that all the pv which we created got claimed by pvc:



![pv3-pvc](https://github.com/user-attachments/assets/f0dd6a3a-d7cf-4273-a950-244aec06faa7)




# Step 3: Deployment:

Now let’s try to create the deployment and attach the above PVC to it.

Create deployment.yaml using the below manifest file:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
      name: mongodb
  template:
    metadata:
      labels:
        app: mongodb
        name: mongodb
    spec:
      containers:
        - image: mongo
          name: mongodb
          imagePullPolicy: Always
          volumeMounts:
            - name: mongodb-data1
              mountPath: /tmp/db1
            - name: mongodb-data2
              mountPath: /tmp/db2
            - name: mongodb-data3
              mountPath: /tmp/db3
            - name: mongodb-data4
              mountPath: /tmp/db4
      volumes:
        - name: mongodb-data1
          persistentVolumeClaim:
            claimName: task-pv-claim1
        - name: mongodb-data2
          persistentVolumeClaim:
            claimName: task-pv-claim2
        - name: mongodb-data3
          persistentVolumeClaim:
            claimName: task-pv-claim3
        - name: mongodb-data4
          persistentVolumeClaim:
            claimName: task-pv-claim4
      nodeSelector:
        kubernetes.io/hostname: i-00eebdfcde16dcdc2
```

In the above manifest file claimName: task-pv-claim1, task-pv-claim2, task-pv-claim3, task-pv-claim4 will be used to attach the PVC to the container.



# lets get into the container to verify the volumes which we mounted through the pvc got mounted or not:




![pv4-pvcmounted](https://github.com/user-attachments/assets/0321ee2d-d52a-4087-9016-f50c4eb689a1)




PV, PVC and Deployments are created.

**Finally, a pod is created that mounts the PVC to its container, enabling the pod to use the persistent storage.**


###########################################################################################################################


# Dynamic Provisioning:

**Introduction:**
Dynamic provisioning in Kubernetes enables automatic creation of PVs when there are no existing PVs that match the requirements of a PVC.


# Lets create a pvc of size 20Gi:

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim6
spec:
  storageClassName: gp2
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```


# We can see that pvc got created and got bounded to a pv of size 20Gi which was not present.



![pv5-dynamic-vol-provisioning](https://github.com/user-attachments/assets/5f3e4d84-bb5b-4f36-a9a9-0ef2fd7675c1)





 We can see in the aws console as well that the volumes got created with alias as **dynamic-pvc**

![pv5-dynamic-vol-provisioning-aws-cons-ebs-vol](https://github.com/user-attachments/assets/0632e713-dd84-47f8-8a5a-bb6082a30161)





PV is created here automatically with the help of storageClass and got bounded to the pvc which we created and claimed.

Here since we used **KOps** to provision our **k8s-Cluster** hence it by default have some **storage classes** by default which can be seen below:



![pv6-dynamic-vol-provisioning-deafult-storageclasses](https://github.com/user-attachments/assets/9402d155-fffd-452a-a48a-37b32926d123)





# Now lets delete these storage classes and try to again create a pvc with vol of size 20Gi and see what happens:



![pv7-dynamic-vol-provisioning](https://github.com/user-attachments/assets/3c647216-af3f-4f15-b807-e7e0cc18dd97)



We can see that since storage classes are not present hence pvc got into **Pending State**


# Now lets create the storage classes and again claim a pvc and see what happens:

```bash
#Creating gp2 storage class and making it default
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: gp2
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4 

#Creating io storage class
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: io1
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4
```


![pv8-dynamic-vol-provisioning-volcreatedafterstorageclassescreated](https://github.com/user-attachments/assets/34c1fb16-01a3-4ef8-877f-40a211ef4bc7)




We could see that the new since storage classes now got created hence the new claim for pvc and the older claim which went into pending state, both got created and got bounded to pv successfully due to **Dynamic Provisioning**.




































