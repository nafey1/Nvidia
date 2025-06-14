# Install CUDA, Nvidia Container Runtime and Docker for Running NMI

## Expand the Root Partition, as CUDA requires 5+ GB of Storage
    sudo /usr/libexec/oci-growfs -y


## Verify you have a CUDA-Capable GPU
    lspci | grep -i nvidia
    00:04.0 3D controller: NVIDIA Corporation GA102GL [A10] (rev a1)

If NO output is shown in the previous step, perform the step below and repeat

    sudo update-pciids

## Verify the System Has gcc Installed
    gcc --version
    gcc (GCC) 11.5.0 20240719 (Red Hat 11.5.0-5.0.1)
    Copyright (C) 2021 Free Software Foundation, Inc.
    This is free software; see the source for copying conditions.  There is NO
    warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

## Download CUDA Toolkit for OEL 9.x
> Check the Link below to downlaod and install the latest toolkit

    sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel9/x86_64/cuda-rhel9.repo

    sudo dnf clean all

    sudo dnf -y install cuda-toolkit-12-9

### NVIDIA Driver Installation
    sudo dnf config-manager --set-enabled ol9_developer_EPEL

    sudo dnf install -y dkms 
    
    sudo dnf -y module install nvidia-driver:latest-dkms
    
    sudo dnf config-manager --set-disabled ol9_developer_EPEL

### Post Installation

    echo export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/cuda-12.9/lib64 >>~/.bash_profile

    echo export PATH=${PATH}:/usr/local/cuda-12.9/bin >>~/.bash_profile

    source ~/.bash_profile

### Reboot the Server
    sudo reboot


### Check the CUDA Installation Version
    $ nvcc --version
    nvcc: NVIDIA (R) Cuda compiler driver
    Copyright (c) 2005-2025 NVIDIA Corporation
    Built on Wed_Apr__9_19:24:57_PDT_2025
    Cuda compilation tools, release 12.9, V12.9.41
    Build cuda_12.9.r12.9/compiler.35813241_0

```
$ nvidia-smi
Sat May 24 20:15:07 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 570.133.20             Driver Version: 570.133.20     CUDA Version: 12.8     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA A10                     Off |   00000000:00:04.0 Off |                    0 |
|  0%   22C    P8              8W /  150W |       0MiB /  23028MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```


### Verify the Installation
Download the CUDA Samples from the Link below

    sudo yum install cmake git -y
    git clone https://github.com/NVIDIA/cuda-samples.git

    

Navigate to the root of the cloned repository

    cd $HOME/cuda-samples

Create a build directory

    mkdir build && cd build

Configure the project with CMake:

    cmake ..

Build the samples:

    make -j$(nproc)

Now, return to the samples root directory and run the test script:

    python3 run_tests.py --output ./test --dir ./build/Samples --config test_args.json

Inspect the Run Summary

    Test Summary:
    Ran 146 test runs for 147 executables.
    All test runs passed!


Running the Binaries from the samples root directory

```
./build/Samples/1_Utilities/deviceQuery/deviceQuery
./build/Samples/1_Utilities/deviceQuery/deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "NVIDIA A10"
  CUDA Driver Version / Runtime Version          12.8 / 12.9
  CUDA Capability Major/Minor version number:    8.6
  Total amount of global memory:                 22599 MBytes (23696375808 bytes)
  (072) Multiprocessors, (128) CUDA Cores/MP:    9216 CUDA Cores
  GPU Max Clock rate:                            1695 MHz (1.70 GHz)
  Memory Clock rate:                             6251 Mhz
  Memory Bus Width:                              384-bit
  L2 Cache Size:                                 6291456 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)
  Maximum Layered 1D Texture Size, (num) layers  1D=(32768), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(32768, 32768), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total shared memory per multiprocessor:        102400 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  1536
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
  Run time limit on kernels:                     No
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Enabled
  Device supports Unified Addressing (UVA):      Yes
  Device supports Managed Memory:                Yes
  Device supports Compute Preemption:            Yes
  Supports Cooperative Kernel Launch:            Yes
  Supports MultiDevice Co-op Kernel Launch:      Yes
  Device PCI Domain ID / Bus ID / location ID:   0 / 0 / 4
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 12.8, CUDA Runtime Version = 12.9, NumDevs = 1
Result = PASS
```

## Install the Proprietary Kernel Module Flavor

dnf config-manager --set-enabled ol8_codeready_builder


    sudo dnf -y module install nvidia-driver:latest-dkms


## Install Nvidia Container Runtime
Configure the production repository

    curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo | \
    sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo

## Install the NVIDIA Container Toolkit packages:
    sudo dnf install -y nvidia-container-toolkit

## Configure the container runtime by using the nvidia-ctk command
    sudo nvidia-ctk runtime configure --runtime=docker

    sudo systemctl restart docker.service

## Install Docker

    sudo wget https://download.docker.com/linux/centos/docker-ce.repo -P /etc/yum.repos.d/

    sudo yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

    sudo systemctl enable docker.service --now

    sudo usermod -aG docker opc

## Run llama3-8b-instruct from Docker

Login to Docker

    docker login nvcr.io
    Username: $oauthtoken
    Password: <PASTE_API_KEY_HERE>

Pull and run the NVIDIA NIM with the command below. This will download the optimized model for your infrastructure

    export NGC_API_KEY=<PASTE_API_KEY_HERE>
    export LOCAL_NIM_CACHE=~/.cache/nim
    mkdir -p "$LOCAL_NIM_CACHE"
    docker run -it --rm \
        --gpus all \
        --shm-size=16GB \
        -e NGC_API_KEY \
        -v "$LOCAL_NIM_CACHE:/opt/nim/.cache" \
        -u $(id -u) \
        -p 8000:8000 \
        nvcr.io/nim/meta/llama3-8b-instruct:1.0.0

You can now make a local API call using this curl command:

    curl -X 'POST' \
    'http://0.0.0.0:8000/v1/chat/completions' \
    -H 'accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{
        "model": "meta/llama3-8b-instruct",
        "messages": [{"role":"user", "content":"Write a limerick about the wonders of GPU computing."}],
        "max_tokens": 64
    }'

# Links
[NVIDIA CUDA Installation Guide for Linux](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/)

[Download CUDA](https://developer.nvidia.com/cuda-downloads)

[CUDA Samples](https://github.com/nvidia/cuda-samples)

[Installing the NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

[Downlaond Nvidia Models](https://build.nvidia.com/models)