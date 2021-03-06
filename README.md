# CCNP OSPF 2
Open Shortest Path First

# 實際操作

基本設定不像是 EIGRP 那樣有 Cisco Router 支援，而是眾多廠牌的路由器均有支援 OSPF，這使得 ISP 端大量採用 OSPF。

     #router ospf + < process-id >
     
     #network + < ip add > + < wildcard-mask area > + < area-id >
 
(1) 單一區域的 OSPF 設定



      如下均在 single area 內


             R2
            /  \                                                                             10.2.2.1 - R
     10.1.1.1. 10.1.2.1                                                                     /
     10.1.1.2  10.1.2.2                                                                    /   
         /         \                                                                      /
        R3          R1(TPE)- 10.64.0.1 ------------- WAN ----------- 10.64.0.2 -(Tainan) R                        
         |           |                                                                    \
        10.1.3.1 10.1.3.2                                                                  \
                                                                                            \
                                                                                             10.2.3.2 - R
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
     
 (4) 顯示鄰近訊息
 
     R1#sh cdp ne
     
     DeviceId      Local interface    Holdtme       Capability      Platform    Port  ID
     
     R2                 s0                             R
     R3                 s1                             R
     R-tainan           e0                             R
     // capability:
     // R - Router
     // B - Source Route Bridge
     // S - Switcher
     // H - host
     // I - IGMP
     // r - repeater
 
 (5) 利用 area-id 查看鄰近路由器資訊
      
      R1#sh ip ospf neighbor detail
      
      Neighbor 10.64.0.2
      in the area 0 via interface E0
      Neighbor Priority is 1, state is full.
      
      Neighbor 10.1.3.1
      in the area 0 via interface s0
      Neighbor Priority is 1, state is full.
      
      Neighbor 10.1.2.1
      in the area 0 via interface s1
      Neighbor Priority is 1, state is full.
      
 (6) 顯示 ospf 的 LSA (Link state db) 鏈結狀態資料庫
 
       R1#sh ip ospf database
       
       ospf router with ID (router id is 10.64.0.1) (process id is 1)

       
          
            Router Link State (Area 0)
       
       Link ID      ADV Router        Age     Seq#     Checksum      Link count
       
     10.64.0.1                                                       4
     10.64.0.2                                                       4
     10.1.2.1                                                        4
     10.1.3.1                                                        4
     10.2.2.1                                                        5
     10.2.3.2                                                        5
       
       
       
              Net Link State (Area 0)
       
       Link ID      ADV Router        Age     Seq#     Checksum     
       10.64.0.2   10.64.0.2          80    0X80000001   0X7990
       
 (7) 查看 R1 學習到的路徑
 
      R1#sh ip route
      codec: O - OSPF
      
      10.0.0.0/24 is subnetted, 7 subnets
      
      C  10.1.3.0 is directly connected, s0
      C  10.1.2.0 is directly connected, s1
      C  10.64.0.0 is directly connected, e0
      O  10.1.1.0  via 10.1.3.1, s0
                   via 10.1.2.1, s1
      O  10.2.3.0  via 10.64.0.2, E0
      O  10.2.2.0  via 10.64.0.2, E0
      O  10.2.1.0  via 10.64.0.2, E0
      
  (8) 觀看 routing info 訊息
      
      
      R1# sh ip protocol
      
      Redistributing: ospf
      Routing for networks:
      10.1.3.2/32
      10.1.2.2/32
      10.64.0.1/32
      
      Routing info Source
      
      GW          Distance       Last Update
      10.1.2.1     110
      10.2.3.2     110
      10.1.3.1     110
      10.2.2.1     110
      10.64.0.2    110
      
      distance : (default is 110)

  (9) 詳見步驟 7, 能觀察到路由學習時，經過 10.1.1.0 網段的，可以透過 s0 出去經過 10.1.3.1 的 hops 或是 10.1.2.1 的 hops
  
        O  10.1.1.0  via 10.1.3.1, s0
                     via 10.1.2.1, s1
   
  (10) 調整介面成本 cost, 可藉由手動下指令，讓路由被動的選擇最佳路徑
  
       R1#conf t
       R1(config)#int s0
       R1(config-if)# ip cost 50
       // 將此介面降低成本 50
       
       ctrl + Z
       
      
   (11) 檢視結果
   
        R1#sh ip route
        
      10.0.0.0/24 is subnetted, 7 subnets
      
      C  10.1.3.0 is directly connected, s0
      C  10.1.2.0 is directly connected, s1
      C  10.64.0.0 is directly connected, e0
      O  10.1.1.0  via 10.1.3.1, s0 // s1 介面相對 s0 成本較高，故被過濾掉。
      O  10.2.3.0  via 10.64.0.2, E0
      O  10.2.2.0  via 10.64.0.2, E0
      O  10.2.1.0  via 10.64.0.2, E0
        
       
   
   

