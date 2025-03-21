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
## User Feedback Loop
We have talked about feedback loops. Let's start by exploring a basic feedback mechanism where users directly provide feedback on food classifications.

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
### Explore User Feedback System
1. Access the Web Interface:
    - Open your browser and navigate to: http://localhost:8000
    - You'll see the Gourmet Gram interface for uploading and classifying food images

2. Test the Classification System:
    - Upload a food image using the interface
    - The system will display its prediction
    - You can either confirm the prediction is correct or select the correct class if it's wrong

3. Explore the MinIO Storage:
    - Navigate to http://localhost:9001
    - Login with the credentials username: minioadmin, password: minioadmin
    - Explore the `production-images` bucket to see how images are organized by class
    - Check the `tracking bucket` to view the JSON files that track user corrections

4. Generate a Test Suite:
    - You can generate a test suite based on user corrections by running:
    ```bash
    curl -X GET "http://localhost:8000/generate_test_suite"
    ```
    - This will create a timestamped directory in the test-suites bucket containing corrected images
:::

::: {.cell .code}
```python
# Command to shut down the application
remote.run("cd gourmetgram; docker-compose down -v")
```
:::


::: {.cell .markdown}
### Disadvantages of User Feedback

However, this approach of taking the user's feedback is not optimal for many reasons:

- Users come to Gourmet Gram seeking food classification, not to provide expert labels. There's a high probability they'll misclassify images, and retraining with this feedback would significantly degrade model performance.

- User feedback in production ML systems introduces subtle biases. For example, in our Gourmet Gram UI, food classes are listed in a fixed order while providing feedback. This creates an implicit bias where options at the top of the menu catch users' attention first and are selected more frequently, regardless of correctness.

- This phenomenon, called "degenerate feedback loops" is widespread in recommendation systems. On Netflix, already-popular shows receive more prominent placement, leading to more views, which then reinforces their "popularity" in the algorithm. These loops don't necessarily reflect quality or relevance to individual users, but rather amplify existing popularity patterns.

:::

::: {.cell .markdown}
## Human in the Loop Approach
For this reason, companies use a Human in the Loop approach to improve their model performance. In this case, predictions which either have a low confidence or user disagrees with are sent to data annotators. These annotators have significant experience in the domain of the model and therefore can provide reliable labels for the tasks at hand. These high-quality labels are then used to retrain the model, leading to better performance.
:::

::: {.cell .markdown}
### Setting up Human in the Loop System
Lets try and setting up a Human in the Loop approach for Gourmetgram! For this, we are using `Label Studio` which is an open source data labeling platform. 

https://labelstud.io/
:::

::: {.cell .code}
```python
# Lets fetch and checkout to feedback_loop_integration branch
remote.run("cd gourmetgram; git fetch origin feedback_loop_integration; git checkout feedback_loop_integration")
```
:::

::: {.cell .code}
```python
# lets bring the minio and label-studio containers first
remote.run("cd gourmetgram; docker-compose up -d minio label-studio")

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
### Explore Human in the Loop System
1. Access the Web Interface:
    - Open your browser and navigate to: http://localhost:8000
    - Upload food images for classification

2. Provide Feedback:
    - After seeing a prediction, you can give thumbs up (correct) or thumbs down (incorrect)
    - Notice that unlike the first system, you don't directly provide the correct class
    - Images with low confidence or negative feedback are sent to Label Studio for expert review

3. Explore Label Studio (Annotation Interface):
    - Navigate to http://localhost:8080
    - Login with username: gourmetgramuser@gmail.com, password: gourmetgrampassword
    - Select the "Food Classification Review" project
    - Click on "Tasks" to see images waiting for expert review
    - For each task, select the correct food category and submit your annotation

4. Create random sampling tasks 
    ```bash
    curl -X POST "http://localhost:8000/sample_random_images" -H "Content-Type: application/json" -d '{"sample_count": 5}'
    ```
5. Process Expert Annotations:
    - After annotating some images in Label Studio, process these annotations:
    ```bash
    curl -X POST "http://localhost:8000/process_labels"
    ```
This will move images to their correct class directories
6. Create test suites:
    - After processing the labels, create test cases based on task type (random sampling, low confidence or user feedback)
    ```bash
    curl -X GET http://localhost:8000/generate_test_suite?task_type=user_feedback
    curl -X GET http://localhost:8000/generate_test_suite?task_type=low_confidence
    curl -X GET http://localhost:8000/generate_test_suite?task_type=random_sampling
    curl -X GET http://localhost:8000/generate_test_suite?task_type=all
    ```
7. Explore MinIO contents:
    - Navigate to http://localhost:9001
    - Examine the different buckets to see how data is organized:
        - `production-images`: Contains all classified images
        - `tracking`: Contains tracking JSONs for different feedback sources
        - `target-bucket`: Where Label Studio exports annotations
        - `test-suites`: Contains organized test suites for evaluation
:::

::: {.cell .code}
```python
# Command to shut down the application
remote.run("cd gourmetgram; docker-compose down -v")
```
:::