title: windows/Office 激活及KMS服务器搭建
date: 2019-11-28 11:00:48
categories: 环境部署
tags: 
	- kms
---

#####  Windows/Office GVLK密钥

<!-- more -->

```
## 操作系统版本   KMS 客户端安装密钥
## 其他版本请在 https://docs.microsoft.com/zh-cn/windows-server/get-started/kmsclientkeys 自行查找
Windows 7 专业版   FJ82H-XT6CR-J8D7P-XQJJ2-GPDD4
Windows 8 专业版   NG4HW-VH26C-733KW-K6F98-J8CK4
Windows 8.1 专业版 GCRJD-8NW9H-F2CDX-CCM8D-9D6T9
Windows 10 专业版  W269N-WFGWX-YVC9B-4J6C9-T83GX

## OFFICE 版本   KMS 客户端安装密钥
## 其他版本请在 https://docs.microsoft.com/en-us/DeployOffice/vlactivation/gvlks 自行查找
Office Professional Plus 2019      NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP 
Office Standard 2019               6NWWJ-YQWMR-QKGCB-6TMB3-9D9HK 
Project Professional 2019          B4NPR-3FKK7-T2MBV-FRQ4W-PKD2B
Project Standard 2019              C4F7P-NCP8C-6CQPT-MQHV9-JXD2M
Visio Professional 2019            9BGNQ-K37YR-RQHF2-38RQ3-7VCBB 
Visio Standard 2019                7TQNQ-K3YQQ-3PFH7-CCPPM-X4VQ2

Office Professional Plus 2016      XQNVK-8JYDB-WJ9W3-YJ8YR-WFG99 
Office  Standard 2016              JNRGM-WHDWX-FJJG3-K47QV-DRTFM 
Project  Professional 2016         YG9NW-3K39V-2T3HJ-93F3Q-G83KT 
Project  Standard 2016             GNFHQ-F6YQM-KQDGJ-327XX-KQBVC 
Visio  Professional 2016           PD3PC-RHNGV-FXJ29-8JK7D-RJRJK 
Visio  Standard 2016               7WHWN-4T7MP-G96JF-G33KR-W8GF4 
```
##### KMS 使用。使用管理员权限打开 cmd 命令行。

```
## 查看系统激活状态
slmgr.vbs -xpr
## 查看系统激活状态详细信息
slmgr.vbs -dlv
## 卸载密钥
slmgr.vbs /upk
## 查看系统版本
wmic os get caption
## 安装密钥，需要使用对应系统版本的激活密钥（GVLK）
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
## 设置激活 KMS 服务器
slmgr /skms ip
## 激活系统
slmgr /ato
```
##### 使用 KMS 激活 OFFICE

```
## 切换至 office 安装目录（依据个人安装路径）
cd "C:\Program Files\Microsoft Office\Office16"
## 安装 Office GVLK 激活密钥
cscript ospp.vbs /inpkey: XQNVK-8JYDB-WJ9W3-YJ8YR-WFG99
## 执行注册 KMS 服务器
cscript ospp.vbs /sethst:ip
## 激活 OFFICE
cscript ospp.vbs /act
```

##### KMS 服务器安装方法

KMS 服务器安装的前提是需要自己有一台虚拟机或者服务器，参考 Github 上开源的一键部署方案安装,省时省力：

```
## 下载一键安装及管理程序
git clone https://github.com/dakkidaze/one-key-kms.git
cd one-key-kms
## CentOS / Redhat / Fedora
sudo sh one-key-kms-centos.sh
## Debian / Ubuntu / Mint
sudo sh one-key-kms-debian.sh
## 管理程序 start/stop/restart/status
./kms.sh start
```

##### FAQ

```
iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT
```