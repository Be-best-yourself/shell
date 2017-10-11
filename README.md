#!/bin/bash

################################################
##########     配置以下参数        #############
################################################

#tomcat主目录
catalinaHome=/usr/local/tomcat7_8080

#webapps运行的war包
fileName="ROOT.war"
#服务器中运行的目录名
fileDir="ROOT"


#上传文件目录，请手动创建，并把war放到此目录下
uploadDir=/upload
#上传文件war包
uploadFile="ROOT.war"


#备份的文件夹，请手动创建
bakDir=/bak

#以下三个时间参数请根据文件大小和系统性能，自己估摸,最小0s,不能为空
#移动文件所需的时间
mvTime=1s

#tomcat服务器正常停止所需时间，此值估摸错了也没事，会执行kill
shutDownTime=5s
#tomcat服务器正常启动所需时间
startupTime=5s


###############################################
############   配置以上参数	###############
###############################################


tomcatBinDir=$catalinaHome"/bin"
webappDir=$catalinaHome"/webapps/"


time=$(date +%Y%m%d%H%M)
echo "当前系统时间："$time


#检查是否有上传目录
if [ ! -d "$uploadDir" ]; then
  echo "上传目录："$uploadDir"不存在，请创建此目录，退出此脚本"
  exit
fi

#检查上传目录中是否有上传文件
uploadFileAllPath=$uploadDir"/"$uploadFile
if [ ! -f "$uploadFileAllPath" ]; then
   echo $uploadDir"目录中没有"$uploadFile"文件,退出此脚本"
   exit
fi

#检查是否有备份目录
if [ ! -d "$bakDir" ]; then
  echo "上传目录："$bakDir"不存在，请创建此目录，退出此脚本"
  exit
fi

#检查服务器中webapp中是否有运行的文件和目录
runingFile=$webappDir$fileName
runingDir=$webappDir"/"$fileDir
if [ ! -f "$runingFile" ]; then
   echo "警告：webapps下没有正在运行项目的"$fileName"，请检查"
fi
if [ ! -d "$runingDir" ]; then
   echo "wepapps下没有运行的目录文件"$fileDir"，请检查，退出此脚本"
   exit
fi



tomcatPid=$(ps -ef |grep $catalinaHome  |grep -v "grep"|awk '{print $2}')


if [ -n "$tomcatPid" ];then
	echo "tomcat服务正在运行,pid是"$tomcatPid
	echo "正常关闭tomcat服务……"
	echo "进入tomcat bin目录："$tomcatBinDir
	cd $tomcatBinDir
	echo "运行tomcat停止脚本shutdown.sh，停止tomcat服务……"
	./shutdown.sh
	echo "等待"$shutDownTime",待tomcat正常关闭完成，再执行下一步"
	sleep $shutDownTime
	echo "检查tomcat进程是否全部关闭"
	checkTomcatPid=$(ps -ef |grep $catalinaHome  |grep -v "grep"|awk '{print $2}')
	if [ -n "$checkTomcatPid" ];then
		echo "检查到tomcat没有正常关闭，强制杀掉进程"$checkTomcatPid
		kill -9 $checkTomcatPid
		echo $checkTomcatPid"进程已杀掉"
	fi
	if [ -z "$checkTomcatPid" ];then
		echo "检查到tomcat已正常关闭"
	fi
	
fi

if [ -z "$tomcatPid" ];then
	echo "tomcat服务没有启动"
fi

echo "备份ROOT.war文件……"
mv $webappDir$fileName $bakDir"/"${fileName%.*}"_"$time"."${fileName#*.}
sleep $mvTime
echo "备份ROOT文件……"
mv $webappDir$fileDir $bakDir"/"$fileDir"_"$time
sleep $mvTime

echo "将上传文件部署到webapps目录下……"
mv $uploadDir"/"$uploadFile $webappDir
sleep $mvTime


echo "检查是否部署成功……"
checkDeployFile=$webappDir$fileName
if [  -d "$checkDeployFile" ]; then
  echo "文件还没部署成功，等待中……"
  sleep 5s
else
  echo "文件部署成功"
fi

echo "启动tomcat服务……"
cd $tomcatBinDir
./startup.sh

sleep $startupTime

echo "检测tomcat是否启动成功"
checkAgainTomcatPid=$(ps -ef |grep $catalinaHome  |grep -v "grep"|awk '{print $2}')
if [ -n "$checkAgainTomcatPid" ];then
   echo "tomcat已启动，pid:"$checkAgainTomcatPid             
fi
if [ -z "$checkAgainTomcatPid" ];then
   echo "tomcat没启动成功"
fi

