# 1.python配置

base环境是Anaconda
在anaconda中有python3 

# 激活pychrm中的虚拟环境
```python
#source ~/venv/虚拟环境名称/bin/activate
source ~/venv/DjangoP/bin/activate
#退出
deactivate
```

# 虚拟环境的划分
科学计算的采用conda 目录：/Opt/anaconda3/envs/
web的采用envs     目录：~/venv/
# conda的虚拟环境
创建环境的命令： 
conda create -n name python=X.X（2.7、3.6等）
conda create --name python3 python=3.6.5
目录：  /Opt/anaconda3/envs/
列表：  conda info -e
激活：  conda activate python3  
安装包： conda install -n your_env_name [package]
关闭：  conda deactivate 
删除：  conda remove -n name --all
删除包： conda remove --name 环境名 包名



# 安装virtualenvwrapper
```bash
sudo pip install virtualenv
sudo pip install virtualenvwrapper

vim ~/.bashrc 不行的话加就添加到vim ~/.zshrc
#这是虚拟环境安装的路径
export WORKON_HOME=$HOME/venv
source /Opt/anaconda3/bin/virtualenvwrapper.sh
VIRTUALENVWRAPPER_PYTHON=/Opt/anaconda3/bin/python
PATH=$PATH:/Opt/anaconda3/bin

export PATH
```

## 使用：
mkvirtualenv -p python wechat (wechat是名称)
workon  列表
workon 名称
deactivate 
rmvirtualenv 名称  删除

【python 参考虚拟变量创建】


# conda与virtualenv区别
conda的好处是不仅能创建虚拟环境，还能创建不同版本（ 2，3 ）的 python 虚拟环境。
还有minianaconda 这个版本，不用科学计算，安装这个就可以，小很多，好像几十兆吧


# X手动创建虚拟环境X (废弃)
```python
python -m venv testpy3
source testpy3/bin/activate
deactivate
#生成在命令执行当前目录
```