---
title: NES 模拟器实现指南（零）
subtitle: 简介，以及 iNES 文件格式
date: 2018-02-25 21:30:01
tags:
    - NES
    - Go 语言
categories:
    - NES
---

## 简介

Family Computer （ファミリーコンピュータ），缩写为 Famicom （ファミコン），是日本任天堂公司推出的一种第一代家用游戏主机，在国内常被称为红白机。红白机有两种，一种是日本版，体积较小，机身以红色和白色为主，俗称“红白机”；另一种是欧美版，体积较大，机身以灰色为主，称为 Nintendo Entertainment System，简称NES。两套机器的主要差别是支持的视频制式不一致，以及卡带的形状不同。在上世纪八十年代，红白机曾是世界上使用最广泛的游戏终端。自其从 1983 年发布至 1993 年停止维护，红白机将电子游戏带入各家各户，并推动了电子游戏最初的发展。

尽管自红白机以来，科技已经进步了不少，我们能使用最新的技术制作出足以以假乱真的游戏画面，能够利用相当于当初 FC 卡带几百万倍的存储空间来存储游戏内容。但是，那个时代的 FC 游戏依然以其卓越的可玩性吸引着各个年龄的玩家。超级马里奥兄弟，洛克人，魂斗罗仍然是难以逾越的经典之作。

当然，现代的操作系统以及硬件已经无法直接运行 FC 游戏。不过好在我们可以通过使用软件模拟 NES 主机的硬件来让游戏运行在现在的电脑上。这类软件便被称为**模拟器**。现有的 NES 模拟器中较为著名的有全平台的 [FCEUX](http://fceux.com/web/home.html)，Android 上的 Nesoid，以及 Windows 专供的 [VirtuaNES](http://virtuanes.s1.xrea.com/)。笔者一直使用的便是 VirtualNES。本文的目的便是实现一个简单的 NES 模拟器。笔者在演示时使用的语言为 Go 语言，当然读者若是想自行实现的话可以使用其擅长的任意语言进行。

## 元始，上帝曰：宜读 iNES 文件。

iNES 文件（拓展名 .nes，大小写均可）是 NES 游戏分发的事实标准。该文件标准的最初是由 [Marat Fayzullin](http://fms.komkon.org/) 为其模拟器 [iNES](http://fms.komkon.org/iNES/) 而开发的文件格式。要实现一个 NES 模拟器，我们要做的第一步就是读取 iNES 文件，并将之映射到内存中以备使用。

我们首先要做的是创建 NES 文件的文件头结构体。NES 文件的前 16 个字节是文件头。其中：

1. 第 0 ~ 3 个字节指定了文件的格式，必须为：

        0 = 0x4E (N)
        1 = 0x45 (E)
        2 = 0x53 (S)
        3 = 0x1A (^Z)
    
模拟器依靠这个确定文件的格式。

2. 第 4 个字节指定了 PRG（程序） ROM 块的个数，PRG ROM 块每个大小为 16KB

3. 第 5 个字节指定了 CHR（图块） ROM 块的个数，CHR ROM 块每个大小为 8 KB

4. 第 6 个字节为指定卡带属性的字节。各个比特位的含义如下：

         0   -> Mirror Type ( 1 为水平， 0 为垂直)
         1   -> 是否存在 battery-backed RAM ( 1 则为存在，映射到 $6000-$7FFF)
         2   -> 是否存在 trainer (同上，映射到 $7000-$71FF)
         3   -> 是否存在 VRAM
         4-7 -> Mapper Type 的低四位

5.  第 7 个字节还是指定卡带属性的字节。各个比特位的含义如下：

        *0    -> 卡带是否含有 VS-System
        *1-3  -> 保留，但必须全为 0
         4-7  -> Mapper Type 的高四位

6. 第 8 个字节指定了 RAM 块的个数，每块为 8KB，如果为 0 ，则假设只有一个 RAM 块。

7. *第 9 个字节指定了视频制式，如果其第 0 个比特值为 0，则为 PAL，否则为 NTSC 制式。

8. 第 10-15 字节为保留区域，必须为 0


在上文中，暂不需要读取的区段笔者已经使用星号（*）标出。出现的词汇的含义会在以后的文章中逐步介绍。根据上文信息，现在我们即可写出文件头的结构体。如下：

``` go

const NESMagicMumber = 0x1a53454e //"NES^Z"

type NESFileHeader struct {
	MagicNumber		uint32	// NES Magic Number,must be 0x1a53454e
	PRGNum			byte	// PRG-ROM banks number
	CHRNum			byte	// CHR-ROM banks number
	Ctrl1			byte	// Control
	Ctrl2			byte	// Control too
	RAMNum			byte	// RAM number (8KB each)
	_				[7]byte // Empty bytes. Not used at this tume but MUST BE ALL ZEROS or games will not work.
}

```

并写出相应的读取文件头片段：

``` go

file,err := os.Open(path)

	if err != nil {
		return nil,err
	}

	defer file.Close()

	header := NESFileHeader{}

	// Read header

	if err := binary.Read(file,binary.LittleEndian,&header) ; err != nil {
		return nil, err
	}

	if header.MagicNumber != NESMagicMumber {
		return nil , errors.New("Magic Number is Wrong.Invilid iNES file.")
	}

```

处理完文件之后我们需要一个暂时的 NES 卡带结构来将我们读取到的内容存储到内存中，我们只需要写出来目前需要读取的部分即可：

``` go

type Cartridge struct {
	PRG     []byte
	CHR     []byte
	Mapper  int
	Mirror  int
	Battery bool
}

```

之后便是按照上面的说明读取各个变量

``` go

    // mapper type
	mapper1 := header.Control1 >> 4
	mapper2 := header.Control2 >> 4
	mapper := mapper1 | mapper2<<4

	// mirroring type
	mirror1 := header.Control1 & 1
	mirror2 := (header.Control1 >> 3) & 1
	mirror := mirror1 | mirror2<<1

	// battery-backed RAM
	battery := (header.Control1 >> 1) & 1

```

以及计算各个 ROM 块的个数，分配空间


``` go

	// read trainer if present (unused)
	if header.Control1&4 == 4 {
		trainer := make([]byte, 512)
		if _, err := io.ReadFull(file, trainer); err != nil {
			return nil, err
		}
	}

	// read prg-rom bank(s)
	prg := make([]byte, int(header.NumPRG)*16384)
	if _, err := io.ReadFull(file, prg); err != nil {
		return nil, err
	}

	// read chr-rom bank(s)
	chr := make([]byte, int(header.NumCHR)*8192)
	if _, err := io.ReadFull(file, chr); err != nil {
		return nil, err
	}

	// provide chr-rom/ram if not in file
	if header.NumCHR == 0 {
		chr = make([]byte, 8192)
	}

```

最后将这些代码片段一封装为函数，即可完成读取 iNES 文件这一操作。十分简单。

本段的全部代码可以在[这里](https://github.com/kuso-kodo/kuso-NES/tree/3a0f0fdc2ab83cdda2514fb72f9c0dbf8e52a455)找到。

Reference:

1. [iNES Header/Format Information File](http://nesdev.com/neshdr20.txt)

2. [Marat Fayzullin](http://fms.komkon.org/)

3. [Wikipedia: Nintendo Entertainment System](https://en.wikipedia.org/wiki/Nintendo_Entertainment_System)