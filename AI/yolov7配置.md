Anaconda+Pycharm+CUDA+CUdnn+PyTorch+Tensorflow

# 安装Anaconda  

# 查看显卡信息

显卡信息是命令`nvidia-smi` windows默认在路径`C:\Program Files\NVIDIA Corporation\NVSMI`内，在其中执行`./nvidia-smi.exe`即可 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/显卡cudaVersion示例.png)

表格的右上角有一个 CUDA Version就是最高支持CUDA版本，只能安装低于version的版本





```
pip install torch==1.7.1+cu101 torchvision==0.8.2+cu101 torchaudio==0.7.2 -f https://download.pytorch.org/whl/torch_stable.html

```


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/pytorch安装版本.png)  








# 参考
- [CSND](https://blog.csdn.net/weixin_64524066/article/details/126840322)