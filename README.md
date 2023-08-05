# IP-C-Class

IPCClass 是一个利用C类IP特性实现的性能非常高的IP查询工具。

## 特点

* IP属性查询，复杂度O(1)，直接内存指定偏移量即可获取IP属性。
* IP属性包括多级归属地，例如国家，省市，区等，包括运营商信息。
* IP相邻度计算，复杂度O(1)，通过单次异或操作可得出两个IP相邻度。
* IP相关属性计算，例如是否是一线城市，是否是小运营商等。
* 可根据自身需要定制数据，默认格式采用4个字节保存，占用64MB内存。

## 目录

```txt
data 数据文件
 + 
sdk 各种语言的sdk
server 提供独立的查询服务
tools IP处理工具集
converter 其他IP库转换
```

## 使用方法

xx

## 原理

* 一个IPv4的IP由32位/4个字节构成，标记成A.B.C.D，其中每一段范围是0～255。
* 本IP库精度到C类IP（CIDR：A.B.C.x/24），因为绝大多数运营商的IP段分配范围都不会比这个前缀更小了。
* 而这个C类IP前缀只有1678（256 * 256 * 256）万个，如果一个IP地址用1个字节表示它的属性，只需要16MB就能保存，并且通过偏移量可以直接取出。
* 每个IP的属性2个字节/16位，可以通过每个位进行多级位置表示，当比较IP相邻度时，可以通过异或操作，即可判断IP的相邻度（因为每级相同都为0，不同则为1）。

## 数据格式

可以根据自身需要进行定制，这里基于国内常用情况，进行划分。

<table>
 <tr><th>偏移(字节)</th><th>0</th><th>1</th><th>&nbsp;&nbsp;&nbsp;&nbsp;2&nbsp;&nbsp;&nbsp;&nbsp;</th><th>&nbsp;&nbsp;&nbsp;&nbsp;3&nbsp;&nbsp;&nbsp;&nbsp;</th></tr>
 <tr><th>内容</th><th>城市属性</th><th>运营商</th><th colspan="2">地理位置</th></tr>
</table>

### 地理位置

<table>
 <tr><th>偏移(字节)</th><th colspan="8">2</th><th colspan="8">3</th></tr>
 <tr><th>偏移(位)</th><th>0</th><th>1</th><th>2</th><th>3</th><th>4</th><th>5</th><th>6</th><th>7</th><th>0</th><th>1</th><th>2</th><th>3</th><th>4</th><th>5</th><th>6</th><th>7</th></tr>
 <tr align="center"><td rowspan="4">地区</td><td>A</td><td colspan="3">BBB</td><td colspan="3">CCC</td><td colspan="2">DD</td><td colspan="3">EEE</td><td colspan="4">FFFF</td></tr>
 <tr><td colspan="16">A：0：国内<br>B：国内大区，如：华南<br>C：省，如：广东<br>D：省内区域，如：粤西，珠三角地区<br>E：市，如：广州市<br>F：县区，如：荔湾区</td></tr>
 <tr align="center"><td>a</td><td colspan="3">bbb</td><td colspan="3">ccc</td><td colspan="3">ddd</td><td colspan="2">ee</td><td colspan="4">ffff</td></tr>
 <tr><td colspan="16">a：1：国外<br>b：大洲，如：亚洲<br>c：大洲区域，如：东亚<br>d：国家，如：日本<br>e：地区，如：关东地区<br>f：市，如：东京</td></tr>
</table>

### 运营商

<table>
 <tr><th>偏移(字节)</th><th colspan="8">1</th></tr>
 <tr><th>偏移(位)</th><th>0</th><th>1</th><th>2</th><th>3</th><th>4</th><th>5</th><th>6</th><th>7</th></tr>
 <tr align="center"><td rowspan="2">地区</td><td>A</td><td>B</td><td>C</td><td>D</td><td colspan="4">EEEE</td></tr>
 <tr><td colspan="16">国内：<br>A：电信连通性<br>B：联通连通性<br>C：移动连通性<br>D：是否小运营商<br>E：运营商<br>国外：<br>ABC分别为对应地区主流运营商的连通性</td></tr>
</table>

### 城市属性

<table>
 <tr><th>偏移(字节)</th><th colspan="8">0</th></tr>
 <tr><th>偏移(位)</th><th>0</th><th>1</th><th>2</th><th>3</th><th>4</th><th>5</th><th>6</th><th>7</th></tr>
 <tr align="center"><td rowspan="2">地区</td><td colspan="2">AA</td><td>B</td><td>C</td><td>D</td><td colspan="3">EEE</td></tr>
 <tr><td colspan="16">国内：<br>A：省级定义，如：省，自治区，直辖市，特别行政区<br>B：是否首都<br>C：是否省会<br>D：是否经济特区<br>E：城市等级，如：一线城市，二线城市<br>国外：<br>参照执行</td></tr>
</table>

## 描述文件格式

描述文件用于进行数据编码工作和转换过滤工作。

### 语法

* 开始是#号的行是注释，地区名字开始#为该名字不进行搜索关联，只做注释使用
* | 逻辑或，表示多个分隔的词只需要匹配一个即可
* ! 逻辑非，表示匹配的同时不存在!后的关键字
* @ 完整匹配

### 分段

使用 ini 文件格式定义。

```ini

; 公共定义
[base]
; 一个IP属性共有几个字节
size=4
; 城市属性偏移是0，长度为1字节
attribute-offset=0
attribute-size=1
; 运营商属性偏移是1，长度为1字节
isp-offset=1
isp-size=1
; 地区属性偏移是2，长度为2字节
district-offset=2
district-size=2

; 城市属性详细编码
[attribute]
; 编码=说明
00000001=新一线城市
00000010=二线城市

[isp]
00000000=未知运营商
10010000=电信
01010010=联通

[district]
00000000=#市辖区
00000000001=海淀|清华大学!台湾!深圳|中国地质大学!武汉!长城学院|中国人民大学
101=@AD

```


