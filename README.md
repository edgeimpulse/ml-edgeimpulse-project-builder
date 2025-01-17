# Edge Impulse Project Builder
This repository contains instructions for integrating an
[Edge Impulse Studio](https://edgeimpulse.com/product) project into an MPLAB X project for any xc32
supported platform.

Notes:
- This script has only been tested under linux (ubuntu), [Git
  Bash](https://gitforwindows.org/), and macOS.
- `*.options.ini` can be modified to set additional project options; for help
  call the mplab script `prjMakefilesGenerator -setoptions=@ -help`
  + NB: all relative paths are considered relative to the project root folder
    (.X folder)
- `*.project.ini` is just a placeholder - the **languageToolchain** and
  **device** variables are replaced when building the project - others will take
  default values if unspecified.


## Prerequisites
* MPLAB® X IDE *>=6.00* (https://microchip.com/mplab/mplab-x-ide)
* XCode *>=12.0* (https://developer.apple.com/xcode/)
* Edge Impulse CLI (https://github.com/edgeimpulse/edge-impulse-cli)
* Edge Impulse Stand-alone Inferencing (C++) (https://github.com/edgeimpulse/example-standalone-inferencing)
* Docker (https://www.docker.com/)
* Docker Compose (https://docs.docker.com/compose/)
* Python *>=3.6* (https://www.python.org/downloads/)

## Software Used
* MPLAB® X IDE *>=6.00* (https://microchip.com/mplab/mplab-x-ide)

## Model Deployment
The first step in building an MPLAB project for Edge Impulse is of course to deploy the Edge Impulse model.

1. Open Edge Impulse Studio.

2. Click on the *Deployment* step in the sidebar menu.

3. Under *Create Library* select *C++ library*, then click *Build* to download
   the library, later used in the Edge Impulse SDK Build Instructions as libedgeimpulse.

   ![Add library object](assets/deploy.png)

## Edge Impulse SDK Build Instructions
The following steps cover compiling the Edge Impulse library into a static library object.

1. Extract the library from the step above directly into this folder.

2. (Optional) Open `options.ini` and modify as needed.

3. Set the environment variables for MPLAB IDE Version`MPLABX_VERSION` XCode Version: `XC_VERSION` `XC_NUMBER_BITS` as
   desired, then run `build.sh` to generate the library object. For example:

   ```bash
   MPLABX_VERSION=6.00 XC_VERSION=4.00 XC_NUMBER_BITS=32 ./build.sh ATSAME54P20A libedgeimpulse .
   ```

   If MPLAB X or the XC compiler are in non-default install locations, set the
   corresponding path directly through the `MPLABX_PATH` and `XC_PATH`
   environment variables.

## Docker Build
To launch a Docker build for a specific target build arguments must be set as
shown in the included example `.args` files. See `docker_build.sh` for a full
example for building the docker image and generating the Edge Impulse
library/project. This script can be run by passing the target name and .args
file e.g.:

```bash
./docker_build.sh ATSAME54P20A args/SAME54.args
```

This will output the result of the build into a folder `dist/` under your
current working directory.

See [packs.download.microchip.com](https://packs.download.microchip.com/) for
device family pack listings.

## Integration Instructions
Below are instructions for integrating the library object, compiled with the
steps above, into an MPLAB X project.

1. Add the library object from the step above into an existing MPLAB project as
   shown below.

   ![Add library object](assets/addlibrary.png)

2. Add the `src/ei_classifier_porting.cpp` file from this repository to your
   project, this implements the required Edge Impulse stubs.

3. Use `src/main.cpp` as a template for integrating the Edge Impulse library
   into your project.

4. Add the path to the directory where the SDK is extracted in your include path
   under *Project Properties* -> *XC32 Global Options* -> *Common include dirs*

   ![Add include directory](assets/include.png)

5. The Edge Impulse SDK uses dynamic memory allocation so allocate additional
   heap memory under *Project Properties* -> *xc32-ld* -> *Heap size* (see (1)
   in the image below). The number will vary based on your Edge Impulse project
   parameters (especially the maximum width of your neural network layers).

6. Ensure that the `Remove unused sections` option under *Project Properties* ->
   *xc32-ld* is enabled (see (2) in the image below); this will eliminate any
   unused data or functions from the SDK to reclaim device memory.

   ![Add include directory](assets/linker.png)

You should now have your Edge Impulse model fully integrated with an MPLAB X
project. In order to update the deployed model, simply repeat the steps from the
[build instructions](#edge-impulse-sdk-build-instructions) section above.

## Additional Notes
Some special care has to be taken to ensure the library is integrated correctly
with your project:

- If the CMSIS libraries are included in the build, make sure your project
  doesn't link the pre-built CMSIS library optionally included with MPLAB
  Harmony projects.

- The Edge Impulse SDK will automatically determine whether to include the CMSIS
  NN/DSP libraries. You can use `EIDSP_USE_CMSIS_DSP=0` and
  `EI_CLASSIFIER_TFLITE_ENABLE_CMSIS_NN=0` to manually disable this behavior.

