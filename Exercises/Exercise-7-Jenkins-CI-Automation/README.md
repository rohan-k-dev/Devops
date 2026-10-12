# Exercise 7: Setting Up Jenkins for Continuous Integration

## CI in Practice
Continuous Integration (CI) is a development practice where developers frequently integrate their code changes into a shared repository. The primary goal of CI is to detect and address errors early in the development process, improving software quality and accelerating delivery.

### Main CI Characteristics
1. **Frequent Code Integration**: Developers commit code multiple times a day.
2. **Automated Builds**: Every commit triggers an automated build.
3. **Automated Testing**: Tests run automatically to ensure functionality remains intact.
4. **Immediate Feedback**: Developers receive quick feedback to address issues promptly.

### Why CI Helps
- Early Bug Detection
- Improved Collaboration
- Faster Development Cycles
- High-Quality Code

## Running Jenkins in Docker
Jenkins is an open-source automation server used to build, test, and deploy software.

### Start the Jenkins Container
```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
```
- `-p 8080:8080`: Exposes the web interface.
- `-p 50000:50000`: Exposes the agent communication port.

### Complete the First-Time Setup
1. Retrieve the initial admin password:
   ```bash
   docker exec -it jenkins bash
   cat /var/jenkins_home/secrets/initialAdminPassword
   ```
2. Access Jenkins at `http://localhost:8080/`.
3. Provide the password and follow the setup wizard to install suggested plugins and create the first admin user.
