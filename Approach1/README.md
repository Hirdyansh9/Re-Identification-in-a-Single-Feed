# Detailed Report: Approach 1 - Foundational Re-Identification

This report details the methodology, techniques, and challenges of the first approach to player re-identification.

---

## 1. Approach and Methodology

The first approach implements a general pipeline for detecting, tracking and re-identifying players in a video. The core methodology is as follows:
1.  **Detect**:  A pre-trained YOLOv11 model processes each frame to detect the classes (ball, players, goalkeepers, and referees).
2.  **Track**: The resulting detections are passed to the **ByteTrack** algorithm, which assigns a temporary `tracker_id` and follows each person from frame to frame.
3.  **Re-identify**: A custom re-identification system is built on top of the tracker. When a tracked person is lost and a new one appears, the system attempts to match them based on visual similarity to maintain a persistent ID.

---

## 2. Techniques and Outcomes

### Core Components:

* **Object Detection**: A pre-trained **YOLOv11 model** is used for initial detection and is capable of detecting the `ball`, `goalkeeper`, `player`, and `referee` classes.
* **Tracking**: The `supervision.ByteTrack` library is used for its efficiency and robustness in handling short-term occlusions.
* **Re-Identification**: This is the key custom component and involves several steps:
    * **Appearance Feature**: An **HSV (Hue, Saturation, Value) Color Histogram** is calculated for the entire bounding box of each detected player. This histogram serves as the primary feature vector for representing a player's appearance.
    * **Feature Adaptation**: To account for changes in lighting or player orientation, the stored feature histogram for each player is updated using a **moving average**. This allows the player's profile to adapt over time.
    * **Matching Lost and New Players**: When the tracker loses a target and detects a new one, a matching process is initiated. The **Hungarian algorithm** (`scipy.optimize.linear_sum_assignment`) is used to find the most likely one-to-one matches between the set of newly appeared players and the set of recently lost players. The matching is based on a cost matrix where the cost is inversely proportional to the **correlation similarity** of their HSV histograms.
    * **ID Assignment**: If a match is found with a similarity score above a threshold (0.7), the new tracker is assigned the permanent ID of the lost player. If no suitable match is found, the new tracker is assigned a new, incremental ID.

### Outcomes:

* The system somewhat tracks players and re-assigns their IDs when they reappear in the frame.
* The final processed video demonstrates the functionality of the pipeline.
* Performance was measured at **~15.4 FPS**, which is suitable for near real-time analysis.

---

## 3. Challenges and Future Work

### Challenges Encountered:

* **Appearance Ambiguity**: The sole reliance on color histograms makes the system vulnerable to errors when players have similar jersey colors or when referees are dressed similarly to one of the teams. The model's initial `class_id` is trusted, which can lead to mislabeling (e.g., a referee being labeled as a player).
* **Simple ID System**: The use of a simple incremental counter (`next_player_id`) for new players does not guarantee adherence to a specific team-based numbering system (e.g., 1-11 for Team A, 12-22 for Team B).

### Incompleteness and Future Work:

This approach provides a proof-of-concept but is not fully robust. With more time and resources, the following improvements could be made:
* **Advanced Appearance Features**: Move beyond color histograms to more powerful deep learning-based appearance embeddings. A model like a Siamese network could be trained to produce feature vectors that are more resilient to changes in lighting, pose, and color similarity.
* **Multi-Cue Tracking**: Incorporate motion-based features (e.g., Kalman filters) and position information into the matching cost matrix alongside appearance similarity. This would improve re-identification accuracy, especially for players who are momentarily occluded.
* **More Sophisticated ID Management**: Implement a stateful system that tracks assigned IDs in distinct pools to enforce the desired 1-22 numbering scheme for players and goalkeepers.
