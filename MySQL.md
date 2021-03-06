## 2022年1月24日

------

- SQL分类
  - **DDL**（Data Definition Languages、数据定义语言），这些语句定义了不同的数据库、表、视图、索
    引等数据库对象，还可以用来创建、删除、修改数据库和数据表的结构。
    - 主要的语句关键字包括 CREATE 、 DROP 、 ALTER 等。
  - **DML**（Data Manipulation Language、数据操作语言），用于添加、删除、更新和查询数据库记
    录，并检查数据完整性。
    - 主要的语句关键字包括 INSERT 、 DELETE 、 UPDATE 、 SELECT 等。SELECT是SQL语言的基础，最为重要。
  - **DCL**（Data Control Language、数据控制语言），用于定义数据库、表、字段、用户的访问权限和
    安全级别。
    - 主要的语句关键字包括 GRANT 、 REVOKE 、 COMMIT 、 ROLLBACK 、 SAVEPOINT 等。
    
      

## 2022年1月26日

- 起列的别名方式：
  - 空格：SELECT employee_id emp_id FROM employees; 
  - as: SELECT employee_id as emp_id FROM employees; 
  - 加双引号: SELECT employee_id "emp_id FROM" employees; 

## 2022年2月1日

- 外连接

  - 左外连接
    - #实现查询结果是A
      SELECT 字段列表
      FROM A表 LEFT JOIN B表
      ON 关联条件
      WHERE 等其他子句;

  - 右外连接
    - FROM A表 RIGHT JOIN B表
      ON 关联条件
      WHERE 等其他子句;
  - 满外连接(FULL OUTER JOIN)
    满外连接的结果 = 左右表匹配的数据 + 左表没有匹配到的数据 + 右表没有匹配到的数据。
    SQL99是支持满外连接的。使用FULL JOIN 或 FULL OUTER JOIN来实现。
    需要注意的是，MySQL不支持FULL JOIN，但是可以用 LEFT JOIN UNION RIGHT join代替。

- UNION
  - 合并查询结果 利用UNION关键字，可以给出多条SELECT语句，并将它们的结果组合成单个结果集。合并
    时，两个表对应的列数和数据类型必须相同，并且相互对应。各个SELECT语句之间使用UNION或UNION
    ALL关键字分隔。
  - 语法格式：
    - SELECT column,... FROM table1
      UNION [ALL]
      SELECT column,... FROM table2

## 2022年2月3日

------

- 语句顺序:

  - SELECT...

    FROM.. . LEFT / RIGHT  JOIN  ... ON... 多表的连接条件

    WHERE..不包含聚合函数的过滤条件

    GROUP BY...分组字段

    HAVING ... 包含聚合函数的过滤条件

    ORDER BY... (ACS / DESC)

    LIMIT... , ...

## **2022年2月8日**

------

- 笔记地址1	 https://blog.csdn.net/m0_37989980/article/details/103413942
- 笔记地址2     https://www.cnblogs.com/-wenli/p/13671386.html
- MySQL架构：连接层、服务层、引擎层
- MySQL的SQL执行流程
  - <img src="C:\Users\博子\AppData\Roaming\Typora\typora-user-images\image-20220208223535599.png" alt="image-20220208223535599" style="zoom:80%;" />
  - 第一步，客户端通过**连接器**与数据库建立连接，获取权限
  - 第二步，建立连接后，执行查询语句前会**查询缓存** ，如果缓存中存在该查询语句以及查询结果，那么会将结果返回给客户端（MySQL默认不查询缓存，可修改参数设置为按需使用）建议在更新较少的**静态表**中使用查询缓存。
  - 如果查询缓存没有命中，则由**解析器**对查询语句进行**词法分析**（识别字符串）和**语法分析**（判断语句是否满足MySQL语法），如果不正确将报错。
  - 第三步，经过解析器后就知道SQL的含义了，在开始执行前还要经过**优化器**的处理，优化器会选择最优的**执行计划**
    - 查询优化分为**逻辑查询优化**和**物理查询优化**
    - 物理查询优化通过**索引**和**表连接方式**等技术来进行优化
    - 逻辑查询优化是通过**SQL等价交换**提升查询效率，也就是换一种查询效率更高的写法
  - 第四步，**执行器**判断该用户是否具有权限，如果有，执行器会调用**存储引擎**API对表进行读写
  - 总结：SQL语句在MySQL中的执行流程：SQL语句 → 查询缓存 → 解析器 → 优化器 → 执行器

## **2022年2月9日**

------

- 笔记地址    https://www.cnblogs.com/aluna/p/15846128.html

- **数据库缓冲池（buffer pool)**

  - **InnoDB**存储引擎是以页为单位来管理存储空间的，我们进行的增删改查操作其实本质上都是在访问页面（包括读页面、写页面、创建新页面等操作)。而磁盘I/O需要消耗的时间很多，而在内存中进行操作，效率则会高很多，为了能让数据表或者索引中的数据随时被我们所用，DBMS会申请**占用内存来作为数据缓冲池**，在真正访问页面之前，需要把在磁盘上的页缓存到内存中的**Buffer Pool**之后才可以访问。
    这样做的好处是可以让磁盘活动最小化，从而**减少与磁盘直接进行I/0 的时间**。要知道，这种策略对提升sQL语句的查询性能来说至关重要。如果索引的数据在缓冲池里，那么访问的成本就会降低很多。

- **缓存池的重要性：**

  对于使用**InnoDB**作为存储引擎的表来说，不管是用于存储用户数据的索引（包括聚簇索引和二级索引)，还是各种系统数据，都是以**页（每页大小为4k）**的形式存放在**表空间**中的，而所谓的表空间只不过是InnoDB对文件系统上一个或几个实际文件的抽象，也就是说我们的数据说到底还是存储在磁盘上的。但是各位也都知道，磁盘的速度慢的跟乌龟一样，怎么能配得上**“快如风，疾如电"的CPU**呢?这里，缓冲池可以帮助我们消除cPu和磁盘之间的**鸿沟**。所以InnoDB存储引擎在处理客户端的请求时，当需要访问某个页的数据时，就会把**完整的页的数据全部加载到内存**中,也就是说即使我们只需要访问一个页的一条记录，那也需要先把整个页的数据加载到内存中。将整个页加载到内存中后就可以进行读写访问了，在进行完读写访问之后并不着急把该页对应的内存空间释放掉，而是将其**缓存**起来，这样将来有请求再次访问该页面时，就可以省去**磁盘IO**的开销了。

- **InnoDB的缓冲池缓存什么？有什么用？**

  - 缓存表数据与索引数据，把磁盘上的数据加载到缓冲池，避免每次访问都进行磁盘IO，起到加速访问的作用

- **缓冲原则**：会优先对使用频次高的热数据进行加载

- **缓冲池的预读特性**

  - 缓冲池的作用就是提升I/O效率，而我们进行读取数据的时候存在一个“局部性原理”，也就是说我们使用了一些数据，`大概率还会使用它周围的一些数据`，因此采用“预读"的机制提前加载，可以减少未来可能的磁盘Ⅳ/O操作。

- **缓冲池如何读取数据**

  - 缓冲池管理器会尽量将经常使用的数据保存起来，在数据库进行页面读操作的时候，首先会判断该页面是否在缓冲池中，如果存在就直接读取，如果不存在，就会通过内存或磁盘将页面存放到缓冲池中再进行读取

- **多个Buffer Pool实例**

  - Buffer Pool本质是InnoDB向操作系统申请的一块`连续的内存空间`，在多线程环境下，访问Buffer Pool中的数据都需要`加锁`处理。在Buffer Pool特别大而且多线程并发访问特别高的情况下，单一的Buffer Pool可能会影响请求的处理速度。所以在Buffer Pool特别大的时候，我们可以把它们`拆分成若干个小的Buffer Pool`，每个Buffer Pool都称为一个`实例`，它们都是独立的，独立的去申请内存空间，独立的管理各种链表。所以在多线程并发访问时并不会相互影响，从而提高并发处理能力。

- 存储引擎：指表的类型（以前叫表处理器），其功能是接收上层传来的指令，然后对表中的数据进行提取或写入操作

- **InnoDB引擎和MyISAM引擎（重点）**

  ```
  InnoDB引擎
          介绍：InnoDB引擎是MySQL数据库的另一个重要的存储引擎，正称为目前MySQL AB所发行新版的标准，被包含在所有二进制安装包里。和其他的存储引擎相比，InnoDB引擎的优点是支持兼容ACID的事务(类似于PostGreSQL)，以及参数完整性(即对外键的支持)。Oracle公司与2005年10月收购了Innobase。Innobase采用双认证授权。它使用GNU发行，也允许其他想将InnoDB结合到商业软件的团体获得授权。
  
  InnoDB引擎特点：
          1.支持事务：支持4个事务隔离界别，支持多版本读。
          2.行级锁定(更新时一般是锁定当前行)：通过索引实现，全表扫描仍然会是表锁，注意间隙锁的影响。
          3.读写阻塞与事务隔离级别相关(有多个级别，这就不介绍啦~)。
          4.具体非常高效的缓存特性：能缓存索引，也能缓存数据。
          5.整个表和主键与Cluster方式存储，组成一颗平衡树。(了解)
          6.所有SecondaryIndex都会保存主键信息。(了解)
          7.支持分区，表空间，类似oracle数据库。
          8.支持外键约束，不支持全文索引(5.5之前)，以后的都支持了。
          9.和MyISAM引擎比较，InnoDB对硬件资源要求还是比较高的。
          
          小结：三个重要功能：Supports transactions，row-level locking，and foreign keys
  ```

  ```
  InnoDB引擎适用的生产业务场景
          1.需要事务支持(具有较好的事务特性，例银行业务)
          2.行级锁定对高并发有很好的适应能力，但需要确保查询是通过索引完成。
          3.数据更新较为频繁的场景，如：BBS(论坛)、SNS(社交平台)、微博等
          4.数据一致性要求较高的业务，例如：充值转账，银行卡转账。
          5.硬件设备内存较大，可以利用InnoDB较好的缓存能力来提高内存利用率，尽可能减少磁盘IO，可以通过一些参数来设置，这个就不细讲啦~~~
          6.相比MyISAM引擎，Innodb引擎更消耗资源，速度没有MyISAM引擎快
  ```

  ```
  InnoDB引擎调优精要
          1.主键尽可能小，避免给Secondery index带来过大的空间负担。
          2.避免全表扫描，因为会使用表锁。
          3.尽可能缓存所有的索引和数据，提高响应速度，较少磁盘IO消耗。
          4.在大批量小插入的时候，尽量自己控制事务而不要使用autocommit自动提交，有开关可以控制提交方式。
          5合理设置innodb_flush_log_at_trx_commit参数值，不要过度追求安全性。
          如果innodb_flush_log_at_trx_commit的值为0，log buffer每秒就会被刷写日志文件到磁盘，提交事务的时候不做任何操作。
          6.避免主键更新，因为这会带来大量的数据移动。
  ```

  ```
  	InnoDB 存储引擎将数据放在一个逻辑的表空间中,这个表空间就像黑盒一样由 InnoDB存储引擎自身来管理。从 MySQL4.1(包括 4.1)版本开始,可以将每个InnoDB存储引擎的表单独存放到一个独立的 ibd 文件中。此外,InnoDB 存储引擎支持将裸设备(row disk)用于建立其表空间。
  	InnoDB 通过使用多版本并发控制(MVCC)来获得高并发性,并且实现了 SQL 标准 的 4 种隔离级别,默认为 REPEATABLE 级别,同时使用一种称为 netx-key locking 的策略来避免幻读(phantom)现象的产生。除此之外,InnoDB存储引擎还提供了插入缓冲(insert buffer)、二次写(double write)、自适应哈希索引(adaptive hash index)、预读(read ahead) 等高性能和高可用的功能。
  	对于表中数据的存储,InnoDB 存储引擎采用了聚集(clustered)的方式,每张表都是按 主键的顺序进行存储的,如果没有显式地在表定义时指定主键,InnoDB 存储引擎会为每一行生成一个6字节的 ROWID,并以此作为主键。
  	InnoDB 存储引擎是 MySQL 数据库最为常用的一种引擎,Facebook、Google、Yahoo 等 公司的成功应用已经证明了 InnoDB 存储引擎具备高可用性、高性能以及高可扩展性。对其 底层实现的掌握和理解也需要时间和技术的积累。
  ```

- MyISAM

- ```
  MyISAM引擎特点：
          1.不支持事务
              事务是指逻辑上的一组操作，组成这组操作的各个单元，要么全成功要么全失败。
          2.表级锁定
              数据更新时锁定整个表：其锁定机制是表级锁定，也就是对表中的一个数据进行操作都会将这个表锁定，其他人不能操作这个表，这虽然可以让锁定的实现成本很小但是也同时大大降低了其并发性能。
          3.读写互相阻塞
              不仅会在写入的时候阻塞读取，MyISAM还会再读取的时候阻塞写入，但读本身并不会阻塞另外的读。
          4.只会缓存索引
              MyISAM可以通过key_buffer_size的值来提高缓存索引，以大大提高访问性能减少磁盘IO，但是这个缓存区只会缓存索引，而不会缓存数据。
          5.读取速度较快
              占用资源相对较少
          6.不支持外键约束，但只是全文索引
          7.MyISAM引擎是MySQL5.5版本之前的默认引擎，是对最初的ISAM引擎优化的产物。
          8.崩溃后无法安全恢复
  ```

  ```
  MyISAM引擎适用的生产业务场景
          1.不需要事务支持的业务(例如转账就不行，充值也不行)
          2.一般为读数据比较多的应用，读写都频繁场景不适合，读多或者写多的都适合。
          3.读写并发访问都相对较低的业务(纯读纯写高并发也可以)(锁定机制问题)
          4.数据修改相对较少的业务(阻塞问题)
          5.以读为主的业务，例如：www.blog,图片信息数据库，用户数据库，商品库等业务
          6.对数据一致性要求不是很高的业务。
          7.中小型的网站部分业务会用。
          小结：单一对数据库的操作都可以示用MyISAM，所谓单一就是尽量纯读，或纯写(insert,update,delete)等。
  ```

  ```
  MyISAM引擎调优精要
          1.设置合适的索引(缓存机制)(where、join后面的列建立索引，重复值比较少的建索引等)
          2.调整读写优先级，根据实际需求确保重要操作更优先执行，读写的时候可以通过参数设置优先级。
          3.启用延迟插入改善大批量写入性能(降低写入频率，尽可能多条数据一次性写入)。
          4.尽量顺序操作让insert数据都写入到尾部，较少阻塞。
          5.分解大的操作，降低单个操作的阻塞时间，就像操作系统控制cpu分片一样。
          6.降低并发数(减少对MySQL访问)，某些高并发场景通过应用进行排队队列机制Q队列。
          7.对于相对静态(更改不频繁)的数据库数据，充分利用Query Cache(可以通过配置文件配置)或memcached缓存服务可以极大的提高访问频率。
          8.MyISAM的Count只有在全表扫描的时候特别高效，带有其他条件的count都需要进行实际的数据访问。
          9.可以把主从同步的主库使用innodb，从库使用MyISAM引擎。主库写，从库读可以(不推荐，有些麻烦的地方，市场上有人这么用)。
  ```

  ```
  MyISAM其他介绍
  	不支持事务、表锁设计、支持全文索引,主要面向一些 OLAP 数据库应用,在 MySQL 5.5.8 版本之前是默认的存储引擎(除 Windows 版本外)。数据库系统 与文件系统一个很大的不同在于对事务的支持,MyISAM 存储引擎是不支持事务的。究其根 本,这也并不难理解。用户在所有的应用中是否都需要事务呢?在数据仓库中,如果没有 ETL 这些操作,只是简单地通过报表查询还需要事务的支持吗?此外,MyISAM 存储引擎的 另一个与众不同的地方是,它的缓冲池只缓存(cache)索引文件,而不缓存数据文件,这与大多数的数据库都不相同。
  ```

  -  InnoDB与MyISAM对比
    - <img src="C:\Users\博子\AppData\Roaming\Typora\typora-user-images\image-20220209132255491.png" alt="image-20220209132255491" style="zoom: 67%;" />

- 其他引擎

  - ```
    #NDB 存储引擎
    NDB 存储引擎的特点是数据全部放在内存中(从 5.1 版本开始,可以将非索引数 据放在磁盘上),因此主键查找(primary key lookups)的速度极快,并且能够在线添加 NDB 数据存储节点(data node)以便线性地提高数据库性能。由此可见,NDB 存储引擎是高可用、 高性能、高可扩展性的数据库集群系统,其面向的也是 OLTP 的数据库应用类型。
    
    #Memory 存储引擎
    正如其名,Memory 存储引擎中的数据都存放在内存中,数据库重 启或发生崩溃,表中的数据都将消失。它非常适合于存储 OLTP 数据库应用中临时数据的临时表,也可以作为 OLAP 数据库应用中数据仓库的维度表。Memory 存储引擎默认使用哈希 索引,而不是通常熟悉的 B+ 树索引。
    
    #Infobright 存储引擎
    第三方的存储引擎。其特点是存储是按照列而非行的,因此非常 适合 OLAP 的数据库应用。其官方网站是 http://www.infobright.org/,上面有不少成功的数据 仓库案例可供分析。
    
    #NTSE 存储引擎
    网易公司开发的面向其内部使用的存储引擎。目前的版本不支持事务, 但提供压缩、行级缓存等特性,不久的将来会实现面向内存的事务支持。
    
    #BLACKHOLE
    黑洞存储引擎，可以应用于主备复制中的分发主库
    
    #CSV 存储引擎
    存储数据时，以逗号分割各个数据项
    ```

- **InnoDB中的索引**

  - InnoDB中的索引的底层数据结构为**B+ Tree**
  - 分为存放目录项记录的数据页、存放用户记录的数据页
  - 默认数据页大小为16KB
  - 叶子节点存放用户记录，非叶子节点存放目录项记录
  - 树的高度越大，IO次数越多，因此数的层次越少越好

- **聚簇索引**

  - 聚簇索引就是按照每张表的主键构造一颗B+树，同时叶子节点中存放的就是整张表的行记录数据，也将聚集索引的叶子节点称为数据页。这个特性决定了索引组织表中数据也是索引的一部分，每张表只能拥有一个聚簇索引。
  - Innodb通过主键聚集数据，如果没有定义主键，innodb会选择非空的唯一索引代替。如果没有这样的索引，innodb会隐式的定义一个主键来作为聚簇索引

- 在 Mysql 中，InnoDB 引擎的表的 `.ibd`文件就包含了该表的索引和数据，对于 InnoDB 引擎表来说，该表的索引(B+树)的每个非叶子节点存储索引，叶子节点存储索引和索引对应的数据。

  - 用户数据页内的用户记录用单向链表按顺序存放，用户数据页之间用双向链表连接；上一层存放目录项记录页，更上一层存放更广的目录项记录页...
  - **聚簇索引的优点**
    - **数据访问更快**，因为聚簇索引将索引和数据保存在同一个B+树中，因此从聚簇索引中获取数据比非聚簇索引更快
    - 聚簇索引对于主键的排序查找和范围**查找速度非常快**
    - 按照聚簇索引排列顺序，查询显示一定范围的数据时，由于数据都是紧密相连的，因此数据库不用从多个数据块中提取数据，**节省了大量的IO操作**

  - **聚簇索引的缺点**
    - **依赖于有序的数据** ：因为 B+树是多路平衡树，如果索引的数据不是有序的，那么就需要在插入时排序，如果数据是整型还好，否则类似于字符串或 UUID 这种又长又难比较的数据，插入或查找的速度肯定比较慢。
    - **更新代价大** ： 如果对索引列的数据被修改时，那么对应的索引也将会被修改， 而且聚簇索引的叶子节点还存放着数据，修改代价肯定是较大的， 所以对于主键索引来说，主键一般都是不可被修改的。
    - **二级索引访问需要两次索引查找**，第一次找到主键值，第二次根据主键值找到行数据。
  - **聚簇索引的限制**
    - MySQL数据库中只有InnoDB引擎支持聚簇索引，而MyISAM并不支持
    - 由于数据物理存储排序方式只能有一种，因此每个表**只能有一个**聚簇索引，一般是该表的主键
    - 如果没有定义主键，InnoDB会选择**非空的唯一索引**代替，如果没有这样的索引，会隐式定义一个主键来作为聚簇索引

- **二级索引（也称辅助索引，属于非聚簇索引）**

  - **非聚集索引即索引结构和数据分开存放的索引。**
  - 二级索引的非叶子节点存放其他列+主键值
  -  **非聚集索引的优点**
    - **更新代价比聚集索引要小** 。非聚集索引的更新代价就没有聚集索引那么大了，非聚集索引的叶子节点是不存放数据的
  - **非聚集索引的缺点**
    - 跟聚集索引一样，非聚集索引也依赖于有序的数据
    - **可能会二次查询(回表)** :这应该是非聚集索引最大的缺点了。 当查到索引对应的指针或主键后，可能还需要根据指针或主键再到数据文件或表中查询。

- **联合索引**：属于非聚簇索引，普通索引中每个节点的键值数量只有1个，而联合索引中每个节点的键值数量大于等于2

- **MyISAM中的索引**
  - MyISAM索引文件和数据文件是分离的，**索引文件仅保存数据记录的地址**
    - 索引文件存在.MYI中；数据文件存在.MYD中
    - InnoDB的索引和数据都存在.IBD中
  - MyISAM只有非聚簇索引
  - 底层结构同样也是B+Tree，data域保存数据记录的地址
  - MyISAM中索引检索算法为：首先按照B+Tree搜索索引，如果指定的Key存在，取出其data域的值，然后以data域的值为地址，读取相应数据记录
  - ![image-20220209163419393](C:\Users\博子\AppData\Roaming\Typora\typora-user-images\image-20220209163419393.png)

## **2022年2月10日**

------

- Hash 增删改查的平均时间复杂度为O(1), 树 增删改查的平均时间复杂度为(Olog2N)

- 页是基本单位，InnoDB中，每页的默认大小为16KB
- 区：每个区包含64个连续的页，大小为1MB
- 段：由1个或多个区组成，区之间不一定在物理空间上连续
- 页按类型划分：数据页（保存B+数节点）、系统页、Undo页和事务数据页

- 索引分类
  - 按类型：普通索引，唯一索引，主键索引，全局索引，空间索引

- 创建索引

  - 在创建表的时候创建

  - ```mysql
    # 创建普通索引
    CREATE TABLE student1(
    id  INT NOT NULL,
    NAME VARCHAR(20)NOT NULL,
    email VARCHAR(30),
    INDEX e_name(NAME) # 声明索引
    );
    
    # 创建唯一索引
    CREATE TABLE student2(
    id  INT NOT NULL,
    NAME VARCHAR(20)NOT NULL,
    email VARCHAR(30),
    UNIQUE INDEX uk_idx_name(NAME) # 声明索引
    );
    
    # 创建主键索引 （只能通过创建主键约束来隐式创建）
    CREATE TABLE student3(
    id  INT PRIMARY KEY ,
    NAME VARCHAR(20)NOT NULL,
    email VARCHAR(30)
    );
    SHOW INDEX FROM student3;
    
    #创建联合索引
    CREATE TABLE student4(
    id  INT PRIMARY KEY ,
    NAME VARCHAR(20)NOT NULL,
    email VARCHAR(30),
    INDEX muti_idx(NAME,email)
    );
    ```

  - 使用ALTER TABLE语句创建索引

  - ```mysql
    ALTER TABLE student4 ADD INDEX e_idx(email)
    ```

  - 使用CREATE INDEX ... ON .. 语句创建索引

  - ```mysql
    CREATE INDEX n_idx ON student3(NAME)
    ```

- 删除索引

  - 使用ALTER TABLE .. DROP INDEX..

  - ```mysql
    ALTER TABLE student3 DROP INDEX e_idx
    ```

  - 使用DROP INDEX ... ON ...
  
  - ```mysql
    DROP INDEX e_idx ON student3
    ```
  
    

