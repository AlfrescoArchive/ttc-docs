# Workshop Recommended Steps
You need to have Jenkins X installed in order to accelerate the deployment of your services into a Kubernetes Cluster. You can definitely achieve the same results without Jenkins X, but the HELM Charts and Kubernetes Descriptors for the Trending Topic Campaign Project were created using Jenkins X. 
Jenkins X
For that you run the following command when you have a CGE account with the Kubernetes Engine Enabled:

> jx create cluster gke --kubernetes-version=1.9


Make sure that you sign in with the google account where you have your kube cluster.

Select your availability zone, Then accept all the defaults. 

Make sure that you record the password for Jenkins X.


Execute

> jx get urls

Access to Jenkins X URL and login

Access to nexus URL and login with the same credentials

Fork TTC-* Projects
https://github.com/activiti/?q=ttc-
 
 
The main idea to fork the projects is to be able to change them by sending PRs or just pushing to these repositories that will be monitored by Jenkins X.
If you have your own cluster:
> jx create cluster gke --kubernetes-version=1.9
> europe-west1c
> n1-standard-2


 
Let’s start with ttc-connectors-dummytwitter
Fork it to your github account
Create a new ttc dir in your laptop
cd ttc/
Clone repo:  
> git clone <URL>
Then import it to Jenkins X: 
> jx import --branches "develop|PR-.*|feature.*"
Go to Jenkins X and check that the new project is registered and being built

The build should fail at this point, and this is because we need to configure Nexus to pick up other Maven repositories to download some dependencies. 
Configuring nexus to have custom repositories
Login with admin/<jenkinsx pass>
Click in the Server Administration and Configuration menu (Gear)
Click on Maven2 (Proxy)
Enter the following information:
Name: activiti
Version Policy: Mixed
URL: https://artifacts.alfresco.com/nexus/content/repositories/activiti-snapshots/
Scroll Down and Create Repository
With the repository created you need to add it to the Maven Group (From the list of repositories
Click on Maven Group
Select the activiti repository from the Available list and move it to the Members list
Scroll down and save

Go back to Jenkins -> <Your User> -> ttc-connectors-dummytwitter -> develop
Hit Build Now

---- While you wait for the pipeline to finish (it takes around 7 minutes ) you can fork and clone the other projects --- 

From the terminal you can execute:
> jx get activities -w -> to watch what Jenkins X is doing
> jx env -> select staging
> kubectl get pods -> you should see your pod and the number of replicas
NAME                                                      READY     STATUS    RESTARTS   AGE
jx-staging-ttc-connectors-dummytwitter-<hash>   0/1       Running   3          8m
> jx logs -> will tail the logs of our only service and it will reveal that the service is looking for RabbitMQ but rabbitMQ is not available

Adding Dependencies to an Environment (RabbitMQ)

From the terminal go to :
cd ~/.jx/environments/<github user>/environment-<env name>-staging
git pull -> to retrieve the latest changes from the staging environment
Edit env/requirements.yaml and add at the end
- name: rabbitmq
  repository: https://kubernetes-charts.storage.googleapis.com
  version: 0.8.0
Edit env/values.yaml and add at the end
rabbitmq:
  rabbitmq:
    username: "guest"
    password: "guest"
  ingress:
    enabled: true
Be careful with the spaces (it is a yaml file) and caps
execute : git status -> you should see two files changed
Do: > git add .
Then: > git commit -m “adding rabbitmq to staging env”
Finally: > git push
You can check now inside the jenkins UI that the staging environment pipeline has been triggered


After the Staging Environment pipeline finish, you should be able to do a > jx logs again inside the staging environment (> jx env staging) and you should see that the service is now connecting to RabbitMQ

Building a Marketing Campaign
Fork github.com/activiti/ttc-rb-english-campaign
Clone it inside your ttc/ directory
Import it to Jenkins X: 
> jx import --branches "develop|PR-.*|feature.*"
Check in Jenkins UI that the project was imported and the initial build is triggered
From the terminal you can do jx logs and now you need to select which project do you want to tail. Remember always to be in jx env staging. 
You can also check that your pod is running by doing “kubectl get pods” and check that you have 1/1 replicas running. 
Finally you can verify that the url returned by “jx get apps” can be opened in your browser and you get a hello message: “Hello from the Trending Topic Campaign Named: my-runtime-bundle with topic: activiti”

This service is going to use RabbitMQ as well, but because we already set it up, there is nothing more required for this service to run. 

Time for some tests
> jx get apps
 Copy the URL for the ttc-connectors-dummytwitter feed app: 
> curl <url>/feed
Should return false -> meaning that the feed is turned off
> curl -X POST <url>/feed/start
This will start the feed. If you do 
> jx logs -> select -> ttc-connectors-dummytwitter

You will see the twitter feed being consumed and sent via Spring Cloud Streams. Every available Campaign (who matches some filters, for example the language filter) will be able to pick up the tweet for processing. 

You can stop the logs with CTRL+C and then 
> jx logs -> select -> ttc-rb-english-campaign

You will see tweets being consumed. Then you can stop the feed while we import more projects:

> curl -X POST <url>/feed/stop

Single Entrypoint ->  Gateway


Switch to the Dev environment in Jenkins X by
> jx env dev
Fork github.com/activiti/ttc-infra-gateway
Clone your fork inside the ttc directory
Import project into Jenkins X:
> jx import --branches "develop|PR-.*|feature.*"
Verify in Jenkins UI that the project was imported and the pipeline is running

Once the Pipeline is finished and the service promoted to staging, you should be able to access the following URL (which you can obtain by doing jx get apps)


http://<Gateway App URL>/campaigns

Also, you can access to: 


http://<Gateway App URL>/actuator/gateway/routes


Which shows the available registered services inside the gateway. 

Now you should be able to access
http://<Gateway App URL>/ttc-connectors-dummytwitter/

And

http://<Gateway App URL>/ttc-rb-english-campaign/

Meaning that now, you can start the feed by pointing the curl command to:
http://<Gateway App URL>/ttc-connectors-dummytwitter/feed/start

And 

http://<Gateway App URL>/ttc-connectors-dummytwitter/feed/stop

Consuming Data
Until this point we have a Service that produce data, in this case simulates a social media feed (ttc-connectors-dummytwitter). A service that process this data and defines the business logic on how to perform a marketing campaign (ttc-rb-english-campaign). Now we need a service that aggregates the data generated by multiple campaigns and allows us to consume that data without affecting the performance of each of the individual campaigns. 
This service is called Query Service and in order to get it up and running we will follow the same approach as before:
Fork github.com/activiti/ttc-query-campaign
Clone your fork inside the ttc directory
Import project into Jenkins X:
> jx import --branches "develop|PR-.*|feature.*"
Verify in Jenkins UI that the project was imported and the pipeline is running
 

At this point, when the pipeline finishes you should be able to query generic information about how many tweets are being processed (in-flight), how many were discarded and how many matched with the campaign and were ranked. 

In-flight processes analyzing tweets
curl http://<Gateway App URL>/ttc-rb-english-campaign/v1/process-instances/

Domain specific data for the campaign, only the tweets that have matched with the campaign. 

curl http://<Gateway App URL>/ttc-rb-english-campaign/processed/activiti


System to System Integrations (Service Orchestration)
Finally, as you will see when you try to consume data from the query service none of the processes are finishing, because they need other services to complete all the operations:

Fork github.com/activiti/ttc-connectors-reward
Fork github.com/activiti/ttc-connectors-ranking
Fork github.com/activiti/ttc-connectors-processing

Clone all of those projects

Import them all to Jenkins X

Wait for the pipelines to finish
