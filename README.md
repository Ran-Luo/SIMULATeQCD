# SIMULATeQCD


[![](https://img.shields.io/badge/docs-dev-blue.svg)](https://latticeqcd.github.io/SIMULATeQCD)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://GitHub.com/Naereen/StrapDown.js/graphs/commit-activity)


*Like I Knew What I am Doing*


SIMULATeQCD is a multi-GPU Lattice QCD framework that tries to make it simple and easy for physicists to implement lattice QCD formulas while still providing the best possible performance.


## Prerequisites

The following software is required to compile SIMULATeQCD:

1. `cmake` (Some versions have the "--phtread" compiler bug. Versions that definetely work are [3.14.6](https://gitlab.kitware.com/cmake/cmake/tree/) v3.14.6 or 3.19.2 
2. `C++` compiler with `C++17` support  (e.g. `g++-9`)
3. `MPI` (e.g. `openmpi-4.0.4`) and
4. `CUDA Toolkit` version 11.0 (NOT 11.1 or 11.2)
5. `pip install -r requirements.txt` to build the documentation

## Download the code

Clone the code to your machine:

```shell
git clone https://github.com/LatticeQCD/SIMULATeQCD.git
```

## Building the code

To setup the compilation, create a folder outside of the code directory (e.g. `../build/`) and *from there* call the following example script (see source:/doc/cmake.sh, you need to manually change this for your machine): 
```shell
cmake ../simulateqcd/ \
-DARCHITECTURE="70" \
-DUSE_CUDA_AWARE_MPI=ON \
-DUSE_CUDA_P2P=ON \
``` 
Here, it is assumed that your source code folder is called `simulateqcd`. *Do NOT compile your code in the source code folder!* 
You can set the path to CUDA by setting the `cmake` parameter `-DCUDA_TOOLKIT_ROOT_DIR:PATH`.
`-DARCHITECTURE` sets the GPU architecture (i.e. [compute capability](https://en.wikipedia.org/wiki/CUDA#GPUs_supported) version without the decimal point). For example "60" for Pascal and "70" for Volta. 
Inside the build folder, you can now begin to use `make` to compile your executables, e.g. 
```shell
make NameOfExecutable
```
If you would like to speed up the compiling process, add the option `-j`, which will compile in parallel using all available CPU threads. You can also specify the number of threads manually using, for example, `-j 4`.



## Documentation

Please check out [the documentation](https://latticeqcd.github.io/SIMULATeQCD) to learn how to use SIMULATeQCD.

## Example: Plaquette action computation

(See [Full code example](https://github.com/LatticeQCD/SIMULATeQCD/blob/main/src/examples/main_plaquette.cu).)

```C++
template<class floatT, bool onDevice, size_t HaloDepth>
struct CalcPlaq {
  gaugeAccessor<floatT> gaugeAccessor;
  CalcPlaq(Gaugefield<floatT,onDevice,HaloDepth> &gauge) : gaugeAccessor(gauge.getAccessor()){}
  __device__ __host__ floatT operator()(gSite site) {
    floatT result = 0;
    for (int nu = 1; nu < 4; nu++) {
      for (int mu = 0; mu < nu; mu++) {
        result += tr_d(gaugeAccessor.template getLinkPath<All, HaloDepth>(site, mu, nu, Back(mu), Back(nu)));
      }
    }
    return result;
  }
};

Gaugefield<floatT, true, HaloDepth> gauge(...);
iterateOverBulk<All, HaloDepth>(CalcPlaq<floatT, HaloDepth>(gauge))
```



