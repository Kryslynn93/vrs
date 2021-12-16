# What is VRS?

VRS is a **file format** optimized to record & playback streams of sensor data, such as images, audio samples, and any other discrete sensors (IMU, temperature, etc), stored in per-device streams of time-stamped records.

VRS was first created to record images and sensor data from early prototypes of the Quest device, to develop the device’s positional tracking system now known as [Insight](https://ai.facebook.com/blog/powered-by-ai-oculus-insight/), and Quest's hand tracking software. It is also the file format used by the [Aria glasses](https://about.facebook.com/realitylabs/projectaria/).

## Main features
* VRS files contain multiple streams of time-sorted records generated by a set of "devices", typically one per stream.
* File and streams contain a set of "tags", which are a set of string name-value pairs, to describe them.
* Streams may contain `Configuration`, `State` and `Data` records, each with a timestamp in a common time domain for the whole file.\
Typically, streams contain with one `Configuration` and one `State` record, followed one to millions of `Data` records.
* Records are structured as a succession of typed content blocks.\
Typical content blocks are metadata, image, audio and custom content blocks.
* Metadata content blocks contain raw sensor data described once per stream, making the file format very efficient. The marginal cost of adding 1 byte of data to each metadata content block of a stream is 1 byte per record (or less, when lossless compression happens).
* Records can be losslessly compressed using lz4 or zstd, which are can be fast enough to do compress during recording.
* Multiple threads can create records concurrently for the same file.
* VRS supports huge file size (tested with multi terabytes use cases).
* VRS supports chunked files: auto-chunking on creation, automated chunk detection for playback.
* Playback is optimized for timestamp order (key for network streaming).
* Random-access playback is fully supported (in memory file and stream indexes).
* Customizable `FileHandler` support, to implement streaming from cloud storage (not provided yet).

# Getting Started

To work with VRS files, the vrs open source project provides a C++ library with external open source dependencies such as [boost](https://github.com/boostorg/boost), [cereal](https://github.com/USCiLab/cereal), [fmt](https://github.com/fmtlib/fmt), [lz4](https://github.com/lz4/lz4), [zstd](https://github.com/facebook/zstd), [xxhash](https://github.com/Cyan4973/xxHash), and [googletest](https://github.com/google/googletest) for unit tests.
To build & run VRS, you’ll need a C++17 compiler, such as a recent enough version of clang or Visual Studio.

The simplest way to build VRS is to install the libraries on your system using some package system, such as [Brew](https://brew.sh/) on macOS, or [apt](https://en.wikipedia.org/wiki/APT_(software)) on Ubuntu, and then use cmake to build & test. VRS supports many other platforms such as Windows, Android, iOS and other flavors of Linux, but we currently only provide instructions for macOS and Ubuntu.

## Instructions (macOS and Ubuntu)

### Install build tools & libraries (macOS)
* install Brew, following the instruction on [Brew’s web site](https://brew.sh/).
* install tools & libraries:
  ```
  brew install cmake ninja ccache boost fmt cereal lz4 zstd xxhash googletest
  brew install node doxygen
  ```
### Install build tools & libraries (Linux)
* install tools & libraries:
  ```
  sudo apt-get install cmake ninja-build ccache libgtest-dev libfmt-dev libcereal-dev liblz4-dev libzstd-dev libxxhash-dev
  sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-thread-dev libboost-chrono-dev libboost-date-time-dev
  sudo apt-get install npm doxygen
  ```

## Build & run (macOS & Linux)

* Run cmake:
```
cmake -S <path_to_vrs_folder> -B <path_to_build_folder> '-GCodeBlocks - Ninja'
```

* Build everything & run tests:
```
cd <path_to_build_folder>
ninja all
ctest -j8
```

## Windows Support

We don’t have equivalent instructions for Windows.
[vcpkg](https://vcpkg.io/en/index.html) looks like a promising package manager for Windows, but the cmake build system needs more massaging to work.\
Contributions welcome! :-)

# Documentation & Support

VRS is documented in three complementary ways:
1. A high level documentation built using [Docusaurus](https://docusaurus.io/), right in the GitHub repo, in the `website` folder.
2. The API documentation built using [Doxgyen](https://www.doxygen.nl/manual/index.html), or by looking at the source code.
   Note that we focus on documenting the API users of VRS should be using, while the VRS internal API is less systematically documented. If you find a class that isn't well documented, or not documented at all, you should probably avoid using that class directly. In doubt, ask.
4. Sample code.

To read the VRS documentation with Docusaurus using a web browser (macOS):
```
cd <top_vrs_repo_folder>/website
npm build
npm serve
```

The API documentation can also be built using:
```
cd <top_vrs_repo_folder>
doxygen vrs/Doxyfile
open website/static/doxygen/index.html # macOS only
```


We plan on having a VRS Users group dedicated on discussing VRS usage. Stay tuned for details.

# Sample Code

* [The sample code](./sample_code) demonstrates the basics to record and read a VRS file, then how to work with `RecordFormat` and `DataLayout` definitions. The code is extensively documented, functional, compiles but isn’t meant to be run.
    * [SampleRecordAndPlay.cpp](./sample_code/SampleRecordAndPlay.cpp)\
        Demonstrates different ways to create VRS files, and how to read them, but the format of the records is deliberately trivial, as it is not the focus of this code.
    * [SampleImageReader.cpp](./sample_code/SampleImageReader.cpp)\
        Demonstrates how to read records containing images.
    * [SampleRecordFormatDataLayout.cpp](./sample_code/SampleRecordFormatDataLayout.cpp)\
        Demonstrates how to use `RecordFormat` and `DataLayout` in more details.
* [The sample apps](./sample_apps) are fully functional apps demonstrate how to create, then read, a VRS file with 3 types of streams.
    * a metadata stream
    * a stream with metadata and uncompressed images
    * a stream with audio images

# Contributing

We welcome contributions! See [CONTRIBUTING](CONTRIBUTING.md) for details on how to get started, and our [code of conduct](CODE_OF_CONDUCT.md).

# Future Plans
In this first release of VRS for open source, only the core components of VRS are provided. We are working on open sourcing more code:
* `VRStool`: a command line tool to manipulate VRS files.
* `VRSplayer`: a basic "player", to view VRS file like multi-stream video files.
* `pyvrs`: a Python library to work with VRS files in Python.
* integration with PyTorch, so ML jobs can consume VRS files as training data.
* tooling to build VRS container files optimized for PyTorch traning.
* video codec compression support.
* building blocks to implement network streaming.

# License

VRS is released under the [Apache 2.0 license](LICENSE).
