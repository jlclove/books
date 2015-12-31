Spring Boot 入门以及进阶
主要面向上海链家SE开发团队

com.lianjia.sh.loupan.spi
   v1
     model
     UserServiceSpiV1     /no feign clients
   v2
     model
     UserServiceSpiV2
     
    @FeignClient(“loupan-server”)
    UserSpi  extends UserServiceSpiV1,UserServiceSpiV2
    
    
  com.lianjia.sh.loupan.server
   controller
       v1
         UserServiceControllerV1
       v2
         UserServiceControllerV2
   dao
     only dao-management / readOnly/default
   service
      zoning:
         UserDao
         UserService
      core:
         LoupanDao
         LoupanService
   config
   common
    
     