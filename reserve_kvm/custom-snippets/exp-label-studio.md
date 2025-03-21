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

We have talked about feedback loops. Lets try and setup a very basic one where the user gives feedback on the correct food class of the image. 

:::

::: {.cell .code}
```python
# Clone the gourmetgram repository with user_corrected branch 
remote.run("git clone -b user_corrected https://github.com/ShaktidharK1997/gourmetgram.git")

```
:::

::: {.cell .markdown}

!NOTE : Once Docker-compose is set up, please take the env file that has been shared with you and copy it into the gourmetgram repository. This is important for the application to work correctly. 

:::


::: {.cell .code}
```python
# Use docker to setup the Flask application
remote.run("cd gourmetgram; docker-compose up -d --build")

```
:::

::: {.cell .markdown}

Please look at the README file to understand more about it

:::

::: {.cell .code}
```python
# Command to shut down the application
remote.run("cd gourmetgram; docker-compose down -v")
```
:::


::: {.cell .markdown}

However, this approach of taking the user's feedback is not optimal for many reasons:

- Users come to Gourmet Gram seeking food classification, not to provide expert labels. There's a high probability they'll misclassify images, and retraining with this feedback would significantly degrade model performance.

- User feedback in production ML systems introduces subtle biases. For example, in our Gourmet Gram UI, food classes are listed in a fixed order while providing feedback. This creates an implicit bias where options at the top of the menu catch users' attention first and are selected more frequently, regardless of correctness.

- This phenomenon, called "degenerate feedback loops," is widespread in recommendation systems. On Netflix, already-popular shows receive more prominent placement, leading to more views, which then reinforces their "popularity" in the algorithm. These loops don't necessarily reflect quality or relevance to individual users, but rather amplify existing popularity patterns.

:::

::: {.cell .markdown}

For this reason, companies use a Human in the Loop approach to improve their model performance. In this case, predictions which either have a low confidence or user disagrees with are sent to data annotators. These annotators have significant experience in the domain of the model and therefore can provide reliable labels for the tasks at hand. These high-quality labels are then used to retrain the model, leading to better performance.
:::

::: {.cell .markdown}

Lets try and setting up a Human in the Loop approach for Gourmetgram! For this, we are using Label Studio which is an open source data labeling platform. 
:::

::: {.cell .code}
```python
# Clone the gourmetgram repository with feedback_loop_integration branch 
remote.run("cd gourmetgram; git fetch origin feedback_loop_integration; git checkout feedback_loop_integration")
```
:::

::: {.cell .code}
```python
# lets bring the minio and label-studio containers first
remote.run("cd gourmetgram; docker-compose up minio label-studio -d")

# Wait 30 seconds for Label studio to get started ...
remote.run('sleep(30)')
```
:::


::: {.cell .code}
```python
# lets bring the flask application now
remote.run("cd gourmetgram; docker-compose up flask-app --build")
```
:::

::: {.cell .markdown}

Please look at the README file to understand more about it

:::