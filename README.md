# blockchain_test

1. 解压缩安装包

2. 检查服务器的端口30300-30303，20200-20203
	- 如果端口被占用，分别修改nodes/127.0.0.1/node0，nodes/127.0.0.1/node1，nodes/127.0.0.1/node2，nodes/127.0.0.1/node3中的config.ini中

    [p2p]
    		        listen_port=
		[rpc]
    			listen_port=

3. 运行nodes/127.0.0.1/start_all.sh，在本地建立一个四节点的区块链测试网络。启动成功会输出如下信息：
   
		try to start node0
		try to start node1
		try to start node2
		try to start node3
 		node3 start successfully pid=36430
 		node2 start successfully pid=36427
 		node1 start successfully pid=36433
 		node0 start successfully pid=36428

5. 检查进程是否启动  ps aux |grep -v grep |grep fisco-bcos。正常情况会有类似下面的输出； 如果进程数不为4，则进程没有启动（一般是端口被占用导致的）：

		fisco        35249   7.1  0.2  5170924  57584 s003  S     2:25下午   0:31.63 /home/fisco/nodes/127.0.0.1/node1/../fisco-bcos -c config.ini -g config.genesis
		fisco        35218   6.8  0.2  5301996  57708 s003  S     2:25下午   0:31.78 /home/fisco/nodes/127.0.0.1/node0/../fisco-bcos -c config.ini -g config.genesis
		fisco        35277   6.7  0.2  5301996  57660 s003  S     2:25下午   0:31.85 /home/fisco/nodes/127.0.0.1/node2/../fisco-bcos -c config.ini -g config.genesis
		fisco        35307   6.6  0.2  5301996  57568 s003  S     2:25下午   0:31.93 /home/fisco/nodes//127.0.0.1/node3/../fisco-bcos -c config.ini -g config.genesis

6. 进入spring-boot-crud文件夹，打开src/main/resources/applicationContext.xml，根据之前设定的rpc端口修改（默认为20200，20201若无修改则不用改动）。

				<entry key="peers">
					<list>
                        <value>127.0.0.1:20200</value>
                        <value>127.0.0.1:20201</value>
					</list>
   

8. src/main/resources/application.yml文件是WebServer，主要配置了监听端口，默认为45000。

9. 执行以下命令：
		# 编译项目
		$ bash mvnw compile

		# 安装项目，安装完毕后，在target/目录下生成fisco-bcos-spring-boot-crud-0.0.1-SNAPSHOT.jar的jar包
		$ bash mvnw install

		# 启动spring-boot-crud(启动成功后会输出create client for group 1 success的日志)
		$ java -jar ./target/fisco-bcos-spring-boot-crud-0.0.1-SNAPSHOT.jar

10. 使用curl工具访问接口如下：
		curl -H "Content-Type: application/json" -X POST --data '{"userName":"Kim","userID":123456,"resourceUsage":800,"timestamp":1719528661,"charge":10000}' http://localhost:45000/store

		返回示例：
		{"blockNumber":"157","isSucceed":true,"transactionHash":"0x333340b50a0b286fa504f706e80aa895824305a77e829208d77793e232cace88","timestamp":1719527661}

		curl -H "Content-Type: application/json" -X GET --data '{"blockHeight":"157","txHash":"0x333340b50a0b286fa504f706e80aa895824305a77e829208d77793e232cace88"}' http://localhost:45000/proof

		返回示例：
		{"txProof":[],"rootHash":"0x333340b50a0b286fa504f706e80aa895824305a77e829208d77793e232cace88","isSucceed":true}
		
		注：当交易并发量不大时，一个block内记录一笔账单，此时txProof为空，只需要对照区块头中的rootHash是否于txHash相同即可，可靠性由区块链的链式结构保证。
		       当交易并发量大时，一个block内记录多笔账单，此时txProof是将这些交易组装成merkle tree的形式，可靠性由计算出的merkleRoot和rootHash是否相同保证。

		


