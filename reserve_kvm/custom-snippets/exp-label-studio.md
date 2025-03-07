::: {.cell .markdown}
## Set up Docker
:::

::: {.cell .code}
```python
remote = chi.ssh.Remote(server_ips[0])
```
:::

::: {.cell .code}
```python
remote.run("sudo apt-get update")
remote.run("sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common")
```
:::

::: {.cell .code}
```python
remote.run("curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg")
remote.run('echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null')
```
:::


::: {.cell .code}
```python
remote.run("sudo apt-get update")
remote.run("sudo apt-get install -y docker-ce docker-ce-cli containerd.io")
```
:::


::: {.cell .code}
```python
remote.run('sudo groupadd -f docker; sudo usermod -aG docker $USER')
remote.run("sudo chmod 666 /var/run/docker.sock")
```
:::

::: {.cell .code}
```python
# check configuration
remote.run("docker run hello-world")
```
:::

::: {.cell .code}
```python
remote.run("sudo apt-get install -y python3 python3-pip")
remote.run("python3 -m pip config set global.break-system-packages true")
```
:::



::: {.cell .code}
```python
# Clone the gourmetgram repository with feedback_loop_integration branch if using labelling setup
remote.run("git clone -b feedback_loop_integration https://github.com/ShaktidharK1997/gourmetgram.git")

# Clone the gourmetgram repository with user_proxy_signal branch if using user signal setup
# remote.run("git clone -b user_proxy_signal https://github.com/ShaktidharK1997/gourmetgram.git")

```
:::

::: {.cell .code}
```python
# install docker compose 
# Download the docker compose plugin
remote.run("sudo curl -L https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose")

# Make it executable
remote.run("sudo chmod +x /usr/local/bin/docker-compose")

# Verify the installation
remote.run("docker-compose --version")
```
:::

::: {.cell .markdown}

!NOTE : Once Docker-compose is set up, please take the env file that has been shared with you and copy it into the gourmetgram repository. This is important for the application to work correctly. 

:::


::: {.cell .code}
```python
# Use docker to setup the minio object store and the label studio
remote.run("cd gourmetgram; docker-compose up -d label-studio minio")

# Wait 15 seconds for Label Studio to get ready..
remote.run("sleep 15")
```
:::

::: {.cell .code}
```python
# Use docker to setup the Flask Application
remote.run("cd gourmetgram; docker-compose up flask-app")
```
:::

::: {.cell .code}
```python
# Use docker to setup the scheduler
remote.run("cd gourmetgram; docker-compose up -d scheduler")
```
:::

::: {.cell .markdown}

If you want to run gourmetgram application using a user proxy signal instead of a labelling setup, proceed with the next steps instead 

:::

::: {.cell .code}
```python
# Use docker to setup 
remote.run("cd gourmetgram; docker-compose up --build")
```
:::