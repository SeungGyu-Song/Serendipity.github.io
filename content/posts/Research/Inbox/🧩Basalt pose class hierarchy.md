imu_types.h
# PoseVelBiasStateWithLin
- `using` [[#PoseVelBiasState]]::VecN

- `bool` linearized
- `VecN` delta, backup_delta
- [[#PoseState]](Scalar) pose_linearized, backup_pose_linearized
- SE3 T_w_i_current, backup_T_w_i_current

## PoseVelBiasState
struct
public [[#PoseVelState]]
`using` VecN = `Eigen::Matrix<Scalar,POSE_VEL_BIAS_SIZE, 1>`

`Vec3` bias_gyro, bias_accel


### PoseVelState
struct
public [[#PoseState]]
`using` VecN = `Eigen::Matrix<Scalar, POSE_VEL_SIZE, 1>`

`Vec3` vel_w_i

#### PoseState
struct
`using` VecN = Eigen::Matrix<Scalar, POSE_SIZE, 1>

`int64_t` t_ns
`SE3` T_w_i


# PoseStateWithLin
struct
`using` VecN = `typename PoseState<Scalar>::VecN` 
`friend` struct [[#PoseStateWithLin]]

`bool` linearized
`VecN` delta, backup_delta
[[#PoseState]](Scalar) pose_linearized, backup_pose_linearized
SE3 T_w_i_current, backup_T_w_i_current