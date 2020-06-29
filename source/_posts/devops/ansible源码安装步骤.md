done部署模块，ansible安装步骤
由于ansible大概每4个月会发布一个新的版本，所有pip、yum安装的版本都是最新版的，而不同版本之间可能对应库函数都变了，所以我们需要通过源码方式安装，安装步骤如下：

除了python环境之外，我们还需要先安装ansible的依赖模块，注意：以下需要安装的模块，并不是每个都是必须的，安装前可以检查下当前环境是否已经存在，通过执行命令:pip list,可以查看已存在的模块。

pppip安装：curl https://bootstrap.pypa.io/2.6/get-pip.py -o get-pip.py

(2)、setuptools模块安装

https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz

### tar xvzf setuptools-7.0.tar.gz

### cd setuptools-7.0

### python setup.py install



(3)、pycrypto模块安装

	https://pypi.python.org/packages/source/p/pycrypto/pycrypto-2.6.1.tar.gz

### tar xvzf pycrypto-2.6.1.tar.gz

### cd pycrypto-2.6.1

### python setup.py install

(4)、PyYAML模块安装

http://pyyaml.org/download/libyaml/yaml-0.1.5.tar.gz

### tar xvzf yaml-0.1.5.tar.gz

### cd yaml-0.1.5

### ./configure --prefix=/usr/local

make --jobs= ` grep processor /proc/cpuinfo | wc -l`

### make install



https://pypi.python.org/packages/source/P/PyYAML/PyYAML-3.11.tar.gz

### tar xvzf PyYAML-3.11.tar.gz

### cd PyYAML-3.11

### python setup.py install



(5)、Jinja2模块安装

https://pypi.python.org/packages/source/M/MarkupSafe/MarkupSafe-0.9.3.tar.gz

### tar xvzf MarkupSafe-0.9.3.tar.gz

### cd MarkupSafe-0.9.3

### python setup.py install



https://pypi.python.org/packages/source/J/Jinja2/Jinja2-2.7.3.tar.gz

### tar xvzf Jinja2-2.7.3.tar.gz 

### cd Jinja2-2.7.3

### python setup.py install



(6)、paramiko模块安装

https://pypi.python.org/packages/source/e/ecdsa/ecdsa-0.11.tar.gz

### tar xvzf ecdsa-0.11.tar.gz

### cd ecdsa-0.11

### python setup.py install



https://pypi.python.org/packages/source/p/paramiko/paramiko-1.15.1.tar.gz

### tar xvzf paramiko-1.15.1.tar.gz

### cd paramiko-1.15.1

### python setup.py install



(7)、simplejson模块安装

https://pypi.python.org/packages/source/s/simplejson/simplejson-3.6.5.tar.gz

### tar xvzf simplejson-3.6.5.tar.gz

### cd simplejson-3.6.5

### python setup.py install



(8)、ansible安装

https://releases.ansible.com/ansible/ansible-1.7.2.tar.gz

### tar xvzf ansible-1.7.2.tar.gz

### cd ansible-1.7.2

### python setup.py install

