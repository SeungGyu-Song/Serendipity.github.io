# basalt
### feed_images
[[#OpticalFlowBase]]의 객체가 뽑아 쓸 queue에 이미지를 넣어주는 역할
`opt_flow_ptr->intput_queue.push(data)`
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

#### Constructor
- config파일에서 option 값 불러오기.
- config에서 vio_sqrt_marg == true로 하면
	- marg_data에 들어가는 값들에 sqrt를 씌워줘서 넣어준다.

#### initialize
<span style="color:green">const Eigen::Vector3d<span style="color:purple">& bg_</span>, const Eigen::Vector3d<span style="color:purple">& ba_</span></span>

Thread에 넣어줌 : `processing_thread.reset(new std::thread(proc_func))`

imu의 acc, gyr covariance를 받아오고, 각각의 bias와 scale값을 받아옴.
- 따라서 결국 raw_measurement + scale * raw_measurement - bias 값을 가져옴.
##### while (true)
`vision_data_queue.pop(current_frame)` : optical flow의 결과를 뽑아옴.

- data = [[#popFromImuDataQueue]]
- 중력과 imu의 acc를 기준으로 world와 body의 초기 회전관계를 정의함 → static 상황을 가정하는 건가?
- 

###### popFromImuDataQueue
imu_data_queue에서 최근 거를 빼서 [[#ImuData]]에 저장함.
이 때, 우리가 사용하는 floating point가 double이면 그냥 뱉어주고, 아니면 float으로 cast를 통해 바꿔서 뱉어줌.

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

그럼 일단 왼쪽에서 뽑힌 거는 그냥 true에 가까운 meausrement라고 생각하는 거네
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



# Pangolin
### ManagedImage
<span style="color: brown">public <span style="color: blue"> Image</span></span> <T> 