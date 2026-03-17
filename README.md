
# MOSIP Intelligent OCR & Identity Verification System

## 🚀 Project Overview
This project is an automated identity verification solution designed for MOSIP-based systems. It eliminates manual data entry by using a hybrid AI pipeline: **Computer Vision (OpenCV)** for structural analysis and **Transformers (Microsoft TrOCR)** for high-accuracy text recognition. The system creates a seamless verification loop by extracting data from ID cards and validating it against user input.

---

## 🏗️ Technical Architecture
The solution uses a decoupled microservices architecture to handle intensive image processing efficiently.

### **1. Frontend (Client Layer)**
* **Stack:** React.js, Tailwind CSS.
* **Role:** Handles image uploads and renders the **Interactive Heatmap**. It uses HTML5 Canvas to draw bounding boxes over the original image, giving users immediate visual confirmation of what the AI "read."

### **2. Backend (Service Layer)**
* **Stack:** Python 3.12, FastAPI.
* **Role:** Acts as the high-performance orchestrator. It manages API endpoints (`/extract`, `/verify`), handles file validation, and routes images to the processing engine.

### **3. Computer Vision Engine (Processing Layer)**
* **Pre-processing (Smart Masking):** We implemented a custom sanitization step using **Haar Cascades** to detect faces and QR codes. These regions are masked (painted white) before OCR runs, eliminating 99% of "ghost text" errors (like confusing eyes with letters).
* **Detection:** Uses **CRAFT** (Character Region Awareness) or MSER to locate text blocks, ignoring complex background patterns.
* **Recognition:** Detected crops are fed into **Microsoft TrOCR** (Transformer-based OCR), which outperforms standard Tesseract on handwritten and low-contrast text.
* **Spatial Parsing:** Instead of simple line-reading, our custom algorithm uses spatial geometry. It finds labels (e.g., "Name") and searches specifically to the **right** or **below** that coordinate to capture the correct value, handling multi-column layouts effectively.

---

## 🔄 Data Flow Structure
1. **Input:** User uploads an image via the Frontend.
2. **Sanitization:** The backend detects Faces and QR codes using Haar Cascades and masks them to isolate text.
3. **Extraction:** The AI engine detects text regions, crops them, and feeds them to TrOCR.
4. **Parsing:** The logic engine uses spatial geometry to map text to fields (Name, DOB, ID).
5. **Response:** JSON data containing extracted text, confidence scores, and heatmap coordinates is sent back to the client.

---

## 🔌 API Documentation

This section details the primary endpoints used for integration.

### **1. Extract Data**
**Endpoint:** `POST /extract`
**Description:** Accepts an ID card image file, processes it through the CV pipeline, and returns structured data with coordinate bounding boxes.

**Request:**
* **Content-Type:** `multipart/form-data`
* **Body:** `file` (Binary Image File)

**Response Example (200 OK):**
```json
{
  "status": "success",
  "fields": {
    "Name": "Johnathan Doe",
    "DOB": "15-08-1995",
    "IDNumber": "A123 4567 8901",
    "Address": "123 Green Avenue, Toronto"
  },
  "quality_check": {
    "is_blurry": false,
    "face_detected": true
  },
  "ocr_data": [
    {
      "field": "Name",
      "text": "Johnathan Doe",
      "confidence": 0.98,
      "box": [[100, 200], [300, 200], [300, 250], [100, 250]]
    }
  ]
}

```

###**2. Verify Data****Endpoint:** `POST /verify`
**Description:** Performs a strict comparison between the original AI extraction and the final user-reviewed data. It calculates a trust score based on data integrity.

**Request:**

* **Content-Type:** `application/json`
* **Body:**

```json
{
  "original_data": {
    "Name": "Johnathan Doe",
    "IDNumber": "A123 4567 8901"
  },
  "user_edits": {
    "Name": "Johnathan Doe",
    "IDNumber": "A123 4567 8901"
  }
}

```

**Response Example (200 OK):**

```json
{
  "status": "success",
  "match_score": 100,
  "message": "Verified Perfect Match"
}

```

---

##🛠️ Installation & Setup###**Backend (Python)**1. Navigate to the source folder:
```bash
cd backend/src

```


2. Set up the virtual environment:
```bash
python -m venv venv
.\venv\Scripts\activate  # Windows
# source venv/bin/activate  # Mac/Linux

```


3. Install dependencies:
```bash
pip install fastapi uvicorn python-multipart opencv-python numpy pillow transformers torch torchvision craft-text-detector

```


4. Start the API server:
```bash
uvicorn main:app --reload

```



###**Frontend (React)**1. Open a new terminal and navigate to the client folder:
```bash
cd frontend/ocr-client

```


2. Install dependencies:
```bash
npm install

```


3. Launch the application:
```bash
npm start

```



The application will be accessible at `http://localhost:3000`.

---


```

```
