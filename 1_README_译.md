# trk234
深空网络 TRK 2-34 跟踪和导航文件 (TNF) 数据格式的 Python 阅读器库和示例使用脚本。该库完全兼容 TRK-2-34 格式的 N 版本，但也适用于其更高版本。[^1]
这些实用程序由加州理工学院喷气推进实验室的行星雷达和射电科学组 (332K) 开发。


## Data Overview 数据架构说明

要使用这个库，必须首先了解 TRK-2-34 数据格式的组织和结构 [^1]，**强烈建议在继续之前阅读软件接口规范文档 [^1]**。

TNF 是深空网络闭环跟踪系统最原始（也是最庞大）的产品。TRK 2-34 文件就是 TNF，因此，根据用户的偏好，可以将其称为 TNF 或 TRK 2-34。每个 TRK 2-34 文件都是在航天器被特定 DSN 站主动跟踪时以近实时 (NERT) 方式生成的。DSN 按时间升序排列 TRK 2-34 文件中的 SFDU。
TRK 2-34 文件是标准格式化数据单元 (SFDU) 格式的二进制文件。SFDU 共有 18 种不同的类型，分为五类。

|  编号  |   数据类型    |              描述                 | SFDU 长度（字节）     |
|--------|--------------|-----------------------------------|----------------------|
|      0 | 上行链路      | 上行链路载波相位                   |     162              |
|      1 | 下行链路      | 下行链路载波相位                   |     358              |
|      2 | 上行链路      | 上行链路顺序测距相位               |     194              |
|      3 | 下行链路      | 下行链路顺序测距相位               |     304              |
|      4 | 上行链路      | 上行链路 PN 测距相位               |     276              |
|      5 | 下行链路      | 下行链路 PN 测距相位               |     388              |
|      6 | 导出          | 多普勒计数                        |     200              |
|      7 | 导出          | 顺序测距                          |     330              |
|      8 | 导出          | 角度                              |     178              |
|      9 | 导出          | 斜坡频率                          |     124              |
|     10 | 干涉测量      |  VLBI                            |      204              |
|     11 | 导出          |  DRVID                           |      182             |
|     12 | 滤波          | 平滑噪声                          |     164              |
|     13 | 滤波          | 艾伦偏差                          |     160              |
|     14 | 导出          | PN 范围                           |     348              |
|     15 | 导出          | 音调范围                          |     194              |
|     16 | 导出          | 载波频率可观测值                   |     182 + 18n        |
|     17 | 导出          | 总计数相位可观测值                 |     194 + 22n        |


## Installation 安装

这是一个包含脚本和配置文件的 Python 库包。
您可以通过将代码库克隆到本地计算机并运行以下命令来安装该库：
```
pip install \path\to\trk234
```
请记下安装路径。将“scripts/”目录中的文件添加到您的执行路径，如果使用“bin/”执行脚本，请相应地更新脚本中的路径。


### Configuration 配置

对于 Python 环境复杂或希望简化安装的用户，“bin/” 目录中提供了几个“bash”脚本。在每个文件中，编辑以下语句以指向库的正确安装目录：
```
# 更新 pythonpath 以获取正确的库
export PYTHONPATH=$PYTHONPATH:/home/source/trk234
```

同时更新脚本的位置：
```
# 添加脚本安装目录的路径
SCRIPTDIR=/home/source/trk234/scripts
```


## Library Architecture 库结构设计

`Reader` 类读取 TRK-2-34 文件；然后，可以通过 `SFDU` 类解码文件并访问其内容。`Reader` 类中的一个属性是 `sfdu_list`，它包含 `SFDU` 类的列表。每个 `SFDU` 类包含：
* `SFDU.label`：SFDU 标签
* `SFDU.agg_chdo`：聚合 CHDO
* `SFDU.pri_chdo`：主 CHDO
* `SFDU.sec_chdo`：辅助 CHDO
* `SFDU.trk_chdo`：跟踪数据 CHDO

如 TRK-2-34 软件接口规范文档 [^1] 中数据表第 2 列所述，每个类的独立属性包含数据。


### Examples 示例
#### Example: Basic file decoding 示例：基础文件解码

该模块读取并解析给定 TRK 2-34 二进制文件中的 SFDU。读取文件的基本语法如下：
```
import trk234

f = trk234.Reader('filename.tnf')
f.decode()

sfdus = f.sfdu_list
```

这将读取原始二进制数据并确定 SFDU 断点，然后将其解析为 SFDU 列表。如果您处理的是大型文件，并且只需要解码 SFDU 的某些部分，则可以选择将以下关键字参数之一放入 decoder() 函数中：

* `decode( label=False )` - 不解码标签
* `decode( agg_chdo=False )` - 不解码聚合 CHDO
* `decode( pri_chdo=False )` - 不解码主 CHDO
* `decode( sec_chdo=False )` - 不解码辅助 CHDO
* `decode( trk_chdo=False )` - 不解码跟踪 CHDO

如果您禁用标签解码，则无法解码其他任何内容（不推荐）。如果您禁用主 CHDO 解码，则无法解码跟踪 CHDO。

例如，如果您只需要知道文件中的数据类型，则可能需要禁用辅助 CHDO 和跟踪 CHDO，例如：
```
f.decode( sec_chdo=False, trk_chdo=False )
```


#### Example: Print Attribute 示例：输出属性
```
import trk234

f = trk234.Reader('15025s026.stdf')
f.decode()
for s in f.sfdu_list:
    print( s.pri_chdo.format_code )
```

#### Example: Get info on a file 示例：获取文件信息
```
import trk234

f = trk234.Reader('15025s026.stdf')
f.decode(trk_chdo=False) # Setting trk_chdo to false will not decode the Tracking CHDO, speeding processing time
info = trk234.Info( f )
print( info )
```

#### Dumping to ASCII 转储为 ASCII

警告：这可能会产生非常大的文本，具体取决于数据文件的大小。

```
import trk234

f = trk234.Reader('15025s026.stdf')
f.decode()
for s in f.sfdu_list:
    print( s )
```

#### Accessing attributes 访问属性

属性可从相应的 CHDO 访问。属性名称与 TRK 2-34 文档中指定的“标识符”相同。

```
import trk234

f = trk234.Reader('15025s026.stdf')
f.decode()

time = []
sky_frequency = []
signal_power = []
for s in f.sfdu_list:
    if s.pri_chdo.format_code == 1:
        time.append( s.timestamp() )
        sky_frequency.append( s.trk_chdo.dl_freq )
        signal_power.append( s.trk_chdo.pcn0 )
```

## Script Usage 脚本用法

### Read Downlink Information 读取下行链路信息

读取下行链路信息（天空频率、系统噪声温度、载波信噪比、环路带宽）并打印到终端。可选择按 DSN 站号、下行链路频段和/或跟踪模式进行过滤。

用法：`trk234_dnlink.py [-h] [-l] [-d DSS] [-b BAND] [-m MODE] [-t] [-c] Input`

基本示例：*打印到文本文件*
```
trk234_dnlink.py GRV_JUGR_2016240_0635X55MC001V01.TNF > downlink.txt
```


### Dump Contents to ASCII 将内容转储为 ASCII

读取 TRK-2-34 数据文件，以类似伪 JSON 的格式打印每个 SFDU 的所有属性。
警告：这可能会产生大量的文本，具体取决于数据文件的大小。请使用最大数量和数据类型选项来限制转储的大小。

用法：`trk234_dump.py [-h] [-f FORMAT_CODE] [-m MAX] Input`

基本示例：*转储到文本文件*
```
trk234_dump.py GRV_JUGR_2016240_0635X55MC001V01.TNF > dump.txt
```

复杂示例：*仅转储航母可观测数据类型的第一个SFDU*
```
trk234_dump.py -m 1 -f 16 GRV_JUGR_2016240_0635X55MC001V01.TNF
```


### Extract an Individual Attribute 提取个体属性

读取 TRK-2-34 文件，并从 TRK-2-34 文档中打印用户指定属性的时间历史记录 [^1]。必须提供属性名称，以及它来自 SFDU 的哪个部分（SFDU 标签、聚合 CHDO、主 CHDO、辅助 CHDO 或跟踪 CHDO）。

用法：`trk234_extract [-h] [-f FORMAT_CODE] [-p] [-t] [-i IDENTIFIER] [--label] [--agg] [--pri] [--sec] [--trk] Input`
必需一项：`--label`、`--agg`、`--pri`、`--sec`、`--trk` 、 `-i IDENTIFIER` 

示例：*从所有 SFDU 中提取航天器 ID*
```
trk234_extract.py --sec -i scft_id GRV_JUGR_2016240_0635X55MC001V01.TNF > scft_id.txt
```


### Print Information from File 从文件打印信息

读取 TRK-2-34 文件并解析有关该文件的高级信息，并将其打印到终端。

Usage: `trk234_info.py [-h] [-p] [-m] [-q] Input`

Example:
```
trk234_info2.py GRV_JUGR_2016240_0635X55MC001V01.TNF
```

Example Output:
```
         Report for File: GRV_JUGR_2016240_0635X55MC001V01.TNF
         Generation Date: 2023-268T22:22:36
              Start Time: 2016-240T06:35:48
                End Time: 2016-240T19:50:00
           Spacecraft ID: 61
         Downlink DSS ID: 55
          Downlink Bands: X, Ka
      Doppler Count Time: 1.0
           Uplink DSS ID: 55
            Uplink Bands: X
           Tracking Mode: 2W, None, 3W/43, 1W
       Number of Records: 283594
    Data Description IDs: C125, C123, C124
    Available Data Types: 0, 1, 2, 3, 7, 9, 11, 16, 17
                      00: Uplink Carrier Phase - 47596
                      01: Downlink Carrier Phase - 66247
                      02: Uplink Sequential Ranging Phase - 36570
                      03: Downlink Sequential Ranging Phase - 113
                      07: Sequential Ranging - 113
                      09: Ramps - 366
                      11: DRVID - 113
                      16: Carrier Observable - 66238
                      17: Total Phase Observable - 66238
```

### Purify/Filter a TRK-2-34 to SIS Compliance 净化/过滤 TRK-2-34 以符合 SIS 规范

从 TRK-2-34 文件中删除不合规的 SFDU，并可选择按下行链路频段、上行链路频段和 DSN 站号筛选数据文件。
**这是标记 TRK-2-34 文件的必需步骤之一**
用法：`trk234_purify [-h] [-v] [-p] [-b DL_BAND] [-a UL_BAND] [-d DL_DSS_ID] [-u UL_DSS_ID] [-f FORMAT_CODE] 输入 输出`

示例：*净化 TRK-2-34 文件，删除因 DSN 标记某些文件为坏文件而导致的不合规 CHDO*
```
trk234_purify.py 232511615SC61DSS35_noHdr.234 232511615SC61DSS35_noHdr.234.pure
```
批量处理示例：*对大量数据文件进行净化*
```
find *234 -exec trk234_purify.py {} {}.pure \; >> logfile.txt
```


### Read Uplink Information 读取上行链路信息

从 TRK-2-34 文件中读取上行链路斜坡历史记录并打印出来。打印的数据是斜坡频率和斜坡速率。
用法：`trk234_ramp.py [-h] [-d DSS] [-b BAND] [-t] 输入`

示例：*打印 DSS-55 中的 X 波段斜坡*
```
trk234_ramp.py -b X -d 55 GRV_JUGR_2016240_0635X55MC001V01.TNF > ramp_X_55.txt
```


### Sort a TRK-2-34 File by Data Type 按数据类型对 TRK-2-34 文件进行排序

按数据类型（又称格式代码）对 TRK-2-34 文件进行升序排序（重组）。这将使 TRK-2-34 文件更易于为 PDS 进行标记，但 SFDU 将不再按时间递增顺序排列。
**这是标记 TRK-2-34 文件的必需步骤之一**
用法：`trk234_regroup.py [-h] [-v] [-p] [--validate] 输入 输出`

示例：
```
trk234_regroup.py 232511615SC61DSS35_noHdr.234.pure 232511615SC61DSS35_noHdr.234.pure.sorted
```

批量处理示例：*对大量数据文件进行排序*
```
find *234.pure -exec trk234_regroup.py {} {}.sorted \; >> logfile.txt
```


# Disclaimer Statement 免责声明
Copyright (c) 2023, California Institute of Technology ("Caltech").
U.S. Government sponsorship acknowledged. Any commercial use must be 
negotiated with the Office of Technology Transfer at the California 
Institute of Technology.
 
This software may be subject to U.S. export control laws. By accepting this 
software, the user agrees to comply with all applicable U.S. export laws 
and regulations. User has the responsibility to obtain export licenses, or 
other export authority as may be required before exporting such information 
to foreign countries or providing access to foreign persons.

All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice,
  this list of conditions and the following disclaimer.
* Redistributions must reproduce the above copyright notice, this list of
  conditions and the following disclaimer in the documentation and/or other
  materials provided with the distribution.
* Neither the name of Caltech nor its operating division, the Jet Propulsion
  Laboratory, nor the names of its contributors may be used to endorse or
  promote products derived from this software without specific prior written
  permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.


[^1]: https://pds-geosciences.wustl.edu/radiosciencedocs/urn-nasa-pds-radiosci_documentation/dsn_trk-2-34/



版权所有 (c) 2023，加州理工学院（“加州理工学院”）。
承认美国政府赞助。任何商业用途必须与加州理工学院技术转让办公室协商。

本软件可能受美国出口管制法律的约束。用户接受本软件即表示同意遵守所有适用的美国出口法律和法规。用户有责任在将此类信息出口到外国或向外国人提供访问权限之前，获得出口许可证或
其他可能需要的出口授权。

保留所有权利。

在满足以下条件的情况下，允许以源代码和二进制形式（无论是否经过修改）进行再分发和使用：

* 源代码的再分发必须保留上述版权声明、
此条件列表以及以下免责声明。
* 重新分发必须在随分发提供的文档和/或其他材料中复制上述版权声明、此条款列表以及以下免责声明。
* 未经事先明确书面许可，不得使用加州理工学院及其运营部门喷气推进实验室的名称或其贡献者的名称来认可或推广基于本软件的产品。

本软件由版权所有者和贡献者“按原样”提供，
并且不承担任何明示或暗示的保证，包括但不限于
对适销性和特定用途适用性的暗示保证。在任何情况下，版权所有者或贡献者均不对任何直接、间接、偶发、特殊、惩戒性或后果性损害（包括但不限于采购替代商品或服务；使用、数据或利润损失；或业务中断）承担责任，无论该损害是如何造成的，也不论是基于何种责任理论，无论是合同、严格责任还是侵权行为（包括疏忽或其他），即使已被告知存在发生此类损害的可能性。

[^1]：https://pds-geosciences.wustl.edu/radiosciencedocs/urn-nasa-pds-radiosci_documentation/dsn_trk-2-34/




# 另：TNF文件数据内容详解
根据DSN TRK-2-34格式文档，TNF文件是NASA深空网络的原始跟踪和导航数据，包含以下主要内容：

## 主要数据类型
TNF文件包含18种不同类型的标准格式化数据单元(SFDU)，分为五大类：
### 1. 上行链路数据
类型0: 上行链路载波相位 (162字节)
包含从地面发送到航天器的载波信号相位信息
类型2: 上行链路顺序测距相位 (194字节)
类型4: 上行链路PN测距相位 (276字节)
### 2. 下行链路数据
类型1: 下行链路载波相位 (358字节)
包含从航天器返回地球的信号相位、频率和功率信息
对无线电掩星最重要
类型3: 下行链路顺序测距相位 (304字节)
类型5: 下行链路PN测距相位 (388字节)
### 3. 导出数据
类型6: 多普勒计数 (200字节)
类型7: 顺序测距 (330字节)
类型8: 角度 (178字节)
类型9: 斜坡频率 (124字节)
类型11: DRVID (182字节)
类型14: PN范围 (348字节)
类型16: 载波观测量
类型17: 总相位观测量
### 4. 干涉测量数据
类型10: VLBI (204字节)
### 5. 滤波数据
类型12: 平滑噪声 (164字节)
类型13: 艾伦偏差 (160字节)
每个SFDU记录的主要字段

## 每个数据记录通常包含：
时间戳: 记录采集时间
DSN站标识: 指示哪个深空网络站采集的数据
频率信息: 信号的频率值和变化
相位数据: 载波相位测量值
功率/强度: 信号强度指标
SNR: 信号噪声比
测距数据: 信号往返时间转换的距离信息