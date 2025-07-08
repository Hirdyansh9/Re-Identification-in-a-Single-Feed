# Detailed Report: Approach 2 - Advanced Classification & ID Management

This report details the methodology of the second, more advanced and robust approach, which was developed to address the limitations of the first.

---

## 1. Approach and Methodology

This approach improves on the foundational pipeline by introducing a more intelligent and robust system for both **initial classification** and **permanent ID assignment**. The core ideology is to separate the task of segmenting an entity (as Team 1, Team 2, GK, or Referee) from the task of re-identifying a specific player. The pipeline operates as follows:
1.  **Detect**: The pre-trained YOLOv11 model processes each frame to detect all entities and assign an initial class (`player`, `goalkeeper`, or `referee`).
2.  **Track**: The `supervision.ByteTrack` algorithm takes these detections and assigns a temporary `tracker_id` to each entity, following it across frames.
3.  **Color Segmentation**: To reduce misclassifications, an automatic color identification step was added. This system analyzes the initial frames to learn the distinct color profiles of teams, goalkeepers, and referees.
4.  **Assign Unique ID**: A centralized ID management system intercepts new trackers. If the entity is a player or goalkeeper, the system assigns it the next available permanent ID from a predefined, globally managed pool. This ID is then permanently associated with that `tracker_id`.
5.  **Re_Identify**:  A custom re-identification system is built on top of the tracker. When a tracked person is lost and a new one appears, the system attempts to allocate a centrally managed ID to maintain a persistent ID.

---

## 2. Techniques and Outcomes

### Core Components:

* **Automatic Color Profile Identification**: The script first processes the initial 100 frames to automatically determine the primary colors of all entities.
    * It uses **K-Means clustering** on the colors of detected `player` objects to find the two dominant team colors.
    * It calculates the average color of all detected `goalkeeper` and `referee` objects to create their respective color profiles.
* **Shoulder-to-Knee ROI for Classification**: Out of multiple body proportions, shoulder-to-knee worked the best to segment players into teams. To improve the accuracy of the color profiles, a specific Region of Interest (ROI) covering the player's torso and shorts (from ~15% to 85% of the bounding box height) is used. This minimizes interference from the pitch.
* **Unique ID Management**: This is a key improvement over Approach 1.
    * The system uses central sets (`assigned_team1_ids`, `assigned_team2_ids`) to maintain a persistent record of all IDs ever assigned.
    * A dedicated function, `assign_new_player_id`, finds the first available ID within the correct numerical range (1-10 for Team 1, 11 for GK1, etc.), ensuring no duplicates are ever created.
* **Full-Body Re-Identification**: While classification relies on the kit color, the re-identification of lost players uses an HSV histogram of the **entire bounding box**. This provides a richer feature set for matching players who reappear in the frame.

### Outcomes:

* The system successfully assigns permanent, unique IDs to players and goalkeepers according to the 1-22 numbering scheme.
* The advanced versions demonstrated a significant reduction in misclassifications between teams and referees compared to the baseline approach.
* Performance remained high, with the final version reporting **~15.5 FPS**.

---

## 3. Challenges and Future Work

### Challenges Encountered:

* **Label Stability**: The final code's reliance on separate, pre-initialized annotators and a direct mapping system suggests that earlier versions may have faced challenges with annotation consistency or library versioning errors. The current structure is a robust solution to such issues.

* **Color Ambiguity**: Initial attempts still struggled with misclassifications in complex scenes. This led to the development of the more advanced, confidence-based logic with temporal smoothing, which proved to be more robust.
  
* **Low Confidence Undeterminism**: While confidence-driven logic provided for a more stable and robust re-indentification but in several cases, such as ours (re-identification in a short span and a dynamic environment), also led to the model not being able to allocate player IDs rather allocating the unclassified ID ('?').


### Incompleteness and Future Work:

The final script is a clean and effective implementation of the given problem, yet not optimal. However, the following steps would be next:

* **Deep Learning Re-ID Features**: For the highest accuracy, replace the color histogram feature with a deep learning-based embedding. This would involve using a pre-trained Re-ID model (or training a custom one on a relevant dataset) to extract a feature vector for each player that is highly discriminative and robust to viewpoint and lighting changes.
* **Integrate Motion Cues**: Enhance the tracker's cost matrix by combining the appearance score (from the Re-ID model) with motion information. Using filters like the Kalman Filter to predict a player's next location would help resolve ambiguities, especially when multiple players with similar appearances are close to each other.
