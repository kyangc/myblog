---
title: 从字符编码说开去
date: 2019-02-24 22:41:26
categories: 技术
tags: [java]
description: 你确定你把字符编码整明白了吗？
---

# 背景

字符和字符编码这种貌似从开始学习计算机开始就知道的东西，在写了这么多年代码之后，回过头看，其实还有很多值得好好说道说道的东西。下面我们就一起来复习一下。

## 字符集和 Unicode

我们都知道计算机是不能直接存储字母，数字，图片，符号等，计算机能处理和工作的唯一单位是 “比特位（bit）”，一个比特位通常只有 0 和 1，是（yes）和否（no），真（true）或者假（false）等等我们喜欢的称呼。利用比特位序列来代表字母，数字，图片，符号等，我们就需要一个存储规则，不同的比特序列代表不同的字符，这就是所谓的 “编码”。反之，将存储在计算机中的比特位序列（或者叫二进制序列）解析显示出来成对应的字母，数字，图片和符号，称为 “解码”。提到编码不得不说到字符集，因为任何一种编码方式都会有一个字符集与之对应。之前的很长一段时间内，字符集和字符编码区分不是很严格，因为一般一种字符集只对应一种编码方式。

然而计算机领域通常所讲的字符集实际上是指字符编码集，而不是传统意义上的字符库，除了包括一个系统支持的所有抽象字符（如国家文字、标点符号、图形符号、数字等）外，还包括字符所对应的一个数值，所以通常是一个二维表，下文提到的字符集都是指字符编码集。常见字符集如 ASCII 字符集、ISO-8859-X、GB2312 字符集（简中）、BIG5 字符集（繁中）、GB18030 字符集、Shift-JIS 等等。

以最为熟知的 ASCII 码为例。ASCII 码是基于拉丁字母的一套电脑编码系统。它主要用于显示现代英语，而其扩展版本 EASCII 则可以勉强显示其他西欧语言，其等同于国际标准 ISO/IEC 646。ASCII 码使用 7 位 bit 一个字节 来标识字符。ASCII 码表如下所示：

![](https://imgs.kyangc.com/2019-02-24-ascii.png "ASCII 码表")

如前所述，为了表达汉字、日文、韩语等等字符，在不断地发展中，又形成了各种各样的字符集，但是这些编码方式又存在以下这些问题：

* 没有一种编码可以覆盖全世界所有国家的字符；
* 各种编码之间也会存在冲突的现象，两种不同编码方式可能使用同一个编码代表不同的字符，亦或用不同的编码代表同一个字符；
* 一个指定的机器（比如我们的服务器）将需要支持许多不同的编码方式，当数据在不同的机器之间传输或者在不同的编码之间转换时，很容易产生乱码问题。

多语言软件制造商组成的统一码联盟 (The Unicode Consortium) 于 1991 年发布的统一码标准（The Unicode Standard)，定义了一个全球统一的通用字符集即 Unicode 字符集解决了上述的问题。统一码标准为每个字符提供一个唯一的编号，旨在支持世界各地的交流，处理和显示现代世界各种语言和技术学科的书面文本。此外，它支持许多书面语言的古典和历史文本，不管是什么平台，设备，应用程序或语言，都不会造成乱码问题， 它已被所有现代软件供应商采用，是目前所有主流操作系统，搜索引擎，浏览器，笔记本电脑和智能手机以及互联网和万维网（URL，HTML，XML，CSS，JSON 等）中表示语言和符号的基础。统一码标准的一个版本由核心规范、Unicode 标准、代码图、Unicode 标准附件以及 Unicode 字符数据库（Unicode Character Database 简写成 UCD）组成，同时也是开发的字符集，在不断的更加和增加新的字符，最新的版本为 Unicode 10.0.0。

在 Unicode 字符集发布的第三年，ISO/IEC 联合发布了通用多八位编码字符集简称为 UCS (Universal Character Set 通用字符集)，也旨在使用全球统一的通用字符集。两套通用字符集使用起来又存在麻烦，后来统一码联盟与 ISO/IEC 经过协商，到到 Unicode 2.0 时，Unicode 字符集和 UCS 字符集基本保持了一致。现在虽然双方都发布各自的版本，但两套字符集是相互兼容的，Unicode 字符集使用比 UCS 字符集更加广泛。

Unicode 将编码空间划分为 17 个平面 (plane), 从 0 到 16. 0 号平面称为基本多文种平面 (BMP), 其他的 16 个平面可以统称为辅助平面 (Supplementary Plane, SP)。虽说 Unicode 的 BMP 只有 2^16=65536 个码位，看起来很少，实际上绝大部分文字都是落在 BMP 里，许多奇怪的语言，比如印度的一些小语种，还有从右往左书写的阿拉伯语，即便是这些我们觉得像鬼画符一般的语言，也都是落在 BMP 里的。

下面是一些相关术语的解释：

- **Coded Character Set（CCS）**：即编码字符集，给字符表里的抽象字符编上一个数字，也就是字符集合到一个整数集合的映射。这种映射称为编码字符集，Unicode 字符集就是属于这一层的概念；
- **Character Encoding Form（CEF）**：即字符编码表，根据一定的算法，将编码字符集（CCS） 中字符对应的码点转换成一定长度的二进制序列，以便于计算机处理，这个对应关系被称为字符编码表，UTF-8、 UTF-16 属于这层概念；
- **Code Point**: 码点，简单理解就是字符的数字表示。一个字符集一般可以用一张或多张由多个行和多个列所构成的二维表来表示。二维表中行与列交叉的点称之为码点，每个码点分配一个唯一的编号，称之为码点值或码点编号，除开某些特殊区域 (比如代理区、专用区) 的非字符码点和保留码点，每个码点唯一对应于一个字符。
- **Code Unit**：代码单元，是指一个已编码的文本中具有最短的比特组合的单元。对于 UTF-8 来说，代码单元是 8 比特长；对于 UTF-16 来说，代码单元是 16 比特长。换一种说法就是 UTF-8 的是以一个字节为最小单位的，UTF-16 是以两个字节为最小单位的。
- **Code Space**：码点空间，字符集中所有码点的集合。

总结一下，关于字符集、字符编码，应用上述概念，可以被简单表示为：

> Characters — CCS -> Code Point — CEF -> Code Unit  


## Unicode 几种常见编码方式

Unicode 字符集中的字符可以有多种不同的编码方式，如 UTF-8、UTF-16、UTF-32、压缩转换等。这里的 UTF 是 Unicode transformation format 的缩写，即统一码转换格式，将 Unicode 编码空间中每个码点和字节序列进行一一映射的算法。下面详细介绍一下这三种常见的 Unicode 编码方式。

### UTF-8

UTF-8 是一种变长的编码方式，一般用 1~4 个字节序列来表示 Unicode 字符，也是目前应用最广泛的一种 Unicode 编码方式，但是它不是最早的 Unicode 编码方式，最早的 Unicode 编码方式是 UTF-16。UTF-8 编码算法有以下特点：

- 首字节码用来区分采用的编码字节数：如果首字节以 0 开头，表示单字节编码；如果首字节以 110 开头，表示双字节编码；如果首字节以 1110 开头，表示三字节编码，以此类推；
- 除了首字节码外，用 10 开头表示多字节编码的后续字节，图 3 列出 UTF-8 用 1 ~ 6 个字节所表示的编码方式和码点范围（实际上 1 ~ 4 个字节基本可以覆盖大部分 Unicode 码点）；
- 与 ASCII 编码方式完全兼容：U+0000 到 U+007F 范围内 (十进制为 0~127) 的 Unicode 码点值所对应的字符就是 ASCII 字符集中的字符，用一个字节表示，编码方式和 ASCII 编码一致；
- 无字节序，在 UFT-8 编码格式的文本中，如果添加了 BOM，则标示该文本是由 UTF-8 编码方式编码的，而不用来说明字节序。

在实际的解码过程中：

- 情况 1：读取到一个字节的首位为 0，表示这是一个单字节编码的 ASCII 字符；
- 情况 2：读取到一个字节的首位为 1，表示这是一个多字节编码的字符，如继续读到 1，则确定这是首字节，在继续读取直到遇到 0 为止，一共读取了几个 1，就表示在字符为几个字节的编码；
- 情况 3：当读取到一个字节的首位为 1，紧接着读取到一个 0，则该字节是多字节编码的后续字节。

UTF-8 编码具有一个很重要的「自同步」特性，其表示在传输过程中如果有字节序列丢失，并不会造成任何乱码现象，或者存在错误的字节序列也不会影响其他字节的正常读取。例如读取了一个 10xxxxxx 开头的字节，但是找不到首字节，就可以将这个后续字节丢弃，因为它没有意义，但是其他的编码方式，这种情况下就很可能读到一个完全不同或者错误的字符。

### UTF-16

UTF-16 是最早的 Unicode 字符集编码方式，在概述 UTF-16 之前，需要解释一下 USC-2 编码方式，他们有源远流长的关系，UTF-16 源于 UCS-2。UCS-2 将字符编号（同 Unicode 中的码点）直接映射为字符编码，亦即字符编号就是字符编码，中间没有经过特别的编码算法转换。

因为 16 位二进制表示的最大值为 0xFFFF，而对于增补平面中的码点（范围为 0x10000 ~ 0x10FFFF，十进制为 65536 ~ 1114111），两字节的 16 位二进制是无法表示的。为了解决这个问题，The Unicode Consortium 提出了通过代理机制来扩展原来的 UCS-2 编码方式，也就是 UTF-16。其编码算法如下：

- 基本多语言平面（BMP）中有效码点用固定两字节的 16 位代码单元为其编码，其数值等于相应的码点，桶 USC-2 的编码方式；
- 辅助多语言平面 1-16 中的有效码点采用代理对（surrogate pair）对其编码：用两个基本平面中未定义字符的码点合起来为增补平面中的码点编码，基本平面中这些用作 “代理” 的码点区域就被称之为 “ 代理区 (Surrogate Zone)”，其码点编号范围为 0xD800 ~ 0xDFFF (十进制 55296 ~ 57343)，共 2048 个码点。代理区的码点又被分为高代理码点和低代理码点，高代理码点的取值范围为 0xD800 ~ 0xDBFF，低代理码点的取值范围为 0xDC00 ~ 0xDFFF，高代理码点和低代理码点合起来就是代理对，刚好可以表示增补平面内的所有码点

因此 UTF-16 也是一种变长编码，其通过将超过 16 位的码点进行高低位拆分，通过增加上值下值前缀的方式将字符编码为 4 字节的内容。简单举个例：

对 Unicode U+10437 进行 UTF-16 编码（𐐷）:

- 0x10437 减去 0x10000, 结果为 0x00437, 二进制为 0000 0000 0100 0011 0111。
- 分割它的上 10 位值和下 10 位值（使用二进制）:0000000001 and 0000110111。
- 添加 0xD800 到上值，以形成高位：0xD800 + 0x0001 = 0xD801。
- 添加 0xDC00 到下值，以形成低位：0xDC00 + 0x0037 = 0xDC37。

![](https://imgs.kyangc.com/2019-02-24-D84612D7-E23B-44C3-9414-4CAD68D83748.png "UTF-16 编码过程")

可以看到，UTF-16 一方面使用变长码元序列的编码方式，相较于定长码元序列的 UTF-32 算法更复杂 (甚至比同样是变长码元序列的 UTF-8 也更为复杂，因为引入了独特的代理对这样的代理机制)；另一方面仍然占用过多字节，比如 ASCII 字符也同样需要占用两个字节，相较于 UTF-8 更浪费空间和带宽。

因此，UTF-16 在 Unicode 字符集的三大编码方式 (UTF-8、UTF-16、UTF-32) 中表现较为糟糕。它的存在是历史原因造成的，引起了很多混乱。不过由于其推出时间最早，已被应用于大量环境中，目前虽然不被推荐使用，但长期来看，作为程序人员都不得不与之打交道。

### UTF-32

相比前两种变长编码，采用 32 位进行编码的 UTF-32 能够完全覆盖 unicode 字符集而无需额外进行处理。相比于前两者，采用定长编码的 UTF-32 的寻址性能更强，但在纠错性和空间利用率上存在较大的缺陷，因此应用场景并不多。

### 关于字节序

字节顺序，又称端序或尾序（英语：Endianness），在计算机科学领域中，指存储器中或在数字通信链路中，组成多字节的字的字节的排列顺序。
对于单一的字节（a byte），大部分处理器以相同的顺序处理位元（bit），因此单字节的存放方法和传输方式一般相同。
对于多字节数据，如整数（32 位机中一般占 4 字节），在不同的处理器的存放方式主要有两种，字节的排列方式有两个通用规则。例如，一个多位的整数，按照存储地址从低到高排序的字节中，如果该整数的最低有效字节（类似于最低有效位）在最高有效字节的前面，则称小端序；反之则称大端序：

![](https://imgs.kyangc.com/2019-02-24-280px-Big-Endian.svg.png "大端序")

![](https://imgs.kyangc.com/2019-02-24-280px-Little-Endian.svg.png "小端序")

在网络传输一般采用大端序，也被称之为网络字节序或网络序。IP 协议中定义大端序为网络字节序。这里需要注意的是，在以太网传输中，虽然在字节层面上是高字节先传（大端序），但每一字节内却是低位先传（小端序）。
在字符完成编码进行传输的时候，对于 UTF-8 编码而言，由于每个处理单元仅包含一个字节，因此在接收时，可以不用调整字节序，按照接收字节顺序依次处理字节即可；而对于 UTF-16 或 UTF-32 这种每个处理单元包含 2 个或 4 个字节的编码，为了不因为字节序的问题导致解析混乱，需要在数据最开始时添加字节序标识（Byte Order Mark，BOM），0xFFEF 代表大端序 BE，0xFFFE 代表小端序 LE，那么在接收到数据的时候，接受者就能够根据标识来决定本机应该以什么方案来对接收到的字节流进行解析。
另外，虽然 UTF-8 不需要考虑字节序，但是不代表在 UTF-8 编码中就不能添加 BOM 了，实际上 BOM 始终是个可选项，而且在很多 Windows 系统下的文本编辑软件中，打开不带 BOM 头的文件会出现乱码的情况，这是因为在 UTF-8 编码下，BOM 实际上是被用于标识编码类型的：在 UTF-8 编码下，0xFFEF 会被转换成 0xEFBBBF 这样三个字节，软件就可以根据这三个字节来判断接下来的字节流是应用了哪种编码方案，进而完成字节流的解析。
用这张图来简单梳理下 BOM 在各个编码方式下的表现形式：

![](https://imgs.kyangc.com/2019-02-24-99C2A6D7-F2E9-4DFD-88D5-E33263F13741.png "BOM 类型和标识")

### 比较一下

看图吧：

![](https://imgs.kyangc.com/2019-02-24-F1D2EC0C-2BF8-4C24-BABC-473781324E0F.png "不同编码方式之间的优劣比较")


## Unicode 和 Java

身为一个程序员，在我们的日常工作中，编写运行代码、使用 char string 等数据结构处理字符串时都会不断地跟字符集和字符编码打交道，下面我们就来看看这里面都需要注意哪些问题：

### Java 和 Modified UTF-8 以及 UTF-16

先上图：

![](https://imgs.kyangc.com/2019-02-24-5N7Sf.jpg "Java->class->JVM")

解释一下。
假定我们有这么一段代码，保存在一个以 UTF-8 格式保存的 .java 文件中，经过指定 encode 为 UTF-8 的 javac 编译后，生成 .class 文件，此时这个文件采用的编码类型为 Modified UTF-8，这个编码格式跟正常的 UTF-8 有一些差异（[JVM规范](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4.7)），主要体现为两点：

- 第一，空字符（null character，U+0000）使用双字节的 0xc0 0x80，而不是单字节的 0x00。这保证了在已编码字符串中没有嵌入空字节。因为 C 语言等语言程序中，单字节空字符是用来标志字符串结尾的。当已编码字符串放到这样的语言中处理，一个嵌入的空字符将把字符串一刀两断。
- 第二，相比于标准 UTF-8 编码，MUTF-8 不再有 4 字节及以上的字符单元，转而使用特有的双三字节对的方式进行表示，在这种情况下，以 BE 顺序存储。

JVM 读取 .class 文件后，抛开纯字节操作的内容不谈，在涉及到字符串操作的代码中，考虑到 String 对象的底层也是 char 数组，而 char 在 JVM 中都是以 UTF-16 格式存储的 2 字节无符号整数（均指向 BMP 中的符号），因此基本所有涉及到的字符串操作，其内部存储的都是 UTF-16，也即涉及到了一次编码转换。

为什么会有这么一次转换呢？搜集了一下网上的一些看法，大概总结下：

- .class 之所以采用 UTF-8 编码，是因为在代码构成大部分为 ASCII 码（英文）范围内容时，每个符号只占用一个字节，相比于其他编码方式更加省空间
- 而在虚拟机中，之所以规定 char 存储 UTF-16 格式的字符，有一部分历史原因在于，最早的 UTF-16 编码也即 USC-2 编码，符号仅仅局限在 BMP 平面上，也即可以认为是定长编码，这在虚拟机中有着良好的寻址性能，只是随着 Unicode 的发展，渐渐增补的符号数不断增加，最终让变长的 UTF-16 取代定长的 USC-2 编码

### Java 中的 String.length() 和 String.getBytes()

这段代码的结果是？

```java
public static void main (String[] args) {
	String s = "😊";
	System.out.println(s.length());
}
```

对，length = 2。
String 的 length() 方法其实是返回的是底层 char[] 数组的长度，如前所述，emoji 表情大多数不在 BMP 空间中，也即需要 4 个字节，也即 2 个 char 来存储，因此长度为 2。

接着，这段代码的结果是？

```java
public static void main (String[] args) {
	String s = "1";
	System.out.println(s.getBytes().length);
	System.out.println(s.getBytes("UTF-16").length);
	System.out.println(s.getBytes("UTF-16LE").length);
}
```

分别是 1，4，2。
在这里，String.getBytes() 在不传入 format 的情况下，默认会使用系统默认的编码方式，在 Android 中，默认是 UTF-8，这里转换得到的就是一字节的编码。
如果指定转换格式为 UTF-16，但是不指定大小端，那么会默认以 UTF-16BE 的方式进行编码，前两字节是 BOM 0xFEFF，然后才是 1 的编码。
如果转换格式指定以小端方式进行，那么转换结果不带 BOM，就是 2 字节。