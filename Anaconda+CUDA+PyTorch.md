#利用anaconda装Pytorch+CUDA，无需手动安装cuda等，十分方便灵活，且可以在同个电脑上实现不同的环境配置。

## 1、更新nvidia驱动
在ubuntu software上手动更新nvidia驱动至最新，并通过nvidea-smi查看是否成功更新驱动。

## 2、安装最新的anaconda:
先去官方地址下载好对应的安装包，然后运行：

`bash ~/Downloads/Anaconda3-5.2.0-Linux-x86_64.sh`

anaconda会自动将环境变量添加到PATH里面，如果后面你发现输入conda提示没有该命令，那么你需要`source ~/.bashrc`这样就是更新环境变量，就可以正常使用了。
如果发现这样还是没用，那么需要收到添加环境变量，打开~/.basrc 文件，在最后面加上：

`export PATH=/home/aeasringnar/anaconda3/bin:$PATH`

保存退出后，运行 `source ~/.bashrc`，再次输入`conda list`测试看看，应该就是没有问题啦！

## 3、给anaconda添加源：
`conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes`
添加完后可以通过`conda info`查看镜像。

## 4、新建conda环境： 
`conda create -n env1 python=3.6`

## 5、激活conda环境：
`conda activate env1`

## 6、安装cudatoolkit：
由于anaconda一直从默认的源检索cuda，导致下载速度特别慢。为了强制从清华源下载，运行如下命令：
`conda install cudatoolkit=9.2 -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/linux-64/`

## 7、安装cudnn：
`conda install cudnn=7.6.5 -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/linux-64/`

## 8、安装pytorch：
`conda install pytorch torchvision pytorch`

## 9、另一种办法：
首先手动下载cudatoolkit等安装包，并将安装包放入/anaconda/pkgs目录下，然后从本地安装：
`conda install --use-local pytorch-1.3.0-py3.6_cuda92_cudnn7_0.tar.bz2 和 conda install --use-local cudatoolkit-9.2-0.tar.bz2`

## 10、运行python：
`import torch`
`torch.cuda.is_available`
