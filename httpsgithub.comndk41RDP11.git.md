---
title: 《树莓派不吃灰》第二十一期：部署开源远程桌面服务rustdesk,内网丝滑,外网流畅控制Windows,macOS,Linux主机 / 《Use Pi》Issue 21 Deploy the open-source remote desktop service rustdesk, smoothly control Windows, macOS, Linux hosts in the intranet and fluently in the extranet
categories:
- 树莓派不吃灰 / Use Pi
---

前段时间, 有一台老式MacBook Pro被我改造成了影视资源解码主机, [《树莓派不吃灰》第十七期：树莓派配合性能更好的闲置笔记本搭建开源免费jellyfin私人影院](https://v2fy.com/p/2023-06-10-jellyfin-1686388142000/) , 我想通过远程桌面访问这台MacBook Pro, 发现Mac虽然原生支持VNC连接，但实际用起来经常画面撕裂，于是我找了一款开源的远程桌面程序rustdesk, 将其服务部署在树莓派上，实现局域网设备丝滑访问, 外网设备也可以通过内网穿透直接访问macOS

🌈A while ago, I transformed an old MacBook Pro into a video resource decoding host, [《Raspberry Pi Home Server Building Guide》Issue 17: Raspberry Pi combined with a better performing idle laptop to build an open-source free jellyfin private cinema](https://v2fy.com/p/2023-06-10-jellyfin-1686388142000/). I wanted to access this MacBook Pro via a remote desktop. Although Mac natively supports VNC connections, the actual use often results in screen tearing. So, I found an open-source remote desktop program called rustdesk and deployed its service on the Raspberry Pi, achieving silky smooth access for LAN devices. Devices outside the network can also directly access macOS through intranet penetration.

rustdesk的Github开源地址 https://github.com/rustdesk/rustdesk

🌈The open-source Github address for rustdesk: https://github.com/rustdesk/rustdesk

## rustdesk的优势 / Advantages of rustdesk

- 开源, 支持私有化部署

- 🌈Open source, supports private deployment

- 不限制连接数量

- 🌈No limit on the number of connections

- 支持Windows, macOS, Linux, 一套方案搞定远程控制

- 🌈Supports Windows, macOS, Linux, one solution for remote control

- 通过内网映射的方案, 让你随时随地远程控制内网设备

- 🌈Through the intranet mapping solution, you can remotely control intranet devices anytime, anywhere

- 内网访问丝滑流畅, 自动切换内外网流量

- 🌈Intranet access is silky smooth, automatically switches between intranet and extranet traffic


## 在树莓派部署rustdesk，实现局域网Windows控制macOS / Deploy rustdesk on Raspberry Pi to achieve LAN Windows control macOS

```
# 创建挂载目录 / Creating Mount Directory
mkdir -p /opt/rustdesk
chmod 755 -R /opt/rustdesk
# 创建用于存放docker-compose.yml的目录 / Creating Directory for docker-compose.yml
mkdir -p /opt/rustdesk-docker-compose-yml
chmod 755 -R /opt/rustdesk-docker-compose-yml
# 创建docker-compose.yml / Creating docker-compose.yml
touch /opt/rustdesk-docker-compose-yml/docker-compose.yml 
```


在`docker-compose.yml`中写入配置内容

🌈Write the configuration content in `docker-compose.yml`

```git
cat << 'EOF' > /opt/rustdesk-docker-compose-yml/docker-compose.yml
version: '3'

networks:
  rustdesk-net:
    external: false

services:
  hbbs:
    container_name: hbbs
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21118:21118
    image: rustdesk/rustdesk-server:latest
    command: hbbs -r rustdesk.example.com:21117
    volumes:
      - /opt/rustdesk:/root
    networks:
      - rustdesk-net
    depends_on:
      - hbbr
    restart: unless-stopped

  hbbr:
    container_name: hbbr
    ports:
      - 21117:21117
      - 21119:21119
    image: rustdesk/rustdesk-server:latest
    command: hbbr
    volumes:
      - /opt/rustdesk:/root
    networks:
      - rustdesk-net
    restart: unless-stopped
EOF
```

- 启动服务 
- 🌈Start the service

```
cd /opt/rustdesk-docker-compose-yml/
sudo docker-compose up -d
```

![image-20230912100942170](https://cdn.fangyuanxiaozhan.com/assets/1694484583353NyeQczMW.png)

我的树莓派局域网IP为`192.168.50.10`, 将`192.168.50.10` 填入rustdesk 的客户端即可（发起端和被控端都需要下载安装rustdesk客户端，并填写好ID Server）, rustdesk客户端下载地址 https://github.com/rustdesk/rustdesk/releases

🌈My Raspberry Pi LAN IP is `192.168.50.10`. Fill in `192.168.50.10` in the rustdesk client (both the initiating end and the controlled end need to download and install the rustdesk client, and fill in the ID Server). The download address of the rustdesk client is https://github.com/rustdesk/rustdesk/releases

![image-20230912101426079](https://cdn.fangyuanxiaozhan.com/assets/1694484866814RTGS03DK.png)

客户端保存后，会显示配置成功

🌈After the client is saved, it will show that the configuration is successful

![image-20230912101459339](https://cdn.fangyuanxiaozhan.com/assets/1694484900078DRZ176RF.png)

配置完成后，我们就可以通过树莓派运行的rustdesk服务，进行局域网内设备相互远程控制了

🌈After the configuration is complete, we can use the rustdesk service running on the Raspberry Pi to remotely control devices within the LAN

![image-20230912102038004](https://cdn.fangyuanxiaozhan.com/assets/1694485238684pnTW0zzc.png)

被控制主机的ID不会变（除非用户主动改），但密码经常会随机变化，如果你想使用固定密码，可以直接按照下图进行设置

🌈The ID of the controlled host will not change (unless the user actively changes it), but the password will often change randomly. If you want to use a fixed password, you can set it directly according to the following figure

![image-20230912102601840](https://cdn.fangyuanxiaozhan.com/assets/1694485563086fZ6EYi1J.png)

![image-20230912102727070](https://cdn.fangyuanxiaozhan.com/assets/1694485647831XMhiM5Bh.png)

显示方面可以自定义设置，局域网的延迟可以到5ms, 用Windows控制macOS非常顺滑跟手

🌈The display can be customized. The delay of the LAN can be as low as 5ms, and it is very smooth to control macOS with Windows

![image-20230912103259426](https://cdn.fangyuanxiaozhan.com/assets/1694485980398sQShxFwa.png)

遇到的一个小坑：如果MacBook不外接显示器合上盖子，rustdesk经常会几十秒就自动断开，如果你也是想控制MacBook，请不要合上盖子。

🌈A small pitfall I encountered: If the MacBook does not connect to an external display and closes the lid, rustdesk will often disconnect automatically in a few seconds. If you also want to control the MacBook, please do not close the lid.


## 让家里树莓派的rustdesk服务支持外网访问 / Make the Rustdesk Service on the Raspberry Pi at Home Support External Network Access

外网访问其实也很简单，用frp做几个端口映射就好了

🌈Accessing from the external network is actually very simple, just use frp to do a few port mappings.

![image-20230912111138990](https://cdn.fangyuanxiaozhan.com/assets/1694488300187hfGn57Qt.png)

- 在树莓派的frpc.ini文件中新增以下配置，重启frpc即可

- 🌈Add the following configuration in the frpc.ini file of the Raspberry Pi, and restart frpc.

```
如果想让**内网主机A**可以接收外网连接，我们需要将**内网主机A**的ID server设置为云服务器IP（云服务器的IP为公网IP, 会接收请求，并转发到树莓派）

🌈If you want **Host A in the intranet** to be able to receive connections from the extranet, we need to set the ID server of **Host A in the intranet** to the IP of the cloud server (the IP of the cloud server is a public IP, it will receive requests and forward them to the Raspberry Pi)

当然，外网发起控制的主机，也要设置外网IP, 才能发起控制。

🌈Of course, the host that initiates control from the external network also needs to set the external network IP in order to initiate control.

![image-20230912135153485](https://cdn.fangyuanxiaozhan.com/assets/1694497914409R6QyH826.png)

如果**发起控制端**在家庭内网环境下，可以将**ID server** 设置为树莓派内网IP，也可以设置为**云服务器公网IP**

🌈If the **control initiating end** is in a home intranet environment, the **ID server** can be set to the intranet IP of the Raspberry Pi, or it can be set to the **public IP of the cloud server**

## 直接将rustdesk服务放到服务器运行不就好了，为何要放在树莓派运行？ / Why Run Rustdesk Service on Raspberry Pi Instead of Directly on the Server?

rustdesk有两个服务，hbbs负责验证签名，hbbr负责转发远程控制产生的数据包。

🌈Rustdesk has two services, hbbs is responsible for verifying signatures, and hbbr is responsible for forwarding data packets generated by remote control.

![170487506-8ef1f025-ad42-47f9-8d82-b49d0ec759ad](https://cdn.fangyuanxiaozhan.com/assets/1694498376154G7B1thhS.png)

我们在设置界面，只填写了hbbs信息（ID server） ，没有填写hbbr信息，hbbr就会自动判断是否走公网流量。

🌈In the setting interface, we only filled in the hbbs information (ID server) and did not fill in the hbbr information. Hbbr will automatically determine whether to use public network traffic.

如果我们的**被控主机**和**发起控制的主机**都在内网，二者就会直接走内网流量，体验将无比丝滑。

🌈If our **controlled host** and the **host initiating control** are both in the intranet, the two will directly use the intranet traffic, and the experience will be incredibly smooth.

当然控制端或被控端，一旦离开了内网，就自动走云服务器转发流量。

🌈Of course, if the control end or the controlled end leaves the intranet, it will automatically use the cloud server to forward traffic.

![image-20230912141949102](https://cdn.fangyuanxiaozhan.com/assets/16944995908258GRFpifA.png)

## 那ID server到底应该怎么填？ / So How Should the ID Server be Filled?

只要在外网，控制端和被控端，一律填写外网IP！ 

🌈As long as it's on the external network, both control end and controlled end, fill in the external IP!

如果在内网，控制端和被控端，依然可以全部填写外网IP；（如果被控端和控制端设备同时在内网，会自动走内网流量，无比顺滑）

🌈If it's in the intranet, the control end and the controlled end can still fill in the external IP; (If the controlled end and the control end device are in the intranet at the same time, it will automatically use intranet traffic, which is incredibly smooth)

如果某台设备，只希望被内网的设备连接，则填写内网IP!

🌈If a device only wants to be connected by devices in the intranet, then fill in the intranet IP!

(内网IP是指树莓派IP, 外网IP是指云服务器IP)

🌈(Intranet IP refers to the Raspberry Pi IP, and the external network IP refers to the cloud server IP)

## 小结 / Summary

我看到有些youtuber把mac mini改成了家庭服务器，由于macOS对VNC协议的负优化，导致画面撕裂，于是只能捏着鼻子用ssh连接服务器，但我感觉放弃macOS顺滑的动画太可惜了, 于是就有了树莓派运行开源rustdesk的方案。

🌈I've seen some YouTubers turn the Mac Mini into a home server. Due to macOS's poor optimization of the VNC protocol, the screen tears, so they can only connect to the server via SSH reluctantly. But I feel it's a pity to give up the smooth animation of macOS, so there's the solution of running the open-source Rustdesk on the Raspberry Pi.

虽然有Teamviewer, ToDesk, 向日葵等方案，但都无法实现内网丝滑流畅的远程桌面体验。

🌈Although there are solutions like Teamviewer, ToDesk, Sunflower, etc., none of them can achieve a smooth and fluent remote desktop experience in the intranet.

这个树莓派rustdesk方案保证了远程桌面内网丝滑，外网流畅，即使没有树莓派，也可以通过虚拟机运行rustdesk服务器实现，让老旧的MacBook焕发流光溢彩。

🌈This Raspberry Pi Rustdesk solution ensures that the remote desktop is smooth in the intranet and fluent in the external network. Even without a Raspberry Pi, it can be implemented by running the Rustdesk server on a virtual machine, letting the old MacBook shine brightly.