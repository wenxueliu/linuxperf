
系统基本工具集

1. TCP 各种连接状态情况统计

    ss -ant | awk '{++s[$1]} END {for(k in s) print k,s[k]}'

2. 查看某个进程打开连接

    while true; do lsof -i 4 -a -p PID; done

    while true; do lsof -i 6 -a -p PID; done
