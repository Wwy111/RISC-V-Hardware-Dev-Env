# RISC-V硬件开发环境配置（随时更新）

- 开发平台：ubuntu20.04（虚拟机、PC、服务器都可）其他版本的ubuntu与下面的工具会有兼容性等问题，后续可能会在工具上花很多不必要的时间。

  [ubuntu20.04镜像下载链接](http://releases.ubuntu.com/20.04/ubuntu-20.04.2.0-desktop-amd64.iso)

  虚拟机或者PC机安装教程网上很多，如果虚拟机建议分配4G内存，50G硬盘以上  



```
git clone --recursive https://github.com/whysodangerous/RISC-V-Hardware-Dev-Env.git
```



## Part1. 准备硬件开发所需的软件测试集及测试集运行时环境

- 把apt源更换成国内源

  1. 备份原source.list

  ``` 
  sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
  ```

  2. 删除原sources.list里的内容，把[这个网站]([ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/))里的东西复制到里面，然后`sudo apt-get update`

- （如果是虚拟机且虚拟机软件为vmware的话需要这一步）安装`vmtools`，这是一个能在主机和虚拟机之间自由复制粘贴文字文件的软件。得先卸载系统自带的vmtools再重新安装

  ```
  sudo apt-get autoremove open-vm-tools
  sudo apt-get install open-vm-tools
  sudo aot-get install open-vm-tools-desktop
  ```

  安装好之后重启系统即可生效

- 下载工具

  ```
  sudo apt-get install build-essential    # build-essential packages, include binary utilities, gcc, make, and so on
  sudo apt-get install man                # on-line reference manual
  sudo apt-get install gdb                # GNU debugger
  sudo apt-get install git                # revision control system
  sudo apt-get install libreadline-dev    # a library to use compile the project 
  sudo apt-get install libsdl2-dev        # a library to use compile the project
  sudo apt-get install libc6-dev-i386     # a library to use compile the project 
  sudo apt-get install qemu-system        # QEMU
  ```

- 配置`git`

  ```
  git config --global user.name "yourname"
  git config --global user.email "youremail"
  git config --global core.editor vim
  git config --global color.ui true
  ```

- 安装并配置vim

  ```
  sudo apt-get install vim
  ```

  把提供的`.vimrc`文件复制到`~ `目录下，可以根据个人增减配置，里面有注释

- 安装并配置tmux

  ```
  cd ~
  sudo apt-get install tmux
  vim .tmux.conf
  ```

  把下面复制到文件中

  ```
  bind-key c new-window -c "#{pane_current_path}"
  bind-key % split-window -h -c "#{pane_current_path}"
  bind-key '"' split-window -c "#{pane_current_path}"
  ```

- 安装riscv64交叉编译器

  ```
  sudo apt-get install g++-riscv64-linux-gnu binutils-riscv64-linux-gnu
  ```

  

## Part2. 测试

- 至此，完成了软件测试集所需的环境搭建，现在需要测试一下

- 从我个人[github仓库](https://github.com/whysodangerous)上下载`nemu/` `abstract-machine/` `am-kernels/`这三个目录

- 打开`~/.bashrc`编辑路径

  ```
  vim ~/.bashrc
  
  # Add the following two lines in the end of the .bashrc
  export AM_HOME=$(YOUR_HOME_PATH)/abstract-machine
  export NEMU_HOME=$(YOUR_HOME_PATH)/nemu
  ```

  保存后 `source ~/.bashrc`生效，此时`echo $NEMU_HOME`或者`echo $AM_HOME`能看到正确路径

- 把`nemu`编译成动态链接库

  ```
  cd ~/nemu
  make menuconfig
  ```

  搜索`SHARE`，选中，修改物理起始地址为`0x80000000`，保存退出

  ```
  make
  ```

  编译成功的话不会有提示。如果有提示缺少某个`.h`文件的话，上网搜一下装对应的库就行了。

- am-kernels里面是一些魔改过的benchmark和一些简单的测试c程序，现在可以来编译一下

  ```
  cd am-kernels/tests/cpu-tests
  make ARCH=riscv64-wwy ALL=add
  ```

  如果能编译成功，则会生成一个build目录，里面有`.elf`可执行文件（当然直接运行肯定跑不了），`.bin`二进制文件，以及`.txt`反汇编文件。上面的命令如果不加ALL选项会编译所有`.c`测试程序。

  `am-kernels/tests/am-tests/`，`am-kernels/benchmarks/`目录下也有一些测试文件，编译方法自行参考对应目录下的`Makefile`

  以上的编译所得到的程序依赖`abstract-machine`所提供的运行时环境

  

## Part3. 硬件调试环境

下面开始硬件调试环境介绍

- verilator安装

  verilator可以将verilog代码集成到c++程序中，实现用c++来仿真硬件电路。

  verilator可以用apt一件安装，但版本较老，推荐使用下面的方法，少踩坑

  把`verilator_install.tar.gz`复制到`~`目录，解压，然后

  ```
  ./install-verilator.sh
  ```

- 安装gtkwave

  gtkwave是一个查看波形的文件，配合verilator能够使硬件调试告别vivado

  ```
  sudo apt-get install gtkwave
  ```



## Part4. 测试

- 从我个人[github仓库](https://github.com/whysodangerous)上下载`oscpu/`这个目录（这个工程是用`verilog`写的`riscv64`经典五级流水线）

  ```
  cd oscpu/cpu
  bash runall.sh
  ```

  `runall.sh`脚本会根据编译`am-kernels/tests/cputests/tests`目录下的所有文件，然后执行verilator仿真程序，依次使用每一个`.c`源程序对`verilog`代码进行`difftest`验证，终端上每一项后面都输出了PASS表明`verilog`代码正确，前面的环境安装正确。

- 如果想查看某一项测试的cpu波形文件

  ```
  make run TEST=add
  gtkwave top.vcd
  ```

  `add`可以换成`am-kernels/tests/cputests/`中的任意一项

  

## Part5. 硬件开发环境

- 硬件开发最熟知的工具就是`verilog+vivado`组合，这里介绍一个用高级语言描述电路的工具：`chisel`

- `chisel`是基于`scala`语言的，而`scala`语言又是基于JVM运行的，所以先安装`java`环境

  ```
  sudo apt-get install openjdk-8-jdk
  ```

  查看`java`版本

  ```
  java -version
  
  openjdk version "1.8.0_292"
  OpenJDK Runtime Environment (build 1.8.0_292-8u292-b10-0ubuntu1~20.04-b10)
  OpenJDK 64-Bit Server VM (build 25.292-b10, mixed mode)
  ```

- 安装`scala`构建工具`sbt`

  ```
  echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
  echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt_old.list
  curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo apt-key add
  sudo apt-get update
  sudo apt-get install sbt
  ```

- 安装`scala`构建工具`mill`。`mill`和`sbt`都是`scala`的构建工具，二者可以在同一个工程下一起使用，互不冲突，个人感觉`mill`的速度更快一点。

  把`mill`文件复制到`/usr/local/bin/`目录下，这个路径可以自己定义，然后打开`~/.bashrc`添加一下全局路径

  ```
  vim ~/.bashrc
  
  # Add the following two lines in the end of the .bashrc
  export MILL=/usr/local/bin/mill
  export PATH=$MILL:$PATH
  ```

  保存后，`source ~/.bashrc`生效，修改`mill`权限

  ```
  sudo chmod 777 /usr/locao/bin/mill
  ```

  

## Part6. 测试

下面需要对上面安装的两个工具进行测试，测试之前，还需要根据个人情况安装一下代码编辑器，编辑器在`linux`上的安装就不赘述了，网上非常多。可以直接在`Ubuntu20`的`snap应用商店`里安装。

- `vscode`，编写`verilog`，`c`，`c++`，`Makefile`文件等

  - 推荐安装插件

  `C/C++`       `Chinese (Simplified) Language`       `Chisel Syntax`      `hexdump for VSCode`     

  `Scala (Metals)`        `Verilog-HDL/SystemVerilog/Bluespec SystemVerilog support for VS Code`    

  `WaveTrace`   

  插件使用都有说明，这里不赘述

- `Intellij idea`，针对`java`类语言有完善的语义补全，`vscode`上编辑`java`类语言语义补全需要繁琐的插件配置

  - 必须安装插件(plugins)

  `scala`

- 从我个人[github仓库](https://github.com/whysodangerous)上下载`riscv64-wwy/`这个目录（这个工程是用`chisel`写的`riscv64`经典五级流水线，同时加上了cache）

  ```
  cd riscv64-wwy
  make distclean
  make sbt           # If you use sbt the first time, this step will download lots of                      packs, may fail due to network environment, try few times
  ```

  上一步成功的话，会看到`[success]`，然后会生成一个`build/`目录，里面就有编译生成的`.v`文件。

- 对`.v`文件进行一下仿真测试

  ```
  bash runall.sh
  ```

  可以看到每一项后面都输出了PASS

- 测试完`sbt`，再来测试一下`mill`

  ```
  make distclean
  make mill          # If you use mill the first time, This step may download lots of                      packs, may fail due to network environment, try few times
  
  ```

  上一步成功的话，会看到`[success]`，然后会生成一个`build/`目录，里面就有编译生成的`.v`文件。

- 对`.v`文件进行一下仿真测试

  ```
  bash runall.sh
  ```

  可以看到每一项后面都输出了PASS

- 如果想查看某一项测试的cpu波形文件

  ```
  make emu-run RUN=CPUTEST TEST=add DIFF=1 VCD=1
  gtkwave add.vcd
  ```

  如果`RUN`参数等于`CPUTEST`，那么`RUN`参数可以省略。当`VCD`参数缺省的时候，不会生成波形文件。当`VCD`参数为1时，无论程序运行结果如何，都会生成一个波形文件。当`DIFF`参数缺省的时候，不会进行`difftest`。

  `add`可以换成`am-kernels/tests/cputests/`中的任意一项。

  还可以在仿真环境下运行`dhrystone`，`coremark`，`microbench`，运行命令如下

  ```
  make emu-run RUN=MICROBENCH DIFF=1
  ```

  强烈建议这里不要设置`VCD`参数，要不然的话波形文件会非常大，会把内存和硬盘跑崩。

  

  

  至此，环境配置完毕                                                                  
  
  ​                                                                                                                                              ——wwy
  
  ​                                                                                                                                              2021.8.22
