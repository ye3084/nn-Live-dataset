# nn-Live-dataset

This repository releases the dataset used in:
"nn-Live: Predicting Runtime Latency for Concurrent DNN Inference on Mobile Devices"

## Overview

We release an interference analysis dataset collected from three mainstream mobile SoCs.
The dataset contains runtime latency measurements of concurrent DNN inference under diverse execution conditions.

# Dataset Description

The dataset consists of two main components: **DNN model files** and **raw measurement data** collected from three commercial mobile devices. It covers DNN model profiling, runtime interference measurements, DVFS dynamics, and adaptive hierarchical sampling.

## 1. DNN Model Files

This directory contains the DNN models used in the profiling and evaluation of nn-Live.

- Model Variants/: Contains model variants used in the experimental evaluation. It includes four model families:
  - ResNet34_variants_tflite/
  - MobileNetV3Small_variants_tflite/
  - MobileNetV3Large_variants_tflite/
  - EfficientNetLite0_variants_tflite/
    Each family contains five model variants corresponding to different FLOPs percentiles (5%, 25%, 50%, 75%, and 95%), which are used in Protocol C (NAS-Variant Testing).
- 24 Base DNN Model/: Contains 24 representative DNN models used as the profiling model pool for the adaptive hierarchical sampling strategy described in Section 4. These models cover diverse neural network architectures and are used to characterize model runtime behaviors and construct the profiling dataset for latency prediction.

## 2. Raw Measurement Data

This directory contains raw experimental data collected from three mobile devices with different SoC architectures:

- Xiaomi Pad 6 Pro (Snapdragon 8+ Gen 1)/
- Pixel Tablet (Google Tensor G2)/
- OnePlus Ace 5 Pro (Snapdragon 8 Elite)/

  Each device directory contains the following six subdirectories:
- Throttled Case/: Measurements collected under frequency-constrained or throttled operating conditions.
- Task Profile/: Profiling data for individual DNN models, including task signatures and reference inference latencies measured on different CPU core types under controlled, contention-free conditions.
- Model Variants/: Runtime measurement data collected using the DNN model variants as the main tasks. These data are used for Protocol C (NAS-Variant Testing) to evaluate the generalization of nn-Live across different model variants.
- DVFS Dynamics/: Multiple sets of long-running measurements collected to construct the frequency prediction function (g(\cdot)) described in Section 3.3. Each measurement run lasts 5 minutes under high-stress workloads, capturing the dynamic behavior of CPU frequency and temperature.
- Core Model/: Measurement data collected using the 10 representative DNN models in Section 2. These data support the measurement study and analysis of runtime interference under different concurrent workload configurations and CPU core placements.
- Adaptive Sampling/: Runtime measurement samples collected using the adaptive hierarchical sampling strategy described in Section 4. These data are used to construct the resource-contention function and the latency prediction model.

# Directory Structure

```
Dataset/
│
├── DNN MODEL/
│   ├── Model Variants/
│   │   ├── ResNet34_variants_tflite/
│   │   ├── MobileNetV3Small_variants_tflite/
│   │   ├── MobileNetV3Large_variants_tflite/
│   │   └── EfficientNetLite0_variants_tflite/
│   │
│   └── 24 Base DNN Model/
│
└── raw data/
    │
    ├── Xiaomi Pad 6 Pro (Snapdragon 8+ Gen 1)/
    │   ├── Throttled Case/
    │   ├── Task Profile/
    │   ├── Model Variants/
    │   ├── DVFS Dynamics/
    │   ├── Core Model/
    │   └── Adaptive Sampling/
    │
    ├── Pixel Tablet (Google Tensor G2)/
    │   └── [Same six subdirectories]
    │
    └── OnePlus Ace 5 Pro (Snapdragon 8 Elite)/
        └── [Same six subdirectories]
```

# Raw Data Organization and File Descriptions

The raw measurement data is organized by device, experimental category, and sampling configuration. Each secondary directory may directly contain sample directories or be further divided into multiple batch subdirectories. The batch directory names are arbitrary identifiers used only to distinguish different data collection batches; they do not represent specific experimental configurations.

### 1. Directory Organization

Each secondary directory contains one or more sample directories, either directly or within batch subdirectories. Each sample directory represents one complete sampling process under a specific experimental configuration.

The directory structure is organized as follows:

```text
raw data/
│
└── <Device>/
    │
    └── <Experimental Category>/
        │
        ├── <Batch Directory>/           (Optional)
        │   ├── <Sample Directory 1>/
        │   ├── <Sample Directory 2>/
        │   └── ...
        │
        ├── <Sample Directory 3>/        (May also exist directly)
        └── ...
```

### 2. Sample Directory Naming Convention

Each sample directory is named according to its sampling configuration, indicating the background tasks, main task, and CPU core assignments.

The naming convention follows this general pattern:

```text
t_<Background Task IDs>_m_<Main Task ID>_cpu[<Background CPU IDs>][<Main CPU Sequence>]
```

The components are explained below:

- **`t`**: Identifies the background tasks running concurrently with the main task.
- **`<Background Task IDs>`**: The identifiers of the background DNN models. Each background task is assigned to and exclusively occupies one CPU core.
- **`m`**: Identifies the main DNN task whose inference latency is being measured.
- **`<Main Task ID>`**: The identifier of the main DNN model.
- **First CPU bracket `[ ]`**: Specifies the CPU core assignments of the background tasks. The CPU IDs correspond to the background task IDs in the same order.
- **Second CPU bracket `[ ]`**: Specifies the sequence of CPU cores on which the main task is executed. The main task runs exclusively on each specified core in sequence, allowing its inference latency to be measured across different CPU cores under the same background workload configuration.

All CPU IDs refer to the logical CPU core indices of the mobile device, ranging from 0 to 7.

**Example:**

```text
t_2_4_m_19_cpu[65][740]
```

This directory name represents the following sampling configuration:

| Component    | Meaning                                                                |
| ------------ | ---------------------------------------------------------------------- |
| `t_2_4`    | Two background tasks: DNN models 2 and 4                               |
| `cpu[65]`  | Background task 2 runs on CPU 6, while background task 4 runs on CPU 5 |
| `m_19`     | The main task is DNN model 19                                          |
| `cpu[740]` | The main task is executed sequentially on CPU 7, CPU 4, and CPU 0      |

During this sampling process, the two background tasks remain assigned to their respective CPU cores. Meanwhile, the main task is executed sequentially on CPU 7, CPU 4, and CPU 0. Its inference latency is collected at each placement under the same background workload environment.

This naming convention allows the experimental configuration of each sample to be identified directly from its directory name, without inspecting the measurement files.

### 3. Files in Each Sample Directory

Each sample directory typically contains eight files, including the original system trace, raw PMU statistics, and processed measurement data.

| File                        | Description                                                                                                                                                                                           |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `*.perfetto-trace`        | The system tracing file recorded during the sampling process, containing detailed runtime events collected using Perfetto.                                                                            |
| `*_perf.txt`              | Raw per-core PMU statistics collected from all eight CPU cores while the background workloads are running.                                                                                            |
| `*_perf.xlsx`             | An Excel representation of the PMU statistics converted from the corresponding `*_perf.txt` file.                                                                                                   |
| `*_meminfo.xlsx`          | System memory information extracted from the `*.perfetto-trace` file during background workload execution.                                                                                          |
| `*_invoke_infertime.xlsx` | Inference latency measurements of the main task when executed on different CPU cores, extracted from the `*.perfetto-trace` file.                                                                   |
| `*_cpuidle.xlsx`          | CPU idle-state information extracted from the `*.perfetto-trace` file during background workload execution.                                                                                         |
| `*_cpufreq.xlsx`          | CPU frequency transition data extracted from the `*.perfetto-trace` file during background workload execution.                                                                                      |
| `*_cpu_invoke_count.json` | Records the number of main-task invocations on each CPU core. Combined with `*_invoke_infertime.xlsx`, it can be used to calculate the average inference latency of the main task on each CPU core. |

**Notes on Data Usage:**

- The `*.perfetto-trace` file is the original system trace and serves as the source for extracting multiple types of runtime information.
- The `*_perf.txt` and `*_perf.xlsx` files provide per-core PMU statistics, enabling analysis of hardware resource utilization and interference from concurrent workloads.
- The `*_meminfo.xlsx`, `*_cpuidle.xlsx`, and `*_cpufreq.xlsx` files provide complementary system-level information, including memory usage, CPU idle states, and frequency dynamics.
- The `*_invoke_infertime.xlsx` and `*_cpu_invoke_count.json` files jointly describe the main task's inference latency across different CPU core placements, supporting per-core latency analysis.



# Dataset License

The nn-Live dataset is released under the Open Use of Data Agreement v1.0 (O-UDA-1.0).


# Citation

If you find that our work helps your research, please consider citing it.
