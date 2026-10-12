# Exercise 8: Running a Simple Script from Jenkins

## Objective
This exercise connects a small shell script to a Jenkins Freestyle project so the script can be checked out and run as a repeatable build step.

## Script Used
- `hello-world.sh`: A simple bash script that outputs "Hello, Jenkins!".

## Build Setup
1. **Create the script**: Added `hello-world.sh` and made it executable (`chmod +x hello-world.sh`).
2. **Commit the script**: Pushed the file to this GitHub repository.
3. **Configure Jenkins**:
   - Created a new **Freestyle project** in Jenkins named `HelloWorld`.
   - Configured **Source Code Management** to use Git, pointing to this repository URL.
   - Added a **Build Step** -> *Execute shell*.
   - Command: `sh Exercises/Exercise-8-Jenkins-Hello-World/hello-world.sh`
4. **Run the job**: Started the build manually with "Build Now".

## Successful Build Output
```
Started by user Admin
Building in workspace /var/jenkins_home/workspace/HelloWorld
 > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/HelloWorld/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/SuchitKS/devops.git # timeout=10
Fetching upstream changes from https://github.com/SuchitKS/devops.git
 > git --version # timeout=10
 > git --version # 'git version 2.39.5'
 > git checkout -f main # timeout=10
[HelloWorld] $ /bin/sh -xe /tmp/jenkins1281930102.sh
+ sh Exercises/Exercise-8-Jenkins-Hello-World/hello-world.sh
Hello, Jenkins!
Finished: SUCCESS
```
