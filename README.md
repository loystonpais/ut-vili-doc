# This is incomplete, DO NOT proceed with the instructions
This is written solely for me to understand, at least for now

## Building Ubuntu Touch for xiaomi 11t pro (vili)

30th May 2024, 3 PM

So I got two kernel source trees,

The first one I found from googling, and seems official is
  https://github.com/MiCode/Xiaomi_Kernel_OpenSource/tree/vili-r-oss

  Seems like of A11 because theres R in the name vili-r-oss

  tldr: A11 kernel, seems optimal for this journey


The second one I found is from the lineage os sources
  https://github.com/chemusqui1/android_kernel_xiaomi_sm8350

  The XDA Link
  https://xdaforums.com/t/rom-oss-12-1-lineageos-19-1-for-xiaomi-11t-pro-vili.4520579/

  I'm not sure whether I can use this becuase its based on A12.1
  But the docs and the people seem to mention A11 more so I'll stick
  with the first one

  tldr: A12.1 kernel



The doc is telling me to clone an existing device source
Ideally a similar one

So im seeing what i can clone.
  I get
  the device source of xiaomi-poco-x3-pro

  https://gitlab.com/ubports/porting/community-ports/android11/xiaomi-poco-x3-pro/xiaomi-vayu

  Because it has a similar SoC, sm8150 while mine is sm8350





Cool, lets clone it.

`git clone https://gitlab.com/ubports/porting/community-ports/android11/xiaomi-poco-x3-pro/xiaomi-vayu`


I proceed to download my own phone's source, the first one I mentioned above

`git clone --depth 1 -b vili-r-oss https://github.com/MiCode/Xiaomi_Kernel_OpenSource`


Edit: and also the second one because why not

`git clone --depth 1 -b ReBorn-Stable https://github.com/chemusqui1/android_kernel_xiaomi_sm8350`


obv depth set to 1 because otherwise it'll be massive and will take a long time for no reason

I change the name of the repos to make them more rememberable

the first one
`mv Xiaomi_Kernel_OpenSource vili-a11-kernel`

second one
`mv android_kernel_xiaomi_sm8350 lineage-a12-vili-kernel`


I made folder called backup and copied all three folders there, just in case

then, I changed the name of the device source Ill be working on (xiaomi-vayu) to
xiaomi-vayu-intermediate

Anyways,


## Finding defconfig

now the doc is telling me to fine something called defconfig.
either from the device or from the kernel source itself
Im doing the later becuase i might end up having a wrong one
if I get it from the phone as I am running a custom rom on it
and theres a chance that it might have it different

I open `vili-a11-kernel` in a kate project
I figure out my device is arm64
I add the file called halium.config in arm64/configs with necessary settings

```sh
CONFIG_DEVTMPFS=y
CONFIG_FHANDLE=y
CONFIG_SYSVIPC=y
CONFIG_IPC_NS=y
CONFIG_NET_NS=y
CONFIG_PID_NS=y
CONFIG_USER_NS=y
CONFIG_UTS_NS=y
CONFIG_VT=y
```

## Changing device source

now i go to change the device source in the intermediate folder
I open that as a kate project too

I try to edit the deviceinfo as much as I can manually
then, as told, I try to take inspiration from the lineage device tree
availabe for my device

I find the cmdlines by doing a search

https://github.com/chemusqui1/android_device_xiaomi_sm8350-common/blob/4642bef168359b88ee3b8212efd464c153c3f448/BoardConfigCommon.mk#L97

```sh
BOARD_KERNEL_CMDLINE += androidboot.console=ttyMSM0
BOARD_KERNEL_CMDLINE += androidboot.hardware=qcom
BOARD_KERNEL_CMDLINE += androidboot.memcg=1
BOARD_KERNEL_CMDLINE += androidboot.usbcontroller=a600000.dwc3
BOARD_KERNEL_CMDLINE += cgroup.memory=nokmem,nosocket
BOARD_KERNEL_CMDLINE += console=ttyMSM0,115200n8
BOARD_KERNEL_CMDLINE += loop.max_part=7
BOARD_KERNEL_CMDLINE += msm_rtb.filter=0x237
BOARD_KERNEL_CMDLINE += service_locator.enable=1
BOARD_KERNEL_CMDLINE += swiotlb=0
BOARD_KERNEL_CMDLINE += pcie_ports=compat
BOARD_KERNEL_CMDLINE += iptable_raw.raw_before_defrag=1
BOARD_KERNEL_CMDLINE += ip6table_raw.raw_before_defrag=1
BOARD_KERNEL_CMDLINE += firmware_class.path=/vendor/firmware
```

putting some of them in a single line (organized by me):
 ```sh
 androidboot.hardware=qcom androidboot.console=ttyMSM0 androidboot.memcg=1 androidboot.usbcontroller=a600000.dwc3 service_locator.enable=1 msm_rtb.filter=0x237 loop.max_part=7 cgroup.memory=nokmem,nosocket console=ttyMSM0,115200n8 swiotlb=0 pcie_ports=compat
```

this is the line from vayu (organized by me)
`deviceinfo_kernel_cmdline="androidboot.hardware=qcom androidboot.console=ttyMSM0 androidboot.memcg=1 androidboot.usbcontroller=a600000.dwc3 service_locator.enable=1 msm_rtb.filter=0x237 loop.max_part=7 lpm_levels.sleep_disabled=1 video=vfb:640x400,bpp=32,memsize=3072000  swiotlb=2048 console=tty0"`

this is from the doc
`deviceinfo_kernel_cmdline="console=tty0 bootopt=64S3,32N2,64N2 systempart=/dev/mapper/system"`
</br>

I used chatgpt to gather some info on these parameters.

See `chatgpt on kernel parameters.md` </br>

With all this in mind, I came to a conclusion. I'll use </br>

`androidboot.hardware=qcom androidboot.console=ttyMSM0 androidboot.memcg=1 androidboot.usbcontroller=a600000.dwc3 service_locator.enable=1 msm_rtb.filter=0x237 loop.max_part=7`
`console=tty0 bootopt=64S3,32N2,64N2 systempart=/dev/mapper/system`
`swiotlb=0`

</br>

then setting the bootimg header verion to 3.
Apparently that's what you need for android11
https://source.android.com/docs/core/architecture/bootloader/boot-image-header

Also, I noticed, the KernelSU page seems to suggest that my device is non-gki??
I also found vili_defconfig in https://github.com/diphons/D8G_Kernel_SM8350 .
Just unsure if If its any useful for me or not
