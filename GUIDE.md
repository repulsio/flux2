# Run `flux2` in Cloud

This is working on the latest `flux2` commit [`ab7cca68018ad3ceadcace9d6ecb1bc1f6f46b4e`](https://github.com/black-forest-labs/flux2/commit/ab7cca68018ad3ceadcace9d6ecb1bc1f6f46b4e).

<br/>

## HyperStack

You can run a `H100-80G-PCIe` (80 GB VRAM) VM on **HyperStack** for **$1.90/hour** with the following specs:

- 1 GPU
- 28 CPUs
- 180 GB RAM
- 100 GB Disk
- 750 GB Ephemeral

<br/>

1. Choose the `Ubuntu Server 22.04 LTS R550 CUDA 12.4` OS Image.
2. Enable **SSH Access** to your VM.
3. Assign a **Public IP Address** to your VM.

<br/>

> [!WARNING]
> `flux2` takes up over 100 GB when running `app.py`, so the 100 GB Disk above will run out of space.
>
> To overcome this, first create a bootable Volume with the `Ubuntu Server 22.04 LTS R550 CUDA 12.4` image. Then create a VM using the bootable Volume as the image.

> [!CAUTION]
> On Hyperstack, incoming traffic to VMs are blocked by default, so you need to create a Firewall with a rule that allows incoming TCP traffic to port `7860` (or all ports) and apply it to the VM.

<br/>

## SSH into VM

```shell
ssh root@<PUBLIC_IP_ADDRESS_OF_VM>
```

<br/>

## Install Conda and Python

**References**:

- [Installing Conda on Ubuntu](https://medium.com/@mustafa_kamal/a-step-by-step-guide-to-installing-conda-in-ubuntu-and-creating-an-environment-d4e49a73fc46)
- [Anaconda versions](https://repo.anaconda.com/archive/)

<br/>

```shell
curl -O https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh
bash Anaconda3-2024.10-1-Linux-x86_64.sh -b -p $HOME/anaconda3
source $HOME/anaconda3/bin/activate
```

> [!NOTE]
> Installing `Anaconda3-2024.10-1` also installs `Python 3.12.7`. I checked that this is the last Anaconda version that comes with `Python 3.12`.

<br/>

## Clone `flux2` fork

```shell
git clone https://github.com/repulsio/flux2.git
cd flux2/
git checkout repulsio/hyperstack
```

> [!IMPORTANT]
> Here is the [link to the diff](https://github.com/black-forest-labs/flux2/compare/main...repulsio:repulsio/hyperstack) between
>
> - the official `black-forest-labs/flux2 - main` and
> - my fork `repulsio/flux2 - repulsio/hyperstack`

<br/>

### Conda Virtual Environment

**References**:

- [PyTorch installation](https://pytorch.org/get-started/previous-versions/)

<br/>

```shell
conda create -n flux2 python=3.12 -y
conda activate flux2

# `torch==2.8.0` is needed, which requires >= `cu126`
# `cu126` somehow works on `Ubuntu Server 22.04 LTS R550 CUDA 12.4`
pip install torch==2.8.0 torchvision==0.23.0 --index-url https://download.pytorch.org/whl/cu126

pip install -e . --extra-index-url https://download.pytorch.org/whl/cu124 --no-cache-dir
```

<br/>

## Run Web Demo

```shell
hf auth login

# Enter your WRITE Access Token
# Type and enter `Y`

# High VRAM model
git clone https://huggingface.co/spaces/black-forest-labs/FLUX.2-dev
cd FLUX.2-dev/
pip install -r requirements.txt

# Supposedly lower VRAM model
git clone https://huggingface.co/spaces/Lakonik/pi-FLUX.2
cd pi-FLUX.2/
pip install -r requirements.txt

GRADIO_SERVER_NAME=0.0.0.0 GRADIO_SERVER_PORT=7860 python app.py
```

Open `http://<PUBLIC_IP_ADDRESS_OF_VM>:7860` in your web browser.
