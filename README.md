# Denso Robot + Haption Integration (ROS 2 Humble)

This document contains setup and run instructions for integrating a **Denso VP6242M robot** with a **Haption 6D Desktop device** via **ROS 2 Humble**. Follow each section in order for full initialization and testing.

---

##  System Requirements

- Ubuntu 22.04 (Recommended)
- ROS 2 Humble
- Denso Robot Drivers + SDK
- Haption SDK/API (v4.60)
- Proper USB/Network permissions

---

## Terminal 1: Denso Robot Bring-Up

```bash
sudo bash
cd ~/denso_workspace/src
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 launch denso_robot_bringup denso_robot_bringup.launch.py model:=vp6242m
---
##  Terminal 2: Haption Device Service

```bash
cd ~/haption_workspace/Haption
sudo ./calibrate_Desktop_6D_n41
sudo ./SvcHaptic_Desktop_6D_n41
sudo tail -n 50 /var/log/SvcHaptic_Desktop_6D_n41.log # check log if haption device is port binded, if blocked continue next process and kill the bind port
sudo lsof -i :5000
sudo kill -9 <PID>
---

##  Terminal 3: Virtuose Demo Tool

```bash
export LD_LIBRARY_PATH=.
./virtuoseDemo -mv 127.0.0.1 v6dv4 1.0

export VIRTUOSE_API_PATH=~/haption_workspace/VirtuoseDemo_v4.60_EN/Linux
./virtuoseDemo -mv localhost v6d35-45

##  Terminal 4: Haption Main App

\`\`\`bash
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH
sudo LD_LIBRARY_PATH=$LD_LIBRARY_PATH ./main
\`\`\`

---

##  Terminal 5: Haption ROS 2 Node

\`\`\`bash
cd ~/haption_workspace/Haption
sudo bash
source /opt/ros/humble/setup.bash
source install/setup.bash
export LD_LIBRARY_PATH=~/haption_workspace/Haption/haption_api/lib
ros2 run haption_ros2 haption_node
\`\`\`

---

##  Terminal 6: Echo Haption Pose Data

\`\`\`bash
sudo bash
colcon build --packages-select haption_ros2 --cmake-clean-cache
source /opt/ros/humble/setup.bash
ros2 topic echo --qos-durability transient_local /haption/pose
\`\`\`

---

##  Terminal 7: Denso-Haption Bridge (Trajectory)

\`\`\`bash
cd ~/denso_workspace/src
sudo bash
source /opt/ros/humble/setup.bash
source install/setup.bash
colcon build --packages-select haption_denso_bridge
ros2 run haption_denso_bridge haption_to_joint_trajectory
\`\`\`

---

##  Terminal 8: Load & Spawn Denso Controllers

\`\`\`bash
ros2 control load_controller --set-state active denso_joint_trajectory_controller
ros2 control list_controllers

ros2 run controller_manager spawner denso_joint_state_broadcaster --controller-manager /controller_manager
ros2 run controller_manager spawner denso_joint_trajectory_controller --controller-manager /controller_manager
ros2 control list_controllers
\`\`\`

---

##  Optional: Run with Kalman Filter

\`\`\`bash
ros2 run haption_denso_bridge haption_to_joint_trajectory --ros-args -p filter_mode:=Kalman
\`\`\`

---

##  Terminal 9: Visualization & Analysis

\`\`\`bash
python3 live.py
python3 analyze.py
\`\`\`

---

##  Recommended Directory Structure

\`\`\`
~/denso_workspace/
    └── src/
        ├── denso_robot_bringup/
        └── haption_denso_bridge/

~/haption_workspace/
    ├── Haption/
    └── VirtuoseDemo_v4.60_EN/
\`\`\`

---

##  Notes

- Always source ROS & workspace setups in new terminals.
- Run \`sudo\` only where necessary (device/service operations).
- Ensure USB/network access is granted to your user.
- Use log files for debugging (\`/var/log/SvcHaptic_Desktop_6D_n41.log\`).

---

##  Author

**Yogeshwar Radhakrishna**  
📧 \`yogeshwar@Yogeshwar\`  
🏢 Project: Denso + Haption ROS 2 Integration  
📅 Updated: August 2025  

---







