# Exercise 5: Applying an AppArmor Profile to a Docker Container

## Objective
This lab uses Python and the Docker SDK to apply an AppArmor policy, then checks that the container cannot read protected files or execute restricted binaries.

## Files in This Exercise
- `app.py`: A simple Flask application to be containerized.
- `Dockerfile`: To containerize the Flask application.
- `my-apparmor-profile`: AppArmor profile to restrict access to `/etc/`, `/var/`, and binaries in `/bin/` or `/usr/bin/`.
- `apply_apparmor.py`: Python script using the Docker SDK to build the image, run the container with the AppArmor profile, and verify its application.
- `test_restricted_actions.py`: Python script to test the restrictions (e.g., trying to read `/etc/passwd` or execute `/bin/bash`).

## Recorded Run

### Applying the Profile with `apply_apparmor.py`
```
Building image from Dockerfile...
[INFO] Sending build context to Docker daemon  3.072kB
[INFO] Step 1/5 : FROM python:3.8-slim
 ---> e83d9d28b2f6
...
[INFO] Successfully built flask-apparmor

Running container with AppArmor profile...
Container started: f8c2a7f9b9b8

Inspecting container to verify AppArmor profile...
AppArmor profile applied: ['apparmor=my-apparmor-profile']

Stopping the container...
```

### Checking Restricted Actions
```
Container started: f8c2a7f9b9b8

Attempt to read /etc/passwd: Exit Code 1, Output: cat: /etc/passwd: Permission denied
Attempt to execute /bin/bash: Exit Code 126, Output: /bin/bash: Permission denied
Container stopped
```

## Security Review

1. **What is the purpose of using AppArmor with Docker containers?**
AppArmor is used to enforce security policies and confine applications to a limited set of resources. With Docker containers, it helps to limit access to system resources, files, and networks, providing an additional layer of security.

2. **How do AppArmor profiles help secure a Docker container?**
AppArmor profiles define what a containerized application can or cannot do. They restrict access to sensitive directories, network capabilities, file execution, and system calls, ensuring the container behaves securely without affecting the host system.

3. **Why is it important to restrict access to sensitive directories such as `/etc/` and `/var/`?**
Sensitive directories like `/etc/` contain configuration files and sensitive information such as user data and system settings. Restricting access prevents the container from reading or modifying important system files, reducing the risk of security breaches.

4. **What other capabilities can you restrict using AppArmor profiles?**
AppArmor can restrict a container's ability to access the network, bind to specific ports, execute binaries, write to specific directories, and use system administration capabilities (`cap_sys_admin`).

5. **How can you verify if an AppArmor profile is successfully applied to a Docker container?**
You can verify if an AppArmor profile is applied by inspecting the container using the Docker SDK (`client.api.inspect_container()`) or the Docker CLI (`docker inspect`). The `HostConfig.SecurityOpt` field will show the applied security options, including the AppArmor profile name.
