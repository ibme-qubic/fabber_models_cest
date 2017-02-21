# Building the CEST model #

The current version of the CEST model is designed to build with the latest version of Fabber. Although it should be possible to build them against FSL 5.0 you may need to edit the source slightly. So your first step should be to build fabber-core from Git. Build instructions for fabber-core are in the Wiki on its Git page. You will make the process straightforward if you create a directory (e.g. 'fabber') and download the Git repositories for fabber_core and fabber_models_cest into this directory. Then the CEST model build should be able to find the Fabber libraries without you having to install them or tell the build scripts where they are.

The CEST model uses cmake as the build tool. CMake is designed for out-of-source builds, so you create a separate build directory and all the compiled files end up there. 
CMake is installed on many Linux distributions by default, or can easily be added. It is also readily available for OSX and Windows.

If you have an existing FSL source directory and want to build the CEST models from within this, [see below](#fslbuild)

You need to ensure that `FSLDIR` is set to point to wherever the FSL dependencies are installed, following that the basic steps to make a build are:

    mkdir build
    cd build
    cmake ..
    make

After the `cmake` command, information on dependencies will be displayed, for example:

    -- FSL headers in /home/martinc/dev/fsl/include /home/martinc/dev/fsl/extras/include/newmat /home/martinc/dev/fsl/extras/include /home/martinc/dev/fsl/extras/include/boost
    -- Fabber headers in /home/martinc/dev/fabber_core
    -- Using Fabber libraries: /home/martinc/dev/fabber_core/Debug/libfabbercore.a /home/martinc/dev/fabber_core/Debug/libfabberexec.a
    -- Using libznz: /home/martinc/dev/fsl/lib/libznz.a
    -- Using libutils: /home/martinc/dev/fsl/lib/libutils.a 
    -- Using miscmaths: /home/martinc/dev/fsl/lib/libmiscmaths.a
    -- Using fslio: /home/martinc/dev/fsl/lib/libfslio.a
    -- Using newimage: /home/martinc/dev/fsl/lib/libnewimage.a
    -- Using niftiio: /home/martinc/dev/fsl/lib/libniftiio.a
        -- Using newmat: /home/martinc/dev/fsl/extras/lib/libnewmat.a /home/martinc/dev/fsl/extras/include/newmat
    -- Using newimage: /home/martinc/dev/fsl/lib/libnewimage.a
    -- Using prob: /home/martinc/dev/fsl/extras/lib/libprob.a
    -- Using zlib: /usr/lib/x86_64-linux-gnu/libz.so

Check that these locations seem reasonable. In particular, make sure the Fabber headers and libraries are for the latest version of Fabber which you have probably just built. If it is picking up the FSL-5.0 version you may need to edit the file CMakeLists.txt and re-run cmake.

After running `make`, if all goes well, the build should conclude with the message

    [ 20%] Building CXX object CMakeFiles/fabber_cest.dir/fwdmodel_cest.cc.o
    [ 40%] Building CXX object CMakeFiles/fabber_cest.dir/fabber_client.cc.o
    [ 60%] Linking CXX executable fabber_cest
    [ 60%] Built target fabber_cest
    Scanning dependencies of target fabber_models_cest
    [ 80%] Building CXX object CMakeFiles/fabber_models_cest.dir/fwdmodel_cest.cc.o
    [100%] Linking CXX shared library libfabber_models_cest.so
    [100%] Built target fabber_models_cest

### It failed with something about `recompile with -fPIC`

In order to build the shared library, the FSL libraries have to contain 'Position-independent code'. If they don't you can't build the library and will need to use 
the fabber_asl executable instead. The only way around this is to recompile the FSL libraries with the -fPIC flag but this may be a challenge. You're better off 
just running make fabber_asl which will just build the executable.

### Successful build?

Two objects are built:

1. An executable fabber_cest, which is a version of the Fabber executable with the CEST model built in
2. A shared library libfabber_models_cest.so, which can be loaded in to the generic Fabber executable using the --loadmodels option.

You may use whichever of these you prefer, however if you are intending to use the command line tool you will probably find the fabber_cest executable more convenient. The shared library is intended for when you want to embed the Fabber library in another application.

You can verify that the CEST model is available whichever method you choose:

    fabber_cest --listmodels

    cest
    linear
    poly
    trivial

or

    fabber --loadmodels=libfabber_models_cest.so --listmodels
    
    cest
    linear
    poly
    trivial
  
Note that you still need to specify --model=cest on the command line to tell Fabber to use the CEST model.

### <a name="fslbuild"></a>Building in an FSL environment

If you have built `fabber_core` within an FSL source distribution which you may prefer to build the CEST model within the same system. You will need to 
do the following:

1. Move the fabber_models_cest directory into the FSL source tree `mv fabber_models_cest $FSLDIR/src/`
2. `cd $FSLDIR/src/fabber_models_cest`
3. make

`fabber_cest` should now be set up to build, provided you have the following environment variables set up:

    FSLDIR
    FSLCONFDIR
    FSLMACHTYPE

Note that the FSL makefile does not attempt to build the shared library.
Introduction

# Running the CEST model #

You will need the following:

 1. Sampled CEST z-spectra (nifti image). 
 2. Mask to indicate what region of the data to be processed (nifti image) 
 3. A file specifying details of the pools to be included in the model: poolmat. 
 4. A file detailing the different samples contained within the data: dataspec. 
 5. A file detailing the pulse used for saturation: ptrain. 

## Command line 

  fabber_cest --data=<input image> --mask=<mask image> --output=<output dir> 
              --pools=<ASCII matrix containing pool details>
              --spec=<ASCII matrix containing details of samples in the data>
              --ptrain=<ASCII matrix containing details of saturation pulse shape>
              
            
## The pool specification matrix

It is necessary to specify details about each of the pools that you wish to include within the model, including the water pool. The key details required are: 

Centre frequency of the pool: for water in Hz for others specify relative to water (in ppm). 
Exchange rate expected for the pool (in Hz). 
T1 value for the pool (in s). 
T2 value for the pool (in s). 
This should take the form of an ascii matrix where each row corresponds to one pool, and each column is a parameter in the order: Frequency | rate | T1 | T2 . 

The first row should refer to the water pool and the freqeuncy should be in Hz according to the field strength, otherwise a ppm value is expected. The exchange rate value for the water pool will be ignored, but should be included, i.e. enter a zero in that column. 

### Example poolmats: 

APT in vivo: water plus amide at 3T. 

   | 1.2800000e+08  |  0.0000000e+00  |  1.3000000e+00   | 5.0000000e-02| 
   | 3.5000000e+00  |  2.0000000e+01  |  7.7000000e-01  |  1.0000000e-02|
   
APT+MT in vivo: water plus amide and assymetric MT at 3T. 

   |1.2800000e+08  | 0.0000000e+00 |  1.3000000e+00 |  5.0000000e-02|
   |3.5000000e+00  | 2.0000000e+01   |7.7000000e-01  | 1.0000000e-02|
   |-2.4100000e+00 |  4.0000000e+01 |  1.0000000e+00 |  3.0000000e-04|
  
## Dataspec: data specification

Each individual volume in the data should contain a single sample from the z-spectrum. It is necessary to specify in the dataspec matrix for each sample: 

 * Sampling frequency relative to water (in ppm). 
 * B1 field strength of saturation (in Tesla). 
 * The number of repetitions of the pulse used for saturation (integer). 
 * The data specification matrix should be an ascii matrix with one row for each volume in the data and the columns representing (in order): Frequency | B1 field | No. pulses . 

For example: 

| 0     |   0   | 1     |
| 3.5   | 1e-06 | 1     |
| 3.5   | 1e-06 | 1     |

In this example the first point is unsaturated, the second occurs at the APT centre frequency and the final sample is symmetrically placed at the other side of the water centre. Here it has been assumed that only a single pulse has been applied, e.g. continuous saturation. 

## Ptrain: pulse specification

The pulse shape used for saturation should be discretized into a series of approximate segments that are then used within the model calculations. This is provided to BayCEST as an ascii matrix where each row is a time point in the approximation, the first entry in the row is the the magnitude of the pulse at that time (in relative units, i.e. the largest value at the peak of the pulse should be 1.0, the whole pulse is scaled by the B1 value specified in the dataspec matrix) and the second entry is time value (in seconds). It is assumed that the pulse starts at zero at time zero. Only a single cycle of the pulse scheme need be specified, BayCEST will automatically produce a pulse train using the number of repeats specified in the dataspec matrix (and in a computationally efficient manner). 

Example pulse matrices: 
pulseshp_cont.mat - Continuous pulse with 2 s duration. 

| 1.0000000e+00 |  2.0000000e+00|
   
Gaussian pulse with a 40 ms duration and 50% duty cycle divided into 32 segments. 

|2.3475797e-02|6.2500000e-04|
|3.8074068e-02|1.2500000e-03|
|5.9724818e-02|1.8750000e-03|
|9.0614358e-02|2.5000000e-03|
|1.3297066e-01|3.1250000e-03|
|1.8872580e-01|3.7500000e-03|
|2.5907370e-01|4.3750000e-03|
|3.4397906e-01|5.0000000e-03|
|4.4173043e-01|5.6250000e-03|
|5.4865489e-01|6.2500000e-03|
|6.5910987e-01|6.8750000e-03|
|7.6583111e-01|7.5000000e-03|
|8.6064651e-01|8.1250000e-03|
|9.3547729e-01|8.7500000e-03|
|9.8346365e-01|9.3750000e-03|
|1.0000000e+00|1.0000000e-02|
|9.8346365e-01|1.0625000e-02|
|9.3547729e-01|1.1250000e-02|
|7.6583111e-01|1.2500000e-02|
|6.5910987e-01|1.3125000e-02|
|5.4865489e-01|1.3750000e-02|
|4.4173043e-01|1.4375000e-02|
|3.4397906e-01|1.5000000e-02|
|2.5907370e-01|1.5625000e-02|
|1.8872580e-01|1.6250000e-02|
|1.3297066e-01|1.6875000e-02|
|9.0614358e-02|1.7500000e-02|
|5.9724818e-02|1.8125000e-02|
|3.8074068e-02|1.8750000e-02|
|2.3475797e-02|1.9375000e-02|
|1.4000000e-02|2.0000000e-02|
|0.0000000e+00|4.0000000e-02|

## Extra options

replacing `--method=vb` with `--method=spatialvb` will turn on the spatial smoothing prior, this is recommended if processing in vivo data, it is 
generally not appropriate if there is no spatial structure in the data e.g. simulations. 

Adding `--t12prior` will incorporate uncertainty/variability in T1 and T2 value in the estimation. This treats T1 and T2 for each of the pools as 
extra parameters to be estimated from the data, but with tightly defined priors to avoid over fitting but allow T1 and T2 variability to be reflected 
in the results. 

# Outputs

The primary outputs of BayCEST are estimates of the various parameters in the multi-pool model. These can be found within the output directory all beginning with 'mean_'. Each of the pools are labelled by a letter in the order they appear in the poolmat matrix. The parameters returned are: 

 * `mean_M0i_r` - the magnetization of pool i relative to water (mean_M0a will refer to the magnetization of the water pool in native units of the data). 
 * `mean_kia` - the exchange rate of pool i with pool a (water). Note the exchanges between pools other than water are assumed to be negligible. 
 * `mean_ppm_off` - the offset of the water centre frequency, due to B0 field inhomogeneity (in ppm). 
 * `mean_ppm_i` - the offset of the pool centre frequency relative to water (in ppm). Note there will not be a result for pool a since this is water and its offset is given in mean_ppm_off. 
 * `mean_T1i` and `mean_T2i` - T1 and T2 values for the pool (if using --t12prior). These should not necessarily be taken as accurate estimates for T1 and T2 values since the data is not well suited to such estimation, but may indicate where T1 and T2 variations occur. However, the uncertainty in T1 and T2 values will have been incororated into the uncertainty estimates on the other parameters. It is most likely that these will reflect the values entered in the poolmat matrix, this is normal. 

A modelfit image is also generated which can be compared to the data to visualise the fit of the model to the data. 