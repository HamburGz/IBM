进入IBM后，leader首先提出的工作任务是搭建一套WAS环境，那么啥是WAS呢？又该如何搭建？

Here we go.


# 理论学习

WebSphere Application Server V8.5.5(以下简称：WAS855)					
WAS是一种Java EE运行环境，（不同于 Tomcat等容器，Tomcat只实现了Servlet和JSP的规范，可以算是轻量级的Java运行时环境）。除了Servlet和JSP，WebSphere还实现了Java EE的一系列规范，比如：Enterprise JavaBeans (EJB)、Java Persistence API (JPA)、Java Message Service (JMS)、JavaBeans Activation Framework (JAF)等。

WAS更多详细介绍请参考：[WAS功能介绍](https://www.ibm.com/support/knowledgecenter/zh/SSAW57_9.0.0/com.ibm.websphere.nd.multiplatform.doc/ae/welc6productov.html)





# 准备工作

 1. 查看OS版本 :  Oracle Solaris 10 1/13 s10s_u11wos_24a SPARC
 2. 下载IBM Installation Manager(简称IIM)安装包:InstalMgr1.6.2SOLSPARC_WAS_8.5.5.zip
 3. 下载WAS安装包: WASND_v8.5.5_1of3.zip、WASND_v8.5.5_2of3.zip、WASND_v8.5.5_3of3.zip

因为安装WAS855需要使用IIM，所以需要先下载对应操作系统的IIM安装包。

同时，本次搭建所选择的WAS安装包是WASND安装包，WASND全称：WebSphere Application Server Network Deployments(简称:WASND).WASND可以说是WAS的集群版本，顾名思义，安装WASND后，可通过配置将多个Server配置成一个大的WAS集群。具体配置在安装后会进行详细说明。

*目前对WAS和WASND二者的差别没有进行彻底地了解，WASND安装包由同事提供，我并未自己下载，所以后面出现的WASND、WAS、WAS855都代表同一个东西。（相关概念上的差异后续会进行学习并补充）*

# 环境搭建

SSH工具: PuTTY (Windows 10)

因本次搭建所使用的PuTTY不支持GUI，所以后面IIM和WAS855的安装，均使用的是Silent安装方式（静默安装）。

## **安装IIM**

因需要使用IIM软件去搭建WAS855，所以先安装IIM（Silent Mode）。下面几步是本次安装IIM所用到的具体步骤，若需要其他信息：[IBM Installation Manager安装](https://www.ibm.com/developerworks/cn/websphere/library/techarticles/1111_zhub_im/1111_zhub_im.html)

1、切换至IIM安装包目录 /tmp/iim	

    cd /tmp/iim

2、解压IIM安装包：InstalMgr1.6.2SOLSPARC_WAS_8.5.5.zip

    unzip InstalMgr1.6.2SOLSPARC_WAS_8.5.5.zip -d /tmp/iim

3、删除IIM安装包（可选）

    rm -f InstalMgr1.6.2SOLSPARC_WAS_8.5.5.zip

4、管理员模式安装

    ./installc -acceptLicense -log /opt/IBM/IIM/logs

-log 后面 /opt/IBM/IIM/logs 是指定的IIM日志目录路径


## 安装WAS855

下面几步是本次搭建WAS855环境所用到的具体步骤，若需要其他信息：[WAS V855静默安装
](https://blog.csdn.net/lczean/article/details/77164207)

1、切换至WAS855安装包目录 /tmp/wasnd855

    cd /tmp/wasnd855

2、解压WAS安装包：WASND_v8.5.5_1of3.zip、WASND_v8.5.5_2of3.zip、WASND_v8.5.5_3of3.zip

使用unzip命令一次性解压至临时目录 /tmp/was

    unzip WASND_v8.5.5_\?of3.zip -d /tmp/was

3、删除WAS安装包

    rm -rf /tmp/wasnd855

4、挂载WAS仓库文件

    cd /opt/IBM/InstallationManager/eclipse/tools
    ./imcl listAvailablePackages -repositories /tmp/was
		
挂载后会显示以下内容：  
com.ibm.websphere.ND.v85_8.5.5000.20130514_1044

5、执行安装命令

    cd /opt/IBM/InstallationManager/eclipse/tools
    ./imcl install com.ibm.websphere.ND.v85_8.5.5000.20130514_1044 \
    -repositories /tmp/was/repository.config \
    -installationDirectory /opt/IBM/WebSphere/AppServer \
    -sharedResourcesDirectory /opt/IBM/IMShared \
    -nl zh \
    -properties cic.selector.nl=zh \
    -acceptLicense \
    -showVerboseProgress

## 配置profile并启动WAS

本次搭建WAS855环境，其实仅需一台Server即可，由于本次安装使用的是WASND安装包，现在需通过添加并配置相应的profile来搭建一个单机WAS环境(A standalone application server)。下面几步是本次配置profile并启动WAS的步骤，若需其他信息：[配置并启动WASND-单机环境](https://www.ibm.com/support/knowledgecenter/SSAW57_8.5.5/com.ibm.websphere.nd.multiplatform.doc/ae/tpro_setup.html)

1、切换至WAS855的bin目录 %app_server_root%/bin

    cd %app_server_root%/bin

其中，%app_server_root%就是WAS的安装目录，本次搭建使用的是默认安装目录：/opt/IBM/WebSphere/AppServer

2、使用profileTemplate（profile模板）创建profile

    ./manageprofiles.sh -create -templatePath %app_server_root%/profileTemplates/default 

其中，-templatePath %app_server_root%/profileTemplates/default 表示指定**default**类型的profile模板来创建profile。

3、切换至WAS855的profiles目录 %app_server_root%/profiles，查看%profile_name%的名称

    cd %app_server_root/profiles
    ls -l
    
本次搭建生成的profile，其%profile_name%为：AppSrv01 （默认）。则用%profile_root%来表示目录：/opt/IBM/WebSphere/AppServer/profiles/AppSrv01

4、切换至%profile_root%/bin，启动Server

    cd %profile_root%/bin
    ./startServer.sh server1 -profileName %profile_name%

5、切换至%profile_root%/properties，查看portdef.props文件

    cd %profile_root%/properties
    less portdef.props


文件内容（可能不同，请仔细查看）：

    SOAP_CONNECTOR_ADDRESS=8880
    SIP_DEFAULTHOST_SECURE=5061
    SIP_DEFAULTHOST=5060
    SIB_ENDPOINT_ADDRESS=7276
    WC_defaulthost_secure=9443
    DCS_UNICAST_ADDRESS=9353
    SIB_MQ_ENDPOINT_SECURE_ADDRESS=5578
    WC_adminhost_secure=9043
    CSIV2_SSL_MUTUALAUTH_LISTENER_ADDRESS=9402
    ORB_LISTENER_ADDRESS=9100
    BOOTSTRAP_ADDRESS=2809
    CSIV2_SSL_SERVERAUTH_LISTENER_ADDRESS=9403
    IPC_CONNECTOR_ADDRESS=9633
    SIB_ENDPOINT_SECURE_ADDRESS=7286
    WC_defaulthost=9080
    SIB_MQ_ENDPOINT_ADDRESS=5558
    SAS_SSL_SERVERAUTH_LISTENER_ADDRESS=9401
    OVERLAY_UDP_LISTENER_ADDRESS=11003
    OVERLAY_TCP_LISTENER_ADDRESS=11004
    WC_adminhost=9060

其中，WC_adminhost是非安全管理控制台的端口号，WC_adminhost_secure是安全的管理控制台端口号。

6、登录WAS管理控制台（本次采用非安全方式）

    http://host_name_or_IP_address:9060/ibm/console/

**至此，WAS855环境搭建完毕**


2019/01/21 - 2019/2/12. Journey of IBM. GaoZhenz
> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI4ODk0MTg5XX0=
-->