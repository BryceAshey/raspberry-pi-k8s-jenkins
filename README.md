# raspberry-pi-k8s-jenkins
A containerized Jenkins instance running on k8s on raspberry pi (ARM). This implementation uses the Eval version of jenkins since that is the [only version that currently has ARM support](https://github.com/BryceAshey/raspberry-pi-kubernetes-cluster/blob/master/docs/gluster-setup.md#gluster-volume-setup).

### Prep Work
This implementation assumes that you have already setup [glusterfs](https://github.com/BryceAshey/raspberry-pi-kubernetes-cluster/blob/master/docs/gluster-setup.md#gluster-volume-setup) in Kubernetes.

### Persistent Volume Setup
Create the persistent volume that will host all files associated with Jenkins.
``` 
> kubectl create -f jenkins-gluster-pv.yaml
```
Create the claim
```
> kubectl create -f jenkins-gluster-claim.yaml
```

### Deploy Jenkins to Kubernetes
This will deploy jenkins to kubernetes while exposing Jenkins on port 8080 within the container. Please note that the actually port will be very different.
```
> kubectl create -f jenkins-deployment.yaml
```
Finally, expose Jenkins outside the cluster with a service:
```
> kubectl create -f jenkins-service.yaml
```

### Determine the Exposed Port
We need to determine what port Kubernetes exposed our services on. In the example below we can see that TCP 30597 is mapped to the Jenkins' 8080 port. So we should be able to hit Jenkins on port 30597 using any node IP (or the Kubernetes IP itself).
```
> kubectl get services

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
jenkins      NodePort    10.99.213.181   <none>        8080:30597/TCP   13d
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          13d

```

### Getting the Jenkins Wizard Secret
Assuming you setup the volume as outlined above the easiest way to get the Unlock key for Jenkins is to navigate to the Persistent Volume and look in the "secrets" directory located therein. In the example above this would be /data/gluster01/jenkins/secrets...

## Additional Reading
[Scaling Jenkins](https://rancher.com/blog/2018/2018-11-27-scaling-jenkins/)
[JNLP Docker Image](https://hub.docker.com/r/jenkinsci/jnlp-slave)
[Kubernetes Plugin](https://plugins.jenkins.io/kubernetes)
[Kubernetes Plugin Github](https://github.com/jenkinsci/kubernetes-plugin)
[Configuring Certificates for Jenkins Kubernetes Plugin](https://illya-chekrygin.com/2017/08/26/configuring-certificates-for-jenkins-kubernetes-plugin-0-12/)