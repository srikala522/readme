
# Integrating Jenkins with Oracle Kubernetes Engine (OKE)(CICD)

## Table of Contents 

[Overview](#overview)

[Pre-Requisites](#pre-requisites)

[Access the OKE Cluster](#access-the-oke-cluster)

[Deploy the Deployment and service manually without CICD](#deploy-the-deployment-and-service-manually-without-cicd)


## Overview

The aim of the lab is to integrating Jenkins with OKE cluster, and configuring github webhook to trigger the Pipeline.

In the Pipeline we have three stages.

1. `Git-clone`--to clone the code from github repo.

2. `Build_n_Push`-- to build the docker image from Dockerfile and push it to the Dockerhub.

3. `Deploy`-- Deploy the latest image to Deployment container.

## Pre-Requisites

* Dockerhub

* Github

## Accessing the OKE cluster

* **Region:** {{Region-name}}

* If your region is `us-ashburn-1` use this console url in browser:
https://console.us-ashburn-1.oraclecloud.com

* If your region is `us-phoenix-1` use this console url in browser: https://console.us-phoenix-1.oraclecloud.com

* **Cloud Tenant:** {{Cloud_Tenant}

1. Sign in to OCI console using **Cloud Tenant:** {{Cloud_Tenant}} and then click `Continue`.

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/oci-console-signin.png?st=2019-11-20T12%3A21%3A50Z&se=2021-11-21T12%3A21%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=uL6%2FogX7rFY3nZpliOw2JMS0PjF%2FTz1Ax7PCyRDLzm0%3D" alt="image-alt-text">

2. Enter the following details of Username and Password to sign in to OCI Console and then click `Sign In`, This will sign in to OCI Console.

 * **Username:** {{User_Name}}

 * **Password:** {{Password}} 
 

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/4.png?st=2019-09-10T09%3A13%3A01Z&se=2022-09-11T09%3A13%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=3IQRuryTJGQqg1cyC4VtTlnMQ7rkfqLg%2BrRcuoPpldQ%3D" alt="image-alt-text">

3. From OCI Services menu, Scroll down choose **Developer Services** --> Container Clusters (OKE).

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/oci-okecluster-portal.PNG?st=2019-09-12T06%3A44%3A07Z&se=2022-09-13T06%3A44%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=zSUTTay7wcM7fTfi0nYGStHzXpMJAfdGDAczbkDXVKQ%3D" alt="image-alt-text">

* **Compartment-name:** {{Compartment}}

* Make sure you should see OKE cluster is deployed, verify it by selecting the compartment from drop down **Compartment:** {{Compartment}} and status should be `Active`

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/5.png?st=2019-09-10T09%3A13%3A36Z&se=2022-09-11T09%3A13%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=X4UyHSZLAqvaHxgUwrI2tgH3l8UJCXWi2EqaLtgRW%2Bs%3D" alt="image-alt-text">

4. Click Apps icon in the toolbar and select **PowerShell** to open a terminal window.

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/powershell-startup.png?st=2019-11-21T14%3A03%3A40Z&se=2021-11-22T14%3A03%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=OSB3e48ObhYaTcMvQAMhGS23NV9eWo2GiQlsWzZpfXQ%3D" alt="image-alt-text">

* Create `.oci` directory using the following commands in `PowerShell`

```bash
mkdir ~/.oci
```
**Note:** If `.oci` directory already exists ignore previous step.

* Change directory to `.oci`

```bash
cd ~/.oci
```
* Enter the command `pwd` to verify the current directory, it must be `/d/PhotonUser/.oci`

```bash
pwd
```

* Clean up the directory by removing any existing files in `.oci` directory

```bash
rm ./*
```

5. OCI-cli is already installed in the lab console, To verify OCI-cli installation execute the following command.

```bash
oci -v
```

**Output:**

```
2.6.3
```

6. Next create the user-config for the OCI Cloud account to connect to the OCI console

```bash
oci setup config
```
* Follow the prompts has below

7. Accept the default location (Press `Enter`) for your config

* Enter the following OCI setup configuration details

* **User OCID:** {{User_Ocid}} 

* **Tenancy_OCID:** {{Tenancy_Ocid}}

* **Region name:** {{Region-name}}

* When asked for **Do you want to generate a new RSA key pair?** Press enter **Y**.
* Press `Enter` and accept default options for directories prompts 
* Enter a directory for your keys to be created [D:\PhotonUser\.oci]:
* Enter a name for your key [oci_api_key]:
* Press Enter when prompted for passphrase (i.e leave it empty)

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/oci-setup-ps.PNG?st=2019-11-21T14%3A11%3A42Z&se=2021-11-22T14%3A11%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=%2F5peXjGeMidMPo6QeCMDegW0cjodWn9CPe4xlztpup8%3D" alt="image-alt-text">

**Output:**
```
    This command provides a walkthrough of creating a valid CLI config file.

    The following links explain where to find the information required by this
    script:

    User OCID and Tenancy OCID:

        https://docs.cloud.oracle.com/Content/API/Concepts/apisigningkey.htm#Other

    Region:

        https://docs.cloud.oracle.com/Content/General/Concepts/regions.htm

    General config documentation:

        https://docs.cloud.oracle.com/Content/API/Concepts/sdkconfig.htm


Enter a location for your config [D:\PhotonUser\.oci\config]:
Enter a user OCID: ocid1.user.oc1..aaaaaaaaeysiy7qborb6fil34q5p6gcxdhewueqtne57g5neuulviw4yvdfq
Enter a tenancy OCID: ocid1.tenancy.oc1..aaaaaaaahfuh3fydnu6rcy7vrsdubkneu3fsok35wqzvlbk3nn2e6kqqydsa
Enter a region (e.g. ap-mumbai-1, ap-seoul-1, ap-sydney-1, ap-tokyo-1, ca-toronto-1, eu-frankfurt-1, eu-zurich-1, sa-sao
paulo-1, uk-london-1, us-ashburn-1, us-gov-ashburn-1, us-gov-chicago-1, us-gov-phoenix-1, us-langley-1, us-luke-1, us-ph
oenix-1): us-ashburn-1
Do you want to generate a new RSA key pair? (If you decline you will be asked to supply the path to an existing key.) [Y
/n]: y
Enter a directory for your keys to be created [D:\PhotonUser\.oci]:
Enter a name for your key [oci_api_key]:
Public key written to: D:\PhotonUser\.oci\oci_api_key_public.pem
Enter a passphrase for your private key (empty for no passphrase):
Private key written to: D:\PhotonUser\.oci\oci_api_key.pem
Fingerprint: 47:cc:c5:6a:41:b8:dd:4d:fe:de:ab:96:7e:49:71:fb
Config written to D:\PhotonUser\.oci\config


    If you haven't already uploaded your public key through the console,
    follow the instructions on the page linked below in the section 'How to
    upload the public key':

        https://docs.cloud.oracle.com/Content/API/Concepts/apisigningkey.htm#How2
```

8. Now we need to upload API key into our OCI account for authentication of API calls. To display the content of API key.

```bash
cat ~/.oci/oci_api_key_public.pem
```

9. Select and copy the public key from as shown below

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/cat-oci-pub.PNG?st=2019-11-21T14%3A15%3A42Z&se=2021-11-22T14%3A15%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=lBlUbEFbL%2BVKtrUuQs5kexbezs06KoAqte%2B%2FkixexwQ%3D" alt="image-alt-text">

**Note:** If you want to navigate multiple windows click on icon as shown below.

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/switch-multiple%20windows.png?st=2019-11-21T14%3A28%3A26Z&se=2021-11-22T14%3A28%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=azZ%2B22Ndu0TWCd5wZMXgbzPKHNBKy2tYucfauj6IgJo%3D" alt="image-alt-text">

* Switch to OCI Console, Click **Profile icon** & select **User settings**

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/usersetting.PNG?st=2019-11-22T10%3A25%3A37Z&se=2022-11-23T10%3A25%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=G%2BWn4zTYAevmcAY9%2F1K%2BZ1fa5RdfGKa5PiirO4emrC8%3D" alt="image-alt-text">

* Click on `Add Public Key`

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/add-pubkey.png?st=2019-11-21T08%3A52%3A30Z&se=2021-11-22T08%3A52%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=%2Fn8GJ69WEZ6QyyZDkkTVuVMSu%2FDmOGXSyyExXrx4hmY%3D" alt="image-alt-text">

* Make sure copy and paste the correct public key, click **Add**

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/14.png?st=2019-09-10T09%3A21%3A25Z&se=2022-09-11T09%3A21%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=D7b5Pub7OakFGjSlpAMhO8HFLtiTuc6N0jXRA9nd%2FAI%3D" alt="image-alt-text">

* If you do not paste Public key properly, you will get the following error.

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/error-pem.png?st=2019-11-21T09%3A17%3A13Z&se=2021-11-22T09%3A17%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=eIsWUC8a5YBNzqiUzYKliF9qfhZLgyi57Py%2FqawOOfw%3D" alt="image-alt-text">

* Verify `kubectl version` command to check the version of kubectl client and server version.

**Note:** Server version should not be displayed, because you are not connected to the cluster yet.

```bash
kubectl version
```

**Output:**
```
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:40:16Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"windows/amd64"}
Unable to connect to the server: dial tcp: lookup cztgmbxgnrd.us-ashburn-1.clusters.oci.oraclecloud.com: no such host
```
### Download get-kubeconfig file and Initialize your environment

* Switch to OCI console window and navigate to your cluster that is deployed in the **Compartment:** {{Compartment}}.

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/oci-okecluster-portal.PNG?st=2019-09-12T06%3A44%3A07Z&se=2022-09-13T06%3A44%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=zSUTTay7wcM7fTfi0nYGStHzXpMJAfdGDAczbkDXVKQ%3D" alt="image-alt-text">

* Select `Access Kubeconfig` as shown below

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/18.png?st=2019-09-10T09%3A25%3A06Z&se=2022-09-11T09%3A25%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=5nA6QKiBiNqoP3j74oMmCjzZBxTOSA0qeAsFHlMo5Us%3D" alt="image-alt-text">

* Copy the second command to access the kubeconfig file & ignore `--token-version 2.0`

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/kubeconfig.png?st=2019-11-21T09%3A08%3A29Z&se=2021-11-22T09%3A08%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=costTAipWd%2Fipa9xS%2BvYcHufVjqTwCYSIDUuBofoYz4%3D" alt="image-alt-text">

* Navigate to PowerShell window, paste the command that was copied

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/save-kubeconfig.PNG?st=2019-11-21T14%3A23%3A36Z&se=2021-11-22T14%3A23%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=PN80BcYRVN%2BjPT6XlLfMxZLoHx2JBpy6bKtLboye%2Bwc%3D" alt="image-alt-text">

* Let's verify `kubectl version` of the client and server.

```bash
kubectl version
```

* Now you should be able to see the both client and server versions of kubectl

**Output:**
```
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.3", GitCommit:"5e53fd6bc17c0dec8434817e69b04a25d8ae0ff0", GitTreeState:"clean", BuildDate:"2019-06-06T01:44:30Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"windows/amd64"}
Server Version: version.Info{Major:"1", Minor:"13+", GitVersion:"v1.13.5-6+2d6d607cc2dd26", GitCommit:"2d6d607cc2dd26104fd338e0e0646ca3e68333b5", GitTreeState:"clean", BuildDate:"2019-09-24T23:30:21Z", GoVersion:"go1.11.13", Compiler:"gc", Platform:"linux/amd64"}
```

* We have initialized the enviornment, Get the all nodes using `kubectl get nodes` command to list-out nodes in a k8's cluster.

```bash
kubectl get nodes
```

**Output:**
```
NAME       STATUS    ROLES     AGE       VERSION
10.0.0.2   Ready     node      1d        v1.12.7
10.0.1.2   Ready     node      1d        v1.12.7
10.0.2.2   Ready     node      1d        v1.12.7
```
* Execute the following command in gitbash, to get the default namespaces in a Kubernetes cluster.

```bash
kubectl get ns
```

**Output:**
```
NAME          STATUS    AGE
default       Active    8m
kube-public   Active    8m
kube-system   Active    8m
```

## Deploy the Deployment and service manually without CICD

* Deploy the following yaml code, save it in ubuntu-pod.yaml file, and deploy it manually using the followig command.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ubuntu-pod
spec:
  replicas: 1
  selector:
    matchLabels:
      run: my-ubuntu
  template:
    metadata:
      labels:
        run: my-ubuntu
    spec:
      containers:
      - name: ubuntu-pod
        image: sysgaininc/jenkinskube:v1
        ports:
        - containerPort: 80
```

`kubectl apply -f deployment.yaml`

* Deploy the service which can access the pod 

```yaml
kind: Service
apiVersion: v1
metadata:
  name: jenkins-ui
spec:
  type: NodePort
  selector:
    run: my-ubuntu
  ports:
    - protocol: TCP
      port: 4000
      targetPort: 80
```

`kubectl apply -f service.yaml`

* Deploy the service using the following command

`kubectl get pods`
```
NAME                READY   STATUS             RESTARTS   AGE
ubuntu-pod          1/1     Running            0          31s
```
* List-out the service

`kubectl get svc`

```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
jenkins-ui   NodePort    10.96.122.242   <none>        4000:30374/TCP   7h5m
```
### Login to Jenkins UI

1. Open a new tab in chrome browser and go to {{Jenkins url}}.

* Jenkins url: {{Jenkins url}}
* Jenkins username: {{Jenkins username}}
* Jenkins password: {{Jenkins password}}

2. The setup wizard will ask you whether you want to install `suggested plugins` or you want to install `specific plugins`.

<img src="https://qloudableassets.blob.core.windows.net/devops/Jenkins/Generic/Intermediate/img/customize-jenkins.png?st=2019-09-06T10%3A31%3A31Z&se=2022-09-07T10%3A31%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=fwljWymO6LKz5xubtKh3mAsK3r858hNP%2Bl6%2FtadP4MM%3D" alt="image-alt-text" >

3. Click on `suggested plugins box`, and the plugins installation process will start.
 

4. Once the plugins are installed, you will be prompted to set the URL for your Jenkins instance. The URL will be generated automatically. Confirm the URL by clicking `Save and Finish` button.  

<img src="https://qloudableassets.blob.core.windows.net/devops/Jenkins/Generic/Intermediate/img/instance-configuration.png?st=2019-09-06T10%3A31%3A31Z&se=2022-09-07T10%3A31%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=fwljWymO6LKz5xubtKh3mAsK3r858hNP%2Bl6%2FtadP4MM%3D" alt="image-alt-text" >

5. Once all the configurations are done, you can see the `Jenkins is ready` screen.

<img src="https://qloudableassets.blob.core.windows.net/devops/Jenkins/Generic/Intermediate/img/jenkins-ready.png?st=2019-09-06T10%3A31%3A31Z&se=2022-09-07T10%3A31%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=fwljWymO6LKz5xubtKh3mAsK3r858hNP%2Bl6%2FtadP4MM%3D" alt="image-alt-text" >

6. Click `Start using Jenkins`, it will redirect to Jenkins dashboard.

<img src="https://qloudableassets.blob.core.windows.net/devops/Jenkins/Generic/Intermediate/img/dashboard.png?st=2019-09-06T10%3A31%3A31Z&se=2022-09-07T10%3A31%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=fwljWymO6LKz5xubtKh3mAsK3r858hNP%2Bl6%2FtadP4MM%3D" alt="image-alt-text" >

### Fork a code GitHub repository

1. On GitHub, navigate to the [MSidda/Hello-world-1](https://github.com/MSidda/Hello-world-1.git) repository.

2. In the top-right corner of the page, click `Fork`.

3. That's it! Now, you have a fork of the original `MSidda/Hello-world-1` repository

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/jenkins-fork.png?st=2019-11-18T17%3A19%3A10Z&se=2021-11-19T17%3A19%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=QKBNNdswce5H4JCNoMGTHctzOPx8M9vgEDkIowy%2BgJ4%3D" alt="image-alt-txt" >

#### Create Pipeline 

1. Click `New Item` on the dashboard.  

<img src="https://qloudableassets.blob.core.windows.net/devops/Jenkins/Generic/Advanced/img/new_item.png?st=2019-09-06T10%3A31%3A31Z&se=2022-09-07T10%3A31%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=fwljWymO6LKz5xubtKh3mAsK3r858hNP%2Bl6%2FtadP4MM%3D" alt="image-alt-text" >

2. Enter the name of your project in the `Enter an item name` field, select `Pipeline` project and click `OK` button.

3. Enter `Description` (optional).  

4.  Go to `Pipeline` section, make sure the `Definition` field has `Pipeline script` option selected. 

5. Copy and paste the following declarative pipeline script into a `script` field and update your `GitHub URL` and specify the branch name as well.
6. Mention your docker-hub login credentials as shown below
  
Click on `Credentials`, click on `Jenkins`--> `Global credentials (unrestricted)`


<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/jenkins-cred.png?st=2019-11-18T16%3A18%3A04Z&se=2021-11-19T16%3A18%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=xDkalXm6YLElT4k5wDJ8RYsZUMgd7B%2B2GaAWGvwh43A%3D" alt="image-alt-text" >

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/jenkins-add-cred.png?st=2019-11-18T16%3A24%3A25Z&se=2021-11-19T16%3A24%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=43JvZ2rGBpUMcIdQKyzkxPvXqZGq2B8C2SIJsIjj1Po%3D" alt="image-alt-text" >

* Mention Username and Password of Dockerhub credentials and mention id and description value as `docker-login`, it will be used in further pipeline

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/jenk-2.png?st=2019-11-18T16%3A30%3A07Z&se=2021-11-19T16%3A30%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=rU6eaauIHWReI6EVWs%2FryO9Me9IRTDnEmtAbI4bMasE%3D" alt="image-alt-text" >

* Click on the pipeline `jenkins`that was created in previous section.

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/jenkins-pipe.png?st=2019-11-18T16%3A38%3A31Z&se=2021-11-19T16%3A38%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=ZI%2BfShLtCmwXNOmBhGGcuL5egB7jsnVMeZE%2B2MZ%2Fag8%3D" alt="image-alt-text" >

* Select `Configure`--> Advanced Project Options-> Pipeline-> Definition-> pipeline script

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/jenkins-config.png?st=2019-11-18T16%3A49%3A02Z&se=2021-11-19T16%3A49%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=SVEkKRpaKCf6mWkpE5vdGgNe4khXNvK4IJ8ZBoSDEDM%3D" alt="image-alt-text" >

* Copy the following script and paste it on script section, mention `docker-login` creds as mentioned in credentials section as ID
* Mention git url and brach-name as yours.

* Stage build and push section mention dockerhub username  and repo name

* Stage deploy mention deployment name and containername with dockerhub username and repo-name, this will update a image and deployed to container, new pod will be created.


```
node('master'){
    docker.withRegistry('','docker-login') {
        stage('Git-Clone') { 
            git (url: 'https://github.com/MSidda/Hello-world-1.git', branch: 'master')
            sh "git status"
        }
        stage('build_n_push') { 
            sh "date +%F > date"
            def date = readFile('date').trim()
            println date
            def img = docker.build('sysgaininc/jenkinskube',".").push "oke-${date}.$BUILD_NUMBER"  
}
        stage('deploy') {
            sh '''
            today=`date +%F`
            echo $today
            kubectl get deployments
                kubectl set image deployment/"ubuntu-pod" "ubuntu-pod"=sysgaininc/jenkinskube:"oke"-"$today"."$BUILD_NUMBER"
            '''
        }
    }
}
```
* Click `Apply` and `Save`

* Click on Build now to queue the build manually

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/jenkins-manulbuild.png?st=2019-11-19T04%3A54%3A03Z&se=2021-11-20T04%3A54%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=TYKz0hTL8KSHd1zSHnx28TCUK9SV06h%2F0VpQGauoQl8%3D" alt="image-alt-txt" >

* After pipeline execution is completed, we can verify the history of executed build under the Build History by clicking the build number e.g. #24

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/build-history.png?st=2019-11-19T04%3A57%3A50Z&se=2021-11-20T04%3A57%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=RE1RvIqKuVAL47iM0d7%2BuE52a3DXR8Gr2etJk5sUfNs%3D" alt="image-alt-txt" >

### Configure Webhooks 

1. We have to configure our Jenkins machine to communicate with our GitHub repository. For that we need to get the Hook URL of Jenkins machine.

2. Go to `Manage Jenkins` and select `Configure System` view.  

3. Find the `GitHub` Plugin Configuration section and click on `Advanced` button.  

<img src="https://qloudableassets.blob.core.windows.net/devops/Jenkins/Generic/Advanced/img/n-ci-advanced.png?st=2019-09-06T10%3A31%3A31Z&se=2022-09-07T10%3A31%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=fwljWymO6LKz5xubtKh3mAsK3r858hNP%2Bl6%2FtadP4MM%3D" alt="image-alt-text" >


4. Select the `Specify another hook URL for GitHub configuration`  

<img src="https://qloudableassets.blob.core.windows.net/devops/Jenkins/Generic/Advanced/img/n-ci-sepcify-hook.png?st=2019-09-06T10%3A31%3A31Z&se=2022-09-07T10%3A31%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=fwljWymO6LKz5xubtKh3mAsK3r858hNP%2Bl6%2FtadP4MM%3D" alt="image-alt-text" >

5. Copy URL in the text box field and unselect it. 

6. Click `Save` it will redirect to the Jenkins dashboard.

7. Open your `GitHub` repository.  

8. Click on `Settings`. It will navigate to the repository settings.  

9. Click on `Webhooks` section.   

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/jenkins-webhook.png?st=2019-11-18T17%3A34%3A30Z&se=2021-11-19T17%3A34%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=78gNNcCv6H1noIRqOslXMZUMpjgbDtvrfYzInuebcfY%3D" alt="image-alt-text" >

10. Click on `Add Webhook` button   
 
<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/add-webhook.png?st=2019-11-18T17%3A37%3A22Z&se=2021-11-19T17%3A37%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=jHbM%2FVH5Hmmu1ytmfGvQkQqDTeAOvq46%2BO4vhz%2BvFec%3D" alt="image-alt-text" >

11. Paste the `Hook URL` on the `Payload URL` field.  

12. Make sure the `trigger webhook` field has `Just the push event` option selected. 

13. Click `Add webhook` it will add the webhook to your repository.  

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/webhook-2.png?st=2019-11-18T17%3A40%3A00Z&se=2021-11-19T17%3A40%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=w8XZ8%2B8BQeDa7DN50BVHLMu0pLUoq75b1JF7sz5uQoE%3D" alt="image-alt-text" >

14. Once you added webhook correctly. You can see the webhook with green tick.  

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/webhook-green.PNG?st=2019-11-18T17%3A40%3A23Z&se=2021-11-19T17%3A40%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=SWdTTrQorrU8Y3PTleBYYEIVt1pBMIoAfGm6UNlrEfE%3D" alt="image-alt-text" >

15. Next Jenkins configuration to run builds automatically when code is pushed to GitHub repository. Jenkins doesn’t run all builds for all projects. To specify which project builds, need to run, we have to modify the project configuration.

16. Select the project and Click on `configure`.  

<img src="https://qloudableassets.blob.core.windows.net/devops/Jenkins/Generic/Advanced/img/n-ci-configure.png?st=2019-09-06T10%3A31%3A31Z&se=2022-09-07T10%3A31%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=fwljWymO6LKz5xubtKh3mAsK3r858hNP%2Bl6%2FtadP4MM%3D" alt="image-alt-text" >


17. Go to `Build Triggers` section and select `GitHub hook trigger for GIT SCM polling`.  

<img src="https://qloudableassets.blob.core.windows.net/devops/Jenkins/Generic/Advanced/img/n-ci-jobconfiguration.png?st=2019-09-06T10%3A31%3A31Z&se=2022-09-07T10%3A31%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=fwljWymO6LKz5xubtKh3mAsK3r858hNP%2Bl6%2FtadP4MM%3D" alt="image-alt-text" >

18.  Click on `Save`, it will redirect to pipeline view page.
  
19. Go to GitHub and Update any of the file. In this lab we update `server.js` file. this will trigger a new build, verify using a build history.

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/github-trigger-jenkins.png?st=2019-11-18T17%3A49%3A09Z&se=2021-11-19T17%3A49%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=sFLheSKgHZmC2h8n7gEpvUmlfWD1AEWP6BBHYCd28gc%3D" alt="image-alt-text" >

* List-out the pods using the following command in Powershell, you can observe whenever changes to a code each time docker image is built and push to dockerhub and deploy it to container, new pod will create and old pod will be terminating state

 ```
 kubectl get pods
 ```
```
NAME                          READY   STATUS        RESTARTS   AGE
ubuntu-pod-55c994cb5d-rntcj   0/1     Terminating   0          25m
ubuntu-pod-64cfdbc6b9-hv7qq   1/1     Running       0          36s
```
* Access the application using the `port-forward` 

 `kubectl port-forward ubuntu-pod-55c994cb5d-rntcj 4000`

**Output:** 
``` 
Forwarding from 127.0.0.1:4000 -> 4000
Forwarding from [::1]:4000 -> 4000
```

* Hit `localhost:4000` in browser you will see following output.

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20oke/usermanual%20images/output-jenkins.PNG?st=2019-11-18T18%3A15%3A15Z&se=2021-11-19T18%3A15%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=pvJCuj3xkVSW5zmcKqjY8T522vunvQE49v%2BB%2Fvtyb3Q%3D" alt="image-alt-txt" >

**Conclusion:** Congratulations! You have successfully completed the Integrating Jenkins with Oracle Kubernetes Engine (OKE)(CICD) lab. In this lab explored Helm charts using Helm commands!

Feel free to continue exploring or start a new lab.

<img src="https://qloudableassets.blob.core.windows.net/aks/images%20for%20aks/sp%20images%20gif/congrats-gif.gif?st=2019-08-26T06%3A22%3A30Z&se=2022-08-27T06%3A22%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=7h%2B1GwYtoaOpGYaKxe%2FrQN3dbKfhoKw%2FbPWiqstjlIc%3D" alt="image-alt-text">

Thank you for taking this training lab

