# General Report: Player Re-Identification Approaches

This report provides a summary and comparison of two of my approaches for the player re-identification task. Both solutions aim to detect, track, and assign unique, persistent IDs to players in a 15-second video clip using a custom-trained YOLOv11 model.

---

## Overview

**Approach 1** represents the foundational methodology. It establishes a complete pipeline using object detection, tracking, and a basic re-identification system. Although unoptimal, its primary strength lies in its simplicity and effectiveness in demonstrating the core concepts of re-id.

**Approach 2** is an iterative and more robust version of the first approach. It builds on the first approach by using techniques such as simple color features by introducing a more intelligent, multi-stage classification and ID management system. Although still not perfect, it improves a ton on the first approach.

---

## Core Technology Comparison

| Feature | Approach 1: Foundational Re-ID | Approach 2: Advanced Classification & ID Management |
| :--- | :--- | :--- |
| **Tracking** | `supervision.ByteTrack` | `supervision.ByteTrack` |
| **Classification** | Based directly on initial YOLO `class_id`. | **Initial:** Multi-stage classification using K-Means clustering on jersey/shorts color to distinguish teams, goalkeepers, and referees. <br> **Final:** Simplified to rely on initial `class_id`. |
| **Re-Identification Feature** | HSV Color Histogram of the entire bounding box. | **Classification:** Average color of the shoulder-to-knee area. <br> **Re-identification:** HSV Color Histogram of the entire bounding box. |
| **ID Assignment** | Incremental ID assignment (`next_player_id`). | Centralized, stateful ID management guaranteeing unique and persistent IDs within specified team ranges. |
| **Key Challenge Addressed** | Handling players leaving and re-entering the frame. | Differentiating between entities with similar appearances (e.g., teams, referees) and ensuring globally unique player IDs. |
| **Reported Performance** | ~15.4 FPS | ~15.5 FPS |

---

## Conclusion

While both approaches successfully fulfill the primary objective of Re-Identification-in-a-Single-Feed, **Approach 2** demonstrates a more sophisticated methodology. It leverages techniques such as automatic color profile identification and stateful, unique ID management, which represent a significant improvement over the foundational methods used in Approach 1.
The challenges encountered during its development were mainly the misclassifications, highlighting the necessity of moving beyond simple feature matching to more intelligent approaches for complex real-world scenarios.
