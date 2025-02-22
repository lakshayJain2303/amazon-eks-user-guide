# Tutorial: Deploy the Kubernetes Dashboard \(web UI\)<a name="dashboard-tutorial"></a>

This tutorial guides you through deploying the [Kubernetes Dashboard](https://github.com/kubernetes/dashboard) to your Amazon EKS cluster, complete with CPU and memory metrics\. It also helps you to create an Amazon EKS administrator service account that you can use to securely connect to the dashboard to view and control your cluster\. If you have issues using the dashboard, you can create an [issue](https://github.com/kubernetes/dashboard/issues) or [pull request](https://github.com/kubernetes/dashboard/pulls) in the project's GitHub repository\.

## Prerequisites<a name="dashboard-prereqs"></a>

This tutorial assumes the following:
+ You have created an Amazon EKS cluster by following the steps in [Getting started with Amazon EKS](getting-started.md)\.
+ You have the Kubernetes Metrics Server installed\. For more information, see [Installing the Kubernetes Metrics Server](metrics-server.md)\.
+ The security groups for your control plane elastic network interfaces and nodes follow the recommended settings in [Amazon EKS security group requirements and considerations](sec-group-reqs.md)\.
+ You are using a `kubectl` client that is [configured to communicate with your Amazon EKS cluster](getting-started-console.md#eks-configure-kubectl)\.

## Step 1: Deploy the Kubernetes dashboard<a name="deploy-dashboard"></a>

Apply the dashboard manifest to your cluster using the command for the version of your cluster\.
+ Version 1\.22

  Some features of the available versions might not work properly with this Kubernetes version\. For more information, see [Releases](https://github.com/kubernetes/dashboard/releases) on GitHub\.
+ Versions 1\.20 and 1\.21

  ```
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml
  ```
+ Version 1\.19

  ```
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.5/aio/deploy/recommended.yaml
  ```

The example output is as follows\.

```
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

## Step 2: Create an `eks-admin` service account and cluster role binding<a name="eks-admin-service-account"></a>

By default, the Kubernetes Dashboard user has limited permissions\. In this section, you create an `eks-admin` service account and cluster role binding that you can use to securely connect to the dashboard with admin\-level permissions\. For more information, see [Managing Service Accounts](https://kubernetes.io/docs/admin/service-accounts-admin/) in the Kubernetes documentation\.

**To create the `eks-admin` service account and cluster role binding**
**Important**  
The example service account created with this procedure has full `cluster-admin` \(superuser\) privileges on the cluster\. For more information, see [Using RBAC authorization](https://kubernetes.io/docs/admin/authorization/rbac/) in the Kubernetes documentation\.

1. Run the following command to create a file named `eks-admin-service-account.yaml` with the following text\. This manifest defines a service account and cluster role binding named `eks-admin`\.

   ```
   cat >eks-admin-service-account.yaml <<EOF
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: eks-admin
     namespace: kube-system
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:
     name: eks-admin
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: ClusterRole
     name: cluster-admin
   subjects:
   - kind: ServiceAccount
     name: eks-admin
     namespace: kube-system
   EOF
   ```

1. Apply the service account and cluster role binding to your cluster\.

   ```
   kubectl apply -f eks-admin-service-account.yaml
   ```

   The example output is as follows\.

   ```
   serviceaccount "eks-admin" created
   clusterrolebinding.rbac.authorization.k8s.io "eks-admin" created
   ```

## Step 3: Connect to the dashboard<a name="view-dashboard"></a>

Now that the Kubernetes Dashboard is deployed to your cluster, and you have an administrator service account that you can use to view and control your cluster, you can connect to the dashboard with that service account\.

**To connect to the Kubernetes dashboard**

1. Retrieve an authentication token for the `eks-admin` service account\.

   ```
   kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
   ```

   The example output is as follows\.

   ```
   Name:         eks-admin-token-b5zv4
   Namespace:    kube-system
   Labels:       <none>
   Annotations:  kubernetes.io/service-account.name=eks-admin
                 kubernetes.io/service-account.uid=bcfe66ac-39be-11e8-97e8-026dce96b6e8
   
   Type:  kubernetes.io/service-account-token
   
   Data
   ====
   ca.crt:     1025 bytes
   namespace:  11 bytes
   token:      authentication-token
   ```

   Copy the `authentication-token` value from the output\. You use this token to connect to the dashboard in a later step\.

1. Start the `kubectl proxy`\.

   ```
   kubectl proxy
   ```

1. To access the dashboard endpoint, open the following link with a web browser: [http://localhost:8001/api/v1/namespaces/kubernetes\-dashboard/services/https:kubernetes\-dashboard:/proxy/\#\!/login](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#!/login)\.

1. Choose **Token**, paste the `authentication-token` output from the previous command into the **Token** field, and choose **SIGN IN**\.

   After signing in, you see the dashboard in your web browser\.  
![\[Kubernetes Dashboard\]](http://docs.aws.amazon.com/eks/latest/userguide/images/kubernetes-dashboard.png)

   For more information about using the dashboard, see [Deploy and Access the Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) in the Kubernetes documentation\.