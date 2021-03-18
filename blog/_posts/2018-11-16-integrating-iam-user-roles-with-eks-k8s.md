---
layout: post
title: Integrating IAM user/roles with EKS
date: 2018-11-16 09:50:20.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: []
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '24320581828'
  timeline_notification: '1542361821'
  _oembed_619e22896b89cb8324534bf2659b65ec: "{{unknown}}"
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2018/11/16/integrating-iam-user-roles-with-eks-k8s/"
---
To be completely honest, this article spawns out of some troubleshooting frustration. So hopefully this will save others some headaches.
The scenario: after having configured an EKS cluster, I wanted to provide permissions for more IAM users. After creating a new IAM user with belonged to the target intended IAM groups, the following exceptions were thrown in the CLI:
kubectl get svc
error: the server doesn't have a resource type "svc"
kubectl get nodes
error: You must be logged in to the server (Unauthorized)
 
<h3>AWS profile config</h3>
First configure your local AWS profile. This is also useful if you want to test for different users and roles.
 

```bash
# aws configure --profile
# for example:
aws configure --profile dev
```
{:.lead}

If this is your firts time, this will generate two files,
<code class="highlighter-rouge">~/.aws/config and <code class="highlighter-rouge">~/.aws/credentials
It will simply append to them, which means that you can obviously edit the files manually as well if you prefer. The way you can alternate between these profiles in the CLI is:
```
#export AWS_PROFILE=
# for example:
export AWS_PROFILE=dev
```
Now before you move on to the next section, validate that you are referencing the correct user or role in your local aws configuration:
```
# aws --profile  sts get-caller-identity
# for example:
aws --profile dev sts get-caller-identity
{
"Account": "REDACTED",
"UserId": "REDACTED",
"Arn": "arn:aws:iam:::user/john.doe"
}
```
 
<h3>Validate AWS permissions</h3>
Validate that your user has the correct permissions, namely the following two are required:
```
# aws eks describe-cluster --name=
# for example:
aws eks describe-cluster --name=eks-dev
```
<h3>Add IAM users/roles to cluster config</h3>
If you managed to add worker nodes to your EKS cluster, then <a href="https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html" target="_blank" rel="noopener">this documentation should be familiar already</a>. You probably AWS documentation describes
```
kubectl apply -f aws-auth-cm.yaml
```
While troubleshooting I saw some people trying to use the clusters role in the "-r" part. However you can <span style="text-decoration:underline;"><em>not</em></span> assume a role used by the cluster, as this is a role reserved/trusted for instances. You need to create your own role, and add root account as trusted entity, and add permission for the user/group to assume it, for example as follows:
```
{
  "Version": "2012-10-17",
  "Statement": [
     {
        "Effect": "Allow",
        "Principal": {
             "Service": "eks.amazonaws.com",
             "AWS": "arn:aws:iam:::user/john.doe"
         },
        "Action": "sts:AssumeRole"
    }
   ]
}
```
 
<h3>Kubernetes local config</h3>
then, generate a new kube configuration file. Note that the following command will create a new file in ~/.kube/config
```
aws --profile=dev eks update-kubeconfig --name esk-dev
```
AWS suggests isolating your configuration in a file with name "config-". So, assuming our cluster name is "dev", then:
```
export KUBECONFIG=~/.kube/config-eks-dev
aws --profile=dev eks update-kubeconfig --name esk-dev
```
 
This will then create a the config file in ~/.kube/config-eks-dev rather than ~/.kube/config
As <a href="https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html" target="_blank" rel="noopener">described in AWS documentation</a>, your kube configuration should be something similar to the following:
https://gist.github.com/diogoaurelio/a2349b3aaecda96bf4bc255c49fe4605
If you want to make sure you are using the correct configuration:
```
export KUBECONFIG=~/.kube/config-eks-dev
kubectl config current-context
```
This will print whatever the alias you gave in the config file.
Last but not least, update the new config file and add the profile used.
Last step is to confirm you have permissions:
```
export KUBECONFIG=~/.kube/config-eks-dev
kubectl auth can -i get pods
# Ideally you get "yes" as the answer.
kubectl get svc
```
 
<h3>Troubleshooting</h3>
To make sure you are not working in a environment with hidden environmental variables that you are not aware and may conflict, make sure you unset them as follows:
unset AWS_ACCESS_KEY_ID &amp;&amp; unset AWS_SECRET_ACCESS_KEY &amp;&amp; unset AWS_SESSION_TOKEN
Also if you are getting as follows:
<div class="js-timeline-item js-timeline-progressive-focus-container">
<div class="timeline-comment-wrapper js-comment-container">
<div id="issuecomment-402343140" class="timeline-comment-group js-minimizable-comment-group js-targetable-comment">
<div class="unminimized-comment comment previewable-edit js-comment js-task-list-container timeline-comment reorderable-task-lists">
<div class="edit-comment-hide">
<table class="d-block">
<tbody class="d-block">
<tr class="d-block">
<td class="d-block comment-body markdown-body  js-comment-body">
could not get token: AccessDenied: User arn:aws:iam:::user/john.doe is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam:::user/john.doe

</td>
</tr>
</tbody>
</table>
<div class="comment-reactions  js-reactions-container js-socket-channel js-updatable-content"></div>
</div>
</div>
</div>
</div>
</div>
<div class="js-timeline-item js-timeline-progressive-focus-container">
<div class="timeline-comment-wrapper js-comment-container"></div>
</div>
Then it means you are specifying the -r flag in your kube/config file. This should be only used for roles.
Hopefully this short article was enough to unblock you, but in case not, here is a collection of further potential useful articles:
<ul>
<li><a href="https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html" target="_blank" rel="noopener">AWS documentation for setting up locally kubectl </a></li>
<li><a href="http://marcinkaszynski.com/2018/07/12/eks-auth.html" target="_blank" rel="noopener">How Auth works in EKS with IAM</a></li>
<li><a href="https://medium.com/pablo-perez/common-errors-when-setting-up-eks-for-the-first-time-a43cbf989a2e" target="_blank" rel="noopener">Common errors when setting up EKS for the first time</a></li>
<li><a href="https://github.com/kubernetes-sigs/aws-iam-authenticator/issues/105" target="_blank" rel="noopener">Troubleshooting credentials integration</a></li>
<li><a href="https://aws.amazon.com/blogs/opensource/integrating-ldap-ad-users-kubernetes-rbac-aws-iam-authenticator-project/" target="_blank" rel="noopener">LDAP integration with K8s and AWS EKS</a></li>
<li><a href="https://kubernetes.io/docs/reference/access-authn-authz/rbac/" target="_blank" rel="noopener">Kubernetes RBAC documentation</a></li>
</ul>
