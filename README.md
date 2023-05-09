Mau Artifact
========

[Mau](https://github.com/) is a grey-box, GPI-accelerated fuzzer for
EVM smart contracts. 


# Setup

We assume that your system has Docker installed. Also, you should be able to run
the `docker` command without `sudo`. The following command will build the
docker image name 'mau-artifact', using our [Dockerfile](./Dockerfile).

```
$ ./build.sh
```
Once the Docker image built, we activate a container.
```
docker run --rm --gpus 1 -it mau-artifact:latest
```
Then you will be in the contrainer's bash.

# Startup
We have released the binary of `mau`, which will use 256 GPU cores. After the paper has been public, we will release the source code enabling users to setup the GPU capability.

We use an example in the ground-truth as the tutorial, i.e., CVE-2018-10299

1. enter base folder `$ cd /home/test`
2. build the ourput directory `$ mkdir -p output/mau/2018-10299`
3. convert the EVM bytecode to LLVM IR 
```bash
$ ~/tools/mau/standalone-ptxsema ./benchmarks/B1/bin/2018-10299.bin -o output/mau/2018-10299/bytecode.ll --hex --fsanitize=intsan --dump
```
4. convert the LLVM IR to LLVM bitcode and link with instrumentation library
```bash
$ llvm-as output/mau/2018-10299/bytecode.ll -o output/mau/2018-10299/main.bc && llvm-link output/mau/2018-10299/main.bc ~/tools/mau/rt.o.bc -o output/mau/2018-10299/kernel.bc
```
5. translate LLVM bitcode to PTX code
```bash
$ llc-16 -mcpu=sm_86 ./output/mau/2018-10299/kernel.bc -o output/mau/2018-10299/kernel.ptx
```
6. fuzz it 120 seconds in GPU#0
```bash
$ ~/tools/mau/Smartian-GPU/build/Smartian fuzz -t 120 -p ~/benchmarks/B1/bin/2018-10299.bin -a ~/benchmarks/B1/abi/2018-10299.abi -k ~/output/mau/2018-10299/kernel.ptx -v 0 -o ~/output/mau/2018-10299/ -g 0
```
Once it found bugs, we output the information to the stdout.
change `verbose` to `2` to see the throughput metric.

# Know Issues
1. The final report does not record the number of detected bugs.
2. After the report outputed, there is a CUDA error but affect nothing to the fuzzing.
