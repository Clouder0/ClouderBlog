---    
title: "Linux 使用哈工深(HITSZ)校园 VPN 指北"
date: 2022-08-16T15:00:57+08:00
slug: "hitsz-vpn"  
type: post  
tags:  
  - 
categories:  
  - Tech
---  

经历了一些莫名其妙的问题之后，我终于连上了校园 vpn.

## 安装 EasyConnect

哈工大的 vpn 大概都是基于 EasyConnect 的。可以安装 EasyConnect 的通用版本进行后续操作。

AUR 上的 EasyConnect 有点问题，md5 校验过不了，要自己改 PKGBUILD.

```bash
git clone https://aur.archlinux.org/easyconnect.git
cd easyconnect
makepkg
sudo pacman -U easyconnect-***.pkg.tar.zst 
# 这里多半会 Fail
updpkgsums # 更新 md5
makepkg
sudo pacman -U easyconnect-***.pkg.tar.zst
```

或者直接下载官网的 deb 包自行安装。([http://download.sangfor.com.cn/download/product/sslvpn/pkg/linux_767/EasyConnect_x64_7_6_7_3.deb](http://download.sangfor.com.cn/download/product/sslvpn/pkg/linux_767/EasyConnect_x64_7_6_7_3.deb))

## 连接

由于未知的原因，我这上 vpn.hitsz.edu.cn 啥都加载不出来。(当然也下载不了客户端(可是为什么可以下载 windows 客户端呢))

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220816144035-dqvz8ky.png)​

总之就是设置一下密码，按照[哈尔滨工业大学深圳校区 VPN 使用指引](http://nic.hitsz.edu.cn/info/1018/1352.htm)中所说，用户名、密码初始都是学号。

然后打开 EasyConnect，填写 Server IP. (vpn.hitsz.edu.cn)

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220816144624-0fjo7hm.png)​

然后填写账号密码登陆即可。

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220816144651-p0su688.png)​

图标稳定下来就连接好了。

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220816144823-f5muuh8.png)​

(中间那个)
