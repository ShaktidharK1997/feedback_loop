# Gourmet Gram: Food Classification Systems

## User Feedback Approach

### System Overview

After running `docker-compose up` for the `user_corrected` branch, you'll have access to the following components:

| Component | URL | Description |
|-----------|-----|-------------|
| Food Classification Web App | http://localhost:8000 | Upload images and get classifications |
| MinIO Console | http://localhost:9001 | View stored images and feedback data |

### Getting Started

1. Start the system: Follow instructions in `reserve_kvm\reserve_chameleon.ipynb`
2. Wait for all services to initialize (usually takes about 20-30 seconds)
3. Access the web app at http://localhost:8000 to begin classifying food images

### Components

#### 1. Food Classification Web App (Flask)

**URL**: http://localhost:8000

When you upload an image, the system will:
1. Classify it into one of 11 food categories using a pre-trained PyTorch model
2. Display the predicted class
3. Allow you to confirm or reject the classification
4. If confirmed, store the image in MinIO in the predicted class directory
5. If rejected, allow you to select the correct class and move the image to that class directory

#### 2. MinIO Storage

**URL**: http://localhost:9001

MinIO organizes:
- Food images (in class-specific directories class_00 through class_10)
- Tracking data (production predictions and user corrections)

Key buckets:
- `production-images`: Contains classified images in class directories
- `tracking`: Contains JSON files tracking system operations
  - `production_data.json`: Records all predicted images and model information
  - `user_corrected_labels.json`: Records user corrections to model predictions
- `test-suites`: Contains organized collections of corrected images for evaluation

#### 3. Test Suite Generator

The system can generate test suites based on user corrections:
- Creates test suites for user-corrected images
- Organizes images into timestamped directories for historical tracking
- Can be triggered through the web interface

### Direct User Feedback Loop

This implementation uses direct user feedback as follows:

1. Users upload images for classification
2. The model makes predictions
3. Users confirm correct predictions or provide corrections for incorrect ones
4. Images are stored in their correct class directories (either as predicted or as corrected)
5. A test suite can be generated from corrected images for model evaluation

The `/generate_test_suite` endpoint can be accessed to create test collections of corrected images for offline model evaluation.

### Limitations of Direct User Feedback

As discussed in the notebook, this direct user feedback approach has limitations:
- Users may misclassify images if they're uncertain about food categories
- UI design elements (like the order of options in the correction dropdown) can introduce selection biases
- Without expert validation, feedback quality can be inconsistent

These limitations are addressed in the improved Data Annotator approach implemented in the `feedback_loop_integration` branch.

## Data Annotator Approach

### System Overview

After running `docker-compose up` for the `feedback_loop_integration` branch, you'll have access to the following components:

| Component | URL | Description |
|-----------|-----|-------------|
| Food Classification Web App | http://localhost:8000 | Upload images and get classifications |
| Label Studio | http://localhost:8080 | Review and correct classifications |
| MinIO Console | http://localhost:9001 | View stored images and tracking data |

### Getting Started

1. Start the system: Follow instructions in `reserve_kvm\reserve_chameleon.ipynb`
2. Wait for all services to initialize (usually takes about 30-60 seconds)
3. Access the web app at http://localhost:8000 to begin classifying food images

### Components

#### 1. Food Classification Web App (Flask)

**URL**: http://localhost:8000

When you upload an image, the system will:
1. Classify it into one of 11 food categories using a pre-trained PyTorch model
2. Display the predicted class
3. Allow you to provide feedback (thumbs up/down) on the classification
4. Automatically send low-confidence predictions to Label Studio for expert review
5. Send images with negative user feedback to Label Studio for expert review

The web app also provides endpoints for:
- Processing Label Studio annotations (`/process_labels`)
- Generating test suites (`/generate_test_suite`)
- Random sampling of images for quality control (`/sample_random_images`)

#### 2. Label Studio

**URL**: http://localhost:8080

Label Studio is where:
- Incorrectly classified or low-confidence images are sent for review
- Human annotators (domain experts) can select the correct food category
- Annotations are automatically exported for system improvement

After logging in:
1. Navigate to the "Food Classification Review" project
2. Click on "Tasks" to see images waiting for review
3. For each task, select the correct food category from the available options
4. Submit your annotation

#### 3. MinIO Storage

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

### Data Annotator Workflow

The system implements the following feedback loop:

1. Users upload images for classification
2. The model makes predictions
3. Images are filtered into three streams:
   - Low confidence predictions are automatically sent to Label Studio
   - Images with negative user feedback are sent to Label Studio
   - Random samples are periodically sent to Label Studio for quality control
4. Expert annotators review and correct classifications in Label Studio
5. The `/process_labels` endpoint processes these annotations and moves images to the correct class directories
6. Test suites can be generated for model evaluation based on different feedback sources

### Running On-Demand Processes

You can manually trigger any of the processes using the Flask endpoints:

```bash
# Process Label Studio annotations
curl -X POST http://localhost:8000/process_labels

# Create random sampling tasks in Label Studio
curl -X POST http://localhost:8000/sample_random_images -H "Content-Type: application/json" -d '{"sample_count": 5}'

# Generate test suites
curl -X GET http://localhost:8000/generate_test_suite?task_type=user_feedback
curl -X GET http://localhost:8000/generate_test_suite?task_type=low_confidence
curl -X GET http://localhost:8000/generate_test_suite?task_type=random_sampling
curl -X GET http://localhost:8000/generate_test_suite?task_type=all
```
