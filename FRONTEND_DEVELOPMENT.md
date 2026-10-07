# LuminaDerm Frontend — How It Was Built

This document explains how the LuminaDerm frontend was designed, built, connected to the AI backend, and developed with Lovable.

The goal is that someone looking at the GitHub repository can understand not only **what the frontend contains**, but also **how the frontend communicates with the AI model and backend**.

---

## 1. What is LuminaDerm?

LuminaDerm is an experimental AI-assisted skin assessment application.

The project combines:

- A web frontend for image upload and user interaction
- A FastAPI backend
- A PyTorch image-classification model
- A second-stage symptom/history questionnaire
- A results interface that keeps AI output separate from user-reported information

The current production flow uses the image model for the actual prediction.

The questionnaire is currently collected as additional context and is **not yet fed into the image prediction endpoint**.

This distinction is intentional so the application does not falsely claim that symptoms changed the AI prediction when the current backend does not support that functionality.

---

# 2. Frontend Technology

The frontend is built as a modern web application using:

- React
- Vite
- JavaScript / JSX
- CSS
- Fetch API
- Responsive design for desktop and mobile browsers

The frontend is responsible for the user interface and communication with the backend.

It does **not** contain the PyTorch model itself.

The model remains on the backend.

### Basic architecture

```text
                    LUMINADERM

             ┌─────────────────────┐
             │       Frontend      │
             │   React + Vite      │
             └──────────┬──────────┘
                        │
                        │ HTTPS
                        ▼
             ┌─────────────────────┐
             │      FastAPI        │
             │      Backend        │
             └──────────┬──────────┘
                        │
                        ▼
             ┌─────────────────────┐
             │    PyTorch Model    │
             │   Image Prediction  │
             └──────────┬──────────┘
                        │
                        ▼
                 Prediction JSON
                        │
                        ▼
             ┌─────────────────────┐
             │      Frontend       │
             │   Results Screen    │
             └─────────────────────┘
```

---

# 3. Role of Lovable

Lovable was used as a development tool to help design and build the LuminaDerm frontend.

It was especially useful for:

- Creating the initial user interface
- Designing the upload experience
- Building the questionnaire screens
- Creating mobile-friendly layouts
- Iterating on the visual design
- Connecting frontend actions to the backend API
- Refining the user flow

Lovable was **not the AI model** and it does not perform the skin-condition prediction.

The actual prediction is performed by the FastAPI/PyTorch backend.

A simplified division of responsibilities is:

```text
Lovable
   │
   ├── UI design
   ├── React frontend development
   ├── User flow
   ├── Components
   └── API integration
          │
          ▼
FastAPI Backend
   │
   ├── Receives image
   ├── Preprocesses image
   ├── Loads PyTorch model
   └── Returns prediction
```

After developing the frontend with Lovable, the frontend code can be kept in GitHub as part of the main project repository.

This makes the project reproducible and allows other developers to inspect and continue development without depending on Lovable.

---

# 4. The Frontend User Journey

LuminaDerm uses a two-stage assessment experience.

## Stage 1 — Image

The user first uploads a skin image.

The frontend:

1. Opens the file selector.
2. Validates that an image was selected.
3. Displays a preview.
4. Sends the image to the backend.
5. Waits for the AI response.
6. Stores the prediction.
7. Moves the user to the questionnaire.

The request is sent as:

```text
POST /predict
```

using:

```text
multipart/form-data
```

with the field:

```text
file
```

The current backend URL is:

```text
https://skin-ai-backend-1myc.onrender.com
```

Therefore the complete endpoint is:

```text
https://skin-ai-backend-1myc.onrender.com/predict
```

---

# 5. How the Image is Sent to the Backend

The frontend uses `FormData`.

Conceptually, the API integration works like this:

```javascript
const formData = new FormData();

formData.append("file", imageFile);

const response = await fetch(
  `${API_BASE_URL}/predict`,
  {
    method: "POST",
    body: formData
  }
);

const result = await response.json();
```

The browser sends the actual image file to FastAPI.

The frontend does not need to know how the PyTorch model works internally.

It only needs to understand the API request and response.

---

# 6. Backend Response

The current image-only backend returns a prediction containing information such as:

```json
{
  "prediction": "Eczema",
  "confidence": 0.82
}
```

The frontend reads that response and displays the result to the user.

Depending on the current backend version, the response can also contain other prediction fields.

The frontend therefore keeps the API communication in a separate file rather than mixing networking code throughout the UI.

---

# 7. Why the API Code is Separate

The frontend contains an API helper:

```text
src/
└── lib/
    └── api.js
```

This keeps backend communication separate from the visual components.

For example:

```text
App.jsx
    │
    └── predictImage(file)
             │
             ▼
          api.js
             │
             ▼
       FastAPI /predict
```

This makes the project easier to maintain.

If the backend URL changes later, it does not require rewriting the whole interface.

---

# 8. Backend URL Configuration

The frontend supports an environment variable:

```text
VITE_API_BASE_URL
```

Example:

```env
VITE_API_BASE_URL=https://skin-ai-backend-1myc.onrender.com
```

If the environment variable is not provided, the frontend uses the current Render backend as its default.

This is useful because development and production can use different backend URLs.

For example:

```text
Local development
        ↓
http://localhost:8000

Production
        ↓
https://skin-ai-backend-1myc.onrender.com
```

---

# 9. Stage 2 — Symptoms and History

After the image prediction is received, the frontend moves to the questionnaire.

The questionnaire contains eight questions.

### Questions

1. When did you first notice the skin problem?
2. Is the affected area itchy?
3. Does the affected area burn or hurt when touched?
4. Has the affected area changed in size?
5. Is there any fluid, pus, bleeding, or crusting?
6. Where is the affected area located?
7. Have you noticed more spots appearing?
8. Have you experienced this type of skin problem before?

The interface shows one question at a time.

---

# 10. Questionnaire UX

The questionnaire was designed to be simple enough for a mobile user.

Features include:

- One question per screen
- Progress indicator
- Back button
- Next button
- Required answers
- Preserved answers when going backward
- Image thumbnail for reference
- Mobile-friendly layout
- Final results screen

Example:

```text
Question 3 of 8
━━━━━━━━━━━━━━━━━━

Does the affected area burn
or hurt when touched?

○ No
○ Mildly
○ Moderately
○ Severely
○ Not sure

        Back       Next
```

---

# 11. Important Model Limitation

The questionnaire is currently **not part of the AI prediction**.

This is an important technical distinction.

The current backend endpoint is:

```text
/predict
```

and it accepts the image.

The questionnaire is handled by the frontend only.

Therefore:

```text
Image
  │
  ▼
AI Model
  │
  ▼
Prediction
```

while:

```text
Questionnaire
  │
  ▼
User-reported context
```

These two outputs are currently kept separate.

The application must not say:

> "Your symptoms caused the AI to predict Eczema."

That would be incorrect.

Instead, the application explains that the symptoms provide additional context for the user but are not currently incorporated into the image model's prediction.

---

# 12. Results Screen

After the questionnaire is completed, the frontend displays three main sections.

## Image AI Result

Shows:

- Predicted condition
- Confidence/score when supplied by the backend

Example:

```text
IMAGE AI RESULT

Eczema

Confidence: 82%
```

The interface also explains that the model output is experimental and should not be interpreted as a medical diagnosis.

---

## User-Reported Symptoms

The user's answers are displayed separately.

Example:

```text
USER-REPORTED SYMPTOMS

First noticed: A few days ago
Itching: Moderately
Pain/burning: Mildly
Size: Getting larger
Fluid/pus/bleeding: None
Location: Arms
More spots: No
Previous similar problem: Yes
```

---

## Interpretation

The frontend explains that:

- The questionnaire provides additional context.
- The current image model did not use those answers for prediction.
- The questionnaire does not confirm or rule out a medical condition.
- LuminaDerm is not a replacement for a healthcare professional.

---

# 13. Frontend Project Structure

The frontend is organized approximately like this:

```text
frontend/
│
├── index.html
├── package.json
├── vite.config.js
├── .env.example
├── .gitignore
├── README.md
│
└── src/
    │
    ├── main.jsx
    ├── App.jsx
    ├── styles.css
    │
    └── lib/
        └── api.js
```

### `index.html`

The main HTML entry point used by Vite.

### `main.jsx`

Starts the React application and renders the main application component.

### `App.jsx`

Contains the main LuminaDerm user flow:

- Image upload
- Image prediction
- Questionnaire
- Answer storage
- Results
- Restart flow

### `styles.css`

Contains the frontend styling and responsive/mobile layout.

### `api.js`

Contains the connection between the frontend and FastAPI backend.

### `.env.example`

Documents the backend URL configuration without exposing secrets.

---

# 14. Why React State is Used

The frontend needs to remember several pieces of information while the user moves through the assessment.

For example:

```text
file
preview
prediction
questionIndex
answers
loading
error
```

React state is used to keep these values available while the user interacts with the application.

For example:

```javascript
const [answers, setAnswers] = useState({});
```

When the user selects an answer:

```javascript
setAnswers(current => ({
  ...current,
  [question.key]: value
}));
```

This is what allows the answers to remain available when the user presses Back.

---

# 15. Image Preview

The selected image is displayed before analysis.

A browser object URL is used for the local preview.

Conceptually:

```javascript
const previewUrl = URL.createObjectURL(file);
```

This allows the user to see the selected image without first uploading it to another image-hosting service.

The same preview is shown as a smaller reference image during the questionnaire.

---

# 16. Error Handling

The frontend also handles common failures.

Examples include:

- No image selected
- Backend unavailable
- HTTP errors
- Invalid backend responses
- Prediction request failures

Instead of silently failing, the application displays an error message to the user.

This is particularly important because the backend is hosted on a free Render instance and may occasionally take time to wake up.

---

# 17. Deployment Architecture

The final project can be deployed as two connected services.

```text
                User's Browser
                       │
                       ▼
              ┌─────────────────┐
              │ LuminaDerm       │
              │ Frontend         │
              │ React / Vite     │
              └────────┬────────┘
                       │
                       │ HTTPS
                       ▼
              ┌─────────────────┐
              │ Render          │
              │ FastAPI         │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ PyTorch Model   │
              └─────────────────┘
```

The frontend can be hosted on a static frontend service such as Vercel, Netlify, GitHub Pages where compatible, or another static hosting provider.

The backend remains on Render.

---

# 18. GitHub Repository Structure

The recommended final repository structure is:

```text
Luminaderm-backend/
│
├── frontend/
│   ├── src/
│   ├── package.json
│   ├── index.html
│   └── ...
│
├── app.py
├── requirements.txt
├── class_names.json
├── skin_model.pth
├── luminaderm_multimodal.pt
└── README.md
```

This allows a visitor to see both sides of the project:

```text
Frontend
   +
Backend
   +
AI Models
   =
LuminaDerm
```

---

# 19. Development Process

The frontend was developed iteratively rather than being created as a single finished page.

The general process was:

```text
Idea
  ↓
Initial UI
  ↓
Image upload
  ↓
Backend connection
  ↓
Image prediction
  ↓
Questionnaire design
  ↓
Results interface
  ↓
Mobile improvements
  ↓
Backend/API testing
  ↓
GitHub organization
  ↓
Deployment
```

Lovable was particularly useful during the UI development and iteration stage.

The frontend was then organized as a normal project so that its source code can be inspected and maintained independently.

---

# 20. Why the Frontend and Backend Are Separate

Keeping the frontend and backend separate provides several advantages.

### Frontend

Responsible for:

- User interface
- Image selection
- Questionnaire
- Results presentation
- User experience

### Backend

Responsible for:

- Receiving uploaded images
- Image preprocessing
- Loading the PyTorch model
- Running inference
- Returning prediction results

This separation also makes it easier to replace or upgrade the model without completely rebuilding the user interface.

---

# 21. Future Multimodal Integration

The project already contains work toward a multimodal LuminaDerm model.

The intended future architecture is:

```text
             Image
               │
               ▼
        Image Encoder
               │
               │
Symptoms ──► Symptom Encoder
               │
               ▼
          Feature Fusion
               │
               ▼
        Multimodal Model
               │
               ▼
        Condition Scores
```

In that version, the frontend questionnaire could send structured symptom information to a dedicated multimodal backend endpoint such as:

```text
/predict-multimodal
```

The backend would then use both:

- Image features
- Symptom/history features

for the model's prediction.

Until that integration is officially enabled and tested, the frontend should continue treating questionnaire answers as separate user-reported context.

---

# 22. AI Model Transparency

LuminaDerm should be presented as an experimental AI project.

The frontend therefore avoids presenting the prediction as a confirmed diagnosis.

A model prediction such as:

```text
Eczema — 82%
```

should be understood as:

```text
The model produced this output for the uploaded image.
```

It should **not** be interpreted as:

```text
You definitely have eczema.
```

This distinction is important when building an AI system for health-related use.

---

# 23. Medical Disclaimer

LuminaDerm is an experimental informational tool.

It does not provide a medical diagnosis and does not replace evaluation by a qualified healthcare professional.

Users should seek appropriate medical care for concerning, severe, rapidly changing, painful, infected, or otherwise unusual skin conditions.

---

# 24. Running the Frontend Locally

Install dependencies:

```bash
npm install
```

Start the development server:

```bash
npm run dev
```

Create a production build:

```bash
npm run build
```

Preview the production build:

```bash
npm run preview
```

The backend must also be running and accessible for image predictions.

---

# 25. Connecting a Different Backend

To use another backend, create a `.env` file:

```env
VITE_API_BASE_URL=https://your-backend-url.com
```

For the current LuminaDerm backend:

```env
VITE_API_BASE_URL=https://skin-ai-backend-1myc.onrender.com
```

The frontend then calls:

```text
POST {VITE_API_BASE_URL}/predict
```

---

# 26. Summary

The LuminaDerm frontend is more than a static questionnaire.

It is a client application that connects a user-friendly interface to a real FastAPI/PyTorch AI backend.

The current architecture is:

```text
                    LUMINADERM

        ┌─────────────────────────────┐
        │          FRONTEND           │
        │                             │
        │ React + Vite                │
        │                             │
        │ Image Upload                │
        │       ↓                     │
        │ Image Prediction Request   │
        │       ↓                     │
        │ Questionnaire               │
        │       ↓                     │
        │ Results                     │
        └──────────────┬──────────────┘
                       │
                       │ HTTPS
                       ▼
        ┌─────────────────────────────┐
        │          BACKEND            │
        │                             │
        │ FastAPI                     │
        │       ↓                     │
        │ Image preprocessing         │
        │       ↓                     │
        │ PyTorch model               │
        │       ↓                     │
        │ Prediction JSON             │
        └─────────────────────────────┘
```

Lovable helped accelerate the frontend design and development process, while React/Vite provides the actual frontend application and FastAPI/PyTorch handles the AI inference.

The project is intentionally structured so that a developer visiting the GitHub repository can understand how the interface, API, backend, and AI model fit together.

---

## Project Status

Current capabilities:

- Image upload
- Image preview
- FastAPI `/predict` integration
- AI image prediction display
- Eight-step symptom/history questionnaire
- Back/Next navigation
- Mobile-friendly interface
- User-reported symptom summary
- AI result and symptom context kept separate
- Medical disclaimer
- Environment-based backend URL configuration

Future work can connect the questionnaire to the multimodal model once the multimodal API is ready and tested.
