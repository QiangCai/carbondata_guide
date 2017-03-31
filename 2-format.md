2 **format模块**

2.1 **文件目录结构**

CarbonData数据存储在carbon.storelocation配置项指定的位置（在carbon.properties中配置；若未配置则默认为../carbon.store）。

文件目录结构如图2-1_1所示：

<img src="media/2-1_1.png" width = "60%" alt="2-1_1" />

1.  carbon.store目录下有ModifiedTime.mdt文件和database目录（默认database名：default）。

2.  default目录下有属于该database的用户表目录（例如：user\_table）。

3.  user_table目录下有Metadata目录和Fact目录；创建表时，在Metadata目录下生成schema文件，记录了表结构；其他文件均在dataloading时生成。其中每个字典编码的列对应生成dict,dictmeta和sortindex三个文件；每一次批量loading生成一个新的segment目录,其中每个task生成一个carbonindx和多个carbondata文件。

2.2 **Schema文件格式**

schema文件内容如图2-2_1中TableInfo类所示:

<img src="media/2-2_1.png" width = "60%"alt="2-2_1" />


1.  TableSchema类 表名由schema文件所在的表目录决定。
    tableProperties用来记录table相关的属性， 例如；table\_blocksize。

2.  ColumnSchema类 encoders用来记录column存储时采用的编码
    columnProperties用来记录column相关的属性。

3.  BucketingInfo类 创建bucket表时可以指定表的bucket数量以及column来split
    buckets。

4.  DataType类 描述了CarbonData支持的数据类型。

5.  Encoding类 CarbonData文件可能用到的几种编码。

2.3 **carbondata文件格式**

1 **blocklet部分**

carbondata file
由多个blocklet和一个footer部分组成。blocklet是carbondata文件内部的数据集（默认配置是32万行）。
carbondata文件目前有如下3个版本： V1: blocket由所有column的data page, RLE page,
rowID
page组成。每个column的这3部分数据是在blocklet内分散存储的，在footer部分需要记录全部page的offset和length信息。

<img src="media/2-3_1.png" width = "25%" alt="2-3_1" />

V2:
blocklet由ColumnChunk2组成。ColumnChunk2将column的3部分数据聚集在了一起，可以使用更少的reader去完成column数据读取。由于DataChunk2记录page的长度信息，因此，footer只需要记录ColumnChunk的offset和length，减小了footer数据量，有利于缩短元数据加载时间，提升first query性能。

<img src="media/2-3_2.png" width = "50%" alt="2-3_2" />

V3:
blocklet由ColumnChunk3组成。ColumnChunk3默认由10个ColumnChunk2组成。在blocklet内ColumnChunk3包含了每个column的全部page。ColumnChunk2新增加了BlockletMinMaxIndex，同时footer只需记录ColumnChunk3级别BlockletMinMaxIndex。进一步缩小footer而且减小了page的数据行数，使索引更精确的命中page。

<img src="media/2-3_3.png" width = "50%"alt="2-3_3" />

2 **footer部分**

footer记录每个carbondata
file内部全部blocklet数据分布信息以及统计相关的元数据信息(minmax,startkey/endkey)。

<img src="media/2-3_4.png" width = "80%" alt="2-3_4" />

1.  BlockletInfo3用来记录全部ColumnChunk3的offset 和length。

2.  SegmentInfo用来记录column数量以及每个column的cardinality。

3.  BlockletIndex包括BlockletMinMaxIndex和BlockletBTreeIndex。BlockletBTreeIndex用来记录block内所有blocklet的startkey/endkey;查询时通过过滤条件结合mdkey生成查询的startkey/endkey，借助BlocketBtreeIndex可以圈定每个block内满足条件的blocklet范围。BlockletMinMaxIndex用来记录blocklet内所有列的min/max值;查询时通过对过滤条件使用min/max检查，可以跳过不满足条件的block/blocklet。

2.4 **carbonindex文件格式**

抽取出“2.3
footer部分”中BlockletIndex部分生成carbonindex文件。Dataloading时，一个node启一个task，每个task生成一个carbonindex文件。carbonindex文件记录segment内每个task生成的所有carbondata文件内部所有blocklet的index信息。BlockIndex记录carbondata
filename，footer offset和BlockletIndex,
数据量要比footer少些,查询时直接使用该文件构建driver侧索引，而无需跳扫数据量更大的footer部分。

<img src="media/2-4_1.png" width = "25%" alt="2-4_1" />

2.5 **dictionary 文件格式**

1.  dict file记录某列的distinct value
    list。首次dataloading时，使用所有的distinct value生成该文件;
    后续采用追加的方式。在DataConvertStep，字典编码列会使用key替换origin value。

<img src="media/2-5_1.png" width = "25%" alt="2-5_1" />

	
2.  dictmeta记录每次dataloading的新增distinct value的元数据描述,字典缓存使用该信息增量刷新缓存。

<img src="media/2-5_2.png" width = "25%" alt="2-5_2" />
	
3.  sortindex记录字典编码按照origin value排序后的key的结果集。基于字典列的过滤查询，需要将value的过滤条等价转换为key的过滤条件，使用sortindex文件可以快速构建有序的value序列，以便快速查找value对应的key值。dataLoading时，如果有新增的字典值，会重新生成sortindex文件。

<img src="media/2-5_3.png" width = "30%" alt="2-5_3" />

2.6 <div id="2_6">**tablestatus 文件格式**</div>
tablestatus记录每次加载和合并的segment相关的信息（采用gson格式），包括加载时间,加载状态,segment名称，是否被删除以及合并入的segment名称等等。每次加载或合并完成后，重新生成tablestatusfile。

<img src="media/2-6_1.png" width = "25%" alt="2-6_1" />
