# VERA dataset

This dataset has been used to design and evaluate the VERA framework proposed in:

```
S. Tripathi, C. Puligheddu, S. Pramanik, A. Garcia-Saavedra and C. F. Chiasserini, "Fair and Scalable Orchestration of Network and Compute Resources for Virtual Edge Services," in IEEE Transactions on Mobile Computing, doi: 10.1109/TMC.2023.3254999.
```
If you use this dataset please consider citing our paper.
## Testing environment

The attached dataset is built using 2 applications at the same time: 
- a radio application - namely srsRAN - that is in charge of managing the LTE link, and 
- a video streaming application, that transcodes a live video feed and transmits it to the client through the LTE network.

The applications are colocated in an Edge host equipped with a 4-core@2.8GHz Intel i7-7700HQ CPU and 16 GB of 2400 MHz DDR4 memory.

## Test parameters

The test parameter set is given by the cartesian product of the following sets:

```
parameters.radio_cpu = [1, 0.9, 0.8, 0.7, 0.6, 0.5, 0.4];
parameters.video_cpu = [2, 1.75, 1.5, 1.4, 1.3, 1.2, 1.1, 1, 0.9, 0.8];
parameters.rbs = [18];
parameters.mcs = [0, 9, 18, 27];
parameters.tx_power = [14, 8, 5, 2, -1, -4, -7, -10, -13];
parameters.out_video_resolution = 720;
parameters.out_video_bitrate = [300, 800, 1500, 5000, 10000];
parameters.out_video_framerate = [10, 20, 30];
```

Note: the radio_cpu parameter 0.4 is not truly tested, it is added as a lower limit to the CPU allocation in order to improve the performance of the RL framework in dealing with bad actions.
The reason for this addition is that the current dataset doesn't include a sufficient amount of bad actions to drive the learning phase of the framework.

Each test is 25 seconds long, the metrics are sampled once each 100 ms.

Results are further filtered to retain only the samples that are generated after the start of the playback. Additionally, in case the transmission of video packets halts due to radio failure, the samples after this event are also discarded. Therefore each test duration may be lower than the mentioned 25 secs.

Furthermore, the dataset is then downsampled at 1 s periodicity.

## Dataset fields
The dataset fields are listed and explained below.

### General fields:
```
'id' experiment unique identifier
'monitoring_slot' timestamp of the sampling instant. Experiment duration is variable.
'tx_power' approximation of the tx power of the USRP. It's equal to the TX_gain - 73. It is kept constant for each stage.
```

### Context metrics:
```
'ctx_radio_throttled_time_norm' radio throttled time normalized wrt the available cpu time of the CPU
'ctx_radio_snr_db' SNR in dB reported from the UE to the BS through the CQI. This is direct result of the chosen tx power.
'ctx_radio_offered_load_kbps' traffic sent through the radio link. Calculated using the actual output bitrate of the video encoder
'ctx_video_throttled_time_norm' video throttled time normalized wrt the available cpu time of the CPU
'ctx_video_inp_bitrate_kbps' bitrate of the video that is given as input to the transcoder. Constant for the entire dataset.
'ctx_video_inp_framerate_fps' framerate of the input video. Constant.
'ctx_video_inp_vertical_resolution_px' resolution of the input video. Constant. To decouple it from the aspect ratio, total number of pixels (horizontal resolution by vertical resolution) could be considered instead.
```

### Actions:
```
'act_radio_cpu_limit' cpu limit of the radio application set in docker. Some threads run with real time priority, so they are not constrained by this limit.
'act_radio_link' link utilized. For this dataset it is always 'LTE'.
'act_radio_mcs_max' Max MCS available for the pdsch. srsLTE is free to chose any mcs below this limit according to the SNR and buffer status.
'act_video_cpu_limit' cpu limit of the video application.
'act_video_out_bitrate_kbps' output video bitrate set as encoding parameter and that the video encoder targets. In edge cases (eg. very low framerate or resolution), the actual value may differ.
'act_video_out_framerate_fps' output video framerate. This is an encoder parameter, NOT an indicator of the speed of the encoder.
```

### Performance metrics:
```
'kpi_radio_latency_ms' unidir latency eNB->UE, measured using a single probe (low overhead) for each monitoring slot
'kpi_radio_segment_error_rate' computed as the fraction of tcp retransmission over sent segment. 0 in case of 0 segment sent.
'kpi_video_wvmaf' the wvmaf score of the encoded video, derived using the mobile phone model. From 0 to 100. It is weighted according to the relative framerate. wvamf = vmaf * output_framerate / input_framerate.
'kpi_video_client_buffer_s' the inferred value of the client buffer occupation in seconds. The target value is 1, if it rises above 1 (may happen only if the player stops first), then the e2e delay rises. If it gets close to 0, there is the risk of buffer underrun (which happens when buffer empties), which means that the client has to stop the playback and the user experience gets worse.
```