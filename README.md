# dfrws2015-challenge

The purpose of this challenge is to foster interest in development of GPU memory analysis tools, to enhance our abilities to understand and mitigate GPU-enhanced malware.  The goal isn't to present insanely difficult to analyze GPU malware--that, unfortunately, is likely to develop on its own, and better that the community starts preparing before it happens, by developing appropriate tools and techniques.  As with most DFRWS challenges, there's both low-hanging fruit and harder stuff to deal with, so regardless of your skill level, we hope you'll have fun and get something out of the experience.

This page provides some basic tools and resources to support the challenge and associated research efforts.  Importantly, and thanks to NVidia's cooperation, downloads of the first-ever tools for comprehensive GPU memory dumping for Linux are provided in the Downloads area, at the bottom of the page.

## Scenario

Niles Boudreaux, the infamous Cajun chef from the DFRWS 2014 Forensics Rodeo (review the DFRWS 2014 Forensics Rodeo materials for background) can't seem to catch a break.  After being hacked by his former IT specialist in that scenario, he retired from the frozen food business, downsized, and is trying to rekindle his cooking mojo, develop new recipes and plan a new venture.  Niles has also upped his computer skills after receiving a brand new Lenovo Y50 from his Cajun nephew (Golden, who configured it and then ran away, fearing for his life, given past experience). Once again, however, tragedy may have struck.  After noticing strange computer behavior, Niles contacted his friend and forensic expert, Steve Carrier, to investigate.  Steve took a photo of Niles' screen using his iPhone (see below) and then a LiME-format physical memory dump of Nile's computer. Because of his increased awareness of GPU-based malware (Steve and Niles have shared more than a few drinks and a paella or two over discussions about Niles' poor luck with computers) and the availability of appropriate forensics tools for dumping GPU memory, Steve took a GPU memory dump.  Given Niles' propensity for bad luck, it's not surprising that Steve choked on a particularly large bite of boudin (supplied by Niles, to speed the investigation, of course) shortly after initiating the forensics investigation. Luckily, the two memory dumps live on, and that's where you come in.  Just steer well clear of Niles and chew carefully while you work, if you know what we mean.

## Investigative Questions

Niles is particularly interested in:
- Identifying any malware that's present on his computer system.  The exact nature of the malware, its apparent purpose, and any potentially malicious actions are of interest.
-  Determining exactly what data, if any, was collected by any malware that's present.  Was any of Niles' personal or work-oriented information collected?

## Challenge Goals and Scoring

The goal of the challenge is to foster development of semi-automated or automated tools for analyzing GPU-enabled malware.  Tools that bridge the gap between traditional memory analysis and analysis of GPU device memory and between reverse engineering of host and GPU malware components are particularly encouraged.  Therefore, while one goal of the challenge is to answer the investigative questions, much stronger weight will be given to submissions that support flexible, reusable, (semi-?)automated analysis than submissions based on manual analysis.  Translation: While mad hex editor skills are cool, write some tools!

## Rules

Contestants may enter individually, or as a team, with no restrictions.  Source code for tools developed to solve the challenge must be openly available under a free software license, such as those listed at (http://www.gnu.org/licenses/license-list.html). The author(s) retain rights to the source code.  Tools may incorporate third-party free software, as long as it is compatible with your license and is included with your submission. However, submissions will be judged on the contribution your own research brings to the challenge.

Submissions must include clear instructions for building tool(s) from source code along with all relevant dependencies.

DFRWS will publish the results of the Challenge, both in detailed and summary form, along with the methodology used and the source code for the tools.

## Notes on Using Volatility

The 3.13.0-32 kernel on Niles' Lenovo Y50 XUbuntu 14.04 installation, used to create the final challenge materials, uses a relocatable kernel, as described here: (http://cateee.net/lkddb/web-lkddb/RELOCATABLE.html).  The Volatility developers hadn't previously encountered kernels compiled with this option, and so some updates to Volatility were necessary.  If you're using Volatility with the profile supplied in the Downloads area, below, to tackle the host memory dump, be sure to git pull to get the latest file versions.

## Using the GPU Memory Dumping Tools

You can work on the challenge with no access to specialized hardware, but because of the low associated costs and the advantages of being able to create your own scenarios and GPU memory dumps, creating a test system is strongly recommended. Currently, the GPU dumping tools work exclusively on Linux and on select NVIDIA GPUs.  A patched driver is necessary to enable GPU memory dumping--this is detailed in the README inside the tools tarball in the Downloads area, below.  The target platform is currently Ubuntu 14.04 and NVIDIA GTX 750 Ti / NVIDIA GTX 860M GPUs.  Other modern NVIDIA GPUs are very likely to work, as are other Linux distributions.  Please relay your experiences with alternate platforms to me via challenge@dfrws.org so that I can update the list below accordingly.  Please let me know if you have any problems with the tools, but please DO NOT bug NVidia!

### Currently tested platforms

- 64-bit XUbuntu 14.04 w/ GTX 750Ti NVIDIA GPU (Golden G. Richard III)
- 64-bit XUBuntu 14.04 on Lenovo Y50 with GTX 860M (Golden G. Richard III)

Detailed installation instructions are included in the tarball--please study the README carefully.  I've provided links to the tarball and to the necessary driver and prerequisite packages that are required at the bottom of the page.  Essentially, all you have to do is install the required packages, apply patches as specified, and then  make all to build  nvidia_fb, which is the memory dumping utility, and a test suite.  The tools must be run as root to access GPU memory.

You can determine the size of GPU device memory using nvidia-smi, e.g., 

```
$ nvidia-smi

nvidia-smi
Thu Sep 18 17:32:32 2014
+------------------------------------------------------+
| NVIDIA-SMI 343.13     Driver Version: 343.13         |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 860M    Off  | 0000:01:00.0     N/A |                  N/A |
| N/A   43C    P0    N/A /  N/A |      7MiB /  2047MiB |     N/A      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Compute processes:                                               GPU Memory |
|  GPU       PID  Process name                                     Usage      |
|=============================================================================|
|    0            Not Supported                                               |
+-----------------------------------------------------------------------------+ 
```

The following command line should be used to dump GPU device memory:

$ ./dump_fb  -o start   -s  length   -f  dumpfile   -g GPU-UUID

where start is a 4K-aligned location in GPU RAM at which dumping should begin, length is a 4K-aligned size (in bytes) for the dump operation, dumpfile is the name of the image file to generate, and GPU-UUID is the identifier for the GPU that's targeted.  Use the following command to determine the GPU ID:

```
$ nvidia-smi -L

GPU 0: GeForce GTX 860M (UUID: GPU-e2153072-ec1d-fa00-6f01-8749912913e2)
```
and use only the bold portion (i.e., omit the "GPU-" prefix).  For example, the following command dumps the first 4K of memory from the specified GPU:

$ sudo ./dump_fb -o 0 -s 4096 -f blarg -g e2153072-ec1d-fa00-6f01-8749912913e2

Note that the package also contains a test suite.  After installation, test by executing the nvidia_fb_test application and specifying a GPU UID (again, without the "GPU-" prefix) using "-g".  A correct execution of the test cases is illustrated below.  Anything else is an indicator of a problem with the installation.

```
$ nvidia-smi -L

GPU 0: GeForce GTX 860M (UUID: GPU-e2153072-ec1d-fa00-6f01-8749912913e2)
niles@niles:~/nvidia-dump-fb-343.13$
$ sudo ./dump_fb_test -g e2153072-ec1d-fa00-6f01-8749912913e2
[==========] Running 20 tests from 2 test cases.
[----------] Global test environment set-up.
[----------] 10 tests from DumpFbTest
[ RUN      ] DumpFbTest.ZeroLength
[       OK ] DumpFbTest.ZeroLength (219 ms)
[ RUN      ] DumpFbTest.NullPtr
[       OK ] DumpFbTest.NullPtr (110 ms)
[ RUN      ] DumpFbTest.RandomPtr
[       OK ] DumpFbTest.RandomPtr (109 ms)
[ RUN      ] DumpFbTest.ReadOnly
[       OK ] DumpFbTest.ReadOnly (114 ms)
[ RUN      ] DumpFbTest.BasicTest
[       OK ] DumpFbTest.BasicTest (197 ms)
[ RUN      ] DumpFbTest.LargeTest
[       OK ] DumpFbTest.LargeTest (213 ms)
[ RUN      ] DumpFbTest.OffsetTest
[       OK ] DumpFbTest.OffsetTest (281 ms)
[ RUN      ] DumpFbTest.ConsistencyCheck
[       OK ] DumpFbTest.ConsistencyCheck (284 ms)
[ RUN      ] DumpFbTest.UnalignedCpuAddress
[       OK ] DumpFbTest.UnalignedCpuAddress (116 ms)
[ RUN      ] DumpFbTest.OverflowTest
[       OK ] DumpFbTest.OverflowTest (136 ms)
[----------] 10 tests from DumpFbTest (1780 ms total)

[----------] 10 tests from PerformanceTest/PerformanceTest
[ RUN      ] PerformanceTest/PerformanceTest.TestBandwidth/0
86.8516ms
4.3922e-05GB/s
[       OK ] PerformanceTest/PerformanceTest.TestBandwidth/0 (87 ms)
[ RUN      ] PerformanceTest/PerformanceTest.TestBandwidth/1
87.1853ms
0.011201GB/s
[       OK ] PerformanceTest/PerformanceTest.TestBandwidth/1 (87 ms)
[ RUN      ] PerformanceTest/PerformanceTest.TestBandwidth/2
106.79ms
0.585264GB/s
[       OK ] PerformanceTest/PerformanceTest.TestBandwidth/2 (107 ms)
[ RUN      ] PerformanceTest/PerformanceTest.TestBandwidth/3
126.247ms
0.990123GB/s
[       OK ] PerformanceTest/PerformanceTest.TestBandwidth/3 (127 ms)
[ RUN      ] PerformanceTest/PerformanceTest.TestBandwidth/4
404.586ms
2.47166GB/s
[       OK ] PerformanceTest/PerformanceTest.TestBandwidth/4 (407 ms)
[ RUN      ] PerformanceTest/PerformanceTest.TestBandwidthLocked/0
86.7522ms
4.39723e-05GB/s
[       OK ] PerformanceTest/PerformanceTest.TestBandwidthLocked/0 (87 ms)
[ RUN      ] PerformanceTest/PerformanceTest.TestBandwidthLocked/1
87.067ms
0.0112162GB/s
[       OK ] PerformanceTest/PerformanceTest.TestBandwidthLocked/1 (87 ms)
[ RUN      ] PerformanceTest/PerformanceTest.TestBandwidthLocked/2
106.163ms
0.588717GB/s
[       OK ] PerformanceTest/PerformanceTest.TestBandwidthLocked/2 (110 ms)
[ RUN      ] PerformanceTest/PerformanceTest.TestBandwidthLocked/3
120.064ms
1.04111GB/s
[       OK ] PerformanceTest/PerformanceTest.TestBandwidthLocked/3 (128 ms)
[ RUN      ] PerformanceTest/PerformanceTest.TestBandwidthLocked/4
349.97ms
2.85739GB/s
[       OK ] PerformanceTest/PerformanceTest.TestBandwidthLocked/4 (411 ms)
[----------] 10 tests from PerformanceTest/PerformanceTest (1638 ms total)

[----------] Global test environment tear-down
[==========] 20 tests from 2 test cases ran. (3535 ms total)
[  PASSED  ] 20 tests.
```

I've also provided an application for stuffing GPU device memory with the contents of a file.  This is useful to get started with GPU memory analysis, because it allows you to benignly stuff arbitrary content (readable strings, binary structures, etc.) into GPU device memory and then test the dumping tools (and your own tools). It's a CUDA application called fillgpu and can be downloaded below.  A simple proof of concept GPU-based keylogger (which works only for USB keyboards and has both userspace and kernelspace components) is also available below.

Again, please let me know if you have any problems with the tools.  Please DO NOT bug NVidia!

## Downloads

### Supporting materials
- [GPU mem dump tools tarball (nvidia-dump-fb-1.0.tar.gz)](materials/nvidia-dump-fb-1.0.tar.gz)        MD5: 84e0f7e0ca9aaece30d0205c8e481536
- [GPU deployment kit (cuda_340_29_gdk_linux_64.run)](materials/cuda_340_29_gdk_linux_64.run)             MD5: 12406064cd1654e8b2f22ceea6c77502
- [343.13 graphics driver](https://www.dropbox.com/s/w1a0pkkedlocst7/NVIDIA-Linux-x86_64-343.13.run?dl=0) MD5: 583597fb4793542a65a889b6262a8816
- [fillgpu tarball (fillgpu.tar.gz)](materials/fillgpu.tar.gz)         MD5: 6c25fea7a9e832a00a5f0839eb2827b5
- [keystroke logger tarball (keylogger.tar.gz)](materials/keylogger.tar.gz)        MD5: ba234b4aa961b8ad83ffab8a926ef939

### Challenge memory images

- [Host memory image](https://www.dropbox.com/s/smyuaoj49ap9dfl/NILES.lime.bz2?dl=0) (NILES.lime.bz2) MD5: 34b888a3982754ded8aab9101564391d
- [GPU memory image](https://www.dropbox.com/s/klkn706yndwadbg/NILES.gpu.bz2?dl=0)  MD5: d3f368882b4a8ea2c07a10cbd6801c35 
- [Volatility 2.4 profile for Niles' machine (Xubuntu1404.zip)](materials/Xubuntu1404.zip)   MD5: c9871e739ed20cc8bfec2d532c4425b6

### Submitted Challenge Solutions

Note: these are partial solutions and do not address all aspects of the challenge.  You should consider this a challenge in itself, to continue development of better tools for GPU memory forensics and malware analysis.

- [Solution # 1](solutions/Challenge.tar)
- [Solution # 2](solutions/dfrws-challenge-cquates.zip)

### GPU malware source code
- [Source ZIP](materials/source.zip)
