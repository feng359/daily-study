```c
一：
操作流程：应用程序操作A机中的MQ队列管理器，将消息发送到B机中的MQ队列管理器，B机中的应用程序从B中的MQ队列管理器中取出消息，进行业务处理，反之亦然。
    
1-一个队列管理器可以有多个队列和多个通道;队列管理器相当于MQ中的虚拟主机。

2-队列分为本地队列，远程队列，传输队列,用来存放取出消息；发送到本地队列上的消息存储在本机上；发送到远程队列上的消息，通过绑定传输队列传输到别的队列管理器上的本地队列上存储。

3-通道分为发送通道、接收通道；通道为消息进出队列的渠道桥梁，发送通道只能出，接收通道只能进。

4-常见的消息中间件：IBM Websphare MQ、TongLink/Q、RocketMQ、Kafka、MQTT ---消息中间件的一个重要作用是可以实现跨平台操作，为不同操作系统上的应用软件集成提供服务。
```

```c
四：本地接收队列的深度，接收消息的长度
    进入另一个同服务终端，进去队列管理器后执行 dis ql(B.A.RT.QL) 展示的属性CURDEPTH(val),中的val值就是当前的队列深度；消息长度直接浏览amqsbcg -----直接展示有Message下有length长度。

```

```C
服务器A:192.168.43.95:3022   用户名和密码：chk/EjpaDD2CSrn3bytV5dRZ8bL5kCJjkFPg  备用端口：6022
服务器B:192.168.43.95:4022   用户名和密码：mqm/pedsE10aaZr9pw0M3kk8XfBYtXfAz7Fq  备用端口：6033

服务器：172.168.65.158 ：1221  用户名和密码corpdsf/corpdsf 一样的
    
查看本机ip：ifconfig - ens33属于本机 -本机A:192.168.200.30 port 6022  本机B：192.168.200.40 port 6033
       
 termius ssh登录：  ssh chk@192.168.43.95 -p 3022 --在history中ADD TO HOST,方便下次直接在HOST中选中连接即可。

	注意：服务于crtmqm，strmqm等操作MQ管理器的命令依赖于同名可执行程序，这些同名程序在目录 /opt/mqm/bin目录下，使用这些命令之前需要先切换到相关目录下，一般：cd /opt/mqm/bin  再执行crtmqm 等命令；同样执行amqsput/发送、amqsbcg/浏览和amqsget/接收消息的命令依赖的同名程序在 /opt/mqm/samp/bin 目录下，在使用这些命令之前需要先切换到相应的目录下；一般一个服务器开两个终端，一个终端执行linux命令，另一个终端进入MQ控制台执行MQ管理器命令。
    
    
1-创建MQ队列管理器      				删除队列管理器：dltmqm QMBANK_A --先endmqm停止，再删除
   A机：crtmqm QMBANK_A -取名队列管理器不要取成MQ-message queue;而是见名知意取为QM-queue manager
   B机：crtmqm QMbank_B
	
2-启动队列管理器： strmqm QMBANK_A      停止MQ：endmqm -i QMBANK_A  --immediately立即停止，不加-i则处理完后再停止。
    
    dspmq  --显示队列管理器的运行状态
    dspmqver  --显示队列管理器的版本和其余详细信息
    
3-进入MQ控制台进行后续创建队列和通道等操作：runmqsc QMBANK_A   退出MQ控制台:exit或者end --直接在控制台使用
    
    -------注:上述123步骤命令均在linux平台执行，而不是进入MQ管理器中
    
4-创建队列：创建队列，通道等-创建监听（监听本机端口号）-启动监听-启动发送通道
        
5-发送浏览取出消息--浏览消息CURDEPTH不会改变，取走消息会改变CURDEPTH的值
   本机中发送消息到本地接收队列--发送：/opt/mqm/samp/bin amqsput B.A.RT.QL QM_A 本机操作，发送消息在本地队列，直接在本地队列读取即可-- 取出：amqsget B.A.RT.QL QM_A 
   本机利用远程队列发送消息到另一服务器MQ管理器中--需要事先建立通道和接收方创建开启监听打通通道running--发送：amqsput A.B.RT.QR QM_A --通过A机发送消息给A机的远程队列，消息经过A机的传输队列到达B机MQ管理器中的本地接收队列，此时浏览或取出消息应在B机的本地接收队列中--在B机中取出：amqsget A.B.RT.QL QM_B

        
常用命令：
        
    dis ql(*)查看所有队列，可能有很多系统队列，自己创建的队列在前面，dis ql(B.A.RT.QL)查看单个具体队列的详细信息；
    dis chl(*)显示所有通道，dis chl(B.A.RT.CHL)查看单个通道信息；
    dis chs(*)显示所有启动的通道状态，dis chs(A,B,RT,CHL)显示单个具体的通道状态信息；
        
    需要的命令用help帮助查看，打印相应的命令根据提示操作。
     
    修改现有通道定义：alter channel(...)...  修改传输通道的最大长度alter ql(A.B.RT.TRANS) MAXMSGL(修改值) 注意修改值是字节单位；
        
    删除通道：delete chl(A.B.RT.CHL) 
        
    删除监听：删除监听如果提示正在使用，应该先stop listener(MQ_A.LISR) 再delete listener(MQ_A.LISR)
  
    依据错误码，查看MQ的错误信息，如2085：mqrc 2085
        2035 -权限认证问题，注意关闭CHLAUTH后，进行正常的操作还出现2035，需要将定义的服务器连接通道的mcauser属性设置成mqm-- 需要将通道的 MACUSER 参数设置为 mqm , mqm 是客户机投印在本地 MQ 的用户 , 在 Linux 上安装 MQ 时 , 都会默认新建这么一个用户 , 因此我通过命令
    
		2058 -name err 说明操作的管理器不对，client模式操作的是远程管理器而不是本地的-eg:本该client模式本该操作远程队列管理器却操作了本地队列管理器,或者操作正确而没有设置环境变量，重新export MQSERVER="svrconn.T1.CHL/TCP/192.168.200.40(6033)"

		2085 -unknown object-eg:操作的队列不对
    
		2540 -没有定义通道或者通道定义有问题
            
	ALTER CHANNEL(QM_CHANNEL) CHLTYPE(SVRCONN) MCAUSER(‘mqm’)
        
    关闭通道权限: ALTER QMGR CHLAUTH(DISABLED)  --ENABLED表示开启状态
     
    查看程序的依赖库：ldd amqsput --amqsput是可执行文件
    
    查看端口连接情况： netstat antlq | grep 1414  --linux使用
    
```

```c
	创建监听后，另一端口查看端口有没有这个端口 netstat antlP | grep 1414，此时仅仅是创建监听没有开始监听，不会显示端口，启动监听后，再次查看，此时会显示这个端口--启动这个端口后，别人就可以连接它--此时dis chs(*)查看通道的状态，dis chs(A,B,BT,CHL)--channel st通道at缩写,查看具体通道的状态信息，没有任何状态信息，通道没有进行触发就是不可知状态，start chl()启动发送通道后，由于B机没有创建监听和启动，通道连接不上，再次dis chs(),会显示status为retrying,直到B机创建监听并启动监听后，A机的发送通道状态才会变为running打通状态。
    
	start 启动通道是启动的发送通道（SDR），接收通道没必要启动，一旦A方启动发送通道，对应的B方接收通道（RCVR）会自动激活；
 
	一方启动发送通道后，如果另一方没有start启动监听，此时发送通道会处于retrying状态，直到另一方start启动监听后，再次dis chs(*)状态会变为running表示连通；
        
```

```c
服务器A：
    
#创建本地实时接收队列
    define ql(B.A.RT.QL)   
#创建本地批量接收队列
    define ql(B.A.BT.QL)
    
#创建实时接收通道，另一方服务器B发送通道和此通道名相同
    define channel(B.A.RT.CHL) chltype(RCVR) trptype(tcp)
#创建批量接收通道，另一方发送通道和此通道名相同
    define channel(B.A.BT.CHL) chltype(RCVR) trptype(tcp)
    
#创建实时发送传输队列
    define ql(A.B.RT.TRANS) usage(xmitq)
#创建批量发送传输队列
    define ql(A.B.QT.TRANS) usage(xmitq)
    
#创建实时发送通道，另一服务器B接收者通道名和此通道名相同
    define channel(A.B.RT.CHL) chltype(SDR) conname('192.168.200.40(6033)') xmitq(A.B.RT.TRANS) trptype(tcp)  --注意：此处ip和port为另一方服务器B的，ip通过config查看，名称ens33下的ip即为本机ip，端口自己任意指定。
#创建实时发送通道，另一服务器B接收者通道名和此通道名相同
    define channel(A.B.BT.CHL) chltype(SDR) conname('192.168.200.40(6033)') xmitq(A.B.BT.TRANS) trptype(tcp)
   
#创建实时发送远程队列，rname为对方接收者（服务器B）的MQ队列管理器的本地接收队列名，rqmname为对方服务器B的MQ队列管理器名；--发送到远程队列上的消息，通过绑定传输队列传输到别的队列管理器上的本地队列上存储。
	define qr(A.B.RT.QR) rname(A.B.RT.QL) rqmname(MQ_B) xmitq(A.B.RT.TRANS)
#创建批量发送远程队列
	define qr(A.B.BT.QR) rname(A.B.BT.QL) rqmname(MQ_B) xmitq(A.B.BT.TRANS)
    
#创建监听
    denfine listener(MQ_A.LISR) trptype(tcp) control(QMGR) port(6022) --监听的port为本机端口
    
#启动监听
    start listener(MQ_A.LISR) 
    
#启动实时发送通道(对方服务器B需启动监听之后，否则处于retrying状态，正常是running状态表示连通)
    start chl(A.B.RT.CHL)
#启动实时发送通道(对方服务器B需启动监听之后，否则处于retrying状态，正常是running状态表示连通)
    start chl(A.B.BT.CHL)
    
    
-----------------------------------------------------------------------------------------
    
    
服务器B：
    
#创建实时发送传输队列
    define ql(B.A.RT.TRANS) usage(xmitq)
#创建批量发送传输队列
    define ql(B.A.QT.TRANS) usage(xmitq)
  
#创建实时发送通道，另一服务器B接收方通道名和此通道名相同
    define channel(B.A.RT.CHL) chltype(SDR) conname('192.168.200.30(6022)') xmitq(B.A.RT.TRANS) trptype(tcp)  --另一方服务器A的ip和port
#创建实时发送通道，另一服务器B接收者通道名和此通道名相同
    define channel(B.A.BT.CHL) chltype(SDR) conname('192.168.200.30(6022)') xmitq(B.A.BT.TRANS) trptype(tcp)
    
#创建实时发送远程队列，rname为对方接收者（服务器B）的MQ队列管理器的本地接收队列名，rqmname为对方服务器B的MQ队列管理器名；--发送到远程队列上的消息，通过绑定传输队列传输到别的队列管理器上的本地队列上存储。
	define qr(B.A.RT.QR) rname(B.A.RT.QL) rqmname(MQ_B) xmitq(B.A.RT.TRANS)
#创建批量发送远程队列
	define qr(A.B.BT.QR) rname(A.B.BT.QL) rqmname(MQ_B) xmitq(B.A.BT.TRANS)

#创建本地实时接收队列
    define ql(A.B.RT.QL)   
#创建本地批量接收队列
    define ql(A.B.BT.QL)
    
#创建实时接收通道，另一方服务器B发送通道和此通道名相同
    define channel(A.B.RT.CHL) chltype(RCVR) trptype(tcp)
#创建批量接收通道，另一方发送通道和此通道名相同
    define channel(A.B.BT.CHL) chltype(RCVR) trptype(tcp)
    
#创建监听
    denfine listener(MQ_B.LISR) trptype(tcp) control(QMGR) port(4022)  --port为本机端口
    
#启动监听
    start listener(MQ_B.LISR) 
    
#启动实时发送通道(对方服务器A需启动监听之后，否则处于retrying状态，正常是running状态表示连通)
    start chl(B.A.RT.CHL)
#启动实时发送通道(对方服务器A需启动监听之后，否则处于retrying状态，正常是running状态表示连通)
    start  chl(B.A.BT.CHL)    
    
 
    
```

```c
	CURDEPTH 队列的深度不是指消息的条数，而是指基于mq消息的大小段--有时候发一个大报文过来，此时CURDEPTH等于好几个数。
    amqsget取出消息时如果没有其余的消息，则在一定时间后会自动退出界面。
```

```c
amqsput的api和MQ队列管理器放在一起，若两者不在一起，则涉及到两种开发模式，即bind模式和client模式。

 bind模式--也可理解为本地模式；应用程序与MQ服务端必须位于同一台机器。
    
 client模式--也可理解为客户端服务器模式；应用程序作为客户端连接MQ服务端。
	client模式1--B机设置服务器连接通道SVRCONN，A机设置环境变量MQSERVER，最后编译改变库链接。此种模式源代码无需进行任何修改，只需修改链接选项即可，将服务端的MQ动态库			  (libmqm_r.so)，更新成客户端链接库(libmqic_r.so),同时设置MQSERVER环境变量即可，依赖于MQSEVER环境变量；--缺点：依赖于MQSERVER环境变量，在同时需要操作多台MQ服务器时不够灵活。
    
	client模式2--同样需要SVRCONN,A机修改bind源码，链接不同库。既可以以客户端方式连接并操作MQ，又可以同时连接操作多台MQ的方式，已解决日常生产应用的复杂情况；--两步改造：1-连接MQ队列管理器选项的修改；2-链接程序时使用客户端的库。
   
  
    源代码-/opt/mqm/samp目录下：eg:amqsput属于amqsput0.c -- 连接队列管理器-打开队列-往队列中放消息-关闭队列-断开队列管理器连接
    发送程序尾名带c的表示是client模式
    _r可有可无，主要是表示线程安全，在多线程中加上，否则无缘无故出现coredump.
    
    bind模式是应用程序在B机和B机中的队列管理器在一起，从B机发送消息到本地队列 amqsput A.B.RT.QL QM_B 应用程序在B机，在B机执行命令
   
    应程序放在A机，是为了模拟应用程序和队列管理器不在一个机器，以后应用程序执行命令在A机操作B机的队列管理器，而bind模式是应用程序在B机和B机中的队列管理器在一起；client模式是应用程序在A机操作B的队列管理器发送消息到B机的本地队列 amqsputc A.B.RT.QL.QM_B 应用程序在A机，在A机执行命令；应用程序还可在A机操作B的队列管理器 amqsputc B.A.RT.QR QM_B  这样A机中应用程序操作B机中队列管理器将消息发送到A的B.A.RT.QL本地队列
    
    
    svrconn 服务器连接通道：服务器连接通道就是给MQ客户端连接进来的一个标识入口,它和其他通道不一样,它是不需要启动的,如果有MQ客户端成功地通过这个服务器连接通道连接进来,它的状态就是活动的了.其他的通道类型可能需要执行启动命令来变成活动,这种通道活动以后,有一个真实的通道进程启动起来,服务器连接通道是没有相应的通道进程的.
 
    
    client模式1：
    
 #在B机上进入QM，建立服务器连接通道：
    define channel(SVRCONN.T1.CHL) chltype(SVRCONN) mcauser('mqm')
    
 #在A机上设置环境变量，在linux端 /opt/mqm/samp/bin进行如下操作：
    export MQSERVER="SVRCONN.T1.CHL/TCP/192.168.200.40(6033)" --B机的 服务连接通道名和指定TCP协议 和B机 ip和port
    执行 amqsputc A.B.RT.QL MQ_B  --报错2035，认证问题
    
#进入MQ_B
    
1-查看认证开关：DIS QMGR

2-关闭通道认证：ALTER QMGR CHLAUTH(DISABLED)
 
3-8.0开始必须设置认证选项可选或直接使用 ALTER QMGR CONNAUTH(' ')置空
    ALTER AUTHINFO(SYSTEM.DEFAULT.AUTHINFO.IDPWOS) AUTHTYPE(IDPWOS) CHCKCLNT(OPTIONAL)
    
4-刷新认证缓存：REFRESH SECURITY TYPE(CONNAUTH)
 


为通道设置 MCAUSER 参数 , 这里有一个需要注意的点是 mqm 用户需要在 mqm 分组里 , 只有在 mqm 分组中的用户才有权限操作 MQ。ps: 可以通过 cat /etc/group 查看分组 , 通过 cat /etc/passwd 查看用户
    

#链接不同的库，编译成bind和client1两种模式    
	export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/mqm/lib64
	gcc -o ~/amqsput_c amqsput0.c -I/opt/mqm/inc -L/OPT/MQM/lib64 -lmqic_r  将源程序编译成client模式1
	gcc -o ~/amqsput_b amqsput0.c -I/opt/mqm/inc -L/OPT/MQM/lib64 -lmqm_r   将源程序编译成bind模式
    可以看出只是链接的库不同，一个为lmqic_r,一个为lmqm_r
    
  
    client模式2：1-添加代码；2-更改链接库
        
#修改代码：
        cd ~/  将bind源代码amqsput0.c(在/opt/mqm/samp 目录下拷贝一份到 ~/ 目录) -- cp amqsput0.c amqsput_client --vim amqsput_client 添加代码：
        
        :set nu - 
        :77 在#include<cmqc>下一行添加：   #include<cmqxc>
        :88 在主函数main里面 MQCNO cno .....下添加：
            MQCD ClientConn = {MQCD_CLIENT_CONN_DEFAULT};
        :158 在if(argc>2) ...下添加4行：
            strncpy(ClientConn.ConnectionName),
					"192.168.200.40(6033)",     //B机的ip和port
					MQ_CONN_NAME_LENGTH);

		    strncpy(ClientConn.ChannelName),
					"SVRCONN.T1.CHL",          //B中定义的服务器连接通道
					MQ_CHANNEL_NAME_LENGTH);
        
         	cno.ClientConnPtr = &ClientConn;  //前面第88行下添加的选项ClientConn;cno为Connect_options的缩写
			cno.Version = MQCNO_VERSION_2;

#更改链接库,最终编译成client2模式：
	先查看先前的环境变量是否存在 echo $MQSERVER  若内容为空可直接在当前操作，若不为空，则另开终端，切换到同样的目录下操作，进行如下操作：
        export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/mqm/lib64
        gcc -o ~/amqsput_client aqmsput_client.c -I/opt/mqm/inv -L/opt/mqm/lib64 -lmqic_r
    
        在A机操作:amqsput_client A.B.RT.QL MQ_B 测试，B机器；amqsget A.B.RT.QL MQ_B查看
    
    
    dis qmgr
    
    dis ql(B.A.RT.QL) 显示的MAXMSGL(4194304) 4194304字节为4M，
    alter ql(qmname) MAXDEPTH(100000)   //增大配置队列深度
    
```

