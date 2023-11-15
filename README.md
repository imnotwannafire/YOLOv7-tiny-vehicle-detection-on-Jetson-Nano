# YOLOv7-tiny-vehicle-detection-on-Jetson-Nano
 YOLOv7-tiny TensorRT on Jetson Nano for vehicle detection
## Train model
* Download dataset from [here](https://drive.google.com/file/d/1Ec4JDwe7abMdOCkKNmA1cg9vXNk51JYY/view?usp=drive_link)
* Create a folder YOLOv7-tiny on your google drive
* Train model with **yolov7_tiny_vehicle_detection.ipynb** file.
## Setup environment on Jetson Nanno B01
Build and run l4t-ml docker on Jetson Nano following below steps:
I recommend to use docker instead of install packages directly on Jetson. Because of the following reasons:
* Ubuntu on Jetson Nano is not an official version released by Ubuntu, it is customize for Jetson Nano only by NVIDIA. When you try to install python package or Ubuntu package on Jetson, you will face with many compatible errors during installing process (some packages cannot find compatible version to download).
* Version update: Package version is a flustrated problem for python and Ubuntu software. When you install a package/software, it automaticall installs dependency packages. It may cause conflict with other packages dependency.
* Docker for Jetson Nano is customised by NVIDIA official. It's highly compatible with Jetson Nano and easy to set up. It reduces your headache during setup process

Log on Jetson Nano over SSH or execute Jetson Terminal
Download and install docker from NVIDIA official github: https://github.com/dusty-nv/jetson-containers.git
#### Set docker default runtime
You need to set Docker's default-runtime to nvidia, so that the NVCC compiler and GPU are available during docker build operations. Add **"default-runtime": "nvidia" to your /etc/docker/daemon.json** configuration file before attempting to build the containers:
```
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    },

    "default-runtime": "nvidia"
}
```
Then restart the Docker service, or reboot your system before proceeding:
```
sudo systemctl restart docker
```
You can then confirm the changes by looking under docker info
```
sudo docker info | grep 'Docker Root Dir'
Docker Root Dir: /mnt/docker
...
Default Runtime: nvidia
```
#### Add Swap
If you're building containers or working with large models, it's advisable to mount SWAP (typically correlated with the amount of memory in the board). Do following steps:
```
cd ${HOME}
git clone https://github.com/JetsonHacksNano/installSwapfile.git
cd installSwapfile
./installSwapfile.sh
```
After that, check swap by 
```
free -m
```
#### Clone Docker Container Repo
```
sudo apt-get update && sudo apt-get install git python3-pip
git clone --depth=1 https://github.com/dusty-nv/jetson-containers
cd jetson-containers
pip3 install -r requirements.txt
```
#### Run container
I use lt4-ml container for this project since it has already contain: pytorch, tensorflow, pycuda and everything else I need to develop a deep learning application
```
mkdir ${HOME}/project
./run.sh -v ${HOME}/project:${HOME}/project dustynv/l4t-ml:r32.7.1
```
Choose to use version r32.7.1 since I'm using Jetpack 4.6
### Install additional package inside docker environment
```
tqdm
imutils
psutil
```
Instal Seaborn
```
apt install python3-seaborn
```
## Run Custom YOLOv7-tiny model on Jetson
Inside Docker environment
### Download YOLOv7-tiny-custom model
Download pre-trained YOLOv7-tiny-custom.pt 
### Generate wts file from pt file
run below command to convert .pt file into .wts file
```
python3 gen_wts.py -w yolov7-tiny-custom.pt -o yolov7-tiny-custom.wts
```
### Make
```
cd yolov7/
mkdir build
cd build
cp ../../yolov7-tiny-custom.wts .
cmake ..
make 
```
### Build TensorRT engine file
```
./yolov7 -s yolov7-tiny-custom.wts  yolov7-tiny-custom.engine t
```
Python Object Detection
Use app.py to do inference on video file
```
python3 app.py
```
If you have custom model, make sure to update categories as per your classes in yolovDet.py
