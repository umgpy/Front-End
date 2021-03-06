# Application Setup

## Table of Contents
- [Requirements](#requirements)
- [Set up the Environment](#set-up-the-environment)
- [Set up the Collaborator](#set-up-the-collaborator)

## Requirements

- OS: Linux (Ubuntu 18.04) [note that Ubuntu 20.04 does **not** work, and we will support Windows for Phase-2]
- Python 3.6+: we have tested on [Python distributed by Anaconda](https://www.anaconda.com/products/individual) 3.6, 3.7
Example installation on Ubuntu 18.04:
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.6 python3.6-venv python3.6-dev
```
- GPU: for faster training and inference
  - [CUDA 9.2 - 10.2](https://developer.nvidia.com/cuda-toolkit)
    - **Note**: CUDA 11 is currently _not_ supported
  - 11GB dedicated VRAM
  - 40GB RAM (**Note**: 120G if you want to run [DeepScan](https://doi.org/10.1007/978-3-030-11726-9_40) inference)
- Read/write access to the data for processing
- Data requirements: 
  - Glioblastoma (GBM) patients
  - 4 structural modalities - T1 pre and post contrast, T2 and T2-Flair
  - No instrumentation, pre-resection
  - Scanned along axial axis
  - At least 90 cases
  - Tumor regions (as defined in BraTS challenge):
    -	Edematous/Invaded tissue + Non-Enhancing-Tumor - ED
    - Enhancing - ET
    - Necrotic tumor core - NET

[Back To Top &uarr;](#table-of-contents)

## Set up the Environment

- Download the pre-built binaries from [this link](https://github.com/FETS-AI/Front-End/releases/tag/0.0.2)
- Run the following commands:
```bash
sudo apt-get install wget zip unzip # required to download and unzip model weights
cd ${download_location}
chmod +x ./FeTS_${version}.bin # optional addition of execution permission
./FeTS_${version}_Installer.bin --target ${install_path} # change ${install_path} to appropriate location
# accept license
cd ${install_path}/squashfs-root/usr/ # this is the ${fets_root_dir}
cd bin/OpenFederatedLearning
nvidia-smi # note the ${cuda_version}
# install pytorch using appropriate ${cuda_version}; see https://pytorch.org/get-started/locally/
./venv/bin/pip install torch torchvision # installs latest stable pytorch using CUDA 10.2 (we have tested with 1.7.1 with cuda-10.2)
# for cuda 9.2, this would be './venv/bin/pip install torch==1.7.1+cu92 torchvision==0.8.2+cu92 torchaudio==0.7.2 -f https://download.pytorch.org/whl/torch_stable.html'
# for cuda 10.1, this would be './venv/bin/pip install torch==1.7.1+cu101 torchvision==0.8.2+cu101 torchaudio==0.7.2 -f https://download.pytorch.org/whl/torch_stable.html'
```
### Note for Ubuntu 20.04 users

We have **not** tested with Ubuntu 20.04 and there might be unforseen stability issues with dependencies. That being said, if there is no other way around it, there are some pointers that can be followed to get FeTS up and running on this platform:

```bash
echo "deb http://archive.ubuntu.com/ubuntu xenial main" | sudo tee /etc/apt/sources.list.d/xenial.list
sudo apt-get update
sudo update-alternatives --remove-all gcc
sudo update-alternatives --remove-all g++
sudo apt-get install gcc-5 g++-5
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 10
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-5 10
```

### Optional instructions for Federation backend

These commands are run along with the installer, but in case you receive an error during the python environment setup, please follow these instructions:
```bash
cd ${fets_root_dir}
cd bin/OpenFederatedLearning
make install_openfl 
./venv/bin/pip install opencv-python==4.2.0.34 # https://stackoverflow.com/a/63669919/1228757
make install_fets
# after this, the federated backend is ready
./venv/bin/pip install -e ./submodules/fets_ai/Algorithms/GANDLF # gandlf
./venv/bin/pip install -e ../LabelFusion # label fusion
```

[Back To Top &uarr;](#table-of-contents)

## Set up the Collaborator

- Run the following commands to generate the [Certificate Signing Request (CSR)](https://en.wikipedia.org/wiki/Certificate_signing_request):
```bash
cd ${fets_root_dir} # see previous step for where this would be
cd bin/OpenFederatedLearning/bin/federations/pki
bash create-collaborator.sh ${collaborator_common_name} # keep a note of the ${collaborator_common_name}, as it will be used for authentication and to send/receive jobs to/from the aggregator at UPenn
# this command will generate the following items, which needs to be saved for collaborator verification:
## a CSR file at `./client/${collaborator_common_name}.csr``
## a string of alpha-numeric numbers for hash verification - SAVE THIS FOR VERIFICATION!!
```

- Send the CSR file (`${fets_root_dir}/bin/OpenFederatedLearning/bin/federations/pki/client/${collaborator_common_name}.csr`), and **not the verificiation hash**, to **admin@fets.ai**.
- The FeTS Team will reach back to set up a call, and you will need to provide the verification hash.
  - *For FeTS Team ONLY*: `bash sign-csr.sh ${collaborator_common_name}.csr ${hashVerification}`
- FeTS Team will send the following file back: `${collaborator_common_name}.crt`
- Copy this file to the following location: `${fets_root_dir}/bin/OpenFederatedLearning/bin/federations/pki/client/`
- (**Verification**) This location should contain the following files:
  - `${collaborator_common_name}.crt`
  - `${collaborator_common_name}.csr`
  - `${collaborator_common_name}.key`

[Back To Top &uarr;](#table-of-contents)

---
[Next: Process Data](./process_data.md)

---
