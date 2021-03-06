# Running Crypto Flask App through Docker, Kubernetes and AWS
<img width="1218" alt="Screen Shot 2021-06-04 at 4 55 29 PM" src="https://user-images.githubusercontent.com/78613742/120873336-9013ad00-c556-11eb-99d9-e5d47e0f5933.png">


### What the project does and how it was made?
This project was built using Python3 to develop Real-Time Candlestick Charts and a Crypto Trading Bot using Binance API and Websockets via Flask based web application. Once the Flask App was developed:
* I created a customized Docker container of the flask application
* Pushed the Docker image to Amazon ECR and Elastic Beanstalk
* Created an AWS ECS and Cluster Task
* Pull image down and run it on a cloud platform cloud shell: AWS Cloud9
* Deployed the application to managed Kubernetes cluster via AWS EKS

### Prerequisites:
* Python3
* [Binance API Key](https://www.binance.com/en) (signup for free)
* [AWS Account](https://aws.amazon.com/free/)
* AWS CLI
* [Docker](https://docs.docker.com/)



### To build this project perform the following steps:

Clone the GitHub repository

### Setup to run the project
1. Flask App

Clone the GitHub repository.
Install the library requirements from the `requirements.txt` file. Change your `config.py` file with the appropriate API_KEY and API_SECRET.  Run the command python 
```bash
python3 app.py
``` 
or 
```python
flask run
``` 
on the terminal. Make sure that you are executing this command from the directory of the file.

<img width="1043" alt="Screen Shot 2021-06-05 at 10 17 25 AM" src="https://user-images.githubusercontent.com/78613742/120899991-56d74d80-c5e7-11eb-9cdd-168917e95c92.png">

2. Docker

#### Dockerfile
Create a Dockerfile, which is a list of commands that the Docker client calls while creating an image. It's a simple way to automate the image creation process. View sample Dockerfile [here](https://github.com/Darenson/Capstone_BinanceFlask_Docker_Kubernetes/blob/master/Dockerfile)

Now that we have our Dockerfile, we can build our image. The docker build command does the heavy-lifting of creating a Docker image from a Dockerfile.
The docker build command is quite simple - it takes an optional tag name with -t and a location of the directory containing the Dockerfile.
<img width="1165" alt="Screen Shot 2021-06-04 at 5 54 18 PM" src="https://user-images.githubusercontent.com/78613742/120874956-f8b25800-c55d-11eb-8024-bcd288e1f958.png">

The last step in this section is to run the image and see if it actually works (replacing my username with yours).

<img width="1159" alt="Screen Shot 2021-06-04 at 6 05 07 PM" src="https://user-images.githubusercontent.com/78613742/120875198-8478b400-c55f-11eb-91e1-114bef3e6225.png">

The command we just ran used port 5000 for the server inside the container and exposed this externally on port 8888. Head over to the URL with port 8888, where your app should be live.

3. Elastic Beanstalk

Now we need to upload our application code. But since our application is packaged in a Docker container, we just need to tell EB about our container. Open the 
`Dockerrun.aws.json` located [here](https://github.com/Darenson/Capstone_BinanceFlask_Docker_Kubernetes/blob/master/Dockerrun.aws.json) and edit the Name of the image to your image's name. When you are done, click on the radio button for "Upload your Code", choose this file, and click on "Upload".
Now click on "Create environment". The final screen that you see will have a few spinners indicating that your environment is being set up. 
(Note: the warning sign doesn't seem to impact the app from running)


<img width="1045" alt="Screen Shot 2021-06-04 at 6 19 23 PM" src="https://user-images.githubusercontent.com/78613742/120875603-83e11d00-c561-11eb-9609-63987f19b175.png">



To view the live site visit: [http://cryptobinancedemo.us-west-1.elasticbeanstalk.com/](http://cryptobinancedemo.us-west-1.elasticbeanstalk.com/)

4. ECR

### Login AWS CLI
To install locally:
```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
``` 


### Login to ECR
`aws ecr get-login-password --region REGIONHERE!!!! | docker login --username AWS --password-stdin ACCOUNTIDHERE!!!!.dkr.ecr.REGIONHERE!!!.amazonaws.com`

<img width="1166" alt="Screen Shot 2021-06-04 at 6 26 18 PM" src="https://user-images.githubusercontent.com/78613742/120875781-7710f900-c562-11eb-8582-fca5ca7df692.png">


### Tag the version
`docker tag test:latest YOURACCOUNT.dkr.ecr.YOURREGION-1.amazonaws.com/YOURREPO:YOURTAG`

<img width="1262" alt="Screen Shot 2021-06-04 at 6 31 14 PM" src="https://user-images.githubusercontent.com/78613742/120876024-5bf2b900-c563-11eb-8dd6-d394a971f32c.png">


### Upload
`docker push YOURACCOUNT.dkr.ecr.YOURREGION.amazonaws.com/YOURREPO:YOURTAG`

<img width="1163" alt="Screen Shot 2021-06-04 at 6 23 32 PM" src="https://user-images.githubusercontent.com/78613742/120875729-33b68a80-c562-11eb-8fa5-eac761105047.png">


To confirm that it worked, you should have something on the AWS console that looks like this:
![Screen Shot 2021-06-05 at 11 08 39 AM](https://user-images.githubusercontent.com/78613742/120901252-82116b00-c5ee-11eb-84ac-9ae46bbf5d9a.png)


5. ECS

Once the ECR portion is complete, next is going into ECS and creating a cluster. The cluster template I selected was EC2 Linux + Networking.
Going through all of the provisions for Instance Configuration, Networking I left everything as Default **EXCEPT** EC2 Instance Type as t3.micro and "Enable" Auto assign Public IP.
Next, go to "Task Definitions" on the ECS menu and create a new task defition for the project. Select EC2 as the launch type, add memory amount for task size

![Screen Shot 2021-06-05 at 11 23 32 AM](https://user-images.githubusercontent.com/78613742/120901540-8048a700-c5f0-11eb-910e-5040981516f6.png)

Then add the container information from ECR (Name, Image) and add Host Port 8888 and Container port 5000 to Port Mappings and click Create.

Next, go back to ECS menu, click on the Cluster and add the new Task to run.
The application that is loaded in ECR as the Docker Image is now going to be deployed on this machine and should look something like this:

![Screen Shot 2021-06-05 at 11 33 06 AM](https://user-images.githubusercontent.com/78613742/120901816-d407c000-c5f1-11eb-853b-ecd1b4b6eb66.png)

Lastly, to get the container running with the public IP address VPC that is hosting this application we will need to adjust some of the secuirty settings to let incoming traffic in. Go to your EC2 instance for this cluster, on the left hand side under Network Security, click on Security Groups. Click on the Default security group and add TCP Port 8888 to Incoming Inbound Rules.
To verify: enter your Public IPv4 DNS, add port 8888 and it should be good to go.
```bash
http://ec2-18-144-53-204.us-west-1.compute.amazonaws.com:8888/
```  
![Screen Shot 2021-06-05 at 11 43 02 AM](https://user-images.githubusercontent.com/78613742/120902097-4927c500-c5f3-11eb-8e26-916eadd8447a.png)


6. EKS

To setup EKS properly, you first have to add the necessary permissions to your IAM user. I added all EKS related admin permissions to the user.
Next, you will need to install a few dependcies for EKS: Weaveworks and EKSCTL. The documentation can be found [here](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
Follow these installation steps:
```bash
brew tap weaveworks/tap
``` 
```bash
brew install weaveworks/tap/eksctl
``` 
to verify it worked:
```bash
eksctl version
``` 
next, add a config.yaml and client-deployment.yaml which can be found [here](https://github.com/Darenson/Capstone_BinanceFlask_Docker_Kubernetes/tree/master/kubernetes)

Then signin to AWS CLI and add the cluster.yaml
```bash
eksctl create cluster -f cluster.yaml
``` 
It will take several minutes to run, but you should eventually see something like this:

![Screen Shot 2021-06-05 at 12 55 44 PM](https://user-images.githubusercontent.com/78613742/120903850-75484380-c5fd-11eb-8f7f-a8e7e4f31d9f.png)

Next, confirm that kubernetes is now running via AWS CLI and add the client-deployment.yaml file as well:

![Screen Shot 2021-06-05 at 12 59 10 PM](https://user-images.githubusercontent.com/78613742/120903909-d7a14400-c5fd-11eb-967d-399bfc6e629c.png)

Lastly, you will now see the Kubernetes cluster on the AWS portal:
![Screen Shot 2021-06-05 at 12 57 07 PM](https://user-images.githubusercontent.com/78613742/120903866-9e68d400-c5fd-11eb-9464-fce99ffa8ed5.png)
 
 
 
 




