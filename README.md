# RIO应用开发手册

1.
# 1.关于本文档

本文档是开发人员使用RIO基础框架进行应用开发的参考手册，通过本手册，可对RIO的构成、基础类、工具等有一个全面的了解，从而能够尽快使用RIO进行开发。

本文档提供给所有开发人员阅读，包括架构师、设计人员、编码开发人员等。

RIO提供了示例项目作为实践指南，相关说明请参考《RIO应用开发实例说明》。

在阅读本文档前，建议读者先了解以下方面的基础知识：

- XML Schema规范
- Hibernate框架
- Spring开发框架、Hibernate整合特性
- Struts2框架

1.
# 2.RIO简介

## 简介

RIO是针对Java Web应用开发提供的基础框架，包含若干基础应用类、工具以及常见应用功能模块，可在大多数应用系统中复用，RIO本身不包含应用逻辑。

使用RIO开发的应用系统，部署时需要将RIO基础包、所依赖的第三方包一同发布。

在RIO基础上开发应用，存在以下优势：

1. 1.
2. 2.
  1. 2.1.

### 快速开发

从零开始构建一个项目需要编写很多基础性代码，RIO提供一系列基础类、代码生成工具，并提供一个可用的基本示例项目，可减少项目初期构建成本，避免编写重复性代码，使开发人员集中精力于业务逻辑。

### 规范性

在RIO上开发需要遵循相应的规范要求，配置、处理机制、程序结构都比较统一，有效降低维护成本。

### 稳定性

重复性代码和配置通过工具自动生成，RIO保证了这些代码的正确性和稳定性，一些基础功能特性能够复用，提升了应用系统整体的稳定性。

### 提升可配置性

RIO提供的基础特性，如可联机刷新的配置读取、序列号生成等，都提供了灵活的配置手段，使用命名查询特性可以将SQL语句从java代码剥离，为应用系统的客户化留下配置空间。

1.
  1. 2.1.第三方依赖

RIO使用JDK1.5构建，要求应用系统使用Java 1.5以上版本开发。

RIO依赖的第三方框架和开发包有：

- Hibernate 3.6.7
- Spring 3.0.6
- Struts 2.2.3
- Castor 1.3.1
- XDoclet 2.0.6

1.
  1. 2.2.发行包说明

RIO发行包是一个压缩文件，解压后包含以下目录：

doc RIO文档，包括版本说明、开发手册等

javadoc RIO API的java doc

lib RIO基础包，其中：RIO-xx.jar是基础包，需要部署在应用Runtime环境，RIO-xx-deps.zip，RIO集成的spring,hibernate等框架依赖包。以及RIO-xx-genlib.zip其中genlib目录下是运行代码生成等编译期任务需要的库文件。这个包解压出来的jar不用放在项目的编译环境里面去。

resources RIO基础配置和资源文件，应用根据实际需要更改配置并部署到CLASSPATH中

sample RIO示例项目，工程项目说明参见《RIO应用开发实例说明》，建议开发人员在实际使用RIO之前对照《RIO应用开发实例说明》阅读项目代码

schema RIO中XML配置对应的Schema，XML文件的编写需要遵照Schema的约束，详细说明请参见附录一

sql 数据库对象创建、初始化等相关的SQL文件，Oracle、SQL Server、MySQL、DB2等不同数据库的脚本分别放在各子目录下

WebRoot RIO为页面提供的基础库和组件，应用需要根据具体使用到的特性选择使用

RIO发行包解压之后即可使用，需要在项目工程中配置相关的环境，具体样例可参见《RIO应用开发实例说明》。

1.
# 3.持久层

按照一般分层原则，持久层用于完成业务实体对象的存取、查询等操作，RIO在这个层面提供多样化的手段支持，在调用同样API的前提下，既能够通过hibernate的OR-Mapping特性和HQL语句访问数据，也能够通过RIO内部封装的JDBC调用及SQL语句访问数据（需要Spring支持），方便开发人员根据应用的实际要求灵活选择。

在对业务实体对象模型的支持上，RIO做了简化处理，只支持单个表的对象映射，一对多、多对多、对象引用等hibernate支持的特性，在RIO提供的PO及JDBC调用上均无法实现，但这并不影响开发人员使用RIO提供的基于Hibernate的DAO，此DAO基类上提供了丰富的数据访问方法（包括分页查询、命名查询等），必要时开发人员可基于Hibernate自行构建PO类及对象映射关系，并借助RIO的基础DAO，快速构建包含丰富Hibernate Mapping特性的持久层。

RIO提供了复合主键的支持，方便在某些特定场合下的使用。

1.
  1. 3.1.构成

RIO采用DAO模式实现数据操作，即：需要持久化保存的业务实体作为PO看待，每个PO都有对应的DAO对象，DAO提供此PO对象的创建、修改、删除、对象获取、查询、分页查询、批量更新等数据操作方法。

如上图所示，针对每个PO，RIO都提供基于Hibernate或JDBC的DAO，DAO配置在Spring容器中管理并从中获取所依赖的Session等对象。

基于Hibernate的DAO需要session对象的支持，映射关系包含在后缀为.hbm.xml的配置文件中；基于JDBC的DAO则需要dbInfoProvider对象的支持，对象与实体表之间的关系包含在各META对象中。

使用命名查询特性时，DAO需要Named SQL Configuration配置文件的支持，命名查询（Named SQL）以配置的形式提供数据操作中需要执行的SQL语句（HQL或SQL），SQL语句的可配置性，大大增强了应用系统的灵活性，也便于项目组SQL语句的审查，详细说明可参见后文描述。

1.
  1. 3.2.基础类

DAO相关的基础类在org.rg.rio.common.dao包下，PO相关的基础类在org.rg.rio.common.po包下，重要的类有：

1. 3.
  1. 3.1.
  2. 3.2.
    1. 3.2.1.GenericDao

DAO接口类。除了单一PO对象的增删改查外，此接口还定义了部分属性更新、属性匹配查询、分页查询、命名查询、HQL/SQL语句查询、更新等方法，在两个基础实现类：BaseHibernateDao、BaseJdbcDao中，都有这些方法的具体实现。

DAO方法的简要说明：

create 创建新的实体对象实例，有单个实例和批量实例两种模式

save 更新已有的实体对象，提供单个实例和批量实例更新，也提供部分属性更新

delete 根据主键删除实例

get 根据主键获取实体对象实例，可控制锁定

exists 根据主键判断实例的存在性

getAll 获取所有对象实例，提供排序及分页查询

getByProperty 根据属性参数做匹配查询，这是一种便捷的查询方法，匹配模式为&quot;=&quot;或&quot;IN&quot;（匹配参数为Collection或数组类型时），提供排序及分页查询，提供唯一结果（uniqueByProperty）及数量查询（countByProperty）

listNamedQuery 执行命名查询，提供分页查询、唯一结果查询

listQuery 根据HQL/SQL参数语句执行查询，提供分页查询、唯一结果查询

execNamedUpdate 执行命名查询，语句为更新或删除操作

execUpdate 根据HQL/SQL参数语句执行更新或删除操作

1.
  1.
    1. 3.2.2.PropertySaveIndicator

使用PO对象传递参数，只修改实体的部分属性时，过程往往比较繁琐，基础DAO中提供了部分属性更新的方法，通过PropertySaveIndicator指定需要更新的属性范围，POSaveIndicator则是具体的实现类。

其它相关类的说明参见javadoc。

1.
  1. 3.3.实体定义

为了简化开发，RIO提供源代码生成工具，不需要开发人员手工编写PO、DAO等代码。生成代码之前，需要对PO进行定义。可以这样理解，对PO进行配置和定义的过程，就是设计人员对应用系统进行分析、抽象，形成业务模型及其存储结构的过程。

PO使用XML文件进行定义，XML对应的Schema在RIO发行包的schema路径下：persistentObject.xsd，其详细说明请参见附录一。

一个PO定义XML中可定义多个业务实体，建议在具体使用时，按照模块划分将PO配置在多个XML文件中，各模块的PO、DAO有各自的package，相关命名参见开发约定章节。

定义PO，有3方面特性需要定义：1.PO对象的属性及存储类型；2.PO的主键；3.PO的外键。其中外键定义是可选的，也可以定义多个外键。

定义PO时可以为其指定一个自行扩展的DAO类，如不指定，则使用RIO生成的DAO作为该PO对应的DAO类，这为开发人员在DAO上定义新的方法提供了空间。例如：需要在OrderDao上增加新的查询方法，则可以自己定义一个order的扩展DAO类，该扩展类从工具生成的DAO继承，利用父类中的方法，可以很方便地实现新的查询方法。将该扩展类名设置在XML文件中，则生成的spring相关配置会使用该扩展类。

1.
  1. 3.3.
    1. 3.3.1.属性定义

由于RIO对PO的映射特性做了限定，只支持单表映射，因此PO对象的属性只能是String、Date等基本类型或复合主键对象类型，不存在对其它PO对象的引用，也不存在List、Set、Map等集合。

RIO支持的属性类型包括：

#### 字符类

PO对象属性为String类型，数据表根据长度采用char、varchar类型。

#### 数值类

包含int、long、float、double，PO对象属性为Integer、Long、Float、Double等类型，数据表采用int、bigint、number、numeric等类型。

#### 日期类

包含date、timestamp，PO对象属性为java.util.Date类型，数据表采用date、datetime、timestamp等类型。

#### 布尔类

包含boolean、truefalse、yesno，PO对象属性为Boolean类型，数据表采用bit、number、char等类型。

#### 大字段类

包含text（文本）、blob（二进制），PO对象属性为String、byte[]类型，数据表采用text、clob、blob等类型。

1.
  1.
    1. 3.3.2.主键定义

每个PO都需要定义主键，用于业务实体对象的唯一性识别，RIO支持复合主键定义，即多个属性组合在一起形成唯一性标识，在XML中的pkMember元素指定主键包含的属性名称。

除了定义主键的构成，还需要配置主键取值的策略，支持的策略包括：

identity 由数据库的自增长特性字段产生唯一标识，值可以从创建完成之后的对象属性中获取

assigned 由调用者自行设置，在创建之前就设置好相关对象属性

sequence 由特定序列产生。不是所有的数据库都支持序列特性，因此RIO提供了可在所有数据库上使用的通用序列号生成手段，详见后文

复合主键由于涉及多个属性的取值，只能使用assigned策略。

1.
  1.
    1. 3.3.3.外键定义

通过外键定义，可在数据库中为业务实体设置关联。RIO支持复合外键的定义，即一个实体的多个属性组合与另外一个实体的多个属性组合形成外键约束。

一个业务实体上可以定义多个外键，外键定义对PO、DAO操作没有影响，仅反映在数据库的约束上。



上述PO配置包含了业务实体的常见使用特性，应用中如果不能满足对业务对象模型的要求，就需要手工编写PO、DAO等类及Hibernate配置，无法享受代码生成工具带来的好处，但可以复用基础DAO上的方法。

1.
  1. 3.4.DAO代码生成

RIO提供代码生成工具，能够根据PO定义XML文件生成PO、DAO等Java源代码及相关配置文件、数据库DDL脚本，由3个含有main方法的Java类构成：

org.rg.rio.tools.dao.HibernateDaoGenerateCmd 生成PO、Hibernate Dao、spring配置文件，PO的Java源文件中包含了XDoclet注释，可进一步产生hibernate配置及数据库DDL脚本

org.rg.rio.tools.dao.JdbcDaoGenerateCmd 生成PO、JDBC Dao、spring配置文件、数据库DDL脚本，生成过程及DAO的执行不需要hibernate的支持

org.rg.rio.tools.dao.SchemaGenerateCmd 单独生成数据库DDL脚本，可提供oracle、SQL Server、MySQL、DB2数据库的支持

上述工具运行需要若干命令行参数，含义如下：

**-i po\_cfg\_file**

指定配置PO的XML文件，根据此配置文件生成代码

**-dep po\_cfg\_file1 po\_cfg\_file2 … po\_cfg\_filen**

指定所依赖的PO配置文件，可以指定多个。在-i参数指定的PO配置中，某些外键可能引用到其它XML中配置的PO对象，就需要在此设置依赖的配置文件；如果没有上述PO引用就无需设置此选项

**-java java\_src\_path**

指定PO、DAO等java代码的保存路径

**-cfg spring\_config\_file**

指定spring配置文件名，不指定则不生成spring配置文件

**-schema schema\_ddl\_file**

指定数据库DDL脚本文件名，不指定则不生成DDL脚本

**-dialect dialect\_name(oracle | sqlserver | mysql | db2 )**

生成数据库DDL脚本时需要设置此参数，指定数据库类型

各工具需要的命令行参数如下：

**HibernateDaoGenerateCmd**

-i 必需；-dep 可选；-java 必需；-cfg可选

**JdbcDaoGenerateCmd**

-i 必需；-dep 可选；-java 必需；-cfg 可选；-schema 可选；-dialect 设置schema选项生成DDL时必需

**SchemaGenerateCmd**

-i 必需；-dep 可选；-schema 必需；-dialect 必需

使用时可以在命令行运行上述工具，如果使用ANT作为项目构建工具，也可以配置到ANT中作为一个TASK使用，具体示例可参见RIO示例项目中的build.xml文件。

以下详细介绍工具所生成的各项内容。

1.
  1.
    1. 3.4.1.PO类

在PO配置中定义的每个PO都会生成一个接口类和一个实现类，如果存在复合主键，也会生成复合主键对象的接口类和实现类，Java类的package名称可在XML的package属性中指定，也可以在PO类的entityName中设置包含package在内的完整类名称。在生成这些Java类时，工具会自动追加po、dao、impl等子包名。Java代码的保存路径在命令行参数中指定，详见命令行参数说明。

PO类中主要包含各属性的Get/Set方法，对于复合主键，也提供复合主键对象的Get/Set方法，另外还有经常用于调试的toString()方法，方便观察属性值。

PO类提供3个构造方法：1.不含参数的空构造方法；2.包含所有属性参数的构造方法，方法中设置各属性的值；3.以PO接口类为参数的构造方法，从参数对象中获取属性值并设置到对应属性中。

1.
  1.
    1. 3.4.2.Meta类

每个PO还生成一个Meta类，用于描述PO的属性，并为JDBC方式执行DAO操作提供元数据支撑。Meta上包含PO对象各属性名称的静态定义，还包含设置PO属性的便捷方法。

Meta类与PO实现类在同一个package中，命名规则是在PO类名上增加后缀Meta。

1.
  1.
    1. 3.4.3.DAO类

每个PO都生成对应的DAO，包含一个接口类和一个实现类，用于处理PO的数据存取访问。DAO类生成在名称为&quot;dao&quot;的子包名中，类名称与PO的名称相关，以DaoHibernate结尾的是基于Hibernate的DAO实现类，以DaoJdbc结尾的是基于JDBC的DAO实现类。Java代码的保存路径在命令行参数中指定，详见命令行参数说明。

DAO中需要的数据操作方法，如create、save、delete、get、exist、getAll、getByProperty、listNamedQuery、listQuery、execNamedUpdate、execUpdate等，在父类中都已经实现，因此生成的DAO类非常简单。

1.
  1.
    1. 3.4.4.Spring Bean配置

DAO的常见使用方式是交给Spring容器管理，因此需要在XML中对这些DAO进行配置，生成工具可以一并生成该配置文件。

Bean配置文件的文件名在命令行参数中指定，如不指定则不生成此配置文件。

1.
  1.
    1. 3.4.5.Schema脚本

构建数据库schema的DDL脚本有两种途径产生：如果使用hibernate，在生成的PO类中包含有XDoclet注释，通过XDoclet及Hibernate提供的相关工具生成DDL脚本，具体使用可参见RIO示例项目；如果不使用Hibernate，RIO提供的工具可直接生成指定数据库类型的DDL脚本。

Schema脚本的文件名在命令行参数中指定，脚本中包含有：创建table及主键约束、创建sequence、创建外键等语句。

1.
  1. 3.5.命名查询

为了增强应用系统的灵活性，RIO提供命名查询，即SQL语句做配置化处理并通过名字进行标识，使用时通过名字获取配置并动态组装SQL语句，这样的剥离会带来很多好处：

- 具备根据参数动态组装SQL语句的能力，避免Java代码中拼装SQL的过程，简化代码。
- 部分业务逻辑的调整或存储结构的变化，可修改查询配置，而不是修改Java代码。
- 方便适应不同类型数据库切换的要求，也便于做SQL审查，出现SQL性能问题时便于梳理。
- 某些SQL的优化只需要更改配置，不需要修改Java代码。

命名查询采用XML配置，可配置在多个文件中，建议按照项目模块的划分为基础进行文件区分，部署时需要将这些XML文件放到CLASSPATH中，默认使用的文件名匹配规则是：\*.namesql.xml，如需调整文件名匹配规则，可修改base\_config.properties配置文件，详见配置文件说明章节。

配置在XML中的查询通过名称识别，配置时需要保证name属性的唯一性。配置文件支持联机刷新，修改后直接覆盖CLASSPATH中的文件即可生效。

可以在命名查询中配置的不仅仅是查询语句，也可以是UPDATE、DELETE语句，既可以是Hibernate中的HQL语句，也可以是某种数据库特有的SQL语句，通过XML配置中的type属性进行区别。

在XML中配置的SQL，可以是完整的一句SQL，也可以是若干SQL片段（sqlpart）的拼接，在实际执行该查询时，根据SQL片段的matchParam属性或matchExpression属性与实际的查询参数进行匹配，匹配成功的片段拼接到最终的SQL语句中，这种模式在应用开发中很常见，例如：经常需要根据UI中用户填写的查询条件动态拼装SQL语句。

matchParam、matchExpression都能用于SQL片段的匹配，matchParam的模式比较简单，只判断对应名称的查询参数是否实际存在并且不是空字符串，这是一种很常见的模式；matchExpression则可以配置EL表达式，组合判断一个或多个查询参数的取值，以决定该SQL片段是否拼装到最终的查询SQL。XML中的完整属性设置，请参见附件一。

命名查询可通过任何一个DAO对象调用，listNamedQuery、uniqueNamedQuery、pageNamedQuery用于执行查询，execNamedUpdate用于执行更新、删除等操作。

1.
  1. 3.6.通用序列号生成

许多应用系统都需要诸如流水号、标识号之类的标识，常见的方式是使用数据库提供的sequence特性，但是并非所有的数据库都支持sequence，为此RIO提供了支持任意数据库的通用序列号生成器。

使用之前，需要在数据库中创建一张数据表，用于保存sequence数据，建表SQL见RIO发行包sql、路径下的schema.txt文件。在生成器的使用过程中，每个sequenceId都会在该数据表中登记一条数据并根据需要刷新。如应用系统有特殊需要，需要调整该数据表的表名、字段名，可在数据库中更改后调整base\_config.properties配置文件中的参数，详见配置文件说明章节。

序列号生成的实现类是SequenceGeneratorImpl，使用时不需要自行构建，从DBInfoProvider中获取即可，DBInfoProvider一般配置在spring容器中，可参见RIO示例项目。序列号生成器主要提供3个方法：lastValue用于取得上一次产生的序列号，nextValue用于获取下一个序号，nextValues用于获取一批序号。

RIO提供的序列号生成器提供分布式环境下的多节点支持，多个应用服务器节点在使用同一个数据库服务器时，可一起使用，不会产生序列号重叠，但是不保证每个节点产生的序列号的连续性。

尽管RIO提供的序列号生成器具备通用性，在数据库支持sequence的情况下，仍然建议优先考虑数据库的sequence特性，数据库sequence具备高效、与事务无关等特点，在复杂场景下不会带来应用事务方面的干扰。在RIO提供的基于JDBC的DAO实现中，sequence的产生机制会根据数据库类型进行选择，数据库支持的情况下调用数据库的sequence访问语句。
