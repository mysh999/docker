## 一、问题描述

操作系统是ubuntu，执行GPU监控插件报

```bash
# docker run -d --gpus all --rm -p 9400:9400 nvidia/dcgm-exporter:2.0.13-2.1.1-ubuntu18.04
Unable to find image 'nvidia/dcgm-exporter:2.0.13-2.1.1-ubuntu18.04' locally
2.0.13-2.1.1-ubuntu18.04: Pulling from nvidia/dcgm-exporter
171857c49d0f: Pull complete 
419640447d26: Pull complete 
61e52f862619: Pull complete 
2a93278deddf: Pull complete 
c9f080049843: Pull complete 
8189556b2329: Pull complete 
293c994cc6c2: Pull complete 
f79d1a4211c3: Pull complete 
fe75137a11ed: Pull complete 
35772a4b9159: Pull complete 
fdd8c9ae911c: Pull complete 
Digest: sha256:31ac69add9788b12f7635d1af23a51b8d740d897a7d4050568190ad8ff6a9a5d
Status: Downloaded newer image for nvidia/dcgm-exporter:2.0.13-2.1.1-ubuntu18.04
b10c2ea0546082af3fd5a120cae1662045d48652e5e973815a87f33ca4c08e6c
docker: Error response from daemon: could not select device driver "" with capabilities: [[gpu]].
```



## 二、解决办法

```bash
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

$ sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
$ sudo systemctl restart docker
```

 再次执行docker run命令OK



## 三、验证

```bash
# curl localhost:9400/metrics
# HELP DCGM_FI_DEV_SM_CLOCK SM clock frequency (in MHz).
# TYPE DCGM_FI_DEV_SM_CLOCK gauge
# HELP DCGM_FI_DEV_MEM_CLOCK Memory clock frequency (in MHz).
# TYPE DCGM_FI_DEV_MEM_CLOCK gauge
# HELP DCGM_FI_DEV_MEMORY_TEMP Memory temperature (in C).
# TYPE DCGM_FI_DEV_MEMORY_TEMP gauge
# HELP DCGM_FI_DEV_GPU_TEMP GPU temperature (in C).
# TYPE DCGM_FI_DEV_GPU_TEMP gauge
# HELP DCGM_FI_DEV_POWER_USAGE Power draw (in W).
# TYPE DCGM_FI_DEV_POWER_USAGE gauge
# HELP DCGM_FI_DEV_TOTAL_ENERGY_CONSUMPTION Total energy consumption since boot (in mJ).
# TYPE DCGM_FI_DEV_TOTAL_ENERGY_CONSUMPTION counter
# HELP DCGM_FI_DEV_PCIE_TX_THROUGHPUT Total number of bytes transmitted through PCIe TX (in KB) via NVML.
# TYPE DCGM_FI_DEV_PCIE_TX_THROUGHPUT counter
# HELP DCGM_FI_DEV_PCIE_RX_THROUGHPUT Total number of bytes received through PCIe RX (in KB) via NVML.
# TYPE DCGM_FI_DEV_PCIE_RX_THROUGHPUT counter
# HELP DCGM_FI_DEV_PCIE_REPLAY_COUNTER Total number of PCIe retries.
# TYPE DCGM_FI_DEV_PCIE_REPLAY_COUNTER counter
# HELP DCGM_FI_DEV_GPU_UTIL GPU utilization (in %).
# TYPE DCGM_FI_DEV_GPU_UTIL gauge
# HELP DCGM_FI_DEV_MEM_COPY_UTIL Memory utilization (in %).
# TYPE DCGM_FI_DEV_MEM_COPY_UTIL gauge
# HELP DCGM_FI_DEV_ENC_UTIL Encoder utilization (in %).
# TYPE DCGM_FI_DEV_ENC_UTIL gauge
# HELP DCGM_FI_DEV_DEC_UTIL Decoder utilization (in %).
# TYPE DCGM_FI_DEV_DEC_UTIL gauge
# HELP DCGM_FI_DEV_XID_ERRORS Value of the last XID error encountered.
# TYPE DCGM_FI_DEV_XID_ERRORS gauge
# HELP DCGM_FI_DEV_POWER_VIOLATION Throttling duration due to power constraints (in us).
# TYPE DCGM_FI_DEV_POWER_VIOLATION counter
# HELP DCGM_FI_DEV_THERMAL_VIOLATION Throttling duration due to thermal constraints (in us).
# TYPE DCGM_FI_DEV_THERMAL_VIOLATION counter
# HELP DCGM_FI_DEV_SYNC_BOOST_VIOLATION Throttling duration due to sync-boost constraints (in us).
# TYPE DCGM_FI_DEV_SYNC_BOOST_VIOLATION counter
# HELP DCGM_FI_DEV_BOARD_LIMIT_VIOLATION Throttling duration due to board limit constraints (in us).
# TYPE DCGM_FI_DEV_BOARD_LIMIT_VIOLATION counter
# HELP DCGM_FI_DEV_LOW_UTIL_VIOLATION Throttling duration due to low utilization (in us).
# TYPE DCGM_FI_DEV_LOW_UTIL_VIOLATION counter
# HELP DCGM_FI_DEV_RELIABILITY_VIOLATION Throttling duration due to reliability constraints (in us).
# TYPE DCGM_FI_DEV_RELIABILITY_VIOLATION counter
# HELP DCGM_FI_DEV_FB_FREE Framebuffer memory free (in MiB).
# TYPE DCGM_FI_DEV_FB_FREE gauge
# HELP DCGM_FI_DEV_FB_USED Framebuffer memory used (in MiB).
# TYPE DCGM_FI_DEV_FB_USED gauge
# HELP DCGM_FI_DEV_ECC_SBE_VOL_TOTAL Total number of single-bit volatile ECC errors.
# TYPE DCGM_FI_DEV_ECC_SBE_VOL_TOTAL counter
# HELP DCGM_FI_DEV_ECC_DBE_VOL_TOTAL Total number of double-bit volatile ECC errors.
# TYPE DCGM_FI_DEV_ECC_DBE_VOL_TOTAL counter
# HELP DCGM_FI_DEV_ECC_SBE_AGG_TOTAL Total number of single-bit persistent ECC errors.
# TYPE DCGM_FI_DEV_ECC_SBE_AGG_TOTAL counter
# HELP DCGM_FI_DEV_ECC_DBE_AGG_TOTAL Total number of double-bit persistent ECC errors.
# TYPE DCGM_FI_DEV_ECC_DBE_AGG_TOTAL counter
# HELP DCGM_FI_DEV_RETIRED_SBE Total number of retired pages due to single-bit errors.
# TYPE DCGM_FI_DEV_RETIRED_SBE counter
# HELP DCGM_FI_DEV_RETIRED_DBE Total number of retired pages due to double-bit errors.
# TYPE DCGM_FI_DEV_RETIRED_DBE counter
# HELP DCGM_FI_DEV_RETIRED_PENDING Total number of pages pending retirement.
# TYPE DCGM_FI_DEV_RETIRED_PENDING counter
# HELP DCGM_FI_DEV_NVLINK_CRC_FLIT_ERROR_COUNT_TOTAL Total number of NVLink flow-control CRC errors.
# TYPE DCGM_FI_DEV_NVLINK_CRC_FLIT_ERROR_COUNT_TOTAL counter
# HELP DCGM_FI_DEV_NVLINK_CRC_DATA_ERROR_COUNT_TOTAL Total number of NVLink data CRC errors.
# TYPE DCGM_FI_DEV_NVLINK_CRC_DATA_ERROR_COUNT_TOTAL counter
# HELP DCGM_FI_DEV_NVLINK_REPLAY_ERROR_COUNT_TOTAL Total number of NVLink retries.
# TYPE DCGM_FI_DEV_NVLINK_REPLAY_ERROR_COUNT_TOTAL counter
# HELP DCGM_FI_DEV_NVLINK_RECOVERY_ERROR_COUNT_TOTAL Total number of NVLink recovery errors.
# TYPE DCGM_FI_DEV_NVLINK_RECOVERY_ERROR_COUNT_TOTAL counter
# HELP DCGM_FI_DEV_NVLINK_BANDWIDTH_TOTAL Total number of NVLink bandwidth counters for all lanes
# TYPE DCGM_FI_DEV_NVLINK_BANDWIDTH_TOTAL counter


DCGM_FI_DEV_SM_CLOCK{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 885
DCGM_FI_DEV_MEM_CLOCK{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 2999
DCGM_FI_DEV_GPU_TEMP{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 41
DCGM_FI_DEV_POWER_USAGE{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 23.528000
DCGM_FI_DEV_TOTAL_ENERGY_CONSUMPTION{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 31035686671
DCGM_FI_DEV_PCIE_REPLAY_COUNTER{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_GPU_UTIL{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_MEM_COPY_UTIL{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_ENC_UTIL{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_DEC_UTIL{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_XID_ERRORS{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_POWER_VIOLATION{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_THERMAL_VIOLATION{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_SYNC_BOOST_VIOLATION{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_FB_FREE{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 7064
DCGM_FI_DEV_FB_USED{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 1057
DCGM_FI_DEV_ECC_SBE_VOL_TOTAL{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_ECC_DBE_VOL_TOTAL{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_ECC_SBE_AGG_TOTAL{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_ECC_DBE_AGG_TOTAL{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_RETIRED_SBE{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_RETIRED_DBE{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_RETIRED_PENDING{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
DCGM_FI_DEV_NVLINK_BANDWIDTH_TOTAL{gpu="0", UUID="GPU-8c16e996-bd76-aa91-17e7-b48bcd0208ba", device="nvidia0"} 0
```

