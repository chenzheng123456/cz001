
----------------------------------------------------------------------I
第十六节课 2018年10月29日09:00:00
----------------------------------------------------------------------I
【进程和计划任务】
内核功用：进程管理、文件系统、网络功能、内存管理、驱动程序、安全功能等
多实例：一个程序运行多次

--资源使用的单位process
Process: 运行中的程序的一个副本，是被载入内存的一个指令集合
	进程ID（Process ID，PID）号码被用来标记各个进程
	UID、GID、和SELinux语境决定对文件系统的存取和访问权限
	通常从执行进程的用户来继承
	存在生命周期


--进程创建：
	init：第一个进程 centos6
	systemd：第一个进程 centos7
	进程：都由其父进程创建，父子关系
	CoW（写实复制，类快照） fork(), clone()


线程之间，由操作系统负责协调，及多个进程之间如何切换
协程（Python）：函数/代码的集合，多个指令分别用协程来管理

进程状态：创建_就绪_执行_阻塞_释放
创建→就绪（等分配cpu资源）→执行（时间片用完）→释放
		就绪（等磁盘I/O读写）←执行
		阻塞→进程（等I/O，MDA读取到内存）
				
PIO:磁盘文件要读到进程（内存）中，必须要经过cpu做转换
DMA:给DMA（设备）发指令，DMA直接把磁盘文件读取，放到内存中，节约CPU资源

内核缓冲区：buffer  
读取文件→传给buffer→传给用户
修改好文件→传给buffer→传给磁盘
系统有权限访问硬件，应用程序不能直接访问硬件



--系统优先级：数字越小，优先级越高
	          【0—99】 【100—139】CentOS4,5
	各有140个运行队列和过期队列数
	          【0—98】 【99        CentOS6
--realtime优先级：【99—0】 （实时优先级）
   值最大优先级最高 （优先级高的一直运行） FIFO机制
--nice优先级：（轮询）RR机制【-20—19】 （nice命令中用到）
--TOP（PR）RT-RT 0—39 （工具）

--big O：时间复杂度   时间↑ →数据 （坐标）
	O(1) 时间成本稳定不变   --最优
	O(logn)      缓慢增长	--次之
	O（n）       线性增长
	O(n^2)       抛物线增长 
	O(2^n)       指数增长
	
--进程相关概念
	页框：存储页面数据，page 4k (分配数据内存单位）
	LRU ：近期最少使用算法，释放内存物理地址空间和线性地址空间
	（线性空间→承诺空间  物理空间→实际空间）
	MMU ：负责转换线性和物理地址（CPU）（CPU↓）
	TLB ：翻译后备缓冲器,用于保存虚拟地址和物理地址映射关系的缓存

--IPC：进程间通讯
	同一主机：signal信号/shm共享内存/semaphore信号量，一种计数器
	不同主机：socket→ip和端口号
		  RPC 远程过程调用 
		  MQ  消息队列
	

【LRU】算法
--假设序列为 4 3 4 2 3 1 4 2
--物理块有3个，则
	第1轮 4调入内存 4
	第2轮 3调入内存 3 4
	第3轮 4调入内存 4 3
	第4轮 2调入内存 2 4 3
	第5轮 3调入内存 3 2 4
	...(内存命中率越高，效率越高）

早期dos:协作式多任务（进程在运行时，一直占用cpu，执行完毕才释放）
（↑导致中途无法处理别的时间，进程死掉，不能响应别的进程）

linux内核：抢占式多任务（基于时间片，如5ms，时间到立即分配给别的进程）
	
守护进程: daemon,在系统引导过程中启动的进程，和终端无关进程
前台进程：跟终端相关，通过终端启动的进程
（注意：两者可相互转化）

【进程状态】
	运行态：running
	就绪态：ready
	睡眠态：可中断interrruptable 不可中断uninterruptable
	停止态：stoped，暂停内存，但不会被调度，除非手动启动
	僵死态：zombie，结束进程，父进程结束前，子进程不关闭
（进程结束会释放所有资源，区别于僵死态）

【系统管理工具】
CPU-bound:密集型，非交互
IO-Bound:IO密集型，交互

Linux系统状态的查看及管理工具：pstree, ps, pidof, pgrep, top, htop, glance, pmap, vmstat, dstat, kill, pkill, job, bg, fg, nohup

【--pstree】 显示树进程 { }：表示线程
	-p 显示更详细

【--ps 】查看进程（/proc/进程编号PID）  ps aux使用较多
	ps → 显示当前所在终端开启的进程列表
	（三种类型风格：UNIX BSD GNU ;UNIX选项用的较多 ）
	如 unix：ls -l  GNU： ls --all  BSD：ps a
	fd文件：此进程打开的文件列表（描述符）

	--BSD 风格
	a 所有终端中的进程
	x 所有和终端无关的进程
	u 进程所有者的信息
	f 显示进程树,相当于--forest森林
	k|--sort属性对属性排序,属性前加-表示倒序
	o 显示定制的信息，如ps axo pid,%cpu,%mem,cmd
	L 显示支持的属性列表，对应关系


【ps常见选项】 ps -ef 使用较多
	-C cmdlist指定命令，多个命令用，分隔
	-L 显示线程
	-e: 显示所有进程，相当于-A
	-f: 显示完整格式程序信息
	-F: 显示更完整格式的进程信息
	-H: 以进程层级格式显示进程相关信息
	-u 生效用户 euser
	-U 真正用户 ruser
	-g gid或groupname指定有效的gid或组名称
	-G gid或groupname指定真正的gid或组名称
	-p pid显示指pid的进程
	--ppid pid  显示属于pid的子进程
	-M 显示SELinux信息，相当于Z
	-t 终端

【ps输出属性】
	VSZ: Virtual memory SiZe，虚拟内存集，线性内存
	RSS: ReSidentSize, 常驻内存集
	STAT：进程状态
	psr：跑在哪颗cpu上
	R：running 运行状态
	S: interruptable   sleeping  可中断
	D: uninterruptable sleeping 睡眠状态，不可中断
	T: stopped 停止状态
	Z: zombie  僵死状态
	+: 前台进程
	l: 多线程进程
	L：内存分页并带锁
	N：低优先级进程
	<: 高优先级进程
	s: session leader，会话（子进程）发起者

--taskset 可绑定cpu进程（编号）
	（111：第2颗cpu，第1颗cpu，第0颗cou ）
	p 4076 当前进程可运行cpu的列表 3:11→表示能跑在第1/0颗cpu上    
	-cp 0 4070 ，把4070进程绑定在第0颗cpu上


ni：nice值  -20—19（大部分优先级在此范围）
pri：数值越大，优先级越高 
rtprio: 实时优先级 99—0（大部分进程不运行在实时优先级） 
renice -n -20 4070 调整优先级 或（renice -20 4070）
  （程序光调整优先级不会明显改善，和程序设计也有密切关系）
示例：
ps axo pid,cmd,psr,ni,pri,rtprio
nice -n -10 ping x.x.x.x 指定-10优先级



【搜索进程】
--pgrep 按预定义的模式
	-t 显示指定终端相关的进程 -t pts/0
	-a 显示完整格式的进程名
	-P 指定进程的子进程 -P 6673
	-u 生效用户 euser
	-U 真正用户 ruser
	-l 显示进程名  

--pidof查看进程对应的编号（/sbin/pidof）
	例：pidof bash 查看bash的进程编号


--uptime 当前系统的时间 
	( 系统开启时间、多少用户登录，负载情况1/5/10分钟)
	（1核cpu 3 进程）

【--top】（进程管理工具）类似windos任务管理器__几秒刷新一次
   【us用户空间 sy系统/内核空间 ni调整优先级 id空闲 wa等待 】
    【hi硬中断 si软中断 st被盗取时间】
	==pr进程优先级 ==NI nice优先级

	排序：
	P：以占据的CPU百分比,%CPU 默认
	M：占据内存百分比,%MEM
	T：累积占据CPU时长,TIME+ 

	首部信息显示：
	uptime信息：l
	tasks及cpu信息：t
	cpu分别显示：1 (数字)
	memory信息：m

	退出命令：q
	修改刷新时间间隔：s
	终止指定进程：k
	保存文件：W   ~/.toprc
	颜色：Z ；z取消颜色

 --top进程管理工具
	选项：
	-d # 指定刷新时间间隔，默认为3秒
	-b 显示所有进程
	-n # 刷新多少次后退出
	-H 线程模式，示例：top -H -p `pidof mysqld`
			   top -H -p 575
			（显示当前线程信息）
	   htop命令：EPEL源

	选项：
	-d #: 指定延迟时间；
	-u UserName: 仅显示指定用户的进程
	-s COLUME: 以指定字段进行排序

	子命令：
	s：跟踪选定进程的系统调用
	l：显示选定进程打开的文件列表
	a：将选定的进程绑定至某指定CPU核心
	t：显示进程树
	
【--free 内存空间】
	-b 字节；-m M单位；-g G单位  默认k为单位
	-h 易读格式
	-o 不显示-/+buffers/cache行 （centos7 没有）
	-t 显示RAM + swap的总和
	-s n 刷新间隔为n秒
	-c n 刷新n次后即退出

	cached  数据读到内存中占用/面对读操作 
	buffers 数据从内存写到硬盘/面对写操作 缓存区

【内存工具】
--vmstat:显示虚拟内存信息
	vmstat 2 5 每2秒扫描一次，报告5次
	=procs  r：可运行进程的个数，3/核
		b：被阻塞的队列的长度

	=memory swpd：交换内存的使用总量
		free：空闲物理内存总量 
		buffer：用于buffer的内存总量
		cache：用于cache的内存总量

	=swap	si：从磁盘交换进内存的数据速率(kb/s)
		so：从内存交换至磁盘的数据速率(kb/s)	
 
	=io	bi：从块设备读入数据到系统的速率(kb/s)
		bo: 保存数据至块设备的速率
	
	=system in: interrupts 中断速率，包括时钟
		cs: context switch 进程切换速率

	=cpu	us：运行非内核代码时间
		sy：运行内核代码时间
		id：空闲时间
		wa：等待io时间
		st：从虚拟机窃取的时间
	-s: 显示内存的统计数据
	
--iostat:统计cpu和设备IO信息
	isstat 1 10 每1秒扫描一次，扫描10次

--pmap （1234）：查看进程使用内存情况→cat /proc/1234/map也能实现
	-x：显示详细格式的信息
	 1 每1秒扫描一次

【系统监控工具】
--glances （需要装epel源）→远程进程跟踪  两端都要装同样版本
	-s  -B ipaddress 开启 监控端 (-B 指定ip，不指定所有IP都能连）
	-c ipaddress     客户端
	
--dstat 系统资源统计，代替vmstat,iostat
	-c 显示cpu相关信息
	-C #,#,...,total
	-d 显示disk相关信息
 	-D total,sda,sdb,...
	-g 显示page相关统计数据
	-m 显示memory相关统计数据
	-n 显示network相关统计数据
	-p 显示process相关统计数据
	-r 显示io请求相关的统计数据
	-s 显示swapped相关的统计数据

--iotop 监视磁盘I/O使用状况
	第一行：Read和Write速率总计
	第二行：实际的Read和Write速率
	第三行：参数如下：
		线程ID（按p切换为进程ID）
		优先级
		用户
		磁盘读速率
		磁盘写速率
		swap交换百分比
		IO等待所占的百分比
		线程/进程命令

--lsof 查看当前系统文件的工具（查看进程使用情况）
	/dev/pts1 查看某个终端相关的进程
	-p 1234 查看指定的进程 
	-i : 80 查看80端口网络连接情况 ▲
	-a：查看存在的进程
	-c<进程名>：列出指定进程所打开的文件
	-g：列出GID号进程详情
	-d<文件号>：列出占用该文件号的进程
	+d<目录>：列出目录下被打开的文件
	+D<目录>：递归列出目录下被打开的文件
	-n<目录>：列出使用NFS的文件
	-i<条件>：列出符合条件的进程(4、6、协议、:端口、 @ip )
	-p<进程号>：列出指定进程号所打开的文件
	-u：列出UID号进程详情
	-h：显示帮助信息
	-v：显示版本信息。
	-n: 不反向解析网络名字
	
	【恢复删除文件】正在使用的文件
	lsof |grep /var/log/messages
	rm -f /var/log/messages
	lsof |grep /var/log/messages
	cat /proc/653/fd/6
	cat /proc/653/fd/6 > /var/log/messages


--kill 向进程发出控制信号 默认-15
	默认不加参数 终止某个信号；例：kill 1234
	-l 显示列表 （trap -l）

	-1 1234 重新加载该进程
	-2 1234 中止该运行的进程（退出）
	-3 1234 ctrl + \	 （退出）
	-9 1234 强制杀死进程
	（删除init进程后，再删除其他进程，无法恢复）
	-18 1234 继续运行
	-19 1234 后台休眠
	-0 1 安全检查 
	（↑eho $?）然后查看该进程是否有问题
	killall httpd 删除该名称所有进程

	pkill（共享选项）
	-u 生效用户 例：pkill -u wang
	-U 真实用户，真正发起运行命令者 例：pkill -U wang
	-t terminal: 与指定终端相关的进程 例：pkill 
	-l: 显示进程名（pgrep可用）
	-a: 显示完整格式的进程名（pgrep可用）
	-P pid: 显示指定进程的子进程

【作业管理】
让作业运行于后台
(1) 运行中的作业： Ctrl+z （停止作业→后端）
		jobs 可查看停止的作业
		bg 1 后端运行；可在前端执行其它命令
		fg 1 前端运行；无法再执行其它命令
(2) 后端执行作业： COMMAND & 例：ping 127.1 &

--fg * 前端执行
--bg * 后端运行
--killall * 终止指定的作业/18 继续运行 /19 后台休眠

--剥离与终端的关系，防止前端崩溃，可用于备份
screen -S bak开启  -r 恢复终端
nohup cmd > /dev/null & 后台运行


--并发运行(同时运行多个进程，提高效率)
方法1: →可ps aux查询，然后killall杀掉进程
	vi all.sh
	f1.sh&
	f2.sh&
	f3.sh&
方法2:
(f1.sh&);(f2.sh&);(f3.sh&)
方法3:
{ f1.sh& f2.sh& f3.sh& }

例：两组命令同时执行
{ { ping 127.1;ping 127.2 }& ; { ping 127.3;ping 127.4 }& };
	

【任务计划】 标准输出的信息会发邮件给该用户▲
--at适合做一次性任务 （/var/spool/at/  自动保存在目录）
	-V 版本
	-t tinme 时间格式 [[世纪]年]月日时分[.秒]
	-l 查看计划任务编号
	-d 删除指定的作业
	-c （任务编号） 查看任务内容
	-f 指定文件中读取任务
	-m 任务执行完成，发空邮件给用户，表示已执行
	now+10minutes/hours/days/weeks 
	10分钟/小时/天/周 后执行
	
	
例： at 18:00 2018-10-31 <<EOF
	rm -rf /data/*
	poweroff
	EOF(ctrl +d)
	-----------------------
     echo 'poweroff;touch /data/at.log' | at 18:00
	
    (ctrl +d)退出；at -l查看任务；at -c 1查看内容
	
▲自动存放目录 /var/spool/at
  确认任务是否开机启动↓
systemctl is-enabled atd crond→查看这两服务是否开机启动   centos7
  chkconfig  --list atd（0-6编号） 3/5模式 on开启 off关闭  centos6
service crond status 确认该计划任务是否启动
service atd status  确认该计划任务是否启动

touch /etc/at.{allow,deny} ； 输入用户名
控制用户是否能执行at任务

白名单：/etc/at.allow 默认不存在，只有该文件中的用户才能执行at命令
黑名单：/etc/at.deny 默认存在，拒绝该文件中用户执行at命令，而没有在at.deny文件中的使用者则可执行 
如果两个文件都不存在，只有 root 可以执行 at 命令



--batch 系统自行选择空闲时间去执行特定任务

--cron  周期性执行任务   默认：或的关系
	systemctl status crond 查看依赖crond.service是否启动
	cat /var/log/cron:周期性计划任务日志信息

【vim /etc/crontab】 普通用户：crontab -e 格式↓
* * * * * 分/时/日/月/周/  用户  cmd
 
其他用户：crontab -e 打开该文件，不用在添加用户
crontab -e -u wang打开wang用户的计划任务
export EDITOR=vim↑文本添加颜色 （要完全生效写到文件 ）
计划任务文件保存

例：30 2 * 3-6,12 0 root /bin/tar cf /data/etc.tar  /etc/ &> /dev/null
    每3-6月,12月的每天2：30，打包/etc目录到/data/etc.tar
*/ * * * * 每10分钟 
10 * * * * 每个小时的第10分钟
30 4 1,15 * 0  1号，15号的4:30，或周日执行
@reboot  下次重启后执行
@yearly  每年执行
@monthly 每月执行
@weekly  每周执行
@daily   每日执行
@hourly  每小时执行

【任意键 a 1：进入单用户（维护）模式，计划任务不会执行】centos 6
[ `date +%w` eq 5 ]

【命令总结】
pstree:显示树进程
ps：查看进程
taskset：可绑定cpu进程
pgrep：按预定义模式搜索进程
pidof查看进程对应的编号
uptime：当前系统的时间
top：进程管理工具
free：内存空间
vmstat:显示虚拟内存信息
iostat:统计cpu和设备IO信息
pmap：查看进程使用内存情况
glances：远程进程监控
dstat 系统资源统计
iotop 监视磁盘I/O使用状况
lsof 查看当前系统文件的工具
kill 向进程发出控制信号
at：计划任务；适合做一次性任务
batch：系统自行选择空闲时间去执行特定任务
cron ：周期性执行任务；默认：或的关系
usleep：微秒

----------------------------------------------------------------------I
第十七节课 2018年10月31日09:00:00 周三
----------------------------------------------------------------------I
anacron：判断是否有些老的程序没有执行
tmpwatch：清除临时文件 

【管理临时文件】
CentOS6使用/etc/cron.daily/tmpwatch定时清除临时文件
CentOS7使用systemd-tmpfiles-setup服务实现


crontab 计划任务  
	-l -u wang 查看wang的计划任务
	-r -u wang 清除wang所有计划任务
	-i 交互提醒
	-e 编辑任务
	-u user 仅root运行
	-c jbonum 显示任务  
黑名单/etc/cron.deny
白名单/etc/cron.allow
(对于cron任务来讲，%有特殊用途；如果在命令中要使用%，则需要转义，将%放置于单引号中，则可不用转义)

【练习】
1、每周的工作日1：30，将/etc备份至/backup目录中，保存的文件名称格式为“etcbak-yyyy-mm-dd-HH.tar.xz”，其中日期是前一天的时间
mkdir /backup 
vim backup_etc.sh ↓
tar jcf /backup/etcbak-`date -d yesterday +%F-%H`.tar.xz  /etc/ 
&> /etv/nullvi
chmod +x backup_etc.sh
crontab -e
30 1 * * 1-5  /root/backup_etc.sh

2、每两小时取出当前系统/proc/meminfo文件中以S或M开头的信息追加至/tmp/meminfo.txt文件中
vim SM.sh
cat /proc/meminfo|egrep '^[S|M]' > /tmp/meminfo.txt
chmod +x SM.sh
crontab -e
0 */2 * * * /root/SM.sh


3、工作日时间，每10分钟执行一次磁盘空间检查，一旦发现任何分区利用率高于80%，就执行wall警报
crontab -e
*/10 * * * *  [ `df|sed -nr '/^\/dev\/sda/s/.* ([0-9]+)%.*/\1/p'|sort -nr|head -1` -gt 80 ] && wall disk will be full



【SHELL脚本编程进阶】



【--if 条件判断】 选择执行 if语句可嵌套

=======if..;then  elif..;then  else..;then  fi

if 判断条件1; then
	cmd
elif 判断条件2; then
	cmd
elif 判断条件3; then
	cmd
else
	cmd
fi(结束）
逐条判断，第一次遇为“真”条件时，执行其分支，而后结束整个if语句


--if示例: 根据命令的退出状态来执行命令
	
if ping -c1 -W2 station1 &> /dev/null;                  then
	echo 'Station1 is UP'
elif    grep "station1" ~/maintenance.txt &> /dev/null; then            
	echo 'Station1 is undergoing maintenance‘
else
	echo 'Station1 is unexpectedly DOWN!'
	exit 1
fi 


【--case条件判断 】
case  变量引用 in
1|2|3)
	cmd1
	
4|5|6)
	cmd2
	;;
*)
	cmd3
esac

case支持glob风格的通配符：
*: 任意长度任意字符
?: 任意单个字符
[]：指定范围内的任意单个字符
a|b: a或b


--练习
1、编写脚本/root/bin/createuser.sh，实现如下功能：使用一个用户名做为参数，如果指定参数的用户存在，就显示其存在，否则添加之；显示添加的用户的id号等信息
vim createuser.sh
read -p "please input username: "user
id $user &> /dev/null
if [[ $? -eq 0 ]];then
	echo user exist
	exit
else 
	useradd $user
	echo 123 | passwd --stdin $user > /deg/null
	id  $user
fi


2、编写脚本/root/bin/yesorno.sh，提示用户输入yes或no,并判断用户输入的是yes还是no,或是其它信息
vim yesorno.sh
read -p "Do you agree(yes or no)? " ans
ans=`echo $ans |tr 'A-Z' 'a-z'`
case $ans in
y|yes)
    echo "you answer is yes"
    ;;
n|no)
    echo "you answer is no"
    ;;
*)
     echo "you input false"
esac

3、编写脚本/root/bin/filetype.sh,判断用户输入文件路径，显示其文件类型（普通，目录，链接，其它文件类型）

vim filetype.sh
read -p "please input a path of the file : " files
if [ -b $files ];then
	echo "该文件是块设备"
elif [ -c $files ];then
	echo "该文件是字符设备"
elif [ -f $files ];then
	echo "该文件是普通文件"
elif [ -h $files ];then
	echo "该文件是符号链接文件"
elif [ -p $files ];then
	echo "该文件是管道文件"
elif [ -s $files ];then
	echo "该文件是套接字文件"
elif [ -d $files ];then
	echo "该文件是目录文件"
else             
	echo "文件或目录不存在"
fi

4、编写脚本/root/bin/checkint.sh,判断用户输入的参数是否为正整数
vim checkint.sh
read -p "please input number : " n
if [[  $n =~ [[:digit:]]+$ ]] && [ $n -gt 0 ];then
	echo "是正整数"
else
	echo "不是正整数"
fi


\c 取消换行 \n 换行 \r回车 \t tab键分隔
(( ))两个小括号里面，默认是数字，变量转换,$(())数字运算  
time cmd :显示执行命令时间

随机生成8位
cat /etc/urandom|tr -dc '[:alnum:]'|head -c8
openssl rand -base64 10


play teasong1.wav播放音乐

【--for 循环】
for 变量 in 列表;  
do  xxx  
done
 
例：

执行机制：
依次将列表中的元素赋值给“变量名”; 每次赋值执行一次循环; 直到列表中的元素耗尽，循环结束。




【练习】:用for实现
1、判断/var/目录下所有文件的类型
vim  ftype.sh
for file in /var/*
do
    if [ -f "$file" ]
    then
        if [ -s "$file" ] 
        then
            printf "File:$file\n"
            cat "$file"
        else 
            rm "$file"
        fi
    else [ -d "$file" ]
        printf "Directory:$file\n"
        ls "$file"
    fi
    printf "\n\n\n"
done

2、添加10个用户user1-user10，密码为8位随机字符

vim adduser.sh
for i in `seq 10`; do
	useradd user$i
	passwd=`#cat /dev/urandom|tr -dc '[:alnum:]'|head -c8`
	echo $passwd|passwd --sdtin user$i
done

3、/etc/rc.d/rc3.d目录下分别有多个以K开头和以S开头的文件；分别读取每个文件，以K开头的输出为文件加stop，以S开头的输出为文件名加start，如K34filename stop S66filename start
vim dup.sh
for i in /etc/rc.d/rc3.d/{K,S}*; do
	mv i


4、编写脚本，提示输入正整数n的值，计算1+2+…+n的总和
vim nsum.sh
read -p "please input number: " n
declare -i sum=0
[[ $n =~ [[:digit:]]+ ]] || { echo "input fault";exit; }                                    
for i in $(seq $n); do
    let sum+=i
done
echo sum=$sum


5、计算100以内所有能被3整除的整数之和
vim sum.sh
sum=0
for i in {1..100};do
	let sum=sum+i
done
echo sum=$sum

6、编写脚本，提示请输入网络地址，如192.168.0.0，判断输入的网段中主机在线状态
read -p "please input ipaddres: " ip
echo $ip|cut -d. -f1-3
netid=192.18.133
for id in {1..254};do
     {
     if ping -c1 -w1 $netid.$id &> /dev/null ;then
         echo $netid.$id is up 
     else
         echo $netid.$id is down
     fi
     }&
done
bash $0 > ipscan.txt

7、打印九九乘法表
for i in `seq 9` ; do
      for j in `seq $i`;do                                                                 
          let sum=i*j
          echo -e "${i}X${j}=$sum\t\c"
      done
      echo
done

------另一种写法
for((i=1;i<=9;i++));do
	for((j=1;j<=$i;j++));do
		echo -en "${i}x${j}=$[$i*$j]\t\c"
	done
	echo
done

8、在/testdir目录下创建10个html文件,文件名格式为数字N（从1到10）加随机8个字母，如：1AbCdeFgH.html
9、打印等腰三角形
vim 3jiao.sh
read -p "please input line: " line
for i in `seq $line`;do
    let star=$i*2-1
    let space=$line-$i
    for j in `seq $space`;do
          echo -n " "
    done
    for k in `seq $star`;do
          echo -n "*"
    done
    echo
done

10、猴子第一天摘下若干个桃子，当即吃了一半，还不瘾，又多吃了一个第二天早上又将剩下的桃子吃掉一半，又多吃了一个。以后每天早上都吃了前一天剩下的一半零一个。到第10天早上想再吃时，只剩下一个桃子了。求第一天共摘了多少？
vim houzi.sh
i=1
for j in `seq 9` ; do
    let i=($i+1)*2                                                     
done
echo "$i"

--打印正方形
read -p "please input lie: " lie
read -p "please input hang: " hang
for i in `seq $hang`; do
    for j in `seq $lie` ; do
	if [ $i -eq 1 -o $i -eq $hang -o $j -eq 1 -o $j -eq $lie ];then
		COLOR=$[RANDOM%7+31]
	  echo -e "\033[$1;5;{COLOR}m*\033[0m\c"  （随机颜色高亮闪烁）
    	else
	  echo -e "*\c"
	fi
    done
    echo
done

--国际象棋

for i in {1..8};do
   for j in {1..8};do
       flag=$[(j+i)%2]
       if [ $flag -eq 0 ]; then
             echo -e "\033[47m  \033[0m\c"
       else
             echo -e "  \c"
       fi
   done
   echo 
done

--拒绝cracker黑客登录
#vim /etc/hosts.deny
#sshd：ip

while: ; do
	iplist=`who | sed -rm '$cracker/s/.*\((.*)\)/\1/p'`
	if [ "$iplist" ] ; then
		pkill -9 -U cracker
		echo "cracker is killed"
		echo sshd:$iplist >> /etc/hosts.deny
	fi
	sloop 10
done

【while 循环】
while read line; do
循环体

--特殊用法↓
while read line; do
循环体
done < /PATH/FROM/SOMEFILE

依次读取/PATH/FROM/SOMEFILE文件中的每一行，且将行赋值给变量line





----------------------------------------------------------------------I
第十八节课 2018年11月2日  周五
----------------------------------------------------------------------I
if ；then  elif ；then   else  ；then   fi 
case 变量 ；in  1|2|3) cmd  *) cmd1 esac  
for 变量 in 列表 do done  能生成列表的循环方式更适合for
	变量${列表} 耗尽：循环结束

while [条件] ；do done
	真  循环

	假  终止循环

until [条件] ；do done            
	假  循环       false
	                        	
	真  退出循环   : / ture

select 菜单 in 列表；do 'cmd' done (可配合case）

【循环条件】
--continue  结束第N层的本轮循环，而直接进入下一轮判断
--break     结束并退出第N层整个循环
--shift     用于将参量列表 list 左移指定次数，缺省为左移一次

【select循环与菜单】 REPLY:用户输入信息 →匹配
select[xx] in [列表] ； do  "cmd" ; done
自动生成 1）cmd1 
	 2）cmd2
	 3）cmd3
	$?

例：PS3=please input digit （更改提示用户输入）原始：#？
    select MENU in lamian huimian gaifan jiaozi quit;do
	case $REPLY in
	1|2)
		echo 15rmb
	3|4） 
		echo 20rmb
	5)
		break
	*）
		echo input false
	esac
done


检查http服务是否启动
sleeptime=30
until false ; do
	if ! killall -0 httpd &>/dev/null ;then
	echo " At `date +'%F %T' httpd restarted" >> /data/httpd.log
    fi
    sleep $sleeptime
done 

--面试题
echo "lh" | read name;echo $name    #空
echo "lh  lh1" | while read name name1;echo $name;done  
	#lh  lh1 能同时识别多个变量


tcrl v ;选中；大写I；输入#；esc两次 → 批量添加注释


iptables -F 清除已有iptables规则
iptables -A INPUT -s 192.168.1.1 -j REJECT 
拒绝IP通过ssh方式登录
vim /etc/hosts.deny ↑同上

a=`ls` ；echo "$a" 当变量调用成命令时，最好加上双引号
 lastb：查看失败登录的日志

【函数】三种格式  cat /etc/init.d/functions 系统函数
info（）{  ;} 
function info { ;} 
function info（）{ ;}

--函数和shell的区别↓↓↓ 
Shell程序在子Shell中运行
而Shell函数在当前Shell中运行。因此在当前Shell中，函数可以对shell中变量进行修改

--特点：
只在当前终端运行，退出失效
在外部执行，在同一个shell
相当于一个集合的命令
. func 读取此文件内容的函数定义
unset 删除函数本身
declare -f sysinfo查看函数 定义内容

$1 $2变量对应第1、第2个参数
先. func引用函数文件，然后再调用函数▲
local:本地变量 只在当前进程生效 local a=1
export:环境变量 子进程也能使用 export -f a=1 
（export -f  或 declare -xf 查看）
 

例：
sysinfo（）{ hostname;cat /etc/centos_release; }
sysinfo

export 环境变量

--return函数返回值
只退出当前函数，不退出脚本

--函数递归
重复执行某些特定的指令

例：阶乘 3=3x2x1
3！=3x2!
2! =2x1!
1! =1x0!
0! =1
↓↓↓↓
fact(){ if[ "$1" -eq 1 ];then
		echo 1
	else
		echo $[$1*`fact $[$1-1]`]
}
fact $1

--fork 炸弹
:(){ :|:& };:
【练习】
编写函数，实现OS的版本判断
编写函数，实现取出当前系统eth0的IP地址
编写函数，实现打印绿色OK和红色FAILED
编写函数，实现判断是否无位置参数，如无参数，提示错误

【练习】
编写服务脚本/root/bin/testsrv.sh，完成如下要求
(1) 脚本可接受参数：start, stop, restart, status
(2) 如果参数非此四者之一，提示使用格式后报错退出
(3) 如是start:则创建/var/lock/subsys/SCRIPT_NAME, 并显示“启动成功”
考虑：如果事先已经启动过一次，该如何处理？
(4) 如是stop:则删除/var/lock/subsys/SCRIPT_NAME, 并显示“停止完成”
考虑：如果事先已然停止过了，该如何处理？
(5) 如是restart，则先stop, 再start
考虑：如果本来没有start，如何处理？
(6) 如是status, 则如果/var/lock/subsys/SCRIPT_NAME文件存在，则显示“SCRIPT_NAME is running...”，如果/var/lock/subsys/SCRIPT_NAME文件不存在，则显示“SCRIPT_NAME is stopped...”
(7)在所有模式下禁止启动该服务，可用chkconfig 和 service命令管理
说明：SCRIPT_NAME为当前脚本名

【练习】
编写脚本/root/bin/copycmd.sh
(1) 提示用户输入一个可执行命令名称
(2) 获取此命令所依赖到的所有库文件列表
(3) 复制命令至某目标目录(例如/mnt/sysroot)下的对应路径下
如：/bin/bash ==> /mnt/sysroot/bin/bash
/usr/bin/passwd ==> /mnt/sysroot/usr/bin/passwd
(4) 复制此命令依赖到的所有库文件至目标目录下的对应路径下： 如：/lib64/ld-linux-x86-64.so.2 ==> /mnt/sysroot/lib64/ld-linux-x86-64.so.2
(5)每次复制完成一个命令后，不要退出，而是提示用户键入新的要复制的命令，并重复完成上述功能；直到用户输入quit退出

【练习】
1)编写函数实现两个数字做为参数，返回最大值
2)斐波那契数列又称黄金分割数列，因数学家列昂纳多·斐波那契以兔子繁殖为例子而引入，故又称为“兔子数列”，指的是这样一个数列：0、1、1、2、3、5、8、13、21、34、……，斐波纳契数列以如下被以递归的方法定义：F（0）=0，F（1）=1，F（n）=F(n-1)+F(n-2)（n≥2）
fei(){ if [ "$1" -eq 0 ] ;then
		echo 0
	if [ "$1" -eq 1 ]; then
		echo 1
	elif 
	echo $[ `fei $[$1-1]`+`$[1-2]` ]
	fi

}

3)利用函数，求n阶斐波那契数列
_汉诺塔（又称河内塔）问题是源于印度一个古老传说。大梵天创造世界的时候做了三根金刚石柱子，在一根柱子上从下往上按照大小顺序摞着64片黄金圆盘。大梵天命令婆罗门把圆盘从下面开始按大小顺序重新摆放在另一根柱子上。并且规定，在小圆盘上不能放大圆盘，在三根柱子之间一次只能移动一个圆盘，利用函数，实现N片盘的汉诺塔的移动步骤

【trap信号捕捉】
1 重载 2中止 3退出 9强制退出 15默认 18继续运行 19后台休眠 
进程收到系统发出的指定信号后，将执行自定义指令，而不会执行原操作
trap '' 信号
忽略信号的操作
trap '-' 信号
恢复原信号的操作
trap -p
列出自定义信号操作
rap finish EXIT
当脚本退出时，执行finish函数

例：当脚本异常退出，照样不中断执行"echo finish"
finish（）{
	echo  "finish"
}
trap finish exit
for ((i=1;i<=10;i++));do
	let sum+=i
	sleep 1
done
echo $sum


【系统启动和内核管理】

linux：单内核，所有功能集成于同一个程序
windows:微内核，每个功能与单独子系统实现
核心文件：/boot/vmlinuz-VERSION-release
模块文件：/lib/modules/VERSION-release
xx.ko.xz →驱动模块

GRUB 1st stage 磁盘上某些数据的二进制数据  1-1.5阶段
     2st stage 加载内核文件路径

centos6启动流程：
加电自检→MBR引导→GRUB→加载内核→启动init文件→读取配置文件→运行脚本	

GRUB:引导操作系统(通过grub定位内核文件）
inittamfs：虚拟操作系统（提供必要驱动，让我们通过内核挂载根）

--启动流程
MBR：第一扇区 
	前446字节 bootloader(GRUB)
	中间64字节 分区表
	最后2字节 55AA


内核←目录←文件系统

/kerner/fs
/lib/modules/2...kerner/fs/ext* 文件系统


【实验】
--centos6 mkinitrd删除修复
作业：rm -f /boot/initramfs-2.**.img
rm -f /boot/initramfs-2.6.32-754.el6.x86_64.img 
mkinitrd /boot/initramfs-`uname -r`.img `uname -r`
(生成文件系统，路径，根据当前内核版本生成配合参数）

↓重启修复
进入救援模式
引导光盘__救援模式
chroot /mnt/sysimage/ 切根
ls /boot -l 确认刚才文件已删除
mkinitrd /boot/initramfs-`uname -r`.img `uname -r` 
sync  等待一会同步
exit 重启！
 

--centos6 vmlinuz-`uname -r`删除修复
/boot/vmlinuz-`uname -r`


【命令总结】
while true do done 真循环
while read；do 真循环
done < file
until false；do done 假循环
cmd | while read ；do 通过管道调取命令

continue 结束第N层的本轮循环，而直接进入下一轮判断
break    结束并退出第N层整个循环

select  循环与菜单
trap    捕捉信号

function 函数
return   函数返回值（只退出当前函数，不退出脚本）

local 变量/lastb 记录错误登录信息/shift 变量缺省左移/mkinitrd重新封包


