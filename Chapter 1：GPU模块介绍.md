### 一、GPU模块介绍

------------------
#### 1、基本信息

- OpenCV GPU模块是一个类和函数的集合，它利用了GPU的计算能力。该模块使用NVIDIA CUDA运行时API来实现，并且只能支持NVIDIA的GPU。OpenCV的GPU模块包含了一些实用函数、视觉处理基础算法及高级算法。这些实用函数和基础算法可以为应用开发人员利用GPU开发出快速的视觉算法包括一些先进的算法（如：立体匹配、人及其面部检测等等）提供强大的基础支持。

- GPU模块被设计为主机（host）级别的API。也就是说，如果你有预编译OpenCV的GPU二进制文件，那么你不需要安装有CUDA工具包或编写额外的代码就可以使用GPU。

- OpenCV的GPU模块被设计得易于使用，并且不需要任何CUDA的知识。不过，拥有这方面的知识将会对处理比较复杂的情况或实现最高性能非常有帮助，也有助于理解各种操作的开销，GPU的运作，以及最合适的数据格式等等。GPU模块一个快速实现计算机视觉算法GPU加速的有效工具。然而，如果你的算法涉及到许多简单的操作，那么，为了性能最优，你可能仍然需要写你自己的内核，以避免对中间结果额外的读写操作。

- 启用CUDA支持，在CMake中配置OpenCV时需要设置WITH\_CUDA=ON当这个标志被设置且安装了CUDA，则就拥有了功能齐全的OpenCV GPU模块。否则，虽仍可建立该模块，但在运行时所有该模块的功能函数（除[gpu::getCudaEnabledDeviceCount()][2]函数，返回GPU数为0外）都会抛出错误代码为CV_GpuNotSupported的[Exception][1]。建立没有CUDA支持的OpenCV不用编译设备代码，所以它不需要CUDA工具包安装，因此，使用[gpu::getCudaEnabledDeviceCount()][2]函数，可以在运行时检测GPU的存在，并选择相应的CPU或GPU来实现高级算法。

------------------
#### 2. 编译不同的NVIDIA平台

- NVIDIA编译器能够生成二进制代码（cubin和fatbin）及中间代码（PTX）。二进制代码往往意味着一个特定的GPU架构和生成，因此不能保证与其他GPU的兼容性。PTX代码由虚拟平台生成，且完全被一系列的功能特征定义。根据所选定的虚拟平台，会模拟或禁用了一些指令，即使真正的硬件支持所有的功能。

- 在第一次调用时，对于特定的GPU，JIT（Just-In-Time）编译器将PTX代码编译为二进制代码，当目标GPU的计算能力低于PTX代码，则JIT编译失败（译者注：由于向后兼容性）。默认情况下，OpenCV的GPU模块获得：
	- 二进制代码：计算能力1.3和2.0（由CMake中CUDA_ARCH_BIN控制）
	- PTX代码：计算能力1.1和1.3（由CMake中CUDA_ARCH_PTX控制）

- 这意味着，计算能力为1.3和2.0的设备运行的是二进制图像。对于所有较新平台，计算能力1.3的PTX代码被JIT编译成二进制图像。计算能力1.1和1.2，被JIT编译成1.1的PTX代码。对于计算能力1.0的设备，没有可用代码并且函数会抛出[Exception][1]。对于首次执行JIT编译的平台而言，运行速度比较慢。
- 在计算能力1.0的GPU上，仍然可以编译GPU模块，且大部分的函数都可以完美的运行。为了实现这一点，需要将1.0加入到二进制列表中。例如：CUDA_ARCH_BIN=”1.0 1.3 2.0”。无法在计算能力1.0的设别上执行的函数，将会抛出异常。
- 你可以在运行时检测OpenCV GPU生成的二进制文件（或PTX代码）是否与你的GPU兼容，利用[gpu::DeviceInfo::isCompatible()][3]函数返回兼容性状态（true/false）。

------------------
### 3. 利用多个GPU

- 在当前版本中，每个OpenCV GPU算法可以使用单个GPU。因此，为利用多个GPU，需手动的分配GPU之间的任务，可以使用[gpu::setDevice()][4]函数来切换激活的GPU设备，有关更多详细信息，请参阅CUDA C编程指南。
在使用多个GPU开发算法时，需要注意数据传递的开销。这对于简单的函数和小尺寸图像，是非常有意义的，这些开销可能会消除利用多个GPU带来的一切优势。但对于高级算法，应该考虑使用多个GPU进行加速。例如：立体块匹配算法已成功使用下述算法并行化：
	- 1、每个立体图像对分割为两个水平重叠的条纹；
	- 2、在单独的Fermi架构GPU中处理每对条纹（左、右图像）；
	- 3、将结果合并成一个单一的视差图。
- 利用该算法，一个Fermi架构的双GPU相比单GPU的提高了180%的性能。源代码示例，见：[https://github.com/opencv/opencv/tree/master/samples/gpu/][5]。

------------------
> `译者`：[**Derek**][6]
> 
> `邮箱`：**jingshuang_hu@163.com**


[1]: http://docs.opencv.org/2.4/modules/core/doc/utility_and_system_functions_and_macros.html#exception

[2]: http://docs.opencv.org/2.4/modules/gpu/doc/initalization_and_information.html#gpu-getcudaenableddevicecount

[3]: http://docs.opencv.org/2.4/modules/gpu/doc/initalization_and_information.html#gpu-deviceinfo-iscompatible

[4]: http://docs.opencv.org/2.4/modules/gpu/doc/initalization_and_information.html#gpu-setdevice

[5]: https://github.com/opencv/opencv/tree/master/samples/gpu/

[6]: https://github.com/hujingshuang
