[README.md](https://github.com/user-attachments/files/35413040/README.md)
# SOL: Finding the Sun with OpenCV

A hands-on Google Colab workshop for Project SunbYte. Students build a classical computer-vision pipeline, one step at a time, that finds the sun in an image from a previous SunbYte mission and calculates its pixel coordinates, its radius, and its offset from the centre of the frame: the numbers a solar telescope needs to stay pointed at the sun.

---

# What the Project Is

- **What does this project do?**
  It's an interactive Jupyter notebook that takes students through a six-step image-processing pipeline: greyscale and blur, thresholding, mask cleaning, contour detection, a roundness check, and coordinate calculation. Each step has an explanation, a short "Try it" experiment and a coding challenge. By the end, the notebook outputs the sun's centre, its radius, and how far it is from the centre of the frame.

- **Who is it for?**
  Students with basic Python who are new to computer vision, including new members of the SunbYte software team. It's designed to run in three settings:
  1. University computer labs on provided computers
  2. A classroom, on students' own laptops
  3. Remotely, alongside a pre-recorded demonstration

- **What problem does it solve?**
  Sun detection is the core of SunbYte's tracking system, but most introductions to OpenCV are either too abstract or jump straight to machine learning. This workshop teaches the fundamentals (image representation, thresholding, morphology, contours and coordinate systems) through a real problem the team actually has to solve, using real mission imagery.

- **What makes it interesting or different?**
  - It's built around a real engineering problem rather than a toy example. The pipeline mirrors the approach used on SunbYte 2 and 3.
  - Students write the key lines themselves (area, perimeter, circularity, fill ratio, centre offsets) instead of running finished code.
  - Each step shows its result visually, for example the brightness histogram with Otsu's cut-off marked, so students can see *why* each step matters.
  - It runs entirely in the browser through Google Colab, with nothing to install.

---

# How It Works

- **What are the main components of the system?**

  | Step | Component | What happens | Why |
  |---|---|---|---|
  | Setup | Imports + `show()` helper | Loads the libraries and `sun.jpg`; defines a helper that displays images side by side | Notebooks can't use `cv2.imshow`, and OpenCV stores colour as BGR |
  | 1 | Greyscale + blur | Converts to 1 brightness value per pixel, then applies a Gaussian blur | Brightness is all we need; blurring hides noise and sunspots |
  | 2 | Threshold | Turns the image into a black-and-white mask, using Otsu's method or a fixed value | Splits the image into "bright" and "not bright" |
  | 3 | Clean the mask | Morphological opening (removes specks) then closing (fills holes) | Gives one clean, solid disc |
  | 4 | Find contours | Outlines every white region and measures its area, perimeter and minimum enclosing circle | Turns bright pixels into candidate shapes |
  | 5 | Roundness check | Scores each candidate on circularity and fill ratio; the largest one that passes is the sun | Rejects clouds, reflections and lens flare |
  | 6 | Coordinates | Calculates the sun's centre and its offset from the frame centre, then draws and saves the result | Gives the telescope its pointing error |

- **How do the pieces interact?**
  Each step is a notebook cell whose output becomes the next step's input:

  ```
  sun.jpg → image → gray → blurred → mask_raw → mask_opened → mask
          → contours → candidates → sun (x, y, r) → dx, dy → sun_detected.png
  ```

  Because the cells share variables, they have to be run **in order**. Changing a parameter means re-running that cell and every cell after it.

- **What is the core logic or architecture?**
  - **Otsu's method** tries every threshold from 0 to 255 and picks the one that best separates the pixels into two groups (dark sky and bright sun).
  - **Circularity** = 4π × area / perimeter². A perfect circle scores 1.0; long, thin or jagged shapes score lower.
  - **Fill ratio** = area / πr², where r is the radius of the minimum enclosing circle. A solid disc scores close to 1.0; ovals and crescents score lower.
  - **Centre** = the centroid from image moments, (m10/m00, m01/m00): the average position of every pixel in the shape.
  - **Offset from the frame centre:** `dx = x − w/2` (positive = right) and `dy = h/2 − y` (positive = up). The *y* is flipped because image *y* increases downwards.

---

# Engineering Decisions

### 1. Classical computer vision instead of a machine-learning detector
- **Decision:** detect the sun with thresholding, morphology and shape checks.
- **Alternatives considered:** a pre-trained object detector (`cvlib` with YOLOv4).
- **Why:** the sun with a solar filter is a bright, round object on a dark background, which classical methods handle well. The YOLO model used by `cvlib` can only recognise 80 everyday object classes, and the sun isn't one of them. `cvlib` also needed TensorFlow and older `setuptools`, and its model format no longer loads in OpenCV 5. Classical methods are also fast enough for a Raspberry Pi, and every step can be explained and seen.
- **Limitations / tradeoffs:** the parameters have to be tuned by hand. Unusual conditions, such as a bright daytime sky, thick cloud or a partly hidden sun, can break individual steps where a trained model might cope.

### 2. Contour-based detection instead of the Hough Circle Transform
- **Decision:** find the sun from contours with circularity and fill-ratio checks.
- **Alternatives considered:** `cv2.HoughCircles`, which SunbYte 3 used to confirm the bright object was circular.
- **Why:** the contour method gives a sub-pixel centre (the centroid) and an accurate radius. Its two scores are simple formulas students can write themselves. It's also easier to see what each part is doing than with Hough's edge voting and its sensitive parameters.
- **Limitations / tradeoffs:** if the sun is partly covered by cloud or cut off by the edge of the frame, its outline stops being round and the contour method fails. Hough copes better because it uses edges. The workshop version has no Hough fallback.

### 3. Otsu's automatic threshold, with a fixed threshold as an option
- **Decision:** `THRESHOLD = None` uses Otsu's method; setting a number uses a fixed cut-off.
- **Alternatives considered:** a fixed, pre-calibrated threshold, which is what SunbYte's flight software used.
- **Why:** Otsu adapts to each image's brightness, so it works on images students haven't seen before. Keeping the fixed option lets students compare the two and see the effect directly on the histogram.
- **Limitations / tradeoffs:** Otsu assumes there are exactly two groups of pixels. With three (dark ground, bright sky, very bright sun) or a very small sun, it can put the cut-off in the wrong place.

### 4. Centroid instead of the enclosing-circle centre
- **Decision:** report the centre as the centroid from `cv2.moments`.
- **Alternatives considered:** the centre of `cv2.minEnclosingCircle`.
- **Why:** the centroid averages every pixel in the shape, so one bump on the outline (noise, or a sunspot on the edge) barely moves it. The enclosing circle depends only on the few outermost points.
- **Limitations / tradeoffs:** if part of the disc is missing (cut off by the frame or covered), the centroid shifts towards the visible part.

### 5. Google Colab notebook instead of local Python scripts
- **Decision:** deliver the workshop as a Colab notebook.
- **Alternatives considered:** `.py` scripts run locally in VS Code, or Jupyter installed on each machine.
- **Why:** nothing to install, it behaves the same in labs, on students' own laptops and remotely, and results appear inline under each step.
- **Limitations / tradeoffs:** it needs a Google account and an internet connection. Uploaded files (including `sun.jpg`) are deleted when the runtime restarts or times out, so students sometimes have to upload the image again.

### 6. Step-by-step cells with TODOs instead of one finished script
- **Decision:** split the pipeline into one cell per step, with the key lines left for students to write.
- **Alternatives considered:** a complete script that students run and read.
- **Why:** students learn more by writing the important lines and seeing each step's result straight away.
- **Limitations / tradeoffs:** cells depend on each other. An unfinished TODO (for example `circularity` in Step 5) causes a `NameError`, and running cells out of order fails too, so demonstrators need to help students who get stuck.

# Tech Stack

- **Languages used:** Python 3.10+
- **Frameworks/libraries:**
  - OpenCV (`opencv-python`, tested on 5.0) for image processing
  - NumPy for arrays
  - Matplotlib for displaying images and histograms
  - `math` (standard library) for π and square roots
- **Tools and platforms:**
  - Google Colab (main platform)
  - Jupyter / VS Code with the Jupyter extension (for running locally)
  - GitHub (for version control)

---

# What I Learned

- **What new skills did I gain?**
  - Building an image-processing pipeline with OpenCV: colour conversion, Gaussian blur, thresholding, morphology, contours and image moments.
  - Designing a workshop: breaking a working program into steps, writing challenges and explanations, and deciding which lines students should write themselves.
  - Debugging Python dependencies, such as `cvlib` needing `pkg_resources` and TensorFlow, and OpenCV 5 dropping the Darknet model loader.

- **What concepts became clearer through building this?**
  - How Otsu's method chooses a threshold, and how to read a brightness histogram (everything right of the red line becomes white in the mask).
  - Why blur kernels have to be odd, and why blurring comes before thresholding.
  - The difference between image coordinates (origin top-left, *y* down) and the centre-based coordinates a telescope needs.
  - How shape measurements like circularity and fill ratio can tell a sun apart from clouds or flare.

- **What surprised me during development?**
  - `cv2.imread` doesn't raise an error when a file is missing. It returns `None`, and the crash only appears later in `cvtColor` as `!_src.empty()`.
  - A perfect circle drawn in pixels scores only about 0.9 for circularity, because its staircase outline makes the perimeter too long.
  - Files uploaded to Colab disappear when the runtime restarts.
  - A ready-made object detector couldn't find the sun at all, because "sun" isn't one of the things it was trained on.

---

# How to Run the Project

- **Prerequisites:**
  - A Google account (for Colab), *or* Python 3.10+ installed locally
  - The sun image `sun.jpg`

- **Installation steps:**
  - **Colab:** nothing to install. OpenCV, NumPy and Matplotlib come pre-installed.
  - **Locally:**
    ```bash
    python -m pip install opencv-python numpy matplotlib jupyter
    ```

- **How to start/run the project:**
  1. Open `SOL.ipynb` in Google Colab (**File → Upload notebook**) or in Jupyter / VS Code.
  2. Upload `sun.jpg`. In Colab, click the folder icon on the left and upload it to the main folder (`/content`), not into `sample_data`.
  3. Run the cells **in order** with **Shift + Enter**, reading the explanation above each one.
  4. Complete each `#TODO` before running its cell.
  5. Step 6 prints the coordinates and saves the annotated result as `sun_detected.png`.

  > If you see `error: (-215:Assertion failed) !_src.empty() in function 'cvtColor'`, the image didn't load. Check that `sun.jpg` is uploaded and spelled exactly right, then re-run the setup cell.

- **Example commands:**
  ```bash
  # Run locally in Jupyter
  jupyter notebook SOL.ipynb

  # Install a missing package from inside a notebook cell
  %pip install opencv-python numpy matplotlib
  ```

---

# Project Structure

```
SOL/
├── README.md           # This file
├── SOL.md              # Workshop plan and facilitator notes
├── SOL.ipynb           # The workshop notebook (setup + Steps 1–6)
├── sun.jpg             # Input image from a previous SunbYte mission
└── sun_detected.png    # Output: annotated result saved by Step 6 (generated)
```
