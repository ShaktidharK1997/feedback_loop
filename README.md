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

Key buckets:
- `production-images`: Contains classified images in class directories (class_00 to class_10)
- `tracking`: Contains JSON files tracking system operations
- - `user_feedback_tasks.json` : Contains details regarding tasks generated from negative user feedback
- - `low_confidence_tasks.json` : Contains details regarding tasks generated from low confidence predictions
- - `random_sampling_tasks.json` : Contains details regarding tasks generated from random sampling
- - `production_data.json` : Contains production data for all predicted images. 
- `target-bucket`: Where Label Studio exports annotations

### 4. Background Scheduler

This component runs in the background to:
- Process completed Label Studio annotations
- Move images to correct class directories based on completed tasks 
- Update tracking files
- Create tasks for random sampling in Label Studio

## Feedback Loop

The system implements a continuous improvement cycle:

1. Users upload images for classification
2. The model makes predictions
3. Users provide feedback on the accuracy
4. Incorrect or low-confidence predictions are sent to Label Studio
5. Annotators review and correct classifications
6. The background processor applies these corrections
7. Future predictions benefit from this additional knowledge


