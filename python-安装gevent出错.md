# python 安装gevent
## 1. 问题
在python2.7中安装gevent的时候出现 error: Unable to find vcvarsall.bat 的错误，
不论使用pip install 还是下载到本地安装都是这个错误.

## 2. 解决办法：
    这个时候需要安装vs2008了，如果安装了更高的版本，可以这样设置：
    Visual Studio 2010 (VS10): SET VS90COMNTOOLS=%VS100COMNTOOLS%
    Visual Studio 2012 (VS11): SET VS90COMNTOOLS=%VS110COMNTOOLS%
    Visual Studio 2013 (VS12): SET VS90COMNTOOLS=%VS120COMNTOOLS%

这些设置需要在安装python的第三方库之前，命令行输入上面的SET即可，再进行第三方库的安装.这个方式可以解决大部分的上述错误.

具体可查看StackOverFlow:
> http://stackoverflow.com/questions/2817869/error-unable-to-find-vcvarsall-bat