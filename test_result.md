# Sysbench oltp_common.lua(default)으로 데이터 로드 후 oltp.lua 테스트 

## 실험환경
- thread : 128


| Option   |  TPS | QPS |READ/S | WRITE/S  |LAT(ms.95%)|
|:----------:|-------------|-------------|-------------|-------------|-------------|
|nonsplit-innodb| 191 | 3828 | 414 | 2676 | 797 | 1903|
|myrocks| 24 | 1104  | 743 |  7.1| 91 -> 107 (16) |
|default-mysql| 38 |   1427 | 1046  |   | 109 -> 127 (18)|
|non-split| 83 | 3450  | 2013 |6.2 | 112 -> 147 (35) | 
