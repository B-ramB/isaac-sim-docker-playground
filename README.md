# isaac-sim-docker-playground
A playground to run the Nvidia Isaac Sim from Docker. This has been tested and succesfully executed on ubuntu 24.04.

## Tested Setup:

## Installation:

### Docker Engine:
[Original installation guide from Docker Engine docs](https://docs.docker.com/engine/install/ubuntu/)
<details> 
  <summary>Or a quick step-by-step guide here</summary>


Add Docker's official GPG key:
```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
Add the repository to Apt sources:
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install the latest version:
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Create docker group and add your user:
```
sudo groupadd docker\
sudo usermod -aG docker $USER
```

Apply changes to the new docker group by running:
```
newgrp docker
```

Verify installation:
```
sudo docker run hello-world
```

</details>

### Isaac Sim:
[Original installation guide from Nvidia Isaac Sim docs](https://docs.isaacsim.omniverse.nvidia.com/latest/installation/install_container.html) 
<details> 
  <summary>Or a quick step-by-step guide here</summary>

**Install NVIDIA driver:**
```
sudo apt-get update
sudo apt install build-essential -y
wget https://us.download.nvidia.com/XFree86/Linux-x86_64/535.129.03/NVIDIA-Linux-x86_64-535.129.03.run
chmod +x NVIDIA-Linux-x86_64-535.129.03.run
sudo ./NVIDIA-Linux-x86_64-535.129.03.run
```

Verify the NVIDIA driver:
```
nvidia-smi
```

**Install the nvidia container toolkit:**
Configure the repository
```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
    && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list \
    && \
    sudo apt-get update
```

Install the NVIDIA Container Toolkit packages
```
sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

Configure the container runtime
```
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```
Verify NVIDIA Container Toolkit
```
docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```
</details> 

### Install Isaac Sim WebRTC Streaming Client (when not utilizing xhost but still want a GUI):

Get the latest Isaac Sim WebRTC Streaming Client from [here](https://docs.isaacsim.omniverse.nvidia.com/latest/installation/download.html#isaac-sim-latest-release)

The linux version is an Appimage which after being made executable can be run.
```
chmod a+x [downloaded file]
```

## Run:

**Pull latest Isaac Sim Container:**
The following docker pull command pulls Isaac Sim version 4.5.0
```
docker pull nvcr.io/nvidia/isaac-sim:4.5.0
```

**Running the docker container while opening a local window using xhost:
**
```
xhost +
docker run --name isaac-sim --entrypoint bash -it --runtime=nvidia --gpus all -e "ACCEPT_EULA=Y" --rm --network=host \
    -e "PRIVACY_CONSENT=Y" \
    -v $HOME/.Xauthority:/root/.Xauthority \
    -e DISPLAY \
    -v ~/docker/isaac-sim/cache/kit:/isaac-sim/kit/cache:rw \
    -v ~/docker/isaac-sim/cache/ov:/root/.cache/ov:rw \
    -v ~/docker/isaac-sim/cache/pip:/root/.cache/pip:rw \
    -v ~/docker/isaac-sim/cache/glcache:/root/.cache/nvidia/GLCache:rw \
    -v ~/docker/isaac-sim/cache/computecache:/root/.nv/ComputeCache:rw \
    -v ~/docker/isaac-sim/logs:/root/.nvidia-omniverse/logs:rw \
    -v ~/docker/isaac-sim/data:/root/.local/share/ov/data:rw \
    -v ~/docker/isaac-sim/documents:/root/Documents:rw \
    nvcr.io/nvidia/isaac-sim:4.5.0 \
  ./runapp.sh
```

**Running the docker container with a headless version and utilizing:
**
```
docker run --name isaac-sim --entrypoint bash -it --runtime=nvidia --gpus all -e "ACCEPT_EULA=Y" --rm --network=host \
    -e "PRIVACY_CONSENT=Y" \
    -v ~/docker/isaac-sim/cache/kit:/isaac-sim/kit/cache:rw \
    -v ~/docker/isaac-sim/cache/ov:/root/.cache/ov:rw \
    -v ~/docker/isaac-sim/cache/pip:/root/.cache/pip:rw \
    -v ~/docker/isaac-sim/cache/glcache:/root/.cache/nvidia/GLCache:rw \
    -v ~/docker/isaac-sim/cache/computecache:/root/.nv/ComputeCache:rw \
    -v ~/docker/isaac-sim/logs:/root/.nvidia-omniverse/logs:rw \
    -v ~/docker/isaac-sim/data:/root/.local/share/ov/data:rw \
    -v ~/docker/isaac-sim/documents:/root/Documents:rw \
    nvcr.io/nvidia/isaac-sim:4.5.0 \
  ./runheadless.sh -v
```
Execute the Isaac Sim WebRTC Streaming Client and connect to 127.0.0.1
