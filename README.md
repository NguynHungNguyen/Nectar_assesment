# Nectar Assessment Workflow

This is a ComfyUI workflow designed for generating and blending reference-based nude images using a multi-step pipeline. The workflow combines image generation, feature extraction, mask processing, and feature blending to create high-quality, customized images based on user prompts and reference images.

## Workflow Overview

The workflow operates in **two main steps**:

### Step 1: Upload Reference Images

* The user uploads one or more reference images which serve as the base for blending features into the generated samples.
* These reference images are processed to extract key features and generate corresponding masks.

### Step 2: Image Generation and Feature Blending Pipeline

#### Process 1: Nude Image Generation from Prompt

* The system generates a nude image sample based on a structured prompt that describes the scene and character details.

* **Prompt Structure:**

  ```
  [Overall scene][How the character looks overall][Character details][Scene details][Some conditions for the images]
  ```

* **Example Prompt:**

  ```
  A sensual realistic woman standing fully nude on the beach, softly backlit, arched back, subtle curves, alluring gaze, natural skin texture, warm ambient lighting, boudoir photography style, tasteful eroticism, high detail, photorealistic, cinematic shadows, realistic skin texture, no plastic skin, realistic, hd
  ```

* The generation uses:

  * **Hidream e1 model** for image synthesis
  * **Llama 3.1** as the CLIP text encoder for conditioning

#### Process 2: Feature Extraction and Blending

* After generating the sample image, the system extracts features from both the sample and the uploaded reference images.

* Masks are generated and preprocessed to prepare for feature blending.

* The blending uses:

  * **Flux fill fp8 model**
  * Two LoRA modules:

    * `comfyui_portrait_lora64`
    * Flux Turbo LoRA

* The prompt for this blending step controls the feature transfer, describing which attributes to retain or change.

* **Example blending prompt:**

  ```
  Retain [character eye colors]. Retain [character lips]. Change the hair color to [character hair].
  ```

---

## Models and Components Used

* **Image Generation:**
  Hidream e1 model (`hidream_i1_dev_uncensored_fp8_v0.2.safetensors`)
  CLIP text encoder: Llama 3.1 (`llama_3.1_8b_instruct_fp8_scaled.safetensors`)

* **Feature Blending:**
  Flux fill model (`fluxFillFP8_v10.safetensors`)
  LoRA modules: `comfyui_portrait_lora64.safetensors` and Flux Turbo LoRA

* **Masking and Segmentation:**
  Florence2-based segmentation models for mask generation and refinement.

* **Utilities:**
  Image resizing, concatenation, cropping, Gaussian blur masks, and stitching nodes are used to preprocess and improve output quality.

---

## How to Use

1. **Upload your reference image(s)** via the designated input node in the UI.
2. **Define your generation prompt** following the structured format described above.
3. **Set the blending prompt** to specify which features to retain or modify based on the reference.
4. Run the workflow to generate the sample image, extract features, apply blending, and produce the final output.
5. Preview and save your generated images.

---

## Limitations

* **Lack of skin features and body size diversity:**
  Due to time constraints, the training data lacks sufficient samples and datasets representing diverse skin colors, body sizes (e.g., fat, overweight), and other physical traits.

* **Automation of prompts:**
  Currently, some parts of the prompt must be manually crafted. In the future, automated extraction of features such as hair color, lip color, and eye color from images can be incorporated using models like Florence or BLIP to dynamically generate these prompt components.

* **Focus on nude characters:**
  This workflow concentrates primarily on nude character generation and feature blending. It is not optimized for detailed or diverse backgrounds.

---

## Future Work

* **Automatic Feature Extraction for Prompt Generation:**
  Implement models such as Florence, BLIP, or other state-of-the-art vision-language systems to automatically analyze uploaded reference images. This would enable the extraction of key visual attributes (e.g., hair color, eye color, skin tone, facial features) that can be programmatically converted into prompt components. Automating this step will reduce manual prompt engineering and improve consistency and accuracy in feature blending.

* **Data Mining and Fine-tuning for Greater Diversity:**
  Acquire and curate larger, more diverse datasets that better represent a wide range of human characteristics, including different skin tones, body types, ages, and ethnicities. Fine-tuning models on such datasets will enhance the realism and inclusivity of generated images. Additionally, developing domain-specific augmentations and synthetic data generation techniques could help overcome current data scarcity.

* **Enhancing Background Detail and Scene Context:**
  Expand the workflow to better integrate and optimize background generation alongside the character. This involves training or fine-tuning models to understand and produce complex, contextually rich environments that complement the character and add to the storytelling or visual appeal. Improving lighting, depth, and environmental effects will also contribute to more immersive scenes.

* **Improved Scene Composition and Realism:**
  Investigate advanced compositional techniques and multi-model pipelines that can dynamically adjust scene elements such as pose, lighting direction, shadows, and camera perspective. This will allow for more natural, photorealistic outputs that respect physical and artistic principles. Additionally, user controls and adaptive feedback loops could enable fine-grained tuning of scene dynamics.

* **Integration of Interactive and Real-time Controls:**
  Future iterations could incorporate user-friendly interfaces to interactively modify generated images or prompts in real time, enabling more creative flexibility. This might include sliders for physical attributes, color pickers for specific features, or semantic editing tools powered by the underlying models.

* **Robustness and Generalization:**
  Work on improving model robustness to handle edge cases, unusual poses, or rare attributes. This includes exploring few-shot learning and transfer learning methods to adapt quickly to new styles or demands with minimal additional training.

---

## Evaluation

### System Hardware and Setup

* The workflow was tested on an **NVIDIA RTX 4090** GPU with 24 GB VRAM.
* All models loaded into GPU memory consume approximately **14.7 GB** VRAM collectively, enabling faster inference.

### Speed Analysis

* **Average inference time per image when models are preloaded:**
  `μ_loaded = 54 ± 1` seconds (mean ± standard deviation over 20 runs)

* **Average inference time per image when models load from disk:**
  `μ_cold = 107.5 ± 3.0` seconds

* The time difference can be modeled as:
  `T_cold = T_loaded + T_load`, where `T_load ≈ 53.5 ± 2.5` seconds accounts for the I/O and initialization overhead.

* **Coefficient of Variation (CV)** for inference time (preloaded):
  `CV = (σ / μ) × 100% = (1 / 54) × 100% ≈ 1.85%`

  This indicates consistent performance with low variability.

### Prompt Adherence Evaluation

* Quantitative prompt adherence was evaluated on a test set of 30 images with varying complexity of actions.

* For **simple actions** (e.g., sitting, standing, swimming):

  * Success rate: 92% ± 4%
  * Mean user satisfaction rating: 4.3 / 5

* For **complex poses** (e.g., specific limb positions):

  * Success rate: 28% ± 6%
  * Mean user satisfaction rating: 2.1 / 5

### Character Consistency

* Character consistency was evaluated manually by comparing generated characters with the reference images in terms of **hair color**, **ethnic appearance**, and **facial structure**.

* We used count-based consistency across 20 generated samples for each sections:

  * **Hair color match rate:** 90% (18/20)
  * **European appearance match:** 100% accuracy (20/20 reference-preserved)
  * **Asian appearance match:** 95% accuracy (19/20 reference-preserved)

* Consistency is high within supported classes (Asian and European), but we observe a drop-off for characters with underrepresented traits or less common species due to data limitations.

### Image Quality Metrics

* The output images at 1024×1024 resolution are acceptable and visually pleasing. In the future, the workflow also supports upscaling to higher resolutions such as HD or 4K to enhance image quality, detail, and sharpness as needed.
