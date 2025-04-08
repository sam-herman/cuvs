# Guide For Building cuVS from Source and the Development Environment

## Hardware Setup On GCP With Debian

### Create a VM instance on GCP:
Specs:

`e2-medium (2 vCPUs, 4 GB Memory)`

SSH into the instance and run the following commands:

`ssh -i <path_to_key> <username>@<ip_address>`

For example:
`ssh -i ~/.ssh/cuvs-key.pem sam.herman@34.28.170.158`

later we can also add the keys to the ssh-agent automatically via ssh config:
```bash
Host cagra_test
  Hostname 34.134.181.237
  AddKeysToAgent yes
  User sam.herman
  IdentityFile ~/.ssh/datastax_ssh_key
```
now we can ssh into the instance with `ssh cagra_test`

Update and Upgrade the system:

`sudo apt update && sudo apt upgrade -y`

Attach the block volume and set it up:
```bash
lsblk | grep sdb
# Confirm the block volume is attached
sudo mkdir /data
sudo mkfs.ext4 /dev/sdb
sudo mount /dev/sdb /data
sudo chmod 777 /data
```

Create the dev workspace:
```bash
mkdir /data/workspace && cd /data/workspace
sudo apt install git
# clone cuvs from your fork repo
git clone https://github.com/sam-herman/cuvs.git
# link the workspace from your home directory
ln -s /data/workspace ~/workspace
```


Install all the dependencies for CUDA toolkit in the previously created workspace:
```bash
# Create special directory for the dependencies
mkdir /data/deps && cd /data/deps

# Dependency for CUDA toolkit
wget https://developer.download.nvidia.com/compute/cuda/12.8.1/local_installers/cuda-repo-debian12-12-8-local_12.8.1-570.124.06-1_amd64.deb
# Dependency for conda to install additional packages
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

sudo dpkg -i cuda-repo-debian12-12-8-local_12.8.1-570.124.06-1_amd64.deb
sudo cp /var/cuda-repo-debian12-12-8-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-8
# Install the cuda drivers
sudo apt-get install -y cuda-drivers
# Install kernel headers to make sure we can load the nvidia drivers properly into the kernel
sudo apt install linux-headers-$(uname -r)
# Load the nvidia module into the kernel
sudo modprobe nvidia

# Install the conda dependencies
bash Miniconda3-latest-Linux-x86_64.sh
```

To build the cuVS library
```bash
cd ~/workspace/cuvs
conda env create --name cuvs -f conda/environments/all_cuda-128_arch-x86_64.yaml
conda init
conda activate cuvs
# Build the cuvs library
./build.sh libcuvs
```

Verify that everything is working by running the tests:
```bash
cd cpp/build
ctest
```


