# CCNP OSPF 2
Open Shortest Path First

# 實際操作

基本設定不像是 EIGRP 那樣有 Cisco Router 支援，而是眾多廠牌的路由器均有支援 OSPF，這使得 ISP 端大量採用 OSPF。

     #router ospf + < process-id >
     
     #network + < ip add > + < wildcard-mask area > + < area-id >
 
(1) 單一區域的 OSPF 設定



      如下均在 single area 內


             R2
            /  \                                                                             R
     10.1.1.1. 10.1.2.1                                                                     /
     10.1.1.2  10.1.2.2                                                                    /   
         /         \                                                                      /
        R3          R1(TPE)- 10.64.0.1 ------------- WAN ----------- 10.64.0.2 -(Tainan) R                        
         |           |                                                                    \
        10.1.3.1 10.1.3.2                                                                  \
                                                                                            \
                                                                                             R
 (2) R1 設定
 
     R1#conf t
     R1(config)#ospf 1
     R1(config-router)#network 10.1.3.2 0.0.0.0 area 0
     R1(config-router)#network 10.1.2.2 0.0.0.0 area 0
     R1(config-router)#network 10.64.0.1 0.0.0.0 area 0
     ctrl + Z
     
     
 (3) 下指令 sh ip protocol 藉以檢視是否設定完成
 
     R1#sh ip protocol
     
     Routing Protocol is "ospf 1" 
     // 這台路由器的 process-id is 1
     
     Redistributing: ospf 1
     Routing for Networks:
     10.1.3.2/32
     10.1.2.2/32
     10.64.0.1/32
     // 目前尚未設定好其他路由器，所以尚未有大量資訊。（如 ）
     
     
