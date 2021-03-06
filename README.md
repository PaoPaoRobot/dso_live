# DSO: Direct Sparse Odometry

For more information see
[https://vision.in.tum.de/dso](https://vision.in.tum.de/dso)

### 1. Related Papers
* **Direct Sparse Odometry**, *J. Engel, V. Koltun, D. Cremers*, In arXiv:1607.02565, 2016
* **A Photometrically Calibrated Benchmark For Monocular Visual Odometry**, *J. Engel, V. Usenko, D. Cremers*, In arXiv:1607.02555, 2016

Get some datasets from [https://vision.in.tum.de/mono-dataset](https://vision.in.tum.de/mono-dataset) .

### 2. Installation

#### 2.1 Required Dependencies

##### suitesparse and eigen3 (required).
Required. Install with

		sudo apt-get install libsuitesparse-dev libeigen3-dev



#### 2.2 Optional Dependencies

##### OpenCV (highly recommended).
Used to read / write / display images.
OpenCV is **only** used in `IOWrapper/OpenCV/*`. Without OpenCV, respective 
dummy functions from `IOWrapper/*_dummy.cpp` will be compiled into the library, which do nothing.
The main binary will not be created, since it is useless if it can't read the datasets from disk.
Feel free to implement your own version of these functions with your prefered library, 
if you want to stay away from OpenCV.

Install with

	sudo apt-get install libopencv-dev


##### Pangolin (highly recommended).
Used for 3D visualization & the GUI.
Pangolin is **only** used in `IOWrapper/Pangolin/*`. You can compile without Pangolin, 
however then there is not going to be any visualization / GUI capability. 
Feel free to implement your own version of `Output3DWrapper` with your preferred library, 
and use it instead of `PangolinDSOViewer`

Install from [https://github.com/stevenlovegrove/Pangolin](https://github.com/stevenlovegrove/Pangolin)


##### ziplib (recommended).
Used to read datasets with images as .zip, as e.g. in the TUM monoVO dataset. 
You can compile without this, however then you can only read images directly (i.e., have 
to unzip the dataset image archives before loading them).

	sudo apt-get install zlib1g-dev
	cd thirdparty
	tar -zxvf libzip-1.1.1.tar.gz
	cd libzip-1.1.1/
	./configure
	make
	sudo make install
	sudo cp lib/zipconf.h /usr/local/include/zipconf.h   # (no idea why that is needed).

#### 2.3 Build

		cd dso_beta
		mkdir build
		cd build
		cmake ..
		make -j
	
this will compile a library `libdso.a`, which can be linked from external projects. 
It will also build a binary `dso_dataset`, to run DSO on datasets. However, for this
OpenCV and Pangolin need to be installed.






### 3 Usage
Run on a dataset from [https://vision.in.tum.de/mono-dataset](https://vision.in.tum.de/mono-dataset) using

		bin/dso_dataset \
			files=XXXXX/sequence_XX/images.zip \
			calib=XXXXX/sequence_XX/camera.txt \
			gamma=XXXXX/sequence_XX/pcalib.txt \
			vignette=XXXXX/sequence_XX/vignette.png \
			preset=0 \
			mode=0

See [https://github.com/JakobEngel/dso_ros](https://github.com/JakobEngel/dso_ros) for a minimal example on
how the library can be used from another project. It should be straight forward to implement extentions for 
other camera drivers, to use DSO interactively without ROS.



#### 3.1 Dataset Format.
The format assumed is that of [https://vision.in.tum.de/mono-dataset](https://vision.in.tum.de/mono-dataset).
However, it should be easy to adapt it to your needs, if required. The binary is run with:

- `files=XXX` where XXX is either a folder or .zip archive containing images. They are sorted *alphabetically*. for .zip to work, need to comiple with ziplib support.

- `gamma=XXX` where XXX is a gamma calibration file, containing a single row with 256 values, mapping [0..255] to the respective irradiance value, i.e. containing the *discretized inverse response function*. See TUM monoVO dataset for an example.

- `vignette=XXX` where XXX is a monochrome 16bit or 8bit image containing the vignette as pixelwise attenuation factors. See TUM monoVO dataset for an example.

- `calib=XXX` where XXX is a geometric camera calibration file. Currently supported:

###### Calibration File for FOV camera model:

    fx fy cx cy omega
    in_width in_height
    "crop" / "full" / "none" / "fx fy cx cy 0"
    out_width out_height

###### Calibration File for Pre-Rectified Images
This one is with no radial distortion, as a special case of ATAN camera model but without the computational cost:

    fx fy cx cy 0
    in_width in_height
    none
    out_width out_height


###### Calibration File for OpenCV camera model

    fx fy cx cy d1 d2 d3 d4
    in_width in_height
    "crop" / "full" / "none" / "fx fy cx cy 0"
    out_width out_height


#### 3.2 Commandline Options
there are many command line options available, see `main_dso_pangolin.cpp`. some examples include
- `mode=X`: 
    -  `mode=0` use iff a photometric calibration exists (e.g. TUM monoVO dataset). 
    -  `mode=1` use iff NO photometric calibration exists (e.g. ETH EuRoC MAV dataset). 
    -  `mode=2` use iff images are not photometrically distorted (e.g. synthetic datasets).

- `preset=X`
    - `preset=0`: default settings (2k pts etc.), not enforcing real-time execution
    - `preset=1`: default settings (2k pts etc.), enforcing 1x real-time execution
    - `preset=2`: fast settings (800 pts etc.), not enforcing real-time execution. WARNING: overwrites image resolution with 424 x 320.
    - `preset=3`: fast settings (800 pts etc.), enforcing 5x real-time execution. WARNING: overwrites image resolution with 424 x 320.

- `nolog=1`: disable logging of eigenvalues etc. (good for performance)
- `reverse=1`: play sequence in reverse
- `nogui=1`: disable gui (good for performance)
- `nomt=1`: single-threaded execution
- `prefetch=1`: load into memory & rectify all images before running DSO.
- `start=X`: start at frame X
- `end=X`: end at frame X
- `speed=X`: force execution at X times real-time speed (0 = not enforcing real-time)
- `save=1`: save lots of images for video creation



#### 3.3 Runtime Options
Some parameters can be reconfigured from the Pangolin GUI at runtime. Feel free to add more.





#### 3.4 Notes
- the initializer is very slow, and does not work very reliably. Maybe replace by your own way to get an initialization.
- see [https://github.com/JakobEngel/dso_ros](https://github.com/JakobEngel/dso_ros) for a minimal example project on how to use the library with your own input / output procedures.
- see `settings.cpp` for a LOT of settings parameters. Most of which you shouldn't touch.
- `setGlobalCalib(...)` needs to be called once before anything is initialized, and globally sets the camera intrinsics and video resolution for convenience. probably not the most portable way of doing this though.




### 4 General Notes for Good Results

#### Accurate Geometric Calibration
- Please have a look at Chapter 4.3 from the DSO paper, in particular Figure 20 (Geometric Noise). Direct approaches suffer a LOT from bad geometric calibrations: Geometric distortions of 1.5 pixel already reduce the accuracy by factor 10.

- **Do not use a rolling shutter camera**, the geometric distortions from a rolling shutter camera are huge. Even for high frame-rates (over 60fps).

- Note that the reprojection RMSE reported by most calibration tools is the reprojection RMSE on the "training data", i.e., overfitted to the the images you used for calibration. If it is low, that does not imply that your calibration is good, you may just have used insufficient images.

- try different camera / distortion models, not all lenses can be modelled by all models.


#### Photometric Calibration
Use a photometric calibration (e.g. using [https://github.com/tum-vision/mono_dataset_code](https://github.com/tum-vision/mono_dataset_code) ).

#### Translation vs. Rotation
DSO cannot do magic: if you rotate the camera too much without translation, it will fail. Since it is a pure visual odometry, it cannot recover by re-localizing, or track through strong rotations by using previously triangulated geometry.... everything that leaves the field of view is marginalized immediately.


#### Computation Speed
If your computer is slow, try to use "fast" settings. Or run DSO on a dataset, without enforcing real-time.


#### Initialization
The current initializer is not very good... it is very slow and occasionally fails. 
Make sure, the initial camera motion is slow and "nice" (i.e., a lot of translation and 
little rotation) during initialization.
Possibly replace by your own initializer.


### 5 License
DSO was developed at the Technical University of Munich and Intel.
The open-source version is licensed under the GNU General Public License
Version 3 (GPLv3).
For commercial purposes, we also offer a professional version, see
[http://vision.in.tum.de/dso](http://vision.in.tum.de/dso) for
details.
