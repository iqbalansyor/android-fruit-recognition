# Fruit Recognition

An Android app that uses machine learning to recognize fruits and vegetables from camera images.

## Preview

<img src="app/src/main/assets/image/preview.png" alt="App Preview" width="250" />

## Features

- Real-time fruit/vegetable recognition using device camera
- Supports 6 classes: Apple, Banana, Lemon, Onion, Potato, Watermelon
- On-device inference using TensorFlow Lite
- Built with Jetpack Compose

## Tech Stack

- **Language**: Kotlin
- **UI**: Jetpack Compose + Material 3
- **ML Framework**: TensorFlow Lite
- **Model Architecture**: MobileNetV2 (transfer learning)
- **Min SDK**: 24 (Android 7.0)

## Model Details

The classification model was trained using transfer learning with MobileNetV2 as the base model:

- **Input**: 150x150 RGB images
- **Output**: 6 class probabilities
- **Training accuracy**: ~97.8%
- **Dataset**: 482 images across 6 classes

- **Model file**: [FruitsClassifier.tflite](app/src/main/assets/model/FruitsClassifier.tflite)
- **Training notebook**: [fruit_recognition_model_creation.ipynb](app/src/main/assets/model/fruit_recognition_model_creation.ipynb)

## Building

```bash
# Debug build
./gradlew assembleDebug

# Release build (minified)
./gradlew assembleRelease
```

## Project Structure

```
app/src/main/
├── java/com/iqbalansyor/fruit_ai/
│   ├── MainActivity.kt           # Entry point
│   ├── FruitRecognitionScreen.kt # Main UI with camera
│   └── FruitClassifier.kt        # TFLite model inference
└── assets/model/
    ├── FruitsClassifier.tflite   # TFLite model
    └── fruit_recognition_model_creation.ipynb  # Training notebook
```

## How It Works

The app uses a simple pipeline to recognize fruits and vegetables from images:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Image Input    │───▶│  Preprocessing  │───▶│  TFLite Model   │───▶│    Result       │
│  (Camera/       │    │  - Resize to    │    │  - MobileNetV2  │    │  - Label        │
│   Gallery)      │    │    150x150      │    │  - 6 classes    │    │  - Confidence % │
│                 │    │  - Normalize    │    │                 │    │                 │
└─────────────────┘    │    RGB (0-1)    │    └─────────────────┘    └─────────────────┘
                       └─────────────────┘
```

### Step-by-Step Process

1. **Image Capture**: User takes a photo with the camera or selects an image from the gallery. The app uses Android's `ActivityResultContracts` to handle both input methods.

2. **Bitmap Conversion**: The captured/selected image is decoded into a `Bitmap` with `ARGB_8888` config to ensure consistent pixel format for processing.

3. **Preprocessing** (`FruitClassifier.kt`):
   - **Resize**: The bitmap is scaled to 150x150 pixels to match the model's expected input dimensions
   - **ByteBuffer Conversion**: Pixel data is extracted and converted to a `ByteBuffer`:
     - Extract RGB values from each pixel (Android stores as ARGB)
     - Normalize each channel to 0-1 range by dividing by 255
     - Store as float32 values in native byte order

4. **Inference**: The TensorFlow Lite interpreter runs the preprocessed data through the model. The model uses **transfer learning** from MobileNetV2 (pre-trained on ImageNet) with a custom classification head trained on a 6-class fruit/vegetable dataset. This approach leverages MobileNetV2's learned feature extraction while adapting the final layers to recognize the specific classes.

5. **Post-processing**: The class with the highest probability is selected as the prediction, and both the label and confidence percentage are displayed to the user.

### Key Components

| Component | File | Purpose |
|-----------|------|---------|
| UI Layer | `FruitRecognitionScreen.kt` | Jetpack Compose UI with camera/gallery buttons and result display |
| ML Classifier | `FruitClassifier.kt` | Loads TFLite model, handles preprocessing and inference |
| Model | `FruitsClassifier.tflite` | Transfer learning from MobileNetV2, fine-tuned on custom 6-class dataset |

## APK Size

<img src="app/src/main/assets/image/apk_size.png" alt="APK Size" width="500" />

## License

MIT