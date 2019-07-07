# Ubuntu18.04-NVIDIA-CUDA10.0-CUDNN-Pytorch-Installation

## 安装Ubuntu18.04
	在固态硬盘上安装ubuntu 16.04, 系统是装好了，不过却出现进不了桌面或者桌面闪现的现象（可以登陆tty界面），在尝试了安装nvidia驱动、修改grub，修改security boot等等方法后均没有解决问题，故放弃。个人猜测是跟bios 的设置有关。改成在机械硬盘上安装ubuntu 18.04，很快就安上了。

## 安装NVIDIA驱动
### 禁用nouveau
```
sudo vim /etc/modprobe.d/blacklist.conf
```
在文件最后部分插入以下两行内容:
```
blacklist nouveau
options nouveau modeset=0
```
更新系统： ```sudo update-initramfs -u```
重启系统（一定要重启）
验证nouveau是否已禁用： ```lsmod | grep nouveau```
若是没有信息显示，说明nouveau已被禁用，接下来可以安装nvidia的显卡驱动。

### 关闭lightdm
ctrl+alt+F1切换到tty1界面, 运行：sudo service lightdm stop
若是出现don’t find lightdm之类的error，我们可以先安装lightdm：
sudo apt-get install lightdm安装之后再关闭服务。

3）运行runfile: 
我们的显卡型号是GTX TITAN X，经过官网查询，适合安装目前最新版的驱动：
sudo ./NVIDIA-Linux-x86_64-430.26.run n -no-x-check -no-nouveau-check -no-opengl-files  
-no-x-check：安装驱动时关闭X服
-no-nouveau-check：安装驱动时禁用nouveau
-no-opengl-files：只安装驱动文件，不安装OpenGL文件  // 其他两个选项可有可无，这个一定要写明，只有禁用opengl才不会出现循环登陆的问题。
运行过程中可能会出现没有找到cc模块之类的错误，这是因为ubuntu18.04没有预装gcc等编译器，我们先给它装上，顺便把g++等也给一并装了：
sudo apt-get install gcc
sudo apt-get install g++
sudo apt-get install make
然后重新运行runfile, 安装过程中的选项：（这是copy别人的，自己的没记住）
	The distribution-provided pre-install script failed! Are you sure you want to continue? 选择 yes 继续。
	Would you like to register the kernel module souces with DKMS? This will allow DKMS to 	automatically build a new module, if you install a different kernel later?  选择 No 继续。
	问题没记住，选项是：install without signing
	问题大概是：Nvidia's 32-bit compatibility libraries? 选择 No 继续。
	Would you like to run the nvidia-xconfigutility to automatically update your x configuration so that the NVIDIA x driver will be used when you restart x? Any pre-existing x confile will be backed up.  选择 Yes  继续
这些选项如果选择错误可能会导致安装失败，没关系，只要前面不出错，多尝试几次就好。

4) 检查驱动是否安装成功：nvidia-smi
5) 重启电脑：sudo reboot

3、安装CUDA:
1）查询型号
在nvidia官网上查询能与我们所安装的驱动型号相匹配的cuda，并且还要查询pytorch所支持的cuda型号，最终我们选定了cuda 10.0。

2）安装
进入刚刚下载的CUDA包的路径，执行命令：
sudo sh cuda_10.0.130_410.48_linux.run
然后按照下面的执行：
    Do you accept the previously read EULA? (accept/decline/quit): accept 
    You are attempting to install on an unsupported configuration. Do you wish to continue? ((y)es/(n)o) [ default is no ]: y 
    Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 390.77? ((y)es/(n)o/(q)uit): n 
    Install the CUDA 10.0 Toolkit? ((y)es/(n)o/(q)uit): y 
    Enter Toolkit Location [ default is /usr/local/cuda-10.0 ]: 
    Do you want to install a symbolic link at /usr/local/cuda? ((y)es/(n)o/(q)uit): y 
    Install the CUDA 10.0 Samples? ((y)es/(n)o/(q)uit): y 
    Enter CUDA Samples Location [ default is /home/luo ]:
其实就是除了第二个安装图形驱动因为之前安装过选择no,其他就是yes或者默认。

3) 配置CUDA的环境变量
sudo gedit ~/.bashrc
在文件的末尾添加下面两行，注意修改成你的安装路径：
    export PATH="/usr/local/cuda-10.0/bin:$PATH" 
    export LD_LIBRARY_PATH="/usr/local/cuda-10.0/lib64:$LD_LIBRARY_PATH"
source ~/.bashrc
测试一下CUDA是否安装成功：nvcc -V 

4）cuda卸载：
在最开始我们就安装了最新的cuda 10.1，然后才发现pytorch不支持该型号，无奈只能将它卸载: 
sudo /usr/local/cuda-10.1/bin/cuda-uninstaller
sudo rm -rf /usr/local/cuda-10.1
如果是卸载cuda 10.0及以下版本：
sudo /usr/local/cuda-10.0/bin/uninstall_cuda_10.0.pl
sudo rm -rf /usr/local/cuda-10.0



4、安装CUDNN:
打开官网https://developer.nvidia.com/cudnn，登陆账号，找到与cuda 10.1和ubuntu18.04相匹配的cudnn型号, 这里我们选择deb安装，所以下载deb包并安装：
sudo dpkg -i libcudnn7_7.6.0.64-1+cuda10.0_amd64.deb
sudo dpkg -i libcudnn7-doc_7.6.0.64-1+cuda10.0_amd64.deb
验证是否成功装上：
sudo cp -r /usr/src/cudnn_samples_v7/ $HOME
cd  $HOME/cudnn_samples_v7/mnistCUDNN
make clean && make
./mnistCUDNN
If cuDNN is properly installed and running on your Linux system, you will see a message similar to the following:
Test passed!
5、安装anaconda:
在anaconda官网上下载对应python 3.7的脚本Anaconda3-2019.03-Linux-x86_64.sh，运行：
sh ./Anaconda3-2019.03-Linux-x86_64.sh 
注意不要加sudo, 否则对Anaconda3里的文件没有权限，导致后面的pytorch安装失败。


6、安装pytorch:
尝试过用conda和源码安装pytorch, 然而总是因为各种各样的原因失败了。
最后找到一个离线安装的办法，首先下载torch安装包，下载地址：https://pytorch.org/previous-versions/，根据自己的python版本和cuda型号，我选择了torch-1.0.1.post2-cp37-cp37m-linux_x86_64.whl， 这里1.0.1 代表torch的版本，cp37代表支持的python 3.7，linux_x86_64代表linux系统下的64位版本。
进入安装包的路径，执行：
 pip install torch-1.0.1.post2-cp37-cp37m-linux_x86_64.whl
通过import torch看是否成功安装torch。安装成功后，我们接着安装torchvision。
然而奇怪的是，虽然成功安装了torch，但是好像不能被识别出来？一旦pip install torchvision，系统又会跑去重新下载并安装torch.whl，然后出现error。我接着尝试从torchvision的源码（https://github.com/pytorch/vision）进行安装，然后又出现错误，发现源码只支持torch1.1.0。我接着就想把torch1.0.1升级到torch1.1.0，执行：
conda config --add channels soumith
conda update pytorch torchvision
发现问题解决了，成功更新了torch和torchvision.
查看pytorch版本：
import torch
print(torch.__version__)

参考网址：
https://blog.csdn.net/wuzhiwuweisun/article/details/82753403
https://blog.csdn.net/red_stone1/article/details/78727096
https://blog.csdn.net/miao0967020148/article/details/80400357
