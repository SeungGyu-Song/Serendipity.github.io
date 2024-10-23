# basalt
### feed_images
[[#OpticalFlowBase]]의 객체가 뽑아 쓸 queue에 이미지를 넣어주는 역할

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

#### initialize 
<span style="color:green">const Eigen::Vector3d& <span style="color:purple">bg_</span>, const Eigen::Vector3d& <span style="color:purple">ba_</span></span>



### Constructor
- config파일에서 option 값 불러오기.
- config에서 vio_sqrt_marg == true로 하면
	- marg_data에 들어가는 값들에 sqrt를 씌워줘서 넣어준다.

### initialize
<span style="color:green">const Eigen::Vector3d<span style="color:purple">& bg_</span>, const Eigen::Vector3d<span style="color:purple">& ba_</span></span>


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




