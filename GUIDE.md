# Run `flux2` in Cloud

This is working on the latest `flux2` commit [`ab7cca68018ad3ceadcace9d6ecb1bc1f6f46b4e`](https://github.com/black-forest-labs/flux2/commit/ab7cca68018ad3ceadcace9d6ecb1bc1f6f46b4e).

## HyperStack

You can run a `RTX-A6000` VM on **HyperStack** for **$0.50/hour** with the following specs:

- 1 GPU
- 28 CPUs
- 58 GB RAM
- 100 GB Disk

<br/>

1. Choose the `Ubuntu Server 22.04 LTS R535 CUDA 12.2 with Docker` image.
2. Enable SSH access to your VM.
3. Assign a Public IP to your VM.

<br/>

> [!WARNING]
> `flux` takes up over 100 GB when running `demo_gr.py`, so the 100 GB Disk above will run out of space.
>
> To overcome this, first create a bootable Volume with the `Ubuntu Server 22.04 LTS R535 CUDA 12.2 with Docker` image. Then create a VM using the bootable Volume as the image.

> [!CAUTION]
> On Hyperstack, incoming traffic to VMs are blocked by default, so you need to create a Firewall with a rule that allows incoming TCP traffic to port `7860` (or all ports) and apply it to the VM.

## SSH into VM

```shell
ssh root@<PUBLIC_IP_ADDRESS_OF_VM>
```

## Install Conda and Python

**References**:

- [Installing Conda on Ubuntu](https://medium.com/@mustafa_kamal/a-step-by-step-guide-to-installing-conda-in-ubuntu-and-creating-an-environment-d4e49a73fc46)
- [Anaconda versions](https://repo.anaconda.com/archive/)

<br/>

```shell
curl -O https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh

bash Anaconda3-2024.10-1-Linux-x86_64.sh

# Press `Enter`
# Press `q` to move to bottom of agreement
# Type and enter `yes`
# Press `Enter`
# Type and enter `yes`

source ~/.bashrc
```

> [!NOTE]
> Installing `Anaconda3-2024.10-1` also installs `Python 3.12.7`.

## Clone `flux` fork

**References**:

- [PyTorch installation](https://pytorch.org/get-started/previous-versions/)

<br/>

```shell
git clone https://github.com/repulsio/flux2.git
cd flux2/
git checkout repulsio/hyperstack
```

> [!IMPORTANT]
> Here is the [link to the diff](https://github.com/black-forest-labs/flux/compare/main...repulsio:repulsio/hyperstack) between
>
> - the official `black-forest-labs/flux - main` and
> - my fork `repulsio/flux - repulsio/hyperstack`

## `flux` setup

**References**:

- [PyTorch installation](https://pytorch.org/get-started/previous-versions/)

<br/>

Choose _**only 1 of the 2 options**_ below depending on whether you want to use a virtual environment:

### Option 1. Inside Conda virtual environment

```shell
conda create -n flux2 python=3.12 -y
conda activate flux2

pip install torch==2.6.0 torchvision==0.21.0 --index-url https://download.pytorch.org/whl/cu124

pip install -e . --extra-index-url https://download.pytorch.org/whl/cu124 --no-cache-dir

pip install pydantic==2.8.2  # downgrade `pydantic` from `2.11.1`
```

> [!WARNING]
> `pydantic` needs to be downgraded due to this [`gradio` issue](https://github.com/gradio-app/gradio/issues/10662).
>
> This does not need to be done for **Option 2**, since it already comes with a non-broken version of `pydantic`.

### Option 2. No virtual environment

```shell
conda install pytorch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 pytorch-cuda=12.1 -c pytorch -c nvidia

# Type and enter `y`

pip install -e ".[all]"
pip install -e ".[tensorrt]"
```

## Run Web Demo

Regardless of whether you chose **Option 1** or **Option 2**, run:

```shell
# For `flux-schnell`
GRADIO_SERVER_NAME=0.0.0.0 python demo_gr.py

# For `flux-dev`
huggingface-cli login

# Enter your Hugging Face User Access Token

GRADIO_SERVER_NAME=0.0.0.0 python demo_gr.py --name flux-dev
```

> [!NOTE]
> If you chose **Option 1**, the virtual environment `flux` should be activated before running the above command.

Open `http://<PUBLIC_IP_ADDRESS_OF_VM>:7860` in your web browser.
