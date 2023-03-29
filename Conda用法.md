文字版教程：https://mp.weixin.qq.com/s/KI-9z7FBjfoWfZK3PEPXJA
CUDA安装地址：https://developer.nvidia.com/cuda-toolkit-archive
CUDA安装版本查看：https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html
Pytorch官网：https://pytorch.org/
Anaconda下载地址：https://www.anaconda.com/products/individual
清华镜像网站：https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/
显卡天梯图：https://www.mydrivers.com/zhuanti/tianti/gpu/

# 管理conda
## 1. 查看 conda 版本
```
conda --version
```
## 2. 添加国内源
- 查看现有源
```
conda config --show-sources
```
- 添加国内清华源
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
```
- 删除默认源
```
conda config --remove channels defaults
```
- 设置搜索时显示通道地址
```
conda config --set show_channel_urls yes
```
## 3.升级 conda
```
conda update conda
```
# 环境管理
## 1. 查看 Python 版本
```
python --version
```
## 2. 创建环境
xxxx为用户名
```
conda create -n xxxx python=3.7.0  

conda create -n xxxx jupyter notebook cudnn
```
新的开发环境会被默认安装在 conda 目录下的 envs 文件目录下。
## 3. 激活环境
```
activate xxxx
```
## 4. 列出所有的环境
```
conda info -e

conda env list
```
## 5. 注销当前环境
```
deactivate
```
## 6. 复制环境
a为要新环境，b为原来的环境
``` 
conda create -n a --clone b
```
## 7.  删除环境
```
conda remove -n b --all
```
# 包管理
## 1. 查看已安装包
```
conda list
```
## 2. 使用 Conda 命令安装包
```
conda install 包名
```
## 3. 通过 pip 命令来安装包
```
pip install 包名
```
4. 移除包
```
conda remove 包名
```

## 安装自用包
1. 先去https://anaconda.org/conda-forge搜索库的安装包
2. 选择的“File”
3. conda install --use-local onnx-1.8.1-py36h6d34f3e_0.tar.bz2