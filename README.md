# Food Classification System

This system provides an end-to-end solution for food image classification with continuous improvement through human feedback. The system automatically classifies uploaded food images into 11 categories and gets better over time by incorporating user feedback and expert annotations.

## System Overview

After running `docker-compose up`, you'll have access to the following components:

| Component | URL | Description |
|-----------|-----|-------------|
| Food Classification Web App | http://localhost:8000 | Upload images and get classifications |
| Label Studio | http://localhost:8080 | Review and correct classifications |
| MinIO Console | http://localhost:9001 | View stored images and tracking data |

## Getting Started

1. Start the system: Follow instructions in `reserve_kvm\reserve_chameleon.ipynb`
2. Wait for all services to initialize (usually takes about 30-60 seconds)
3. Access the web app at http://localhost:8000 to begin classifying food images

## Components

### 1. Food Classification Web App (Flask)

**URL**: http://localhost:8000

When you upload an image, the system will:
1. Classify it into one of 11 food categories using a pre-trained PyTorch model
2. Display the predicted class
3. Allow you to confirm or reject the classification
4. Store the image in MinIO in a class-specific directory

### 2. Label Studio

**URL**: http://localhost:8080

Label Studio is where:
- Incorrectly classified or low-confidence images are sent for review
- Human annotators can select the correct food category
- Annotations are automatically exported to improve the system

After logging in:
1. Navigate to the "Food Classification Review" project
2. Click on "Tasks" to see images waiting for review
3. For each task, select the correct food category from the available options
4. Submit your annotation

### 3. MinIO Storage

**URL**: http://localhost:9001

MinIO has specific buckets for:
- Food images (organized by class directory)
- Tracking data (feedback, tasks, etc.)
- Label Studio export data
- Test suites (organized collections of images for evaluation)

Key buckets:
- `production-images`: Contains classified images in class directories (class_00 to class_10)
- `tracking`: Contains JSON files tracking system operations
  - `user_feedback_tasks.json`: Contains details regarding tasks generated from negative user feedback
  - `low_confidence_tasks.json`: Contains details regarding tasks generated from low confidence predictions
  - `random_sampling_tasks.json`: Contains details regarding tasks generated from random sampling
  - `production_data.json`: Contains production data for all predicted images
- `target-bucket`: Where Label Studio exports annotations
- `test-suites`: Contains organized collections of images for model evaluation and testing

### 4. Background Scheduler

This component runs in the background to:
- Process completed Label Studio annotations
- Move images to correct class directories based on completed tasks 
- Update tracking files
- Create tasks for random sampling in Label Studio
- Generate test suites daily for quality assurance

### 5. Test Suite Generator

The Test Suite Generator creates organized collections of images based on their feedback history:
- Creates test suites for user feedback, low confidence, and random sampling images
- Organizes images into timestamped directories for historical tracking
- Runs automatically once per day
- Can be triggered manually for on-demand test suite creation

## Feedback Loop

The system implements a continuous improvement cycle:

1. Users upload images for classification
2. The model makes predictions
3. Users provide feedback on the accuracy
4. Incorrect or low-confidence predictions are sent to Label Studio
5. Annotators review and correct classifications
6. The background processor applies these corrections
7. Future predictions benefit from this additional knowledge
8. Test suites capture the system's performance over time

## On-Demand Script Execution

You can manually trigger any of the background processes using Docker exec commands:

### Access the Scheduler Container

```bash
docker exec -it <scheduler-container-name> bash
```

Replace `<scheduler-container-name>` with the actual container name or ID. You can find this by running `docker ps`.

### Run Individual Scripts

Once inside the container, you can execute any of these scripts:

```bash
# Process Label Studio annotations
/usr/local/bin/python /app/process_labels.py

# Create random sampling tasks in Label Studio
/usr/local/bin/python /app/random_sampler.py

# Generate test suites (with task type parameter)
/usr/local/bin/python /app/test_suite_generator.py user_feedback
/usr/local/bin/python /app/test_suite_generator.py low_confidence
/usr/local/bin/python /app/test_suite_generator.py random_sampling
/usr/local/bin/python /app/test_suite_generator.py all
```

Each test suite contains:
- Original images
- A metadata.json file with creation timestamp and source information
- Organization by feedback source (user feedback, low confidence, random sampling)

Access test suites through the MinIO Console at http://localhost:9001, navigating to the `test-suites` bucket.