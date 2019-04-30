# Module 4: LIDAR Sensing

LIDAR (light detection and ranging) sensing is an enabling technology for self-driving vehicles. LIDAR sensors can ‘see’ farther than cameras and are able to provide accurate range information. This module develops a basic LIDAR sensor model and explores how LIDAR data can be used to produce point clouds (collections of 3D points in a specific reference frame). Learners will examine ways in which two LIDAR point clouds can be registered, or aligned, in order to determine how the pose of the vehicle has changed with time (i.e., the transformation between two local reference frames).

### 学习目标

- Explain how LIDAR sensors are able to generate range data from time-of-flight measurements
- Understand inverse and forward sensor models for a basic scanning LIDAR
- Apply standard transfomations (translation, rotation, and scaling) to LIDAR point cloud data
- Summarize how the problem of motion distortion can affect LIDAR scans acquired from a moving vehicle
- Utilize the iterative closest point (ICP) algorithm to align point clouds from consecutive LIDAR scans

---

## Lesson 1: Light Detection and Ranging Sensors

