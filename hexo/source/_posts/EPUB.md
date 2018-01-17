---
title: EPUB
date: 2018年1月17日15:26:42
---
# EPUB
## 什么是EPUB？
e-pub的意思是电子出版，是IDPF制定的一个标准，这个标准是基于xml格式的电子书或者其它数字出版物。

.epub文件其实就是一个简单的zip文件可以使用winrar或者linux使用归档管理器打开来看目录结构和基本文档。内容都是xml格式的。

## epub文件的目录结构和基本文档
#### 1. mimetype文件
这个文件十分简单，里面只有

	application/epub+zip
    
 但是这个文件必须作为zip文件中的第一个文件，并且不能被压缩，必须在根目录下。同时这个文件不能有回车和换行。
#### 2. META-INFO
META-INFO文件夹跟mimetype一样，也是必须存在的文件夹，也必须在根目录下，在文件夹下有一个container.xml文件.epub阅读系统会首先查看该文件，他指向文件图书的源数字文件的位置。这个container.xml格式如下

	<?xml version="1.0"?>
    <container version="1.0" xmlns="urn:oasis:names:tc:opendocument:xmlns:container">
    <rootfiles>
    <rootfile full-path="OEBPS/content.opf"
    media-type="application/oebps-package+xml" />
    </rootfiles>
    </container>

> mimetype 和 container 是 EPUB 档案中仅有的两个需要严格限制位置的文件。建议将其他文件保存到 EPUB 的子目录下（通常被称为OEBPS但不是必须的）。

#### 3. 元数据文件(.opf packaging format)
META-INFO文件夹下的container.xml所指定的rootfile的文件。该文件文件名没有特殊要求，扩展名为.opf。他指定了图书中所有内容的位置，如文本和多媒体目录。另外他还会给出一个元数据文件，它是Navigation Center eXtended (NCX) 表，有的称其为逻辑目录，opf内容示例如下所示:
	
    <?xml version='1.0' encoding='utf-8'?>
    <package xmlns="http://www.idpf.org/2007/opf"
    xmlns:dc="http://purl.org/dc/elements/1.1/"
    unique-identifier="bookid" version="2.0">
    <metadata>
    <dc:title>Hello World: My First EPUB</dc:title>
    <dc:creator>My Name</dc:creator>
    <dc:identifier id="bookid">urn:uuid:12345</dc:identifier>
    <meta name="cover" content="cover-image" />
    </metadata>
    <manifest>
    <item id="ncx" href="toc.ncx" media-type="text/xml"/>
    <item id="cover" href="title.html" media-type="application/xhtml+xml"/>
    <item id="content" href="content.html" media-type="application/xhtml+xml"/>
    <item id="cover-image" href="images/cover.png" media-type="image/png"/>
    <item id="css" href="stylesheet.css" media-type="text/css"/>
    </manifest>
    <spine toc="ncx">
    <itemref idref="cover" linear="no"/>
    <itemref idref="content"/>
    </spine>
    <guide>
    <reference href="cover.html" type="cover" title="Cover"/>
    </guide>
    </package>

##### 3.1 命名空间 <package>
文档本身必须使用命名空间 xmlns="http://www.idpf.org/2007/opf",
metadata也可以在这里指定命名空间,元数据一般使用 Dublin Core Metadata Initiative (DCMI) 
xmlns="http://purl.org/dc/elements/1.1/"
##### 3.2 源数据 <metadata>
 Dublin Core Metadata Initiative (DCMI)定义了一组常见的元数据，可以用来描述各种不同的数字资料，下边是元数据摘要：
	
    ...
    <metadata>
    <dc:title>Hello World: My First EPUB</dc:title>
    <dc:creator>My Name</dc:creator>
    <dc:identifier id="bookid">urn:uuid:12345</dc:identifier>
    <meta name="cover" content="cover-image" />
    </metadata>
    ...
    
有两个术语是必须的，分别是title和identifier。这个identifier必须是唯一的，对于图书出版商来说这个字段一般包括ISBN编号，可以使用URL或者很大的随机生成的uuid。
> 属性unique-identifier的值必须和dc:identifier元素的id属性匹配

epub规范没有要求包含name的属性值为cover的meta元素，但是为了增加封面和图像的可移植性，一般建议加上。meta元素的content属性的值对应在这个opf文件manifest元素的id号，
##### 3.3 清单数据 <manifest>
opf文件中的manifest节点列出了epub的所有资源，通常是组成电子书的一组xhtml文件和相关的媒体文件。epub鼓励使用css设定图书内容的样式，因此manifest中也包含css。进入数字图书的所有内容必须在manifest中列出。
下边是manifest示例：

	...
    <manifest>
    <item id="ncx" href="toc.ncx" media-type="text/xml"/>
    <item id="cover" href="title.html" media-type="application/xhtml+xml"/>
    <item id="content" href="content.html" media-type="application/xhtml+xml"/>
    <item id="cover-image" href="images/cover.png" media-type="image/png"/>
    <item id="css" href="stylesheet.css" media-type="text/css"/>
    </manifest>
    
所有的想必须有对应的media-type

epub支持三种核心图像文件类型:JPEG,PNG,GIF和Scalable Vector Graphics(SVG矢量图)

##### 3.4 spine
spine的意思是脊柱，作用是线性阅读顺序，可以将OPF spine看做书中的页面的顺序，按照文档顺序从上到下一次读取spine，下边是spine示例：

	... 
    <spine toc="ncx"> 
    <itemref idref="cover" linear="no"/> 
    <itemref idref="content"/> 
    </spine> 
    ... 
    
每个itemref元素都需要一个idref属性，并且和manifest中的id想匹配。toc属性也是必须的，他引用manifest中表示ncx表文件名的id。

spine中的linear属性表明该项作为线性阅读顺序的一部分，还是和先后顺序无关。建议将封面定义为no。符合epub规范的阅读系统将首次打开spine中设置linear=no的第一项。
##### 3.5 guide
这个是可选的，最好保留，可以为epub阅读系统提供语义功能。下边是示例:
	
    ...
    <guide>
    <reference href="cover.html" type="cover" title="Cover"/>
    </guide>
    ...
    
> 上边就是opf文件中的所有内容，manifest文件定义epub所有物理资源，spine定义这些资源的顺序信息，guide负责解释这些部分的含义。

#### 4. NCX(Navigation Control file for XML applications)
ncx定义了数字图书的目录表，在复杂的图书中，目录表通常采用层次结构，包括嵌套的内容，章节等信息。
下边是简单的toc.ncx文件内容:
	
    <?xml version='1.0' encoding='utf-8'?>
    <!DOCTYPE ncx PUBLIC "-//NISO//DTD ncx 2005-1//EN"
    "http://www.daisy.org/z3986/2005/ncx-2005-1.dtd">
    <ncx xmlns="http://www.daisy.org/z3986/2005/ncx/" version="2005-1">
    <head>
    <meta name="dtb:uid" content="urn:uuid:12345"/>
    <meta name="dtb:depth" content="1"/>
    <meta name="dtb:totalPageCount" content="0"/>
    <meta name="dtb:maxPageNumber" content="0"/>
    </head>
    <docTitle>
    <text>Hello World: My First EPUB</text>
    </docTitle>
    <navMap>
    <navPoint id="navpoint-1" playOrder="1">
    <navLabel>
    <text>Book cover</text>
    </navLabel>
    <content src="title.html"/>
    </navPoint>
    <navPoint id="navpoint-2" playOrder="2">
    <navLabel>
    <text>Contents</text>
    </navLabel>
    <content src="content.html"/>
    </navPoint>
    </navMap>
    </ncx>

EPUB使用的DAISY的NCX DTD的命名空间
xmlns="http://www.daisy.org/z3986/2005/ncx/" 

##### 4.1 head
DTD要求ncx文件的head元素包括四个meta元素:

* dtd:uid // 数字图书的唯一id，和opf文件中的dc:identifier对应
* dtd:depth // 目录中层次的深度
* dtd:totalPageCount // 仅用于纸质图书，保留0即可
* dtd:maxPageNumber // 仅用于纸质图书，保留0即可

##### 4.2 docTitle
是图书的标题，和opf文件中的dc:title对应

##### 4.3 navMap
navMap是ncx文件中最重要的部分，定义了电子书的目录。navmap中包括一个或者多个navPoint元素，每个navPoint都要包括下列元素：
- playOrder属性 // 说明文档的阅读顺序，和opf文件中spine中itemref元素的顺序相同
- navLabel/text元素 // 给出章节的标题通常是章的标题或者数字，如"第一章"
- content元素 // 他的src属性指向这些内容的物理资源。
- 还可以有一个或者多个navPoint元素 // 表示层次结构

#### 5. 问题

##### 5.1 NCX和OPF文件元数据交叉
由于 NCX 源自其他标准，使用 NCX 编码的信息和 OPF 内容之间存在重复。如果通过程序生成 EPUB，这算不上什么问题，因为同样的代码可输出到两个文件中。两个位置的信息要一致，不同的 EPUB 读者可能使用不同位置的值。（可如果手工去编写呢？）

##### 5.2 NCX和OPF spine有什么不同
opf spine主要用来描述电子书章节的顺序，比如第一章后第二章再后第三章 ...

ncx文件描述电子书的目录结构。

一条法则就是ncx包括的节点比spine多。实际上spine的所有项都会出现在ncx中，但是ncx可能更详细。
