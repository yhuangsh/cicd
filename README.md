# Setup Jenkins based CI/CD Pipeline

## Install Jenkins

### Create persistent volumne and persistent volume claim for Jenkins data store

1. Use `kubectl -f create yaml/local-storage-class.yaml` to create a local storage class with `volumeBindingMode: WaitForFirstConsumer`. This step is optional. The `WaitForFirstConsumer` mode instructs Kubernetest not to bind to a persistent volume unitl there is a consumer, i.e. a pod that has claim on this volume. If this step is skipped, the creation of the persisten volume claim in the 3rd step will immeidatly bind to the persistent volume.
2. Use `kubectl -f create yaml/local-storage-w0-pv.yaml` to create a persistent volume with 15Gi capacity of local-storage class on worker node w0. This is using the space from the system volume provisioned to the w0 ECS instance. 
3. Use `kubectl -f create yaml/jenkins-pvc.yaml` to create a persistent volume claim of 10Gi that will be mounted as `/var/jenkins_home` in the pod hosting Jenkins. 

### Fetch Jenkins `helm` charts and customize defaults

1. Use `helm fetch stable/jenkins` wiht VPN to download the Jenkins charts to local.
2. Use `helm inspect values ./jenkins-0.25.0.tgz > jenkins-values.yaml` to dump all default parameters `helm` uses to deploy Jenkins or any charts. 
3. Open `jenkins-values.yaml` and We want to customize a few things for our Jenkins installation. 
   - Jenkins prefix. We want to use dev.davidhuang.top/jenkins as the web root instead of dev.davidhuang.top/.
   - Use ClusterIP as service type rather than the default LoadBalancer type since we will use `nginx-ingress` to terminate TLS for the `dev.davidhuang.top` domain and front all `http` services.
   - Use `jenkins-pvc` to make the `jenkins_home` data directory persistent
   - Admin user name and password. You may want to change these to your own.
4. Use `helm install --name=jenkins -f jenkins-values.yaml --set Master.AdminUser=<your user name> --set Master.AdminPassword=<your admin password> ./jenkins-0.25.0.tgz` to deploy Jenkins to your Kubernetes cluster.

Note that by default Jenkins deploy will include the master Jenkins node listening on port 8080 and a Jenkins agent listening on 5000 for Jenkins slaves.

The Jenkins charts includes an `initContainer` script which downloads several Jenkins plug-ins and write the right initial contents into the `/var/jenkins_home` directory. This way you skip the initial admin passcode screen and the plug-in download screen. Because of this, your Jenkins deployment and pods will take a few minutes to settle down and enter ready state. 

### Add Jenkins to ingress for HTTPS access

By now your Jenkins master is up running but is confined within the cluster if you used ClusterIP as the service type. This was intentional. With ingress and its support for TLS, you should always consider deploying a http-based service as ClusterIP service, then let ingress handle host/path-based service routing and TLS termination. 

Do a diff between `ingress-tls-jenkins.yaml` and `ingress-tls.yaml`. It's really easy. Also note that we use `/jenkins` as the path under `dev.davidhuang.top` so that the Jenkins URL from external perspective becomes `https//dev.davidhuang.top/jenkins`. For me, this saves my time because I don't have to add another DNS name and its certificate for Jenkins or for any other service I will add later. Use the method that suits you best. 

### Solve the "It appears that your reverse proxy set up is broken"

Since the Jenkins service is already accessible from external with our ingress. The only you need to do to eliminate this warning is to go to Jenkins's dashboard, Manager Jenkins, Configure System, Jenkins Location, Jenkins URL. There you fill in the root URL you use from external. Refresh the page and the warning will go away.


## dev container: 

- same image standard for all developers: 
- each developer maps his own git work copy to a standard container path /project and run as root there
- developer test their code locally before commit/push. 
- Jenkins also uses this container to pull code, build, test, make release, lastly create deployment image. Everything runs within this container
- Dockerfile used to create this dev container is in the cicd project.
- Building of this image is also automated by the same cicd process 

# release container:
docker build -t 