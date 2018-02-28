# 编程语言及工具配置

---

<!--sec data-title="Python" data-id="lan_0" data-show=true ces-->
### 安装基于GPU(GT 750M)的tensorflow
建议参照最新的[官网说明](http://docs.nvidia.com/cuda/cuda-installation-guide-mac-os-x/#system-requirements)一步一步做。

1. 下载并安装与MacOS系统相匹配的显卡驱动：`http://www.macvidcards.com/drivers.html`
2. 下载并安装与MacOS系统相匹配的CUDA Toolkit：`https://developer.nvidia.com/cuda-downloads`
3. 下载并解压CUDNN：`https://developer.nvidia.com/rdp/cudnn-download`
4. 将解压后的文件夹cuda内的文件拷贝到`/usr/local/cuda`下：
```bash
cd cuda
sudo cp lib/* /usr/local/cuda/lib
sudo cp cudnn.h /usr/local/cuda/include/
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib/libcudnn*
sudo ln -s /usr/local/cuda/lib /usr/local/cuda/lib64
```
5. 配置环境变量：
```bash
export CUDA_HOME=/usr/local/cuda
export DYLD_LIBRARY_PATH="$CUDA_HOME/lib:$CUDA_HOME/extras/CUPTI/lib"
export LD_LIBRARY_PATH=$DYLD_LIBRARY_PATH
export PATH=$DYLD_LIBRARY_PATH:$PATH
export flags="--config=cuda --config=opt"
```
6. 测试cuda是否安装成功：
```bash
cp -r /usr/local/cuda/samples ~/cuda-samples
pushd ~/cuda-samples
make
popd
~/cuda-samples/bin/x86_64/darwin/release/deviceQuery
```
7. 下载tensorflow源码：`git clone https://github.com/tensorflow/tensorflow`
8. 配置：`cd tensorflow; ./configure`
9. 编译：`bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package`
10. 生成pip安装包：`bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg`
11. 安装tensorflow：`sudo pip3 install /tmp/tensorflow_pkg/tensorflow-1.3.0-cp27-cp27m-macosx_10_7_x86_64.whl`

### Jupyter Notebook
* **安装theme**：`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple jupyterthemes`；
* **设置主题样式**：`jt -t onedork -f roboto -fs 12 -nfs 12 -ofs 10 -T -N`；
* **启动笔记本**：`jupyter notebook`；
<!--endsec-->

<!--sec data-title="Go" data-id="lan_1" data-show=true ces-->
### 安装Go语言环境
1. 下载Go：<https://golang.org/dl/>；
2. 将Go安装到`/usr/local`下：`tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz`；
3. 编辑`/etc/profile`，增加`export PATH=$PATH:/usr/local/go/bin`；
4. 发布修改：`source /etc/profile`；
<!--endsec-->