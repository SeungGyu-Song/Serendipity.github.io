---
title: <Fill me w/ a compact title>
draft: true
tags:
---

| Name            | Sensors                             | Environment          | Platform | Web Link                                                                   | ROS   | Properties                       | Paper                                                                 |
| --------------- | ----------------------------------- | -------------------- | -------- | -------------------------------------------------------------------------- | ----- | -------------------------------- | --------------------------------------------------------------------- |
| EuRoC           |                                     |                      |          |                                                                            |       |                                  |                                                                       |
| UrbanNavDataset | Stereo/ Velodyne LiDAR/ Event/ GNSS | City(Honkong, Tokyo) | Car      | [Link](https://github.com/IPNL-POLYU/UrbanNavDataset?tab=readme-ov-file)   | Yes   |                                  |                                                                       |
| TUM-VIE         | Stereo Event/STtereoCam/IMU         | Indoor               | Handheld | [Link](https://cvg.cit.tum.de/data/datasets/visual-inertial-event-dataset) | No :( | HDR, corridor, slide, bike-night | [Paper](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9636728) |

---
TUM-VIE는 ROS bag이 아닌데 Event의 경우 추천을 하지는 않음. 
그래도 rosbag으로 만들고싶으면 [링크](https://github.com/tub-rip/events_h52bag)참고.
그리고 다운 받은 데이터를 활용할 수 있도록 [python코드](https://tumevent-vi.vision.in.tum.de/access_visualize_h5.py)도 제공