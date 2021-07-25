---    
title: 'Docker 部署思源实现多人协作'    
date: 2021-07-25T15:42:43+08:00    
slug: "docker-siyuan"    
type: post    
tags:    
  - siyuan-note  
  - docker  
categories:    
  - Other    
---    

## 前言  

最近准备将协作平台从 HackMD 迁徙到思源笔记。  
这主要是因为我个人笔记使用思源，而团队笔记使用 HackMD，不免有些混乱。  
再加上思源的**块信息**实在是很好的平台，基于丰富的信息有很多可操作性，后续也有一定的相关的计划，所以现在打算开始迁徙。  

---  

首先买了一台服务器、、、1v 512MB，英国，凑合用用。  

安装了 Debian10.3，准备开始操作。  

## （可选操作）准备工作  

一下都是可选的步骤，主要是为了安全性等考虑。  

先更新一下系统：`apt update ` 然后 `apt upgrade`，更新完重启一下，如果出问题了，则重装系统、、、  

设置一下时区：`timedatectl set-timezone Asia/Shanghai`  

更改 SSH 端口：`vim /etc/ssh/sshd_config`，修改 `Port 22`，然后 `systemctl restart sshd`  
断开连接，使用新的端口重新连接。  

然后最好取消掉密码登陆。  
在本机配置 SSH 密钥，然后上传到服务器。可以参考[这篇文章](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)。  

最后配置一下防火墙，打开 SSH 端口和思源端口 `6806` 即可，可能你的服务商会提供安全组策略。  

## 部署思源 Docker  

先参考[这篇文章](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-debian-10)安装 Docker  

```bash  
apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common   
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -  
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"  
apt update  
apt install docker-ce  
```  

然后 Docker 装好了，开始部署思源笔记。  

```bash  
docker pull b3log/siyuan  
```  

然后考虑一下参数，我看的帮助文档里的参数有些过时了，可以通过 `-h` 参数输出帮助。  
需要注意的地方：指定 `workspace` 的位置方便后续处理，设置授权码 `password`，这里替换为你想设置的密码。  

```bash  
docker run -v /siyuanworkspace:/siyuanworkspace -p 6806:6806 b3log/siyuan -resident -workspace /siyuanworkspace -authCode password  
```  

理论上换个端口会安全一点、、、  

浏览器访问 `http://ip:6806:/`，提示需要输入账号密码，账号是 `siyuan`，密码是你刚刚设置的授权码。  

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20210725135826-kb3nlg8.png)  

大功告成。  

测试完毕后，可以这样运行：  

```bash  
docker run -d -v /siyuanworkspace:/siyuanworkspace -p 6806:6806 b3log/siyuan -resident -workspace /siyuanworkspace -authCode password  
```  

这样在 `detached mode` 中运行，就可以后台运行了。  

> 试了一下，虽然用的是国外服务器，但没有什么延迟的感觉。  
>  

## （可选操作）绑定域名  

绑定个域名好记一点、、、实际上，添加一条 `A` 记录就可以了，没有什么额外的工作。  

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20210725140426-dppxd17.png)  

## （可选操作）使用 MegaSync 同步  

我的最初目的是多人协作编辑一个笔记本，我还希望这个笔记本能被同步到本地，这样我可以在别的笔记本中引用它。  

首先需要注意，在笔记本根目录下创建一个 `assets` 文件夹，这样资源文件就会单独保存到这个文件夹，而与 Workspace 隔离开。  

我现在使用的同步盘是 Mega，因此我考虑用 Mega 同步对应的文件夹到本地。可以使用 [Mega CMD](https://mega.nz/cmd) 进行操作。  

下载好之后，通过 `scp -P port localpath root@ip:remotepath` 进行上传。根据官网教程进行安装。  

这个 MegaCMD 是个全功能客户端，我们只需要 Sync 就好了，无需过多了解。事实上有一个非常详细的 [UserGuide](https://github.com/meganz/MEGAcmd/blob/master/UserGuide.md).  

```bash  
mega-login email password  
```  

登陆完成之后，准备开始同步。由于我已经在本地创建好了对应的笔记本并同步到 Mega，因此路径上需要稍作调整。  
**请在操作前备份。**  

MEGA 路径：`/MEGAsync/SiYuan/data/Gaokao/`  
服务器路径：`/siyuanworkspace/data/Gaokao`  

需要注意，在同步前要创建对应的文件夹。例如 `mkdir /siyuanworkspace/data/Gaokao`  

然后开始同步。  

```bash  
mega-sync /siyuanworkspace/data/Gaokao /MEGAsync/SiYuan/data/Gaokao  
```  

稍微等待一会，进去 `ls` 就可以看到文件全部同步过去了。  

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20210725142510-namxjjz.png)  

顺利同步、、、  

> 大概尝试了一下，新建文档之类的是可以顺利同步过去的，但是有一定的延迟。  
>  
> 但是如果同时修改某个文档，是无法同步的。  
>  
> **这不是可靠性很高的同步方案，请谨慎使用。**  
>  

## （可选操作）打包一个第三方客户端  

曾经我写过一篇[在 Linux 下使用 Nativefier 打包思源笔记](https://ld246.com/article/1622898305015/comment/1623135737884#comments)的文章，其实这个操作在任何平台都适用。  
对于任意的网页，我们都可以打包成一个 Electron 应用。  

因此我们可以打包一个 Windows 程序分享给他人使用。（虽然我是 Linux 用户、、、）  
这样可以做到**类似客户端**的体验，比如托盘、常驻后台、单例模式之类的，详细请访问原文章吧。  

这里有一个注意的点：必须使用 `http://address:6806/` 才能正常授权。  

在 Nativefier 中传入账号密码进行授权。  

```bash  
nativefier --name "RemoteSiyuan"  -i /home/clouder/Pictures/Icons/Siyuan.ico --single-instance --tray -e 13.1.0 http://address:6806 --basic-auth-username siyuan --basic-auth-password password  
```  

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20210725150827-ahb1fas.png)  

Linux 下测试正常，转到 Windows 虚拟机测试。  

![image.png](https://b3logfile.com/siyuan/1609132319768/assets/image-20210725151100-fkbszzp.png)  

看起来很正常。在 Windows 下同样损失了思源自带的标题栏，因为这是网页打包。  
~~往好处想，这也让 UI 变得更协调美观了……~~  

## 结语  

实际上用 Docker 部署思源笔记的 Kernel 是非常简单的。    
其实直接运行 Kernel 的二进制文件是一种更简单的方法，但还是建议大家按官方文档来吧。  

目前尚未投入使用，日后可能会有附加反馈。  
