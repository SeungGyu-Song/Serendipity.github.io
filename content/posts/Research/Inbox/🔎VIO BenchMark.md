
| Name                                      | Feature Extractor   | Feature Tracking | Backend         | CPU  | Preintegration |     |
| ----------------------------------------- | ------------------- | ---------------- | --------------- | ---- | -------------- | --- |
| [[🧩VINS-Fusion Map.canvas\|VINS-Fusion]] | GoodFeaturestoTrack | OpticalFlowLK    | Optimization    | 270% | MidPoint       |     |
| [[👑BASALT.canvas\|BASALT]]               | FAST                | own-made         | Optimization    | 70%  | Euler방식        |     |
| [[🧩OpenVins.canvas\|OpenVINS]]           | FAST                | OpticalFlowLK    | Error-state EKF | 37%  | MidPoint       |     |
|                                           |                     |                  |                 |      |                |     |
|                                           |                     |                  |                 |      |                |     |
BASALT ros wrapper에 뭐가 문제가 있나봄 (ROS로 돌리면 300% 이렇게 찍히던데)