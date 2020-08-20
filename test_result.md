# Sysbench oltp_common.lua(default)으로 데이터 로드 후 oltp.lua 테스트 

mysql-5.7.24(mysql/innodb), mysql-5.6(myrocks), sysbench 1.0.12

## 실험환경
- thread : 128
- memory: 5G
- warehouse: 1000
- run time: 72h


| Option   |  TPS | QPS |READ/S | WRITE/S  |
|:----------:|-------------|-------------|-------------|-------------|
|nonsplit-innodb| 177 | 3553| 2441 | 691 | 
|myrocks| 32 |   642 | 449  | 128  | 
|default-mysql| 45 |  913 | 639  | 182  |


