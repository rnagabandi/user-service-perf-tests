# user-service-perf-tests

Objective:

o Scalable
o Sustainable
o Stability

Dev Set Up:


* Down load the JMETER from https://jmeter.apache.org/download_jmeter.cgi
* Extract the zip file
* Download the plugin manager JAR and place it in lib/ext folder
* Download the below plugins  from plugin manager 
	https://jmeter-plugins.org/wiki/ConcurrencyThreadGroup/
	https://jmeter-plugins.org/wiki/UltimateThreadGroup/
	https://jmeter-plugins.org/wiki/JMeterPluginsCMD/
	https://jmeter-plugins.org/wiki/ActiveThreadsOverTime/
	https://jmeter-plugins.org/wiki/PerfMon/
	https://jmeter-plugins.org/?search=jpgc-synthesis
	https://jmeter-plugins.org/?search=jpgc-filterresults
* Debugger plug in to debug the script from Jmeter UI
       https://www.blazemeter.com/blog/step-step-debugger-jmeter-          it%E2%80%99s-not-dream-anymore

Tuning the Jmeter:

Its very important to tune the JVM arguments so that the resources of JMeter are fully utilized to provide the best performance.

Heap Size: Increase the Heap size of JVM which JMeter will be using and set this parameter according to the resources allocated to the server. You can set this parameter in JMeter’s configuration file. Heap size is the space which JVM uses to create the necessary objects during the test run. By default, Its 512 Megabytes which is very less and your test might end up failing raising outofmemory error.
Tuning the Eden space parameter: If you are increasing the heap size then you should also tune the NEW parameter in JMeter startup script as it says the eden space should also be widened heap space is getting wider. It helps in decreasing the frequency of minor GC.
Add below parameters in jmeter/bin/jmeter.sh or jmeter/bin/jmeter.bat file

set HEAP=-Xms2048m -Xmx4096m
set NEW=-XX:NewSize=2048m -XX:MaxNewSize=4096m
set SURVIVOR=-XX:SurvivorRatio=8 -XX:TargetSurvivorRatio=50%
set TENURING=-XX:MaxTenuringThreshold=2
set PERM=-XX:PermSize=64m -XX:MaxPermSize=128m -XX:+CMSClassUnloadingEnabled


JMeter Cluster Set Up

JMeter was used to simulate load on the app server. Here we used JMeter distributed architecture, It has 1 master and X number of slave machine which generates the load on application server(Target Server).


* Identify the slave machines 
* Make sure master and slave machines are in same subnet
the firewalls on the systems are turned off.
* Make sure JMeter can access the server.
* Make sure you use the same version of JMeter on all the systems. Mixing versions may not work correctly.
* Make sure you use he same JDK version in master and slave machines.
* Set up the SSL in master machine
* Zip the Jmeter and copy to all slave machines to avoid the duplicate work and avoid hand shake issues
* Update the remote_hosts ip's in master machine (192.168.0.[10-14] are slave machines)
       remote_hosts=192.168.0.10,192.168.0.11,192.168.0.12,192.168.0.13,192.168.0.14


Execute below script in master and slave machine


#!/bin/bash
# Increase system file descriptor limit
sudo sysctl -w fs.file-max=100000
 
# Increase Linux autotuning TCP buffer limits
# Set max to 16MB for 1GE and 32M (33554432) or 54M (56623104) for 10GE
# Don't set tcp_mem itself! Let the kernel scale it based on RAM.
sudo sysctl -w net.core.rmem_max=33554432
sudo sysctl -w net.core.wmem_max=33554432
sudo sysctl -w net.core.rmem_default=33554432
sudo sysctl -w net.core.wmem_default=33554432
sudo sysctl -w net.core.optmem_max=25165824
sudo sysctl -w net.ipv4.tcp_rmem="87380 16777216 33554432"
sudo sysctl -w net.ipv4.tcp_wmem="65536 16777216 33554432"
 
# Make room for more TIME_WAIT sockets due to more clients,
# and allow them to be reused if we run out of sockets
# Also increase the max packet backlog
sudo sysctl -w net.core.netdev_max_backlog=50000
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=30000
sudo sysctl -w net.ipv4.tcp_max_tw_buckets=2000000
sudo sysctl -w net.ipv4.tcp_tw_reuse=1
sudo sysctl -w net.ipv4.tcp_fin_timeout=10
sudo sysctl -w net.ipv4.tcp_syncookies=1
sudo sysctl -w net.ipv4.tcp_synack_retries=3
sudo sysctl -w net.core.somaxconn=8096
 
# Increase ephermeral IP ports
sudo sysctl -w net.ipv4.ip_local_port_range="18000 65535"
#sudo sysctl -w net.ipv4.netfilter.ip_conntrack_tcp_timeout_time_wait=1
 
# Number of times SYNACKs for passive TCP connection.
sudo sysctl -w net.ipv4.tcp_synack_retries=2
 
# Disable TCP slow start on idle connections
sudo sysctl -w net.ipv4.tcp_slow_start_after_idle=0
 
# If your servers talk UDP, also up these limits
sudo sysctl -w net.ipv4.udp_rmem_min=8192
sudo sysctl -w net.ipv4.udp_wmem_min=8192
 
# Log packets with impossible addresses for security
sudo sysctl -w net.ipv4.conf.all.log_martians=1
