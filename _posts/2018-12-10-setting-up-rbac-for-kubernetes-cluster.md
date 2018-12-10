---
layout: post
title:  "Setting up RBAC for Kubernetes Cluster"
date:   2018-10-10 14:07:23 +0530
categories: kubernetes
comments: true
---

Once you've set up Kubernetes Cluster and provisioned your applications, next in line is giving permissions to
developers and other employees. Usually these requests range from __ssh access to logs and consoles__.

It's needless to say that giving complete __kubectl__ access for everyone in your organization. We can use  RBAC in Kubernetes to
restrict access based on roles. We are using __AWS IAM__ as the authorization provider in this tutorial.

## What is RBAC

RBAC(Role-based access control) is exactly what the name suggests, it let's give fine grained permissions to users/groups
in Kubernetes. You can create __Role__ objects in Kubernetes which is just like a permission set and attach it to a user
or group.

You should note that Kubernetes doesn't recognise _User_ and _Group_ as native objects, which means you can't just
create those like `kubectl create user`. You'll have to use a third party or cloud solution(AWS IAM in this demo) to manage it.

## Prerequisites

- Kubernetes Cluster (Create one with [kops](https://github.com/kubernetes/kops))
- AWS Account with [IAM](https://aws.amazon.com/iam/) users.
- AWS [Cli](https://github.com/aws/aws-cli)
- [aws-iam-authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator)

## How do we do it ?

We have to set up 2 logical parts for this to work, we've to set up __Authentication__ part to verify identity of
users and __Authorization__ for defining specific permissions based on role.

### Authentication
We are installing and configuring [aws-iam-authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator) in this
step, there is a detailed installation guide in [Github](https://github.com/kubernetes-sigs/aws-iam-authenticator) page,
feel free to skip this if you want to check that out.


First we need to create a IAM role for __KubernetesAdmin__ which will be attached to your AWS IAM profile. You can either
create role named KubernetesAdmin through UI or use AWS cli to create one.
```
# get your account ID
ACCOUNT_ID=$(aws sts get-caller-identity --output text --query 'Account')

# define a role trust policy that opens the role to users in your account (limited by IAM policy)
POLICY=$(echo -n '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"AWS":"arn:aws:iam::'; echo -n "$ACCOUNT_ID"; echo -n ':root"},"Action":"sts:AssumeRole","Condition":{}}]}')

# create a role named KubernetesAdmin (will print the new role's ARN)
aws iam create-role \
  --role-name KubernetesAdmin \
  --description "Kubernetes administrator role (for AWS IAM Authenticator for Kubernetes)." \
  --assume-role-policy-document "$POLICY" \
  --output text \
  --query 'Role.Arn'
```

[aws-iam-authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator) consists of a __ConfigMap__ and __DaemonSet__
. ConfigMap is where we map IAM user account with username, DaemonSet is to be installed in all master nodes.

You can find both configurations from [here](https://github.com/kubernetes-sigs/aws-iam-authenticator/blob/master/example.yaml).
Apply above configurations before moving on to next steps.

Once both objects are created, now we need to setup aws-iam-authenticator in local machine. Download
__aws-iam-authencator [binary](https://github.com/kubernetes-sigs/aws-iam-authenticator/releases)__ based on your OS, give
executable permission and copy to $PATH(Usually /usr/local/bin).

We need to update our local _kubectl_ configuration to authenticate user everytime before issuing any _kubectl_ command.
Update your kubernetes config file(Usually ~/.kube/config) and add/update user section to use _aws-iam-authentication_.
```
users:
- name: kubernetes-admin
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      command: aws-iam-authenticator
      args:
        - "token"
        - "-i"
        - "CLUSTER_ID"
        - "-r"
        - "AWS_IAM_ROLE_ARN"
```
Replace _CLUSTER\_ID_ with clusterID you've provided while creating ConfigMap above, replace _AWS\_IAM\_ROLE\_ARN_ as well.
Now your KubernetesAdmin is ready for action.

### Authorization

In authorization, we'll have to create 2 kinds of objects in Kubernetes to get our job done. First one is called
[__Role__](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole) and second one is 
[__RoleBinding__](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole).

__Role__ is nothing but permission to access resources __kubectl__ provides. __RoleBinding__ is an object which links 
__Role__ and __User__/__Groups__.

For example,
You want to create a __Group__ of users named __quality-tester__  who can only __read__ logs of __pods__.

We'll start with creating role named __QA__
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: staging # You can provide any arbitory namespace.
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"] # You can target any other resources like deployments
  verbs: ["get", "watch", "list"]
```
You can see from above configuration, this role allows to perform any of "get", "watch", "list" actions to "pods".

Next we need to bind this role with a group with __RoleBinding__,
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: staging
subjects:
- kind: Group # We can also use User to bind role to Users directly.
  name: quality-tester
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```
Just like we read role, this _RoleBinding_ binds __pod-reader__ role with __quality-tester__ group.
Check out Kubernetes [Docs](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) for more role variations.
Create both objects by applying above .yamls through kubectl.

We've successfully created __Role__ and __RoleBinding__. Now any _User_ in _quality-tester_ group will be able to _read_
logs.

Now let's assume we've an employee named _jeff\.johns_ who work as a quality tester and he needs to read pod logs,
we'll see how we can add _jeff_ to _quality-tester_ group.

We need to update __ConfigMap__ we created initially along with __DaemonSet__ to include _jeff_,
```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: kube-system
  name: aws-iam-authenticator
  labels:
    k8s-app: aws-iam-authenticator
data:
  config.yaml: |
    clusterID: some-cool-cluster.klastar.com
    server:
      mapUsers:
      - userARN: arn:aws:iam::000000000000:user/jeff.johns
        username: jeff.johns
        groups:
        - quality-tester
```
Once you've apply these change, _jeff.jones_ will be really happy to checkout all the logs by himself.

Make sure you've setup __AWS CLI__, __aws-iam-authenticator__ and kubernet configuration in jeff's
machine(Refer steps we've already done to setup our own mechine).


Thanks for taking time to read this, if you've had any issues while setting up, more than happy to help, suggestions are welcome.

## Resources

- Kubernetes Docs for [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [AWS IAM Authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator)

![meme](https://media.giphy.com/media/fCKGDAmigdB2o/giphy.gif)
