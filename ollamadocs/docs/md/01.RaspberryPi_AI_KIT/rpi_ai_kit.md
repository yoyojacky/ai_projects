
# **入门指南**

本指南将帮助您使用Raspberry Pi 5设置Raspberry Pi AI套件。这将使您能够使用Hailo AI神经网络加速器运行rpicam-apps相机演示。

## **先决条件**
对于本指南，您将需要以下内容：

* - Raspberry Pi 5
* - Raspberry Pi AI套件，其中包括：
* - M.2 HAT+
* - 预装的Hailo-8L AI模块
* - 64位Raspberry Pi OS Bookworm安装
* - 任何官方Raspberry Pi相机（例如Camera Module 3或High Quality Camera）

## **硬件设置**
* 1. 按照在Install a Raspberry Pi Camera上的说明，将相机连接到您的Raspberry Pi 5板上。您可以跳过重新连接您的Raspberry Pi到电源，因为您需要在下一步中将Raspberry Pi从电源断开。

* 2. 按照安装说明将您的AI套件硬件连接到您的Raspberry Pi 5。

* 3. 按照说明启用PCIe Gen 3.0。这一步是可选的，但是强烈推荐以实现与您的AI套件的最佳性能。

安装使用AI套件所需的依赖项。从终端窗口运行以下命令：

```
sudo apt install hailo-all
```

这将安装以下依赖项：

* - Hailo内核设备驱动程序和固件
* - HailoRT中间件软件
* - Hailo Tappas核心后处理库
* - rpicam-apps Hailo后处理软件演示阶段

最后，使用`sudo reboot`重新启动您的Raspberry Pi，以使这些设置生效。

为确保一切运行正常，请运行以下命令：

```
hailortcli fw-control identify
```

如果您看到类似于以下输出，那么您已成功安装了AI套件及其软件依赖项：

```
Executing on device: 0000:01:00.0
Identifying board
Control Protocol Version: 2
Firmware Version: 4.17.0 (release,app,extended context switch buffer)
Logger Version: 0
Board Name: Hailo-8
Device Architecture: HAILO8L
Serial Number: HLDDLBB234500054
Part Number: HM21LB1C2LAE
Product Name: HAILO-8L AI ACC M.2 B+M KEY MODULE EXT TMP
```

此外，您可以运行`dmesg | grep -i hailo`来检查内核日志，应该得到类似于以下的输出：

```
[    3.049657] hailo: Init module. driver version 4.17.0
[    3.051983] hailo 0000:01:00.0: Probing on: 1e60:2864...
[    3.051989] hailo 0000:01:00.0: Probing: Allocate memory for device extension, 11600
[    3.052006] hailo 0000:01:00.0: enabling device (0000 -> 0002)
...
[    3.052070] hailo 0000:01:00.0: Successfully disabled ASPM L0s
[    3.221043] hailo 0000:01:00.0: Firmware was loaded successfully
[    3.231845] hailo 0000:01:00.0: Probing: Added board 1e60-2864, /dev/hailo0
```

为确保相机正常工作，请运行以下命令：

```
rpicam-hello -t 10s
```

这将启动相机，并显示预览窗口十秒钟。一旦您验证了一切安装正确，就是时候运行一些演示了。

## **演示**
rpicam-apps套件的相机应用程序实现了一个后处理框架。本节包含一些演示后处理阶段，这些阶段突出了AI套件的一些能力。

以下演示使用rpicam-hello，默认情况下显示预览窗口。但是，您可以使用其他rpicam-apps，包括rpicam-vid和rpicam-still。您可能需要添加或修改一些命令行选项，以使演示命令与替代应用程序兼容。

首先，下载演示所需的后处理JSON文件。这些文件确定运行哪些后处理阶段，并配置每个阶段的行为。例如，您可以启用、禁用、加强或减弱目标检测演示中时间过滤的强度。或者，您可以在分割演示中启用或禁用输出掩模绘制。

要下载整个后处理JSON文件集合，请克隆rpicam-apps repo。运行以下命令以仅克隆repo中的最新提交，节省空间：

```
git clone --depth 1 https://github.com/raspberrypi/rpicam-apps.git ~/rpicam-apps
```

## **提示**
后续部分中提供的命令使用此存储库中的JSON文件。为了便于引用这些文件，此命令在您的主文件夹中创建了克隆的rpicam-apps目录。如果您修改了此目录的位置，则还必须更改下面的演示命令，以引用JSON文件的新位置。

## **对象检测**
此演示显示神经网络检测到的对象周围的边界框。要禁用视图查找器，请使用-n标志。要返回纯文本输出描述检测到的对象，请添加-v 2选项。请运行以下命令在您的Raspberry Pi上尝试演示：

```
rpicam-hello -t 0 --post-process-file ~/rpicam-apps/assets/hailo_yolov6_inference.json --lores-width 640 --lores-height 640
```

或者，您可以尝试另一个模型，该模型在性能和效率之间有不同的权衡。

要使用Yolov8模型运行演示，请运行以下命令：

```
rpicam-hello -t 0 --post-process-file ~/rpicam-apps/assets/hailo_yolov8_inference.json --lores-width 640 --lores-height 640
```

要使用YoloX模型运行演示，请运行以下命令：

```
rpicam-hello -t 0 --post-process-file ~/rpicam-apps/assets/hailo_yolox_inference.json --lores-width 640 --lores-height 640
```

要使用Yolov5人和面部模型运行演示，请运行以下命令：

```
rpicam-hello -t 0 --post-process-file ~/rpicam-apps/assets/hailo_yolov5_personface.json --lores-width 640 --lores-height 640
```

## **图像分割**
此演示执行对象检测，并通过在视图查找器图像上绘制颜色掩模来分割对象。请运行以下命令在您的Raspberry Pi上尝试演示：


```
rpicam-hello -t 0 --post-process-file ~/rpicam-apps/assets/hailo_yolov5_segmentation.json --lores-width 640 --lores-height 640 --framerate 20
```

## **姿态估计**
此演示执行17点人体姿态估计，绘制连接检测点的线。请运行以下命令在您的Raspberry Pi上尝试演示：

```
rpicam-hello -t 0 --post-process-file ~/rpicam-apps/assets/hailo_yolov8_pose.json --lores-width 640 --lores-height 640
```

## 其他资源
Hailo还为Raspberry Pi 5创建了一套演示，这些演示可以在hailo-ai/hailo-rpi5-examples GitHub仓库中找到。

你可以在hailo-ai/hailo_model_zoo GitHub仓库中找到Hailo的广泛模型库，它包含大量神经网络。

查看Hailo社区论坛和开发者区域，以获取有关Hailo硬件和工具的进一步讨论。

## 产品简介
* 有关AI套件的更多信息，包括机械规格和操作环境限制，请查看产品简介。

---
PS: 我特喵的还没有拿到设备，只能先把文档搞定。

文档来自树莓派官方文档，对于copyright的nerd，just go fuck yourself, do not waste my time!
