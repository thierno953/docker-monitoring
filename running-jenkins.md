# How To Run Jenkins Server in Docker Container

Running Jenkins Server on a Docker has few dependencies that you’ll need to satisfy.

- Linux or macOS

- Docker Engine installed and running

- A user account with sudo privileges

# Step 1: Install Docker Engine

Start by installing Docker engine on your base operating system. You can use our previous guide for docker installation.

After the installation, you can confirm the version installed by running:

```jsx
docker version
```

# Step 2: Add jenkins user

Next is to add Jenkins system user to your server. This user will manage Jenkins service.

```jsx
sudo groupadd --system jenkins
sudo useradd -s /sbin/nologin --system -g jenkins jenkins
sudo usermod -aG docker jenkins
```

Ensure this user is added to the docker group.

```jsx
id jenkins
```

Pull the LTS docker image release of Jenkins

```jsx
sudo docker pull jenkins/jenkins:lts
```

Confirm that the image download was successful.

```jsx
docker images
```

# Step 3: Create a Jenkins Data directory & container

We need a persistent storage for Jenkins data to ensure that the data is made to remain intact and can, therefore, be re-used in the event that the container shuts down or crashes.

```jsx
sudo mkdir /var/jenkins
sudo chown -R 1000:1000 /var/jenkins
```

# Step 4: Create a Jenkins Container Systemd service

Create a new systemd service unit file for Jenkins.

```jsx
sudo vim /etc/systemd/system/jenkins-docker.service
```

add:

```jsx
[Unit]
Description=Jenkins Server
Documentation=https://jenkins.io/doc/
After=docker.service
Requires=docker.service

[Service]
Type=simple
User=jenkins
Group=jenkins
TimeoutStartSec=0
Restart=on-failure
RestartSec=30s
ExecStartPre=-/usr/bin/docker kill jenkins-server
ExecStartPre=-/usr/bin/docker rm jenkins-server
ExecStartPre=/usr/bin/docker pull jenkins/jenkins:lts
ExecStart=/usr/bin/docker run \
          --name jenkins-server \
          --publish 8080:8080 \
          --publish 50000:50000 \
          --volume /var/jenkins:/var/jenkins_home \
          jenkins/jenkins:lts
SyslogIdentifier=jenkins
ExecStop=/usr/bin/docker stop jenkins-server

[Install]
WantedBy=multi-user.target
```

Reload systemd and start jenkins service

```jsx
sudo systemctl daemon-reload
sudo systemctl start jenkins-docker
```

On checking the status, you should get a running message.

```jsx
systemctl status jenkins-docker
```

# Step 5: Unlock Jenkins Server installation

Browse to the URL to access web installation wizard:

```jsx
http://[serverip|hostname]:8080
```

When you first access a new Jenkins instance, you are asked to unlock it using an automatically generated password.

Get the password from

```jsx
cat /var/jenkins/secrets/initialAdminPassword
```