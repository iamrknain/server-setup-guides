# Using a C++ Server on Remote VM
## Table of Contents:
1. [Login To VM](#login-to-vm)
2. [Transfer Files To Remote Machine](#transfer-files-to-remote-machine)
3. [Installation](#installation)
4. [Running Server In Background](#running-server-in-background)

## Login To VM
```bash
sudo ssh -i ~/key.pem user_name@public_ip
```
<!--  ssh -i cppvm.pem ravi@20.40.47.132 -->

## Transfer Files To Remote Machine
- Using  tar to zip and scp to transfer file to remote server and then extract files:
```bash
tar -czvf archive.tar.gz -C path/to/source/code # archiving  
scp -i ~/key.pem archive.tar.gz user_name@public_ip:~ # sending
tar -xzvf archive.tar.gz # extract
```

- Now proceed as per requirement. E.g: building c++ files
```bash
cd archive
make/cmake/
```

## Installation
```bash
# individual package installtion

sudo apt update
sudo apt install g++
sudo apt install libboost-all-dev -y # Boost
sudo apt install nlohmann-json3-dev -y # Nlohmann-json
sudo apt install libssl-dev -y # Openssl
```

## Running Server In Background
```bash
# using screen : start screen and run server

screen
./server

# using nohup (no hung up)

nohup ./server &