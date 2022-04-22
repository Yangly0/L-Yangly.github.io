# 包管理 Pip

摘要：pip 是一个现代的，通用的 Python 包管理工具。提供了对 Python 包的查找、下载、安装、卸载的功能。
<!--more-->

# 包管理 Pip

## 换源：推荐临时源
1. 国内源
```tex
阿里源：https://mirrors.aliyun.com/pypi/simple/
中科大源：https://pypi.mirrors.ustc.edu.cn/simple/
豆瓣源：https://pypi.douban.com/simple/
清华源：https://pypi.tuna.tsinghua.edu.cn/simple/
```

2. 临时源
```bash
$ pip install XXXX -i https://pypi.tuna.tsinghua.edu.cn/simple
$ pip install XXXX -i http://mirrors.aliyun.com/pypi/simple/  --trusted-host mirrors.aliyun.com
```

3. Windows：修改C:/Users/xx/pip/pip.ini (没有就创建一个)。
```
[global]
timeout = 6000
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn

[global]
timeout = 6000
index-url = https://mirrors.aliyun.com/pypi/simple/
trusted-host=mirrors.aliyun.com
```

4. Linux：`$ vim ~/.pip/pip.conf`
```bash
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

## 操作：安装、更新、降级、卸载
- 在线安装
```bash
$ pip install packagename
```
- 多包安装
```bash
$ pip install -r requests.txt
```
- 本地安装
```bash
# 资源链接
# https://pypi.org/
# https://www.lfd.uci.edu/~gohlke/pythonlibs/#lxml
# https://download.pytorch.org/whl/torch_stable.html
$ pip install ～/Downloads/a.whl
```
- 更新
```bash
$ pip install --upgrade packagename
```

- 降级
```bash
$ pip install packagename=3.5 
```

- 卸载
```bash
$ pip uninstall packagename
```

## 批量操作：导出、安装、卸载
- 批量导出
```bash
$ pip freeze > requirements.txt
```
- 批量安装
```bash
$ pip install -r requirements.txt
```
- 批量卸载
```bash
$ pip uninstall -r requirements.txt
```
## 环境清除：删除所有安装包
```bash
# 删除所有库，如果遇到无法删除的库，需要手动删除requirements.txt重名字
$ pip freeze > requirements.txt
$ pip uninstall -r requirements.txt -y

$ pip freeze | grep -v "^-e" | xargs pip uninstall -y
```
## 实例：pytorch-gpu
```bash
# 前提：已安装cuda和cudnn
$ pip install http://download.pytorch.org/whl/cu80/torch-0.4.1-cp36-cp36m-linux_x86_64.whl -i http://mirrors.aliyun.com/pypi/simple/  --trusted-host mirrors.aliyun.com
```
## 查询：库安装时间
```python
# Prints when python packages were installed
from __future__ import print_function
from datetime import datetime
import os
from pip._internal.utils.misc import get_installed_distributions

if __name__ == "__main__":
    packages = []
    for package in get_installed_distributions():
        package_name_version = str(package)
        try:
            module_dir = next(package._get_metadata('top_level.txt'))
            package_location = os.path.join(package.location, module_dir)
            os.stat(package_location)
        except (StopIteration, OSError):
            try:
                package_location = os.path.join(package.location, package.key)
                os.stat(package_location)
            except:
                package_location = package.location
        modification_time = os.path.getctime(package_location)
        modification_time = datetime.fromtimestamp(modification_time)
        packages.append([modification_time, package_name_version])
    for modification_time, package_name_version in sorted(packages):
        print("{0} - {1}".format(modification_time, package_name_version))

```

## 环境：打包
1. `__init__.py`：包的引导初始化文件，内容可以为空。
2. `setup.py`：打包脚本，name的值（打包的包名）和文件夹的名称要一致。
```python
from setuptools import setup


setup(
    author='UNKNOWN',
    author_email="UNKNOWN\nblas_opt_info:\nblas_mkl_info:\n  FOUND:\n    libraries = ['mkl_intel_lp64', 'mkl_intel_thread', 'mkl_core', 'iomp5', 'pthread']\n    library_dirs = ['/Users/filo/anaconda/lib']\n    define_macros = [('SCIPY_MKL_H', None)]\n    include_dirs = ['/Users/filo/anaconda/include']\n\n  FOUND:\n    libraries = ['mkl_intel_lp64', 'mkl_intel_thread', 'mkl_core', 'iomp5', 'pthread']\n    library_dirs = ['/Users/filo/anaconda/lib']\n    define_macros = [('SCIPY_MKL_H', None)]\n    include_dirs = ['/Users/filo/anaconda/include']\n\nUNKNOWN",
    description='A set of python modules for machine learning and data mining',
    install_requires=['scikit-learn'],
    long_description='''Use `scikit-learn <https://pypi.python.org/pypi/scikit-learn/>`_ instead.''',
    maintainer='UNKNOWN',
    maintainer_email="UNKNOWN\nblas_opt_info:\nblas_mkl_info:\n  FOUND:\n    libraries = ['mkl_intel_lp64', 'mkl_intel_thread', 'mkl_core', 'iomp5', 'pthread']\n    library_dirs = ['/Users/filo/anaconda/lib']\n    define_macros = [('SCIPY_MKL_H', None)]\n    include_dirs = ['/Users/filo/anaconda/include']\n\n  FOUND:\n    libraries = ['mkl_intel_lp64', 'mkl_intel_thread', 'mkl_core', 'iomp5', 'pthread']\n    library_dirs = ['/Users/filo/anaconda/lib']\n    define_macros = [('SCIPY_MKL_H', None)]\n    include_dirs = ['/Users/filo/anaconda/include']\n\nUNKNOWN",
    name='sklearn',
    platforms=['all'],
    py_modules=['wheel-platform-tag-is-broken-on-empty-wheels-see-issue-141'],
    url='https://pypi.python.org/pypi/scikit-learn/',
    version="0.0",
    zip_safe=False,
)
```

3. 打包。
```bash
#打包为egg文件
python setup.py bdist_egg
#打包为whl文件
python setup.py bdist_wheel
```
