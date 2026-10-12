# Exercise 9: Automating a Flask Delivery Pipeline with Jenkins

## Objective
This final lab uses a declarative Jenkins Pipeline to move a sample Flask application through build, test, deployment, execution, and application-test stages.

## Pipeline Files
- `app.py`: Simple Python Flask application.
- `requirements.txt`: Defines the Python dependencies (`flask==2.1.2`).
- `test_app.py`: Unit tests using the `unittest` framework.
- `Jenkinsfile`: Defines the declarative Jenkins pipeline with stages: Build, Test, Deploy, Run Application, and Test Application.

## Jenkins Environment Setup
To run the pipeline successfully, the Jenkins environment needs Python and Flask installed. This is achieved by running the following inside the Jenkins Docker container:
```bash
apt-get update
apt install python3 python3-pip python3.11-venv python3-flask -y
```

## Example Successful Run

```
Started by user admin
Obtained Jenkinsfile from git https://github.com/SuchitKS/devops.git
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/jenkins_home/workspace/Python-MultiStage-Pipeline
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
...
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] echo
Creating virtual environment and installing dependencies...
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] echo
Running tests...
[Pipeline] sh
+ cd Exercises/Exercise-9-Jenkins-Multi-Stage-Pipeline
+ python3 -m unittest discover -s .
.
----------------------------------------------------------------------
Ran 1 test in 0.004s

OK
Hello, Jenkins Multi-Stage Pipeline!
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy)
[Pipeline] echo
Deploying application...
[Pipeline] sh
+ mkdir -p /var/jenkins_home/workspace/Python-MultiStage-Pipeline/python-app-deploy
+ cp /var/jenkins_home/workspace/Python-MultiStage-Pipeline/Exercises/Exercise-9-Jenkins-Multi-Stage-Pipeline/app.py /var/jenkins_home/workspace/Python-MultiStage-Pipeline/python-app-deploy/
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Run Application)
[Pipeline] echo
Running application...
[Pipeline] sh
+ echo 5064
+ nohup python3 /var/jenkins_home/workspace/Python-MultiStage-Pipeline/python-app-deploy/app.py
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Test Application)
[Pipeline] echo
Testing application...
[Pipeline] sh
+ python3 /var/jenkins_home/workspace/Python-MultiStage-Pipeline/Exercises/Exercise-9-Jenkins-Multi-Stage-Pipeline/test_app.py
.
----------------------------------------------------------------------
Ran 1 test in 0.004s

OK
Hello, Jenkins Multi-Stage Pipeline!
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Declarative: Post Actions)
[Pipeline] echo
Pipeline completed successfully!
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```
