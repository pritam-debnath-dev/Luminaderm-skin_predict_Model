# LuminaDerm --- AI-Powered Skin Assessment

LuminaDerm is an AI-assisted skin assessment project that uses an
uploaded skin image to suggest possible skin conditions. It also
explores how user-reported symptoms can provide additional context
during an assessment.

The goal is to make an initial skin check more accessible through a
simple interface and an image-based machine learning model. LuminaDerm
is an informational tool, not a substitute for professional medical
advice or diagnosis.

## 🎯 Overview

Skin concerns can look similar, and an image alone cannot tell the full
story. LuminaDerm provides an image-based prediction and, in the planned
assessment flow, collects symptom information separately so users can
review it alongside the model's output.

**Project approach:** - Upload a skin image and receive a model
prediction - Keep image-based predictions separate from user-reported
symptoms - Present model scores carefully rather than treating them as
medical probabilities - Make the assessment flow straightforward on
mobile and desktop - Communicate the limitations of AI-based skin
assessment clearly

## 🧠 How It Works

### Image-based prediction

The backend uses a ResNet18 image classification model to analyze an
uploaded image and return predicted skin-condition results.

The main prediction endpoint is:

`POST /predict`

The image is sent as `multipart/form-data` using the field name `file`.

### Symptom questionnaire

The planned two-stage assessment adds a short questionnaire after the
image prediction. It asks about when the concern started, itching, pain
or burning, changes in size, visible discharge or crusting, location,
whether more spots are appearing, and whether the user has experienced
it before.

These answers are shown as user-reported context. They are **not sent to
the existing `/predict` endpoint** and are not combined with the image
model's score.

### Experimental multimodal model

The backend also includes an experimental `/predict-multimodal` endpoint
that was developed using image and symptom/history features. It is
separate from the main image-only flow and is not required for the
standard assessment described above.

## 🏗️ Architecture

LuminaDerm is built around a frontend assessment experience and a
FastAPI backend.

``` text
LuminaDerm
├── Frontend
│   ├── Skin image upload and preview
│   ├── Image prediction request
│   ├── Symptom questionnaire
│   └── Results and symptom summary
│
└── FastAPI backend
    ├── POST /predict
    └── POST /predict-multimodal (experimental)
```

The frontend is developed in Lovable. The backend is hosted on Render.

## 🚀 Try the API

### Hosted backend

-   **API base URL:** https://skin-ai-backend-1myc.onrender.com
-   **Interactive API documentation:**
    https://skin-ai-backend-1myc.onrender.com/docs
-   **Backend repository:**
    https://github.com/pritam-debnath-dev/Luminaderm-backend

### Predict from an image

Send an image to the `/predict` endpoint as multipart form data. The
uploaded file must use the field name `file`.

Example using `curl`:

``` bash
curl -X POST "https://skin-ai-backend-1myc.onrender.com/predict" \
  -F "file=@path/to/skin-image.jpg"
```

Replace `path/to/skin-image.jpg` with the path to an image on your
machine.

Check the interactive API documentation for the current request and
response schemas.

## 🧪 Model and Data

The primary image model is based on **ResNet18**. Its class labels are
defined by the model's class-name configuration, and its trained weights
are loaded by the backend.

An experimental multimodal model was trained using the SCIN dataset. The
experiment combines image features with symptom and history features and
covers a selected set of 20 skin-condition classes. It is a separate
model from the main image-only predictor.

The experimental checkpoint is available here:

[Download the multimodal
checkpoint](https://github.com/pritam-debnath-dev/Luminaderm-backend/releases/download/v1.0.0/luminaderm_multimodal.pt)

## 📋 Supported Prediction Classes

The experimental multimodal model includes these 20 classes:

-   Eczema
-   Allergic Contact Dermatitis
-   Insect Bite
-   Urticaria
-   Psoriasis
-   Folliculitis
-   Irritant Contact Dermatitis
-   Tinea
-   Herpes Simplex
-   Drug Rash
-   Herpes Zoster
-   Acute dermatitis, NOS
-   Impetigo
-   Hypersensitivity
-   Pigmented purpuric eruption
-   Leukocytoclastic Vasculitis
-   Acne
-   Lichen planus/lichenoid eruption
-   Viral Exanthem
-   CD - Contact dermatitis

This list describes the experimental multimodal model's classes. The
classes available from the image-only endpoint may differ. Vitiligo is
not included in the experimental model's current class list.

## 🛠️ Development Journey

LuminaDerm has been developed in stages, from building the image-based
prediction service to exploring how symptom information could be
presented alongside its results.

### 1. Building the image-classification foundation

The backend uses a ResNet18 image-classification model to analyze an
uploaded skin image and return predicted condition results. The model's
labels and trained weights are loaded by the backend.

### 2. Developing the API

A FastAPI backend was created to make the model available through an
HTTP API. The main `POST /predict` endpoint accepts an image as
multipart form data, using the field name `file`.

### 3. Exploring image and symptom information together

A separate experimental multimodal model was developed using the SCIN
dataset. It combines image features with symptom/history features and
covers a selected set of 20 skin-condition classes. This experiment is
exposed through `/predict-multimodal`; it remains separate from the
standard image-only prediction flow.

### 4. Deploying the backend

The backend is hosted on Render, with interactive API documentation
available through FastAPI's `/docs` page. The hosted API allows the
frontend to send image-prediction requests to the backend.

### 5. Creating the frontend experience

The frontend is being developed in Lovable. The planned assessment flow
separates the steps: users first upload an image and receive the image
model's result, then complete a symptom questionnaire. The questionnaire
answers are presented as user-reported context rather than being sent to
`/predict` or combined with its score.

### 6. Documenting limitations and next steps

Because skin conditions can look similar, the project emphasizes that
predictions may be wrong and that model scores are not necessarily
calibrated probabilities. Future work includes evaluating performance
across varied skin tones and image conditions, improving uncertainty
communication, and further evaluating the experimental multimodal
approach before considering it for the main user flow.

## ⚠️ Important Limitations

LuminaDerm is a learning and development project. Its outputs should not
be used as a medical diagnosis.

-   **Predictions can be wrong.** Similar-looking skin conditions may be
    difficult to distinguish from an image.
-   **Scores are not necessarily calibrated probabilities.** A model
    score should not be interpreted as the probability that a person has
    a condition.
-   **Image-only assessment has limits.** Lighting, image quality, skin
    tone, body location, and other factors may affect results.
-   **The questionnaire is contextual.** In the standard flow, user
    answers are displayed separately and do not change the image model's
    prediction.
-   **The multimodal endpoint is experimental.** Its output should not
    be treated as clinically validated.
-   **Not a replacement for a clinician.** Consult a qualified
    healthcare professional for diagnosis or treatment, especially if a
    skin concern is worsening, spreading, painful, or causing
    significant concern.

## 🔮 Possible Future Improvements

-   Improve the image upload and assessment experience
-   Add clearer explanations of model scores and uncertainty
-   Evaluate performance across different skin tones, image conditions,
    and body locations
-   Validate models using suitable, representative datasets
-   Improve error handling and monitoring for the hosted API
-   Further evaluate the experimental multimodal approach before
    considering it for the main user flow

## 🙌 Project Note

LuminaDerm explores how computer vision can support an initial skin
assessment while keeping the limits of AI clear. The project is intended
to provide useful information and context---not to make a medical
decision for the user.

## Working URL
: https://derm-predict-ai.lovable.app
