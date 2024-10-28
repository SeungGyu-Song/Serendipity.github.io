
| Name                                      | Feature Extractor   | Feature Tracking | Backend         | CPU  | Preintegration | Feature표현         |
| ----------------------------------------- | ------------------- | ---------------- | --------------- | ---- | -------------- | ----------------- |
| [[🧩VINS-Fusion Map.canvas\|VINS-Fusion]] | GoodFeaturestoTrack | OpticalFlowLK    | Optimization    | 270% | MidPoint       | [[inverse depth]] |
| [[👑BASALT.canvas\|BASALT]]               | FAST                | own-made         | Optimization    | 70%  | Euler방식        |                   |
| [[🧩OpenVins.canvas\|OpenVINS]]           | FAST                | OpticalFlowLK    | Error-state EKF | 37%  | MidPoint       | 3D                |
|                                           |                     |                  |                 |      |                |                   |
|                                           |                     |                  |                 |      |                |                   |
BASALT ros wrapper에 뭐가 문제가 있나봄 (ROS로 돌리면 300% 이렇게 찍히던데)

openvins는 error state kf가 아닌가?