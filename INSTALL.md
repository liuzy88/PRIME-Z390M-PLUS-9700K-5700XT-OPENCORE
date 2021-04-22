# OPENCORE 0.6.3

## BIOS

> BIOS版本 PRIME-Z390M-PLUS-ASUS-2808.CAP

- 高级
    - CPU设置
        - SGX -> 禁用
        - VMX -> 启用
    - 北桥
        - VT-d -> 禁用 (装完后启用)
        - 大于4G的内存解码 -> 启用
        - 显示设置
            - 首选显卡 -> 自动
            - 初始化IGPU -> 启用
            - DVMT -> 128M
    - 内置设备
        - 串口 -> S.Port.C -> 禁用
    - USB设置
        - XHCI Hand-off -> 启用
- 启动
    - 快速启动     -> 禁用
    - 自检延迟     -> 1秒
    - 安全启动菜单 -> 系统类型  -> 其他
    - 密钥管理     -> 清除密钥
- 工具
    - Q-Installer -> 禁用

## OC EFI

> OPENCORE版本 0.6.5

```
├── BOOT                             
│  └── BOOTx64.efi                  (必须)
├── OC                               
│  ├── ACPI                         
│  │  ├── SSDT-AWAC.aml            (必须)修复RTC禁止AWA
│  │  ├── SSDT-PLUG.aml            (必须)加载CPU原生电源管理 开启节能四项
│  │  ├── SSDT-PMC.aml             开启NVRAM 原生支持的不需要
│  │  └── SSDT-EC-USBX.aml         修复USB(由hackintool生成)
│  ├── Drivers                      
│  │  ├── HFSPlus.efi              (必须)
│  │  ├── OpenRuntime.efi          (必须)
│  │  └── OpenCanopy.efi           用于加载Resources
│  ├── Kexts                        
│  │  ├── AppleALC.kext            (必须)
│  │  ├── IntelMausiEthernet.kext  有线网卡
│  │  ├── Lilu.kext                (必须)
│  │  ├── NVMeFix.kext             NVMe增强
│  │  ├── RealtekRTL8111.kext      其他USB网卡
│  │  ├── SMCProcessor.kext        SMC
│  │  ├── SMCSuperIO.kext          SMC
│  │  ├── USBPorts.kext            修复USB(由hackintool生成)
│  │  ├── VirtualSMC.kext          SMC
│  │  └── WhateverGreen.kext       (必须)
│  ├── OpenCore.efi                 
│  ├── Resources                    
│  │  ├── Font                     
│  │  ├── Image                    
│  │  └── Label                    
│  └── config.plist                 (必须)
└── INSTALL.md                        
```

## CONFIG

> 以快为主

- ACPI
  - `SSDT-EC-USBX-DESKTOP.aml`和`USBPorts.kext`启用其中之一
- Booter
  - 取消日志
- DeviceProperties
  - 核显PciRoot(0x0)/Pci(0x2,0x0)
  - 独显PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)
  - 声卡PciRoot(0x0)/Pci(0x1F,0x3)
- Kernel
  - 有了`USBPorts.kext`就不需要`XhciPortLimit`
- Misc
  - Builtin
  - 扫描策略2621699不要显示Windows
- NVRAM
  - 注入三码加速引导
- PlatformInfo
  - 自己生成三码
- UEFI
  - 仅3个

## STEP

```
添加核显设备参数
    PciRoot(0x0)/Pci(0x2,0x0)
添加独显固件参数
    PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)

修改启动参数，注入声卡ID
    debug=0x100 keepsyms=1 alcid=1 agdpmod=pikera shikigva=80
    确定没问题后，声卡ID用DeviceProperties注入效率更高

添加自己的PlatformInfo信息
    MLB ROM SystemSerialNumber SystemUUID

修改SystemUUID同步Windows，使用命令
    ioreg -d2 -c IOPlatformExpertDevice | awk -F\" '/IOPlatformUUID/{print $(NF-1)}'
    888153AA-9716-5B0D-8A5D-F0BDDBC820E3
```

## USBX

```
https://heipg.cn/tutorial/custom-usbports-for-hackintosh.html

HS = HighSpeed = USB2，SS = SuperSpeed = USB3

1.下载最新USBInjectAll.kext https://bitbucket.org/RehabMan/os-x-usb-inject-all/downloads/
2.在config.plist中启用USBInjectAll.kext
3.在config.plist中修改XhciPortLimit=是,开启所有USB端口
(X系列几十个USB要分别测试2.0和3.0)在config.plist中添加USBInjectAll.kext引导参数：-uia_exclude_ss,不启用USB3.0支持
4.重启,打开hackintool,切换到USB,开始插拔,记录信息
后置1: 左:HS02      右:HS01
后置2: 左:HS04-SS04 右:HS03-SS03
后置3: 左:HS06-SS06 右:HS05-SS05
前置1: 左:HS09 SS09
前置2: 左:HS11
前置3: 左:HS12
内置    HS14      蓝牙
5.导出,桌面上几个文件
SSDT-EC-USBX.aml
SSDT-EC-USBX.dsl
SSDT-UIAC.aml
SSDT-UIAC.dsl
USBPorts.kext
6.复制到ACPI文件夹，在ACPI中添加SSDT-EC-USBX和SSDT-UIAC
7.复制到Kexts文件夹，禁用USBInjectAll,XhciPortLimit=否,启用USBPorts.kext
8.重启完成
```


# Big Sur中Parallels Desktop联网问题已解决,可以升级啦!
```bash
cd /Library/Preferences/Parallels
sudo vim network.desktop.xml
# 修改: UseKextless => 0
```