## 처음 시작 
(run_subscribe_msckf.cpp 기준)
1. VIOManager가 global 변수로 선언이 되어있다. : `std::shared_ptr<VioManager> sys;`
2. parser를 통해 yaml파일에 들어있는 option들을 가져오는데, 이 때 내부에서 순서로는 launch 파일의 parameter → yaml 파일로 돼있어서 결국에는  yaml파일의 정보로 값을 불러온다.
3. *viz→setup_subscribers(parser)*
4. AsyncSpinner를 불러옴
# ov_msckf
## Type
### check_if_subvariable (virtual)
<span style="color:green">const std::shared_ptr(Type) <span style="color:purple">check</span></span> 
<span style="color:red">return std::shared_ptr(Type)</span>

class [[#imu]]에 대해 argument check와 pose, v, bg, ba가 **같은 객체를 가리키고 있으면** 그 애를 return.
class [[#PoseJPL]]에 대해서는 `_q, _p`  중에서 맞는 게 있으면 걔를 return.
## State
[[#ov_type::Type]] 객체를 기본으로 해서 state를 member 변수로 가지고 있는 class
### variables
```C++ 
/// Mutex for locking access to the state

std::mutex _mutex_state;

/// Current timestamp (should be the last update time in camera clock frame!)

double _timestamp = -1;

/// Struct containing filter options

StateOptions _options;

/// Pointer to the "active" IMU state (q_GtoI, p_IinG, v_IinG, bg, ba)

std::shared_ptr<ov_type::IMU> _imu;

/// Map between imaging times and clone poses (q_GtoIi, p_IiinG)

std::map<double, std::shared_ptr<ov_type::PoseJPL>> _clones_IMU;


/// Our current set of SLAM features (3d positions)

std::unordered_map<size_t, std::shared_ptr<ov_type::Landmark>> _features_SLAM;

  

/// Time offset base IMU to camera (t_imu = t_cam + t_off)

std::shared_ptr<ov_type::Vec> _calib_dt_CAMtoIMU;

  

/// Calibration poses for each camera (R_ItoC, p_IinC)

std::unordered_map<size_t, std::shared_ptr<ov_type::PoseJPL>> _calib_IMUtoCAM;

  

/// Camera intrinsics

std::unordered_map<size_t, std::shared_ptr<ov_type::Vec>> _cam_intrinsics;

  

/// Camera intrinsics camera objects

std::unordered_map<size_t, std::shared_ptr<ov_core::CamBase>> _cam_intrinsics_cameras;

  

/// Gyroscope IMU intrinsics (scale imperfection and axis misalignment)

std::shared_ptr<ov_type::Vec> _calib_imu_dw;

  

/// Accelerometer IMU intrinsics (scale imperfection and axis misalignment)

std::shared_ptr<ov_type::Vec> _calib_imu_da;

  

/// Gyroscope gravity sensitivity

std::shared_ptr<ov_type::Vec> _calib_imu_tg;

  

/// Rotation from gyroscope frame to the "IMU" accelerometer frame (kalibr model)

std::shared_ptr<ov_type::JPLQuat> _calib_imu_GYROtoIMU;

  

/// Rotation from accelerometer to the "IMU" gyroscope frame frame (rpng model)

std::shared_ptr<ov_type::JPLQuat> _calib_imu_ACCtoIMU;
```
### margtimestep

`_clones_IMU`에서 제일 오래된 time return. 
\
## StateHelper
### EKFPropagation
<span style="color:green">std::shared_ptr(State) <span style="color:purple">state</span> const std::vector(std::shared_ptr(Type)) <span style="color:purple">&order_New</span>, const std::vector(std::shared_ptr(Type)) <span style="color:purple">&order_OLD</span>, const Eigen::MatrixXd <span style="color:purple">&Phi</span>, const Eigen::MatrixXd <span style="color:purple">&Q</span></span>

order_New와 order_OLD에는 state의 imu 값, intrinsic 값이 들어가고, Phi에는 [[#predict_and_compute]]의 결과로 나온 `Phi_summed`값이 들어감. 

뭔가 $Cov * \Phi^T$ 랑 $\Phi * Cov * \Phi^T$를 구해서 state의 *_Cov* 에 넣어줌.

그러고 이 Covariance가 positive semi definite인지 체크.

### augment_clone
<span style="color:green">std::shared_ptr(State) <span style="color:purple">state</span>, Eigen::Matrix(double, 3, 1) <span style="color:purple">last_w</span></span>

[[#StateHelper#clone|StateHelper::clone]](state, state→_imu→pose())  함수로 state의 covariance 마지막에 <span style="color:purple">state</span>, 즉 state→_imu→pose()의 해당 covariance를  clone해주고, 이에 해당하는 새로운 block을 return.

`state->_clones_IMU[state->_timestamp] = pose;` 
즉, 여기에는 propagation된 pose가 들어있음.

camera_timeoffset을 calibration 할거면 어떻게 해야하는지 제시가 되어있음.

### clone
<span style="color:green">std::shared_ptr(State) <span style="color:purple">state</span>, std::shared_ptr(Type)<span style="color:purple">variable_to_clone</span></span>
<span style="color:red">return std::shared_ptr(Type)</span>


`Resize both our covariance to the new size
`state->_Cov.conservativeResizeLike(Eigen::MatrixXd::Zero(old_size + total_size, old_size + total_size));`

새로 키운 만큼의 원소는 0으로 할당

[[#Type#check_if_subvariable (virtual)|Type::check_if_subvariable]]로 Type variable에서도 그 안에 어떤 변수에 해당하는지 체크. 

state→_Cov 마지막 부분에 parameter <span style="color:purple">variable_to_clone</span>에 해당하는 부분을 추가해주고, 
`new_clone = type_check->clone(); new_clone->set_local_id(new_loc)`.으로 새로운 변수 new clone을 return. 

즉, 또 state→_variables 에 해당하는 block을 하나 만들어서 return해준다고 생각하자.

### marginalize_old_clone
<span style="color:green">std::shared_ptr(State)<span style="color:purple"> state</span></span>
state의 IMU clone 개수 > 미리 설정한 max_clone_size 옵션
	`double marginal_time = state->margtimestep(); // state에서 제일 오래된 시간.`
	[[#StateHelper#marginalize|StateHelper::marginalize]](state, state→_clones_IMU.at(marginal_time))
	 state의 `_clones_IMU`에서 marginal_time에 해당하는 값 삭제.

### marginalize
<span style="color:green">std::shared_ptr(State) <span style="color:purple">state</span>, std::shared_ptr(Type) marg<span style="color:purple"> marg</span></span>
state의 기존의 covariance에서 marginalization할 부분만 오려서 새로 만들기.
그리고 state→_variables에도 local id를 수정해주기 (marginalize한 변수만 없애서)

---

## VioManager

여기 member 변수 `std::shared_ptr<State> state`가 master state object임. 즉 얘가 진짜 필요한 state값을 가지고 있는 거.
### feed_measurement_Imu
<span style="color:green">(const ov_core::ImuData)</span>

1. oldest_time = [[#margtimestemp|State::margtimestep]] : imu_clones 중 제일 오래된 시간 찾기.
2. [[#feed_imu|Propagator::feed_imu]]로 msg 넣어주고 oldest보다 이전 값 삭제하기.
4. !is_initialized_vio 
	1. initializer→feed_imu
5. is_initialized_vio && updaterZUPT != nullptr && (!params.zupt_only_at_beginning || !has_moved_since_zupt)
	1. updaterZUPT→feed_imu
### feed_measurement_camera
<span style="color:green">(const ov_core::CameraData)</span>
[[#track_image_update]]

### track_image_update
<span style="color:green">(const ov_core::CameraData)</span>
1. image downsample 해야하면 진행
2. [[#feed_new_camera|ov_core::trackBase::feed_new_camera]] (trackKLT)
3. <span style="color:#B9B9B9"> initialized && trackARUCO → ARUCO tracking</span>
4. <span style="color:#B9B9B9"> zupt update 하면 후에 clean_old_imu_measurement </span>
5. [[#try_to_initialize]] 
6. [[#do_feature_propagate_update]] 

### get_features_SLAM
이거도 state의 _features_SLAM을 가져다가 쓰는 거

### get_historical_viz_image
<span style="color:red"> return cv::Mat </span>
state에서 `_feature_SLAM`에 담긴 feature를 visualize
[[#TrackBase#display_history|TrackBase::display_history]]
### do_feature_propagate_update
<span style="color:green">const ov_core::CameraData <span style="color:purple">&message</span></span>

features_SLAM은 현재도 발견되고, max_clones 개수보다 많이 발견된 feature들의 모임
feats_slam과 구별해야함.
MSCKF feature : slam update에 사용하지 않는 feature.

1. state의 timestamp ≠ imu message의 timestamp → [[#propagate_and_clone|Propagator::propagate_and_clone]] ?state의 timestamp가 언제 갱신되더라? #점검 
2. `state->_clones_IMU.size() < std::min(state->_options.max_clone_size, 5)` → 그냥 return하고 종료
	1. 이유는 그냥 넉넉히 5개가 있어야 triangulation을 할 수 있으니까 그런 것 같다. *왜 5개지? *    #점검 
3. `feats_lost, feats_marg, feats_slam` 변수 구하기.
	1. `feats_lost = trackFEATS->get_feature_database()->features_not_containing_newer(state->_timestamp, false, true);` 
	  state의  timestamp보다 최신인 게 없는 feature들 모음.
	  [[#features_not_containing_newer|FeatureDatabase::features_not_containing_newer]]
	2. *_clones_IMU*가 5개 이상이거나 state의 *_clones_IMU*가 max_clone_size보다 많을 때
		1. `feats_marg = trackFEATS->get_feature_database()->features_containing(state->margtimestep(), false, true)`
	     [[#features_containing|FeatureDatabase::features_containing]] 로 margtimestep에 있는 애들을 검출.
	     2. `feats_slam = trackARUCO->get_feature_database()->features_containing(state->margtimestep(), false, true);`
	3. `feats_lost`에 있는 feature가 message.camera sensor_id에서 발견이 안 되면 `feats_lost`에서 삭제하고, 발견이 됐으면 그대로 진행.
			*E.g: if we are cam1 and cam0 has not processed yet, we don't want to try to use those in the update yet
			E.g: thus we wait until cam0 processes its newest image to remove features which were seen from that camera* 
	4. `feats_marg`에서 `feats_lost`에 있는 애들 삭제하기. (*현재 안 발견된 애들 삭제*) (line 401)
	5. `feats_marg`에 있는 feature들 중에 왼 / 오 둘 중 하나 `max_clone_size`보다 많이 발견되면
		1. `feats_marg`에서 삭제, `feats_maxtracks`에 추가.
			- `feats_maxtracks`은 state와 같은 시점, max_track보다 많이 발견된 feature들의 모임 , feature_SLAM으로 사용될 가능성이 있나보다.
	6. initialization으로부터 1초 이상 지나고, `state->_features_SLAM < option.max_slam_features + curr_aruco_tags`이면 
		1. `amount_to_add = option.max_slam_features + curr_aruco_tags - state->_features_SLAM.size()` :  feature_SLAM 늘릴 개수
		2. `valid_amount = amount_to_add와 feats_maxtracks` 중 더 적은 거
			1. state에 들어갈 feature를 보수적으로 잡는 것 같아.
		3. `feats_slam`에는 `valid_amount`만큼 `feats_maxtracks`의 뒤에서부터 넣고, `feats_maxtracks`에서는 삭제. 
	7. `state→_features_SLAM` loop
			1. `trackFEATS`(Feature로 뽑힌 애들)에서 state의 landmark와 겹치는 애들은 `feats_slam`에 넣어주고
			2. 겹치지 않고, `current_unique_cam == true` → `landmark.second→should_marg = true
			3. `landmark.second→ update_fail_count > 1 →landmark.second→should_marg = true`
	8. [[#StateHelper#marginalize_slam|StateHelper::marginalize_slam]](state)
		- old SLAM feature들을 marginalize하는데, 이 때
		- old SLAM feature들은 현재 시점에 tracking이 잘 안 된 애들
	9. `feats_slam_DELAYED, feats_slam_UPDATE`
		1. `feats_slam`에 있는 feature가 원래 *_features_SLAM*에 있었다면 `feats_slam_UPDATE`에 추가
		2. 원래 없었다면 `feats_slam_DELAYED`에 추가.
	10. `featsup_MSCKF`에 `feats_lost, feats_marg, feats_maxtracks`순으로 넣기.
	11. 


### try_to_initialize
<span style="color:green">const ov_core::CameraData</span> <span style="color:purple">&message</span>
1. `thread_init_running → camera_queue_init.push_back(message.timestamp)`
		initialization 과정에서 image 들어오면 camera_queue_init에 쌓아놓기.
2. thread를 하나 생성함으로써 main thread에 방해되지 않게 함.
	1. [[#initialize|ov_init::InertialInitializer::initialize]]
	2. initialize 성공했을 때
		1. [[#set_initial_covariance|ov_msckf::StateHelper::set_initial_covariance]]
		2. state의 timestamp를 맞춰주고
		3. [[#cleanup_measurements|trackFEATS→get_feature_database()→cleanup_measurements]]로 initialize 된 시간보다 오래된 feature 제거.
		4. [[#set_num_features|trackFEATS::set_num_features]] : yaml파일에 있는 num_pts / num_cameras
		5. 로봇이 움직인다면 ZUPT(zero velocity update) 안 하기.
		6. initialize 시점보다 이후에 들어온 카메라들 (caemra_queue_init)에 대해 
			1. [[#propagate_and_clone|Propagator::propagate_and_clone]]
			2. [[#marginalize_old_clone|StateHelper::marginalize_old_clone]]
	3. 실패했을 시, camera_queue_init.clear()
	4. thread join / detach

## Propagator
### feed_Imu
<span style="color:green">(const ov_core::ImuData, double oldest_time)</span>
1. std::vector<[[#ImuData|ov_core::ImuData]]> imu_data 에 값 넣기.
2. oldest_time 보다 더 이전의 값들은 버리기
### fast_state_propagate
<span style="color:green" >(StatePtr , double timestamp, Eigen::Matrix(double, 13,1) &state_plus, Eigen::Matrix(double, 12, 12) &covariance)</span> 
1. state에서 imu 부분에 해당하는 timestamp, value, covariance, dt를 가져옴. (이전의 값)
	1. 이 때 [[#get_marginal_covariance|StateHelper::get_marginal_covariance]]함수로 imu에 해당하는 Covariance만 복사해옴.
	2. [[#select_imu_readings|Propagator::select_imu_readings]]로 state와 현재 imu msg까지 imu값들 가져오기.
	3. state의 imu bias, intrinsic 값들 복사생성.
	4. IMU instrinsic 보정 : [Online Self-Calibration 분석 논문](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10145468) 
		1. ![[Pasted image 20240828222201.png]]
			- 한편 저가 IMU에서는 imu에서 gyro, acceleration 축이 같지 않을 수가 있다. 
			- 그래서  $\prescript{I}{\omega}R$, $\prescript{I}{a}R$ 는 각각 imu 축에서 gyro, acceleration 축으로의 회전을 의미함.
			- 보통은 gyro나 acceleration 축 하나를 imu 축으로 두고 나머지 하나와의 R을 도입하긴함. 그래서 여기서는 imu model이 kalibr이면 $\prescript{I}{\omega}R$를 Identity로, RPNG면 $\prescript{I}{a}R$를 Identity로 놓음.
			- $D_w, Da$는 scale imperfection과 axis misalignment를 의미한다고 하는데 잘 와닿지는 않는다.
	5. **Discrete-time Error-state Propagation** [discrete propagation](https://docs.openvins.com/propagation_discrete.html)
		- Discrete-time에서는 imu intrinsic을 고려한 propagation은 다루지 않음.
		2. ![[Pasted image 20240828221720.png]]
		 ![[Pasted image 20240828221940.png]]
		 ![[Pasted image 20240828221734.png]]
		- [ ] 근데 `Qd = 0.5 * (Qd + Qd.transpose());` 하는 이유를 모르겠어. #점검
		- 여기서도 VINS-Fusion과 마찬가지로 quaternion으로 함. 
		1. cache_state_est에 값 업데이트. 


### select_imu_readings
<span style="color:green">const std::vector(ov_core::ImuData) <span style="color:purple"> &imu_data</span>, double <span style="color:purple">time0</span>, double <span style="color:purple">time1</span>, bool <span style="color:purple">warn</span></span>
**return** <span style="color:red">  std::vector(ov_core::ImuData)</span>

- time0 ~ time1 시간 동안의 imu를 `prop_data`라는 local 변수에 저장해서 return하는 함수.
- 근데 맨 처음과 맨 마지막은 linear interpolation해서 넣어줌.
### compute_Xi_sum
<span style="color:green">std::shared_ptr(State) <span style="color:purple">state</span>, double <span style="color:purple">dt</span>, const Eigen::Vector3d <span style="color:purple">&w_hat</span>, const Eigen::Vector3d <span style="color:purple">&a_hat</span>, Eigen::Matrix(doublem 3, 18) <span style="color:purple">&Xi_sum</span></span>



### predict_and_compute
<span style="color:green">std::shared_ptr(State) <span style="color:purple">state</span>, const ov_core::ImuData <span style="color:purple">&data_minus</span>, const ov_core::ImuData <span style="color:purple">&data_plus</span>, Eigen::MatrixXd <span style="color:purple">&F</span>, Eigen::MatrixXd <span style="color:purple">&Qd</span></span>

1. data_minus는 i 번째, data_plus는 i+1 번째. [[#fast_state_propagate]]와 마찬가지로 IMU unbias화를 시킴.
2. integration_method option이 RK4, Analytical이면
	1. [[#compute_Xi_sum]] 을 통해 mean과 covariance integration을 미리 계산. (`a_hat_avg, w_hat_avg`를 넘겨줌.)






### propagate_and_clone
<span style="color:green">std::shared_ptr(State) <span style="color:purple"> state</span>, double <span style="color:purple">timestamp</span> </span>

1. state의 timestamp + `last_prop_time_offset`, 현재 imu_message timestamp + 현재 td 간의 [[#select_imu_readings]] 하고 이 데이터들을 `prop_data`에 저장.
2. `prop_data`에 있는 imu 데이터 하나 당 [[#predict_and_compute]]
# ov_core

##### CameraData
```C++

double timestamp;
//camera 번호
std::vector<int> sensor_ids;
// image 쌍을 하나의 CameraData 객체에서 가지고 있음.
std::vector<cv::Mat> images;

std::vector<cv::Mat> masks;

```
##### ImuData
```C++

double timestamp;

Eigen::Matrix<double, 3, 1> wm;

Eigen::Matrix<double, 3, 1> am;
```

#### Feature
`std::unordered_map<size_t, std::vector<Eigen::VectorXf>> uvs`
`std::unordered_map<size_t, std::vector<Eigen::VectorXf>> uvs_norm;`
`std::unordered_map<size_t, std::vector<double>> timestamps;`
#### FeatureDatabase
`std::unordered_map<size_t, std::shared_ptr<Feature>> features_idlookup` :  하나의 feature의 observation에 대해 저장한 변수.

##### features_not_containing_newer
<span style="color:green">double <span style="color:purple">timestamp</span>, bool <span style="color:purple">remove</span>, bool <span style="color:purple">skip_deleted</span></span>
**return** <span style="color:red">std::vector(std::shared_ptr(Feature))</span>


- features_idlookup에 있는 모든 feature의 observations에 대해
	1. `skip_deleted` && (* it.second→to_delete) continue;
	2. 각 feature 들이 parameter `timestamp`보다 최근 게 있는지 체크.
	3. 없으면 `feats_old.push_back` , parameter `remove==true` 면 `features_idlookup`에서 삭제.
- return `feats_old`, 즉 outlier를 return하는 거임.

##### features_containing
<span style="color:green">double <span style="color:purple">timestamp</span>, bool <span style="color:purple">remove</span>, bool <span style="color:purple">skip_deleted</span></span>
**return** <span style="color:red">std::vector(std::shared_ptr(Feature))</span>

[[#features_not_containing_newer]]랑 비슷함. 
1. 그냥 parameter `timestamp`의 값과 feature_idlookup의 feature들 observation 중에서 같은 게 있는지 찾고
2. `feats_has_timestamp`에 저장, return `feats_has_timestamp`

## FeatureHelper
### compute_disparity
<span style="color:green">std::shared_ptr (ov_core::FeatureDatabase) <span style="color:purple">db</span>, double <span style="color:purple">&disp_mean</span>, double <span style="color:purple">&disp_var</span>, int <span style="color:purple">&total_feats </span>double <span style="color:purple">newest_time = -1</span>, double <span style="color:purple">oldest_time = -1</span></span>

- -1은 disable이라고 생각하면 됨.
- [[#FeatureDatabase]]에 있는 feature들의 observation에 대해서 image 상의 disparity 즉, parallax의 mean과 variance를 구한다고 생각하면 됨. 

 


## TrackBase 

### display_history
`highlightened` 변수가 `_features_SLAM`이라고 생각하면 됨.
아 초록색 box가 `_features_SLAM`
나머지는 [[#FeatureDatabase]]에서 `_Features_SLAM`와 같은 feature들을 찾은 다음에, 표시를 하는 건데, 
색이 흰 색일 수록 예전 것임.
그리고 이건 느낌인데, 양쪽에서 검출된 feature는 파란색, 한 쪽에서만 검출된 것은 빨간색으로 표현하는 것 같다.
코드를 봐도 잘 모르겠음.
![[Pasted image 20241007172336.png]]

## TrackKLT ← TrackBase
### feed_new_camera
<span style="color:green">(const CaemraData</span> <span style="color:purple"> &message </span>) 

	==주석에서 Parallize를 하면 속도가 매우 느려진다는 경고가 있다. 왜인지는 모르겠음==
- image 개수만큼
1. histogram equalization
2. image pyramid 만들기.
3. [[#feed_monocular]] or [[#feed_stereo]]

### feed_stereo
<span style="color:green">(const CameraData </span>  <span style="color:purple">&message</span>, <span style="color:green">size_t </span> <span style="color:purple">msg_id_left, <span style="color:green">size_t </span> msg_id_right</span> )
- 여기서 특이한 게 lock_guard를 2개를 사용함으로써 왼쪽 image, 오른쪽 image를 서로 관여하지 않게 함.
	- cam_id_left, right가 각각 index 0,1이라 이후 모든 index 0,1에 따라서 mutex가 되는 거인듯 ???
1. message의 각 image, pyimg, mask를 local 변수로 복사.
2. pts_last, ids_last도 똑같이 local 변수로 복사.
3. [[#perform_detection_stereo]] 에서 corner detection
4. [[#perform_matching]]을 parallel하게 돌려서 왼/오 이미지 각각 temporal opticalflow tracking. <span style="color:red">VINS-Fusion과는 다르게 같은 시간 left to right tracking은 하지 않음.</span>
5. 위에서 뽑은 good feature들을 각각 good_left, good_ids_left / good_right, good_ids_right에 넣어줌.
6. [[#FeatureDatabase|ov_core::FeatureDatabase]]에 있는 feature들을 update. (distorted, undistorted 좌표 다 )
7. 현재 쓰인 값들을 img, imgpyramid, img_mask, pts, ids의 각종_last 로 저장.
### feed_monocular

### perform_detection_stereo
<span style="color: green">std::vector(cv::Mat) <span style="color: purple">&img0pyr </span>,
const std::vector(cv::Mat) <span style="color: purple">&img1pyr</span>, const cv::Mat <span style="color: purple"> &mask0</span>, const cv::Mat<span style="color: purple"> &mask1</span>,
size_t <span style="color: purple">cam_id_left</span>, size_t <span style="color: purple">cam_id_right</span>, 
std::vector(cv::KeyPoint) <span style="color: purple">&pts0 </span>, std::vector(cv::KeyPoint) <span style="color: purple">&pts1</span>, 
std::vector(size_t) <span style="color: purple">&ids0 </span>, std::vector(size_t) <span style="color: purple">&ids1,</span></span>
- input값들은 다 old이고, 0이면 left, 1이면 right.
- 왼쪽에서 뽑은 새로운 feature를 오른쪽에 복사해서 optical flow를 하니까 baseline이 큰 상황에서는 이게 성립하지 않음.
- feature를 새로 뽑아야할 때도 현재 이미지가 아니라 old image로 뽑는 게 맞는 건지 의문.

<왼쪽 이미지>
1. feature 검토 
	- edge feature 삭제
	- img범위 밖 feature 삭제
	- 해당 좌표 mask값이 128이상이면 삭제 (왜 255도 아니고 127일까? #점검)
2. grid, mask image 처리
	- grid_2d_close0에는 10으로 나눈 좌표에 흰 색 색칠
	- grid_2d_grid0에는 해당 grid +1
	- mask_update0에는 min_px_dist 크기만큼 흰색 색칠
3. feature를 더 뽑아야 하면 
		[[#perform_griding|Grider_GRID::perform_griding]]
		뽑은 point를 오른쪽에 복사해서 Opticalflow, image 범위를 벗어나면 제거
		나머지 이상 없는 feature 각 pts0, pts1, ids0, ids1에 push_back.
<오른쪽 이미지>
4. 1~3번 반복. : optical flow 제외하고.

### perform_matching
<span style="color: green">std::vector(cv::Mat) <span style="color: purple">&img0pyr </span>, const std::vector(cv::Mat) <span style="color: purple">&img1pyr</span>, std::vector(cv::KeyPoint) 
<span style="color: purple">&kpts0 </span>, std::vector(cv::KeyPoint) <span style="color: purple">&kpts1</span>, 
size_t <span style="color: purple">id0 </span>, size_t <span style="color: purple">id1,</span>std::vector(uchar)<span style="color: purple">&mask_out</span></span>
- img0pyr, kpts0은 last고, img1pyr, kpts1이 현재.

`cv::findFundamentalMat(pts0_n, pts1_n, cv::FM_RANSAC, 2.0 / max_focallength, 0.999, mask_rsc);` 
즉, RANSAC을 하는데 그 기준을 2 / max_focallenght로 함. 
- 2.0/max_focallenght에서 focal length로 나눠주는 이유가 normalized plane으로 calibration 해줄 때 focal length를 나눠서 해주니까 여기서의 의미는 <span style="color:red">normalized plane에서 오차를 고려하겠다는 의미</span>
- RANSAC을 할 때 undistorted point를 가지고 하는데 projection이 nonlinear해서 nonlinear 특성을 없앤 점을 가지고 진행.  비선형성은 더 복잡해서 RANSAC 오차가 더 클 수 있음.

- 💡focal length가 낮아지면 scale이 낮고(image가 커지고), focal length가 크면 scale이 높다(image가 작다.) 이러면 실제로 움직임을 크게 해도 scale이 큰 이미지에서 작게 움직인 것처럼 반영이 되기 때문에 초점 거리를 기준으로 threshold를 설정하게 되면 scale 변화에 더 일관된 결과를 얻을 수 있다고 함.
	- undistorted point로 고려했으니까 intrinsic을 한 거라서 focal length로 thresholding을 해야함. (2픽셀)

## TrackDescriptor ← TrackBase
### feed_new_camera
<span style="color:green">const CameraData <span style="color:purple">&message</span></span>

*stereo, monocular version에 따라 부르는 게 달라지고, binocular tracking이면 mono를 parallize해야함.*

[[#TrackDescriptor ← TrackBase#feed_stereo|TrackDescriptor::feed_stereo]]
### feed_stereo
<span style="color:green">const CameraData <span style="color:purple">&message</span>, size_t <span style="color:purple">msg_id_left</span>, size_t<span style="color:purple">msg_id_right</span></span>

[[#TrackKLT ← TrackBase#feed_new_camera|TrackKLT::feed_new_camera]]에서 했던 것처럼 (근데 왼, 오 동시에 함, thread X)
1. histogram equalizer → clearer image with obvious boundaries
	1. or createCLAHE
2. `img_left, img_right, mask_left, mask_right` 를 local 변수로 저장.

[[#TrackDescriptor ← TrackBase#perform_detection_stereo|TrackDescriptor::perform_detection_stereo]] : 새로운 descriptor 추출

[[#TrackDescriptor ← TrackBase#robust_match|TrackDescriptor::robust_match]]를 parallel로 돌림.
## Grider_GRID
### perform_griding
<span style="color:green"> const cv::Mat <span style="color:purple">&img</span>, const cv::Mat <span style="color:purple">&mask</span>, const std::vector(std::pair(int, int)) <span style="color:purple">&valid_locs</span>, std::vector(cv::KeyPoint) <span style="color:purple">&pts</span>, int <span style="color:purple">num_features</span>, int <span style="color:purple">grid_x</span>, int <span style="color:purple">grid_y</span>, int <span style="color:purple">threshold</span>, bool <span style="color:purple">nonmaxSuppression</span></span>

- 각 grid로부터 FAST feature를 뽑는 함수. 
- valid_locs가 뽑아야하는 grid에 해당함.
- [ ] 왜 grid를 feature 개수에 따라서 다시 정했는지 잘 모르겠습니다. 이러면 기존의 grid와는 위치가 달라져서 원하는 곳에 안 뽑힐 수도 있는 게 아닌가 싶습니다. #점검 


![[SmartSelect_20240829_175228_Samsung Notes.jpg]]
한편, 만약 feature 개수가 낮으면 grid 간격이 원래보다 넓어져서 기존의 index로 접근하면 image 범위 밖으로 나갈 가능성이 있습니다. 
그래서 `if (x + size_x > img.cols || y + size_y > img.rows)` 코드를 통해 이를 처리합니다.

1. 각 grid 별로 `paralell_for_`를 통해 FAST 알고리즘을 병렬로 구현
2. 해당 좌표 mask 값 > 127인 feature 제외
3. `cv::cornerSubPix` 를 통해 feature 좌표 정보를 보정합니다.

# ov_init
## InertialInitializer
### initialize
<span style="color:green">double <span style="color:purple">& timestamp</span>, Eigen::MatrixXd <span style="color:purple">&covariance</span> std::vector(std::shared_ptr(ov_type::Type)) <span style="color:purple">&order</span>, std::shared_ptr(ov_type::IMU) <span style="color:purple">t_imu</span>, bool <span style="color:purple">wait_for_jerk</span></span>
1. [[#FeatureDatabase]]에 저장되어 있는 feature들의 timestamp를 조사해서 `newest_cam_time`을 구함.
2. init_window_time만큼 빼줘서 oldest_time을 구하고, 이거보다 오래된  observation들 / feature를 삭제.
3. imu도 마찬가지로 오래된 값들 삭제.
4. [[#compute_disparity|FeatureHelper::compute_disparity]]  두 번 실행.
	 ![[compute_disparity.jpg|400]]
	 사진과 같이 1,2번에 대해 각각 parallax의 mean과 variance를 구함.
	 이 mean으로 움직였는지 여부를 판단해서 initialize 종류를 선택.
- has_jerk = 가만히 →움직임.
- is_still = 가만히 → 가만히
6. [[#StaticInitializer#initialize|StaticInitializer::initialize]] or [[#DynamicInitializer#initialize|DynamicInitializer::initialize]]
7. 여기서 실패하면 terminal 창에 <span style="color:rgb(200, 200, 100)">노란 글씨</span>로 cout이 이루어짐.
## StaticInitializer
### initialize
<span style="color:green">double <span style="color:purple">& timestamp</span>, Eigen::MatrixXd <span style="color:purple">&covariance</span>, std::vector(std::shared_ptr(Type)) <span style="color:purple">&order</span>, std::shared_ptr(IMU) <span style="color:purple">t_imu</span>, bool <span style="color:purple">wait_for_jerk</span></span>
이게 <span style="color:red">window 의 예전 절반은 가만히, 최근 절반은 움직였을 때 </span>하는 것임을 기억하자.
1. init_window에 대해서 오래된 절반은 `window_2to1`, 최근에 절반은 `window_1to0`으로 둔다.
2. `a_var_1to0` < threshold, `a_var2to1` > threshold 여야 아래가 돌아감. <span style="color:gray">(`!wait_for_jerk`일 때를 고려한 조건도 있긴 함.)</span>
3. 처음에 가만히 있으니까 `a_avg_2to1 / a_avg_2to1.norm()`로 z_axis를 구하고, 이를 gram_schmidt로 나머지 x,y axis를 구한다음 Ro에 넣어준다.
4. `q_GtoI = rot_2_quat(Ro)` : <span style="color:red">이거 ItoG가 아니라 GtoI임에 유의하기.</span>
5. bias 구하기
	1. `bg = w_avg_2to1;`
	2. [ ] `ba = a_avg_2to1 - quat_2_Rot(q_GtoI) * gravity_inG` #점검  : noise mean이 0이니까 그 가정 하에 이렇게 쓴 거 같아.
6. t_imu에 `q_GtoI, bg, ba`넣어주고, set_fej까지.
7. `order.push_back(t_imu)`
8. covariance 값을 넣어주는데 그냥 magic number로 가중치 곱해서 넣어주는 거 같다.
## DynamicInitializer
[[📦️OpenVINS State Initialization - Technical Report]]

### initialize
[[#InertialInitializer#initialize|InertialInitializer::initialize]]에서 했던 것처럼
	 보유하고있는 feature들 중 최근 시간을 건져내고, oldest_time도 구한 후 
	 오래된 imu 값들 버리기.

가지고 있는 feature를 `feat_new`로 복사해준 후 새로운 local변수 `Features`에 저장. 
	*initialization은 따로 thread를 둬서 돌아가니까 feature tracking과 저촉하지 않게끔 함.* 

한 feature loop를 돌면서 
	미리 설정한 initialize window 주기 (`int_window_time / init_dyn_num_pose`)에 맞춰서 `times, camids`에 값을 넣어줌. 
	이 때, `times` 원하는 주기에 맞춰진 시간 vector(cam time)
	 `camids`는 왼/오에 대해 true/false를 저장.
	 
<span style="color:blue"> 중요한 것은 내가 설정한 주기의 카메라를 저장하는 거임. 마치 어떻게 보면 naive한 keyframe. </span>

#고칠부분
`const int min_num_meas_to_optimize = (int)params.init_window_time; : 고칠 여지가 있음. : 그냥 적어도 1초에 하나 씩은 있어야한다 이런 의미인 거 같아.`

oldest_time과 newest_time에 있는 imu를 가져옴 : [[#InitializerHelper#select_imu_readings|InitializerHelper::select_imu_readings]]

init_dyn_min_deg만큼 회전이 이루어져야 initialization이 진행됨. 왜 gyro값을 기준으로 할까? → 그냥 충분한 baseline을 확보하려고 하는 것 같음.

#### 이 아래서부터는 reprojection error 기반[[📦️OpenVINS State Initialization - Technical Report#3.4 Linear Ax = b Problem|Linear Ax=b problem]]으로 푸는 게 나옴.
<span style="color:green">std::map(size_t, int) <span style="color:orange"> map_features_num_meas</span></span> : 한 feature에 몇 개의 observation이 있는지
<span style="color:green">std::map(double, bool) <span style="color:orange">map_camera_times</span></span> : feature의 시점을 모두 저장해놓은 자료구조
<span style="color:green">std::map(size_t, bool)<span style="color:orange"> map_camera_ids</span></span> : 저장된 feature들의 카메라 번호 존재 여부
<span style="color:green">double<span style="color:orange"> pose_dt_avg</span></span> : 미리 설정한 원하는 initialization window 주기

`std::map<double, std::shared_ptr<ov_core::CpiV1>> map_camera_cpi_I0toIi` 선언.
map_camera_times의 시점들을 loop로 돌면서
- CpiV1  객체를 하나 생성한 후
- 객체 내 linearization point 값에 argument값을 넣어줌
- oldest_time과 current_time 사이에 있는 imu값을 가져옴 . [[#InitializerHelper#select_imu_readings|initializerHelper::select_imu_readings]]
	- 한편, `I0toIi1`: oldest_camera_time ~ current_time
	- `IitoIi1`: last_camera_time ~ current_time 임을 기억하자.
- `I0toIi1`과 `IitoIi1`에 대해서 각각 
	- bg, ba에 대해 linearization point로 설정해준 후, 
	- 해당 범위의 imu값들을 불러와서
	- CpiV1내의 객체에서 preintegration을 진행함. [[🧩OpenVINS Code Analysis#CpiV1#feed_IMU|CpiV1::feed_IMU]]
- `map_camera_cpi_I0toIi, map_camera_cpi_IitoIi1`에 각각 해당하는 값을 따로따로 넣어줌. → 코드에서는 해당하는 시간의 pose를 가지고 있다고 판단.
##### Linear Ax=b
300 번째 줄 부터 
- `A_index_features`에 feature 값을 넣어줌. 
- `feature`를 돌면서 feature 좌표의  uv값과 preintegration term을 가져옴.
- 나머지는 [[📦️OpenVINS State Initialization - Technical Report#3.4 Linear Ax = b Problem|Linear Ax=b problem]]여기에 기술되어있는 식을 그냥 옮겨놓은 거라 따라가면 된다.
- 선형 방정식을 풀고 난 후, gravity, feature position, velocity를 얻을 수 있는데, gravity의 경우, [[#StaticInitializer#initialize|StaticInitializer::initialize]]에서 했던 [[Gram-Schmidt]] 방식으로 진행해서 gravity-align coordinate를 설정함.
- 추가적으로 맨 처음 시점에 대한  나머지 window의 Rotation, position, velocity를 recovery

#### MLE, Full optimization (Ceres Solver)
679번 째 코드를 보면 제일 처음 pose를 fix로 두는 게 아니면 window가 매우 작아서 full rank가 되지 않는다고 한다. #점검 

Prior term : [[#Factor_GenericPrior]]
IMU term : [[#Factor IMU]]
Camera term : [[#Factor_ImageReprojCalib]]

---
Optimization  성공 시, 
	`timestamp = newest_cam_time`으로 넣어주며, 함수가 끝난 이후 state에 이 시간보다 오래된 것들은 다 버릴 예정.
	`state_imu <- map_states[timestamp]`
	`_imu<-state_imu의 값`
	`_clones_IMU <-map_states의 값 모두 (map_camera_times에 들어있던 시점들)`
	`_features_SLAM <-map_features, ceres 결과 feature position.`
	Ceres를 통해서 covariance recovery를 진행 (1005 ~ 1074번 째 줄)
	config file의 inflation 값으로 각 covariance 파트에 뻥튀기를 해줌.
	`covariance = 0.5*(covariance + covariance.transpose())` 
		<span style="color:red">이거는 covariance의 대칭성이 깨지는 경우도 있는 것 같아서 이렇게 해주는 것 같다.</span>
		


## InitializerHelper
### select_imu_readings
<span style="color:green">std::vector(ov_core::ImuData) <span style="color:purple">&imu_data_tmp</span>, double <span style="color:purple">time0</span>, double <span style="color:purple">time1</span></span>
<span style="color:red"> return std::vector(ov_core::ImuData) </span>
해당 시간의 imu 값들을 return. 
이 때 양 끝의 시간은 linear interpolate함. [[#interpolate_data]]

### interpolate_data
해당 시간에 linear interpolate해주는 함수.


# ROS1Visualizer
## callback_stereo
1. ros sensor_msgs 를 [[#CameraData|ov_core::CameraData]]에 저장함. (image, mask)
2. lock_guard 걸고 camera_queue에 push, sort로 시간 순으로 정렬. 이 때 operator 함수를 선언해주었음.

## callback_inertial
1. ros msg를 [[#ImuData|ov_core::ImuData]]에 저장함.(w_m, a_m)
2. [[#feed_measurement_imu|VioManager::feed_measurement_imu]]에서 각 클래스로 feed_imu(propagator, initializer, updaterZUPT)
3. [[#visualize_odometry|visualize_odometry]]에서 pre_integration을 진행함.
4. <span style="color:red">Thread : </span>[[🧩OpenVINS Code Analysis#feed_measurement_camera|VioManager::feed_measurement_camera]] && [[🧩OpenVINS Code Analysis#visualize|visualize]]
### visualize_odometry
1. initialized가 됐는지 확인. ❌ → return false.
2. [[#fast_state_propagate|ov_msckf::Propagator::fast_state_propagate]]로 imu propagation.
3. 위 결과와 covariance값을 odomIinM 메시지 안에 넣어주기.
	- [ ] covariance 처리하는 거 잘 이해가 안 됨. 0으로 거의 다 박혀야하는 거 아닌가? #점검 	-![[Screenshot_20240828_225547_Samsung Notes 1.jpg|300]]
4. pub_odomimu.publish(odomIinM)


### visualize
- [[#publish_images]]
- [[#publish_state]]
- [[#publish_features]]
- [[#publish_groundtruth]]
- [[#publish_loopclosure_information]]
#### publish_images
아래 사진의 표시된 feature들 의미 파악하기.
![[Pasted image 20241007172336.png|300]]

`cv::Mat img_history = _app->get_historical_viz_image` [[#VioManager#get_historical_viz_image|VioManager::get_historical_viz_image]]
