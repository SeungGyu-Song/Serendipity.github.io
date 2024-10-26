# basalt
### feed_images
[[#OpticalFlowBase]]의 객체가 뽑아 쓸 queue에 이미지를 넣어주는 역할
`opt_flow_ptr->intput_queue.push(data)`

## BundleAdjustmentBase
[[#LandmarkDatabase]](Scalar) lmdb


## SqrtBundleAdjustmentBase
<span style="color: brown">public <span style="color: blue">BundleAdjustmentBase</span></span>
[[#BundleAdjustmentBase]]

## VioEstimatorBase

### Constructor

#### factory_helper
<span style="color:green">const VioConfig<span style="color: purple">& config</span>, const Calibration(double)<span style="color: purple">& cam</span>, const Eigen::Vector3d<span style="color: purple">& g</span> bool<span style="color: purple"> use_imu</span></span>
return <span style="color: red"> VioEstimatorBase::Ptr </span>

Scalar 악몽의 시작은 여기서부터임.
```C++
if (use_imu) {

res.reset(new SqrtKeypointVioEstimator<Scalar>(g, cam, config));

} else {

res.reset(new SqrtKeypointVoEstimator<Scalar>(cam, config));

}
```


### VioEstimatorFactory
#### getVioEstimator
결국 [[#SqrtKeypointVioEstimator]]를 불러오는 거.

<span style="color: green">const VioConfig<span style="color: purple">& config</span>, const Calibration(double)<span style="color: purple">& cam</span>, const Eigen::Vector3d<span style="color: purple">& g</span>, bool<span style="color: purple"> use_imu</span>, bool<span style="color: purple"> use_double</span> </span>
return <span style="color: red"> VioEstimatorBase::Ptr </span>

use_double에 따라 
return [[#factory_helper]](float)(config, cam, g, use_imu)
`return factory_helper<double>(config, cam, g, use_imu);`

### SqrtKeypointVioEstimator 
<span style="color: brown">public <span style="color: blue">VioEstimatorBase, </span>public <span style="color: blue">SqrtBundleAdjustmentBase(Scalar_)</span></span>
[[#SqrtBundleAdjustmentBase]] 

`using` [[#BundleAdjustmentBase]](Scalar)::frame_poses
`using` [[#BundleAdjustmentBase]](Scalar)::frame_states
`using` [[#BundleAdjustmentBase]](Scalar)::lmdb
`using` [[#BundleAdjustmentBase]](Scalar)::obs_std_dev
`using` [[#BundleAdjustmentBase]](Scalar)::huber_thresh
`using` [[#BundleAdjustmentBase]](Scalar)::calib

`std::set<int64_t>` kf_ids

### popFromImuDataQueue
imu_data_queue에서 최근 거를 빼서 [[#ImuData]]에 저장함.
이 때, 우리가 사용하는 floating point가 double이면 그냥 뱉어주고, 아니면 float으로 cast를 통해 바꿔서 뱉어줌.


#### Constructor
- config파일에서 option 값 불러오기.
- config에서 vio_sqrt_marg == true로 하면
	- marg_data에 들어가는 값들에 sqrt를 씌워줘서 넣어준다.

#### initialize
<span style="color:green">const Eigen::Vector3d<span style="color:purple">& bg_</span>, const Eigen::Vector3d<span style="color:purple">& ba_</span></span>

Thread에 넣어줌 :
- `processing_thread.reset(new std::thread(`[[#proc_func]]`))`

#### proc_func

##### while (true)
`vision_data_queue.pop(current_frame)` : optical flow의 결과 (transforms)를 뽑아옴.
###### initialize 안 됐을 때
- `data` = [[#popFromImuDataQueue]]
- [[#getCalibrated]] 함수를 통해서 
	- imu의 acc, gyr covariance를 받아오고, 각각의 bias와 scale값을 받아옴.
	- 따라서 결국 raw_measurement + scale * raw_measurement - bias 값을 가져옴.
- 중력과 imu의 acc를 기준으로 world와 body의 초기 회전관계를 정의함 → static 상황을 가정하는 건가?
###### Initialize 후
[[#IntegratedImuMeasurement]]::Ptr을 `meas`라는 local  변수로 둠.
- parameter : prev_frame→t_ns, last_state의 gyro, accel bias.

[[#IntegratedImuMeasurement#integrate|IntegratedImuMeasurement::Integrate]] 함수를 통해서 [[Research/Zotero/Papers/@Forster_OnManifold_2017|Manifold preintegration]] 방식으로 preintegration을 진행
이상한 게 integrate 함수에 넣어주기 전에 **config에서 저장한** bias를 빼주는데, 들어가서도 **우리가 추정한** bias를 또 빼줌.


[[#measure]](curr_frame, meas)함수로 optimization, marginalization 진행

### measure
<span style="color:green">OpticalFlowResult::Ptr <span style="color:purple">& opt_flow_meas</span>, IntegratedImuMeasurement(Scalar)::Ptr<span style="color:purple"> &meas</span> </span>
opt_flow_meas가 현재 시점에서의 observation 담고있는 거.

- [[#IntegratedImuMeasurement#predictState|IntergatedImuMeasurement::prdictState]] (`frame_states.at(last_state_t_ns).getState(), g, next_state`)함수를 통해 last_state_t_ns시점 기준으로 preintegration 값을 이용해서 pose를 구하고 이를 next_state로 저장.
- `last_state_t_ns` = `opt_flow_meas->t_ns`
- `next_state.t_ns` = `opt_flow_meas->t_ns`

`int` connected0 : 왼쪽 카메라에서 발견된 feature 개수
`std::map<int64_t, int>` num_points_connected
`std::unordered_set<int>` unconnected_obs0


- `opt_flow_meas`의 모든 observation에 대해 for loop
	- 한 observation에 있는 feature 별로 아래의 작업 수행
	
	- landmark에 속한 feature면 
		- host kf과 feature id를 `tcid_host`에 저장.
		- [[#KeypointObservation]] 을 local 변수 `kobs`로 선언. (for individual obs)
		- `lmdb`.[[#addObservation]](`tcid_target, kobs`)으로 [[#LandmarkDatabase]]의 `kpts` 변수와 `observations`변수에 추가해주기.
		- `num_points_connected`로 feature의 host_keyframe count +1
		- `connected` ++
	
	- Landmark에 없던  새로운 왼쪽에서 뽑힌 feature면 
		- `unconnected_obs0.emplace(kep_id)`

`connected0`과 `unconnected_obs0` 개수의 비율로 keyframe 여부 판단.

뭔가 다 왼쪽에서 새로 뽑힌 것만 triangulation하고 이럼. 

#### Triangulate 진행 
`kf_ids.emplace(last_state_t_ns)`로 keyframe으로 등록.

`for (int lm_id : unconnected_obs0)` 
- `std::map<TimeCamId, KeypointObservation<Scalar>>` kp_obs (로컬변수) 등록

triangulation 한 feature의 depth가 0과 3.0 사이에 있어야만 
- `lmdb.addLandmark(lm_id, kpt_pos)`로 landmark database에 넣어줌.
- `valid_kp = true`

`valid_kp`가 true인 feature들에 대해서 과거의 모든 위치를 넣어준다.
`lmdb.addObservation(kv_obs.first, kv_obs.second)`

`num_points_kf[opt_flow_meas->t_ns] = num_points_added` 
- 각 시점별로 추가된 landmark 개수를 저장하는 변수.
#### Triangulation 
만약 config에서 `vio_marg_lost_landmarks`를 true로 해놓았다면
- [[#LandmarkDatabase#getLandmarks|LandmarkDatabase::getLandmarks]]를 이용해서 현재 시점에 발견 안 된 landmark들 `lmdb`에서 제거.

[[#optimize_and_marg]](`num_points_connected, lost_landmarks`)
- `num_points_connected`는 landmark 중에서 host_keyframe 빈도수를 나타낸 거.

visualize를 위해 

### optimize_and_marg

## IntegratedImuMeasurement
preintegration.h에 있음.

### Constructor
`prev_frame`의 시간, gyro와 accel의 bias를 받아와서 초기화.

### Integrate


## OpticalFlowBase
### getOpticalFlow
<span style="color: green">const VioConfig<span style="color: purple">& config</span>, const Calibration(double)<span style="color: purple">& cam</span></span>

return <span style="color:red">OpticalFlowBase::Ptr</span>

config에서 optical_flow_type을 
- patch
- frame_to_frame
- multiscale_frame_to_frame
으로 했냐에 따라  맞는 객체를 return해줌.

ex ) `res.reset(new PatchOpticalFlow<float, Pattern24>(config, cam))`

### PatchOpticalFlow
#### Constructor
patch 크기에 따라 PatchT 변수 선언
`T_i_j`는 j번째→i번째 transform이니까 오른쪽→왼쪽 카메라 변환
그래서 Essential matrix `E` 구함.

#### ProcessingLoop
이거 Thread임.
[[#processFrame]] (input_ptr→t_ns, input_ptr)호출

#### processFrame
<span style="color:green"> int64_t <span style="color:purple">curr_t_ns</span> OpticalFlowInput::Ptr<span style="color:purple"> &new_img_vec</span></span>
⚠️ `calib.intrinsics.size()`는 camera 개수를 의미함.
##### 함수가 처음 돌아갈 때 초기화 해주는 작업
`transforms.reset(new` [[#OpticalFlowResult]]`)`
transforms의 observation 초기화, 시간(`t_ns`) 초기화 

`pyramid.reset(new std::vector<`[[#ManagedImagePyr|basalt::ManagedImagPyr]]`<uint16_t>>)`
`pyramid->at(i).`[[#setFromImage]](* new_img_vec→ img_data[i].img, config.optical_flow_levels) 함수를 통해 pyramid 구성

[[#addPoints]]
[[#filterPoints]]로 오른쪽 카메라의 feature를 왼쪽과 Epipolar geometry error 기준으로 삭제

1. output_queue 가 존재하고,
2. `frame_counter % config.optical_flow_skip_frames ==0` 
→ `output_queue->push(transforms)`
##### 함수가 처음 말고 그 이후로 돌아갈 때
1. 전 pyramid를 old로 바꾸고
2. [[#trackPoints]]함수로 opticalFlow가 [[#addPoints]], [[#filterPoints]] 이전에 들어가는 것만 다름.

`old_pyramid = pyramid`

`pyramid.reset(new std::vector<`[[#ManagedImagePyr|basalt::ManagedImagPyr]]`<uint16_t>>)`
`pyramid->at(i).`[[#setFromImage]](* new_img_vec→ img_data[i].img, config.optical_flow_levels) 함수를 통해 pyramid 구성

[[#OpticalFlowResult]]::Ptr new_transforms
`new_transforms.reset(new` [[#OpticalFlowResult]]`)`
transforms의 observation 초기화, 시간(`t_ns`) 초기화 

카메라 개수만큼 [[#trackPoints]](`old_pyramid->at(i), pyramid->at(i), transforms->observations[i], new_transforms->observations[i])

[[#addPoints]]
[[#filterPoints]]로 오른쪽 카메라의 feature를 왼쪽과 Epipolar geometry error 기준으로 삭제

1. output_queue 가 존재하고,
2. `frame_counter % config.optical_flow_skip_frames ==0` 
→ `output_queue->push(transforms)`

#### filterPoints
⚠️ 카메라 개수가 1개면 return을 해벌임;;

- 오른쪽에 있는 feature(`transforms->observations.at(1)`)가 왼쪽에도 있는지 확인
	- 만약 없으면
	- `proj0`에 해당 좌표 넣기
	- `proj1`에 해당 좌표 넣기
	- `kpid`에 해당 keypoint 넣기

- `calib.intrinsics[0].`[[#unproject]] `(proj0, p3d0, p3d0_success)`
- `calib.intrinsics[1].`[[#unproject]] `(proj1, p3d1, p3d1_success)`

- for loop : `p3d_success.size()` 
	- `p3d0, p3d1`모두 success인 것중에
		- 초기화 때 구했던 `E`(왼쪽-오른쪽 카메라 간 Essential Matrix)로 error 확인
			- `epipolar_error` =  $p_1^T E p_0$
		- `epipolar_error`가 config에서 설정한 `optical_flow_epipolar_error`보다 크면 `lm_to_remove.emplace(kpid[i])`
	- `p3d0, p3d1`둘 중 하나라도  success가 아니면
		- `lm_to_remove.emplace(kpid[i])`

`lm_to_remove`에 있는 keypoint들을 `transforms->observations.at(1).erase(id)`로 오른쪽 observation에서 삭제.

그럼 일단 왼쪽에서 뽑힌 거는 그냥 true에 가까운 measurement라고 생각하는 거네
### OpticalFlowResult
struct이고 `using KeypointId = size_t` 
`std::vector<Eigen::alinged_map<KeypointId, Eigen::AffineCompact2f>>` observations
`std::vector<std::map<KeypointId, size_t>>` pyramid_levels
`OpticalFlowInput::Ptr` input_images

### OpticalFlowInput
- int64_t t_ns
- std::vector<[[#ImageData]]> img_data

### ManagedImagePyr
이거는 thirdparty 디렉토리 안 image_pyr.h 파일 안에 있음 
대충 imagepyr를 만든다고 생각하자.

### ImageData
이건 io파일 안에 있음
- [[#ManagedImage]]<uint16_t>::Ptr img
- double exposure


## landmark_database.h
### LandmarkDatabase 
class
`Eigen::aligned_unordered_map<KeypointId`, [[#Keypoint]]`<Scalar>>` **kpts**
- `using` KeypointId = size_t (optical_flow.h)
- keypoint당 host_keyframe 및 observation을 모아둔 거 같은데

`std::unordred_map<TimeCamId, std::map<TimeCamId, std::set<KeypointId>>>` **observation**
- 첫 TimeCamId : host_kf_id
- 다음 TimeCamId : 발견된 시점
- KeypointId : feature id
#### addObservation 
<span style="color:green">const TimeCamId<span style="color:purple">& tcid_target</span>,  const KeypointObservation(Scalar)<span style="color:purple">&o </span></span>
KeypointObservation은 그냥 한 keypoint의 observation이라 생각하자.
tcid_target은 currframe 시간과 cam_id

그래서 kpts에서 내가 찾고싶은 keypoint가 있는지 찾아보고 
`kpts`의 Keypoint의 obs에 넣어줌 (`it->second.obs[tcid_target] = o.pos`)

`observations[it->second.host_kf_id][tcid_target].insert(it->first)` 
observation 변수에도 넣어줌.
### Keypoint
struct, 한 keypoint의 observation들을 가지고 있는 자료구조.

`using` Scalar, Vec2
`using` ObsMap = `Eigen::aligned_map<TimeCamId, Vec2>`
`using` MapIter = `typename ObsMap::iterator`

`Vec2` direction
`Scalar` inv_dist
`TimeCamId` host_kf_id
`ObsMap` obs

### KeypointObservation

struct
`int` kpt_id
`Eigen::Matrix<Scalar, 2, 1>` pos

## Common_types.h

`using`  FrameId = int64_t
`using` CamId = std::size_t
`using` FeatureTrack = std::map<TimeCamId, FeatureId>
`using` FeatureId = int

### TimeCamId
- [[#FrameId]] frame_id
- [[#CamId]] cam_id

### Landmark
- `Eigen::Vector3d ` p
- [[#FeatureTrack]] obs
- [[#FeatureTrack]] outlier_obs


# Pangolin
### ManagedImage
<span style="color: brown">public <span style="color: blue"> Image</span></span> <T> 