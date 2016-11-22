=======================================
搭建Golang的Vim开发环境
=======================================
工欲善其事必先利其器，若想把Golang写得溜，首先得有个顺手得IDE不是，本文教你打造强大得vim-go环境。

首先安装好Go，然后设置gopath，比如可以这样::

    $ apt-get install golang
    # 设置gopath 内容如下
    $ cat /etc/profile.d/golang.sh
    export GOPATH=/mnt/workspace/mygo
    export PATH=$PATH:$GOPATH/bin
    $ source /etc/profile

安装Vim插件并设置相关配置，Vim的版本至少7.3以上::

    $ git clone https://github.com/monnand/vimrc
    $ cd vimrc
    $ ./deploy.sh
    $ sudo apt-get install build-essential cmake python-dev libclang-dev
    $ cd ~/.vim/bundle/YouCompleteMe
    $ ./install.sh --gocode-completer

安装过程需要到github上下载插件比较慢，请耐心等待。vim-go依赖很多包，这些包需要到glang.org上下载，很遗憾被墙了，你进入vim里执行GoInstallBinaries可以看到。

通常GoInstallBinaries是优先到golang官网上找包，但是官网在大陆是无法访问的，所以你可以手动安装依赖::

    # 主要安装的依赖包是这些
    $ go get github.com/bradfitz/goimports
    $ go get github.com/rogpeppe/godef
    $ go get github.com/nsf/gocode
    $ go get github.com/jstemmer/gotags
    $ go get github.com/golang/lint/golint

如果仍然不能正常安装，且没有科学上网大法，可以使用 http://gopm.io/  把对应的包下载到go的环境中，也可以使用gopm的命令行工具，我们以安装goimports为例::

    # 将goimports安装到gopath中
    $ gopm get -g -d golang.org/x/tools/cmd/goimports
    # 使用go install 来安装
    $ go install golang.org/x/tools/cmd/goimports
     
参考资料:

- http://tonybai.com/2014/11/07/golang-development-environment-for-vim/ 


    











