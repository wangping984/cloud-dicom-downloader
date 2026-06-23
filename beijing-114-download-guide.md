# 北京 114 影像下载说明

用该项目下载会面临如何得到链接的问题。
```
https://github.com/Kaciras/cloud-dicom-downloader/tree/master
```

这份说明针对一个已经验证可用的场景：

- 在北京地区医院拍片。
- 通过 114 小程序查看报告和影像。
- 影像实际由海纳医信 `medicalimagecloud.com` 提供。
- 使用本仓库下载原始 DICOM 文件。

## 先说结论

如果你已经在 114 小程序里打开了影像页面，直接复制地址栏里类似下面这种链接，通常不能直接下载：

```text
https://bjsyxpt.medicalimagecloud.com:10808/Study/StudyView?... 
https://bjsyxpt.medicalimagecloud.com:10808/Study/StandAloneReport?... 
```

这两种通常只是当前会话里的页面地址，不是本项目需要的分享入口。

本项目真正需要的是通过“分享”功能生成出来的链接，格式通常类似：

```text
https://bjsyxpt.medicalimagecloud.com:10808/t/<share_id>
```

并且还需要对应的访问密码。

## 为什么会踩坑

114 小程序里点进影像后，页面会跳转到海纳医信的系统，但这个最终页面地址往往带有临时会话状态。

实际踩坑中遇到过下面两类地址：

```text
https://bjsyxpt.medicalimagecloud.com:10808/Study/StudyView?id=<uuid>&...
https://bjsyxpt.medicalimagecloud.com:10808/Study/StandAloneReport?id=<uuid>&...
```

这些地址看起来像是报告页和影像页，但单独拿出来往往会失效，或者离开当前登录状态后就没有权限。

所以不要把它们当成最终下载链接。

## 正确拿链接的方法

### 1. 先进入 114 小程序里的检查详情

一般会先看到一个页面，里面有“报告”和“影像”两个入口。

### 2. 不要直接复制影像页地址

即使你已经能看到影像，也先不要急着复制 `StudyView` 或 `StandAloneReport` 这类地址。

### 3. 使用页面里的分享功能

关键点是使用海纳医信页面或相关入口里的分享功能，生成：

- 一个二维码
- 一个访问密码

扫描二维码后，或者从分享结果里复制出来的链接，才是本项目需要的链接。

实际验证可用的格式如下：

```text
https://bjsyxpt.medicalimagecloud.com:10808/t/<share_id>
```

有时你复制出来可能会变成：

```text
https://bjsyxpt.medicalimagecloud.com:10808//t/<share_id>
```

这种双斜杠通常也能用；如果担心兼容性，可以手工改成单斜杠 `/t/`。

### 4. 记下密码

分享功能除了生成链接，还会给一个访问密码。下载命令必须同时提供链接和密码。

## 下载前准备

先安装依赖。若你使用的是 Miniforge/Conda 环境，可以先激活环境：

```powershell
conda activate mybase
python -m pip install -r .\requirements.txt
```

如果依赖没装到你真正运行的 Python 环境里，常见报错是：

```text
ModuleNotFoundError: No module named 'yarl'
```

这通常不是项目本身有问题，而是你运行命令时用错了解释器。

## 下载命令

普通下载：

```powershell
python .\downloader.py "https://bjsyxpt.medicalimagecloud.com:10808/t/<share_id>" "<password>"
```

下载 raw 像素数据：

```powershell
python .\downloader.py "https://bjsyxpt.medicalimagecloud.com:10808/t/<share_id>" "<password>" --raw
```

`--raw` 表示下载未压缩像素。已经验证在北京 114 这个海纳医信入口上可用。

## 下载成功后会得到什么

程序会在仓库根目录下创建 `download` 目录，并按检查信息生成子目录，例如：

```text
download/
└── [患者姓名]-[检查项目]-[日期]/
    ├── [序列号]-[序列名]/
    │   ├── 00001.dcm
    │   ├── 00002.dcm
    │   └── ...
    └── ...
```

每个子目录对应一个序列，里面是一组 `.dcm` 文件。

## 常见问题

### 只有 `StudyView` 地址，没有 `/t/` 地址怎么办

回到 114 或海纳医信页面，重新找“分享”功能。只有 `StudyView` 或 `StandAloneReport` 地址，通常不够。

### 链接能打开，但脚本下载失败怎么办

先检查两点：

1. 你传给脚本的是不是 `/t/<share_id>` 这种分享链接。
2. 你传的密码是不是分享时显示的那个密码。

### 明明执行过 `pip install -r requirements.txt`，还是报缺模块

大概率是装到了别的 Python 环境里。先激活你真正要运行脚本的环境，再执行安装命令。

### 这个仓库能直接看片吗

不能。这个仓库是下载器，不带 DICOM 查看器。下载完成后，需要用单独的 DICOM 查看软件打开，或者用仓库里的 `tools/export.py` 转成图片或视频。

## 适用范围

这份文档只覆盖一个已经实测成功的流程：

- 北京地区
- 114 小程序
- 海纳医信 `bjsyxpt.medicalimagecloud.com`
- 通过分享功能拿到 `/t/<share_id>` 和密码

如果医院换了别的云影像系统，或者 114 后续改了跳转逻辑，这份说明可能就不再适用。