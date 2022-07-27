tmp:

```c++

pdd-概要设计-概要设计说明书：各个具体业务的详细说明
pdd-测试-测试总结报告-支付业务、信息业务、业务查询三大模块核心功能。
pdd-接口文档-各个报文格式。
招商银行：
	单笔代付：211 line
			按人行对账日期对账
			按时间间隔对账 331 line
	批量代付：1027 line
			按人行对账日期对账
			按时间间隔对账
	批扣单笔代付：1316 line
			按人行对账日期对账
			按时间间隔对账1448 line
			
			
	accCHK(){  //业务处理入口
			
	    std::vector<CheckUnit> vecCheckUnit = findGroup(); //CheckUnit是一个结构体，里面有企业代码，对账日期两个数据成员

        CheckUnit strCheckUnit;
		//重点：上面用动态数组来存储CheckUnit元素，然后循环遍历数组里面的元素，每一次循环中对每一个元素调用三个生成对账文件的函数进行相应的处理。
        for(size_t i = 0; i < vecCheckUnit.size(); i++)
        {
            strCheckUnit = vecCheckUnit[i];
            ULog(eTRACE, "");
            ULog(eTRACE, "BEGIN***************对账日期[%s]委托方[%s]开始对账			    ***************BEGIN", strCheckUnit.m_sCheckDate.c_str(),                      				     	strCheckUnit.m_sCorpId.c_str());
            ULog(eTRACE, "");
            //生成(上一日的)调整文件
            adjustFile(strCheckUnit);   

            //doCheck(strCheckUnit);
            //生成(当前日)对账文件
            recountFile(strCheckUnit);  //内部主要包含单笔代付、批量代付、折扣单笔代付(每一个代付里面包括 按人行对账时间对账 和 按时间间隔对账两种方式)

            //生成(当前日)退汇文件
            returnFile(strCheckUnit);
            ULog(eTRACE, "");
            ULog(eTRACE, "END*****************对账日期[%s]委托方[%s]结束对账*****************END", strCheckUnit.m_sCheckDate.c_str(), strCheckUnit.m_sCorpId.c_str());
            ULog(eTRACE, "");
        }
			
}


乱码：输出乱码--更改系统编码格式； log乱码--使用vim指令打开
服务启动失败：
	--main.sendfe_mpss.com.log-数据库连接失败
 	-- merchant.rcv.ws.log-获取商户公匙失败
    --com.szfs.rcv.com.log-MQ客户端初始化失败
    --com.szfs.batch.rcv.com.log-MQ客户端初始化失败
	-- main.recvfe.com.log-连接队列管理器[QMCORP]失败
    --main.recvfe.batch.com.log-连接队列管理器[QMCORP]失败
    
更改了/home/corpdsf/product/Linux64/etc/current
     dbconfig.xml.bak   -ip
     dbconfig_oracle.xml
     dbconfig_db2.xml   dbconfig.xml
/home/corpdsf/product/Linux64/etc/current/services/main
	service.main.recvfe.batch.com.xml  -ip
    service.main.recvfe.com.xml
    service.main.sendfe_mpss.com.xml
	service.main.sendfe_mpss.com.xml.bak20211011

  

    








```

```c++
中国银行：
 深同二验收文档：
    改造需求说明书：13-12-06
    测试案例：14-2-20
    操作手册：14-1-26
    数据库设计：13-11-19
 验收文档：
    系统技术方案：3.系统概述(接口定义) -13-10-10 对应代码段：bocs-api-fsxml-include
    
    
功能：string提供的方法：trim() 去掉字符串首尾空格 -对象调用；
     to_string(val)将整形转换为字符串型 -单独使用；
     cstdlib头文件提供的将一个字符串转换为整数： atoi(str) -单独使用
    
    MFC提供的CString类型处理串，其中对象调用方法GetBuffer()和GetBuffer(0)效果一样，都是指向原来CString对象，GetBuffer(val)当val的长度小于原字符串本身长度时，则不分配内存，还是指向原字符串；当val长度大于原字符串本身长度时就要重新分配一块比较大的空间出来。
    
问题：xml文件需要完全掌握解析吗
    
   
    
```

