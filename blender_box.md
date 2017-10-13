# Blender + Caffe2 box
## Ubuntu install
1. [Download](https://help.ubuntu.com/community/Installation/MinimalCD) ubuntu minimal iso.
2. Burn to USB drive
	3. `diskutil list`
	4. unmount USB drive from list
	3. `sudo dd if=<image.iso> of=/dev/rdisk2 bs=1m`
		4. `r` is important
5. Boot and install
	6. Make sure you install `gcc` and other important stuff

## Build latest Cmake
1. `wget https://cmake.org/files/LatestRelease/cmake-3.9.0.tar.gz`
2. `tar xvzf cmake-3.9.0.tar.gz`
3. `cd cmake-3.9.0`
4. `./configure --prefix=/opt/cmake`
5. `make`
6. `sudo make install`
7. Add `/opt/camke/bin` to path


## Install CUDA
1. Since this is on headless Ubuntu, follow the post [here](https://askubuntu.com/questions/830983/how-to-winstall-nvidia-drivers-to-use-cuda-without-also-installing-x11)
	1. See third answer
		1. `wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/patches/2/cuda_8.0.61.2_linux-run`
		2. `wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run`
		3. [Blacklist Noveau (conflicts)](https://askubuntu.com/questions/841876/how-to-disable-nouveau-kernel-driver)
			1. `sudo apt-get remove nvidia* && sudo apt autoremove`
			2. `sudo apt-get install dkms build-essential linux-headers-generic`
			3. `sudo vi /etc/modprobe.d/blacklist.conf`
				1. `# NVIDIA drivers conflict with Noveau`
				2. `blacklist nouveau`
				3. `blacklist lbm-nouveau`
				4. `options nouveau modeset=0`
				5. `alias nouveau off`
				6. `alias lbm-nouveau off`
				7. `echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf`
				8. `sudo update-initramfs -u`
				9. `sudo reboot`
		4. `chmod u+x cuda_8.0.61_375.26_linux-run cuda_8.0.61.2_linux-run`
		5. `sudo ./cuda_8.0.61_375.26_linux-run`
			1. Everything to default and install Samples and Toolkit
		6. [Add following lines to `~/.profile`](https://www.cs.colostate.edu/~info/cuda-faq.html)
		7. `source ~/.profile`
		8. `sudo ./cuda_8.0.61.2_linux-run`

## Install Blender
Because we're going to do things remotely without X11, we need to build Blender from source.

### [CMake approach](https://wiki.blender.org/index.php/Dev:Doc/Building_Blender/Linux/Ubuntu/CMake)
1. [Get the source](https://wiki.blender.org/index.php/Dev:Doc/Building_Blender/Linux/get_the_source)
	1. `mkdir ~/blender-git`
	2. `cd ~/blender-git`
	3. `git clone https://git.blender.org/blender.git`
	4. `cd blender`
	5. `git submodule update --init --recursive`
	6. `git submodule foreach git checkout master`
	7. `git submodule foreach git pull --rebase origin master`
2. Build and resolve dependencies
	1. Dependencies
		1. `sudo apt-get install libjpeg-dev libpng-dev libpython3-dev libopenexr-dev libopenjpeg-dev libtiff-dev libopenal-dev libjemalloc-dev libspnav-dev libboost-all-dev libopenimageio-dev mesa-utils freeglut3-dev libboost-system-dev`
	2. Build instructions
		1. `cd ~/blender-git/blender`
		2. `mkdir build`
		3. `cd build`
		4. Update `intern/cycles/CMakeLists.txt` from [here](https://blender.stackexchange.com/questions/68698/headless-blender-using-cycles-without-x-server)
			1. Change `set(WITH_CYCLES_DEVICE_OPENCL TRUE)` to `set(WITH_CYCLES_DEVICE_OPENCL FALSE)`
		5. `cmake -DWITH_HEADLESS=ON -DWITH_CYCLES_DEVICE_OPENCL=OFF ../`
		6. Update `build/source/creator/CMakeFiles/blender.dir/link.txt` file like [here](https://developer.blender.org/T49231)
			1. Add `-lboost_locale -lboost_thread -lboost_system` to end of line
		7. `make -j4`
		8. `sudo make install`
		9. `cd ../`
		10. `mkdir build_py`
		11. `cd build_py`
		12. `export PATH="$HOME/blender-git/blender/build/bin:$PATH"`
		13. `cmake -DWITH_HEADLESS=ON -DWITH_CYCLES_DEVICE_OPENCL=OFF -DWITH_SYSTEM_GLEW=OFF -DWITH_PYTHON_INSTALL=OFF -DWITH_PLAYER=OFF -DWITH_PYTHON_MODULE=ON -DPYTHON_VERSION=3.5 ../`
		14. `make`
		15. `make install`
		16. `export PYTHON_PATH="$HOME/blender-git/blender/build_py/bin:${PYTHON_PATH}"`

## Enabling remote GPU for Blender
### Create script for Blender
1. [List GPU devices](https://blender.stackexchange.com/questions/74075/headless-render-cant-find-compute-device-in-2-78b)
2. [Write a script](https://blender.stackexchange.com/questions/5281/blender-sets-compute-device-cuda-but-doesnt-use-it-for-actual-render-on-ec2)
	1. [render all cameras script (render_cam.py)](https://www.blender.org/forum/viewtopic.php?t=19102&highlight=batch+render)
		1. Add following to set Cycles render
			1. `bpy.context.user_preferences.addons['cycles'].preferences.compute_device_type = 'CUDA'`
			2. `bpy.context.user_preferences.addons['cycles'].preferences.devices[0].use = True`
			3. `bpy.context.scene.render.engine = 'CYCLES'`
3. `blender -b ~/blend.blend -P render_cam.py`

### Setup network connection
1. Look at scripts in `scripts/blender` directory (both local and remote)

## Install Caffe2
### [Instructions](https://caffe2.ai/docs/getting-started.html?platform=ubuntu&configuration=compile)
Make sure to do the following in the instructions

1. Install cuDNN
2. Install optional packages
3. Update environment variables
	1. `export PYTHONPATH=/usr/local:$PYTHONPATH`
	2. `export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH`

### [Follow MNIST example](https://caffe2.ai/docs/tutorial-MNIST.html)

1. Dependencies
	1. `sudo apt-get install python-tk`
2. Build command
	1. `cmake -DUSE_CUDA=OFF -DLMDB_INCLUDE_DIR:PATH=/anaconda/include -DLMDB_LIBRARIES:FILEPATH=/anaconda/lib/liblmdb.so ../`
