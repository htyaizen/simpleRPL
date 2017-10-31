SimpleRPL
=========
SimpleRPL是基于Linux的低功耗和有损路由协议的实现[RFC 6550]（https://tools.ietf.org/html/rfc6550）中定义的网络（RPL）。
它旨在通过带来一个完整的Linux无线传感器网络生态系统（希望）完全兼容的RPL实现。

实现了什么？实现了什么选择？
----------------------------------------------------------
*无组播支持的存储模式（MOP值为2）
*作为DODAG根或RPL路由器
*实现目标函数零（RFC 6552）。然而，rank增加总是一样固定的价值。这是因为没有反馈
  第2层或第3层（尚未），这意味着无法指示链接质量。
*存储无限数量的DIO父母
*存储一个DAO父母at time
*支持多个接口（即节点可以充当两个链路层之间的桥梁）

尚未实现的
------------------------
*路由metrics（如RFC 6551所定义）未被实现，这是因为目前没有办法从IEEE 802.15.4链路中检索链路质量信息
*不支持浮动DODAG
*不支持安全机制（因此，如果需要，应该实施在链路层）
*不支持叶功能
* DAO消息中没有路径控制支持

须知限制
-----------------
*一次只有一个DODAG根可以存在于网络中：如果同一个DODAG存在两个根，则它们将永久竞争
*一次只能加入一个DODAG

安装
------------

### List of dependencies

SimpleRPL是用Python 2.x编写的，需要安装下列库：
* [libnl3](http://www.infradead.org/~tgr/libnl/): netlink库
* [pyzmq](http://pypi.python.org/pypi/pyzmq): 用来绑定著名的ZeroMQ库和Python
* [Routing](http://github.com/tcheneau/Routing/): 一个可以管理路由，地址和链路层地址的libnl3上的python包
* [RplIcmp](http://github.com/tcheneau/RplIcmp/): 一个通过Python中的ICMP套接字来简化操作的python模块。
* [python-zmq](http://www.zeromq.org/bindings:python): Python绑定ZeroMQ（0mq）库
* [python-argparse](https://pypi.python.org/pypi/argparse):一个python参数解析器模块（只需要你的Python版本是<2.7）


### Sytem-wide installation

From the root directory:

    $ python setup.py install


### Bulding up RPMs
您可能更喜欢构建软件包（以便您可以轻松部署，升级或删除SimpleRPL）。所有你需要做的，从根目录是：

    $ python setup.py bdist_rpm

如何使用
----------

### General information

SimpleRPL的设计只能通过命令行参数进行配置。
以下是simpleRPL识别的参数列表:

    $ simpleRPL.py --help
    usage: simpleRPL.py [-h] [-d DODAGID] [-i IFACE] [-R] [-v] [-p PREFIX]
    
    A simplistic RPL implementation
    
    optional arguments:
      -h, --help            show this help message and exit
      -d DODAGID, --dodagID DODAGID
                            RPL DODAG Identifier, has to be an IPv6 address that
                            is assigned on the node (optional)
      -i IFACE, --iface IFACE
                            network interfaces that RPL will listen on
      -R, --root            indicates if the nodes is the DODAG Root
      -v, --verbose         verbose output
      -p PREFIX, --prefix PREFIX
                            Routable prefix(es) that this node advertise (only for
                            DODAG root, optional)

请注意，由于其功能简单，SimpleRPL需要在系统中进行root访问。

### Running a RPL Router

如果要启动在所有接口上侦听的RPL路由器：

     $ simpleRPL.py

在这种情况下，节点将加入收到DIO消息的第一个DODAG。

如果您想要更详细的输出：使用“-v”参数打开调试（可以重复多达五次以获得更详细的输出（“-vvvvv”））。

### Running a DODAG Root
DODAG根需要一个全局IPv6地址，它在其一个接口上分配为DODAG ID。以下是DODAG ID为2001的DODAG示例：
aaaa :: 0202：0007：0001（在任何接口上分配），它们公布了2001：aaaa :: / 64前缀。

    $ simpleRPL.py -vvvvv -R -d 2001:aaaa::0202:0007:0001 -p 2001:aaaa::

### Getting information on a running instance
SimpleRPL附带一个可以与正在运行的实例通信的辅助工具，以便检索内部值或触发一些管理功能
（本地修复，全球修复等）。此工具名为_cliRPL.py_。

当SimpleRPL启动时，它将IPC套接字绑定到当前目录。这涉及_cliRPL.py_需要在同一目录中被调用。

获取可用命令列表：

    $ cliRPL.py help
    show-preferred-parent: List the currently preferred (DIO) parent
    list-parents-verbose: List the (DIO) parents and their corresponding DODAG
    list-downward-routes: List the downward routes for the currently active DODAG
    local-repair: Trigger a local repair on the DODAG
    list-routes: List the routes assigned by the RPL implementation
    list-parents: List the (DIO) parents
    show-current-dodag: Show the currently active DODAG
    show-dao-parent: Show the DAO parent (for the currently active DODAG)
    subdodag-dao-update: Trigger the DODAG to increase its DTSN so that the sub-dodag will send a DAO message
    global-repair: Trigger a global repair on the DODAG (only valid for DODAG root)
    list-dodag-cache: List the content of the DODAG cache
    list-neighbors: List the neighbors
    help: List this help
    list-neighbors-verbose: List the neighbors and their corresponding DODAG

例如，如果你想从DODAG root触发一个全局修复：
    $ cliRPL.py global-repair
    global repair triggered, bumping new version for DODAG:
    DODAGID: 2001:aaaa::202:7:1; new version: 241
    全球修复触发，碰撞新版本的DODAG：
    DODAGID：2001：aaaa :: 202：7：1;新版本：241

与其他实现的交互
-------------------------------------------
暂时没有执行交互性测试。这是因为目前，Linux内核随附的6LoWPAN堆栈遭受了一些困扰
局限性。一旦与其他操作性系统，如Contiki，实现最基本的交互性测试。交互性测试将成为最高的
优先。

A word on security
------------------

SimpleRPL is expected to be run on a secure environment (either completely
isolated, or using link-layer security). This is because we use it as a
prototype implementation. There is a lot of case where the implementation will
(purposely) stop working (because a function is not implemented yet). This
means that someone with evil intents could craft packets designed to shut down
the implementation.
SimpleRPL预计将在安全的环境中运行（完全隔离运行或使用链路层安全）。这是因为我们把它当作一个
原型实现。执行会有很多情况（故意）停止工作（因为功能尚未实现）。这个意味着有邪恶意图的人可以制作出关闭的数据包实施。
Authors
-------

* Tony Cheneau (tony.cheneau@nist.gov or tony.cheneau@amnesiak.org)

Acknowledgment
--------------

This work was supported by the Secure Smart Grid project at the Advanced
Networking Technologies Division, NIST.

Conditions Of Use
-----------------

<em>This software was developed by employees of the National Institute of
Standards and Technology (NIST), and others.
This software has been contributed to the public domain.
Pursuant to title 15 Untied States Code Section 105, works of NIST
employees are not subject to copyright protection in the United States
and are considered to be in the public domain.
As a result, a formal license is not needed to use this software.

This software is provided "AS IS."
NIST MAKES NO WARRANTY OF ANY KIND, EXPRESS, IMPLIED
OR STATUTORY, INCLUDING, WITHOUT LIMITATION, THE IMPLIED WARRANTY OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, NON-INFRINGEMENT
AND DATA ACCURACY.  NIST does not warrant or make any representations
regarding the use of the software or the results thereof, including but
not limited to the correctness, accuracy, reliability or usefulness of
this software.</em>
