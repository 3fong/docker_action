### sonarqube  ###

server
要求:
jdk 11

下载:
https://www.sonarqube.org/downloads/

作为非root解压它.因为elasticsearch获取不到root权限,会造成启动失败

运行:
# On Windows, execute:
C:\安装目录\bin\windows-x86-xx\StartSonar.bat

# On other operating systems, as a non-root user execute:
安装目录/bin/[OS]/sonar.sh console

默认访问:
http://localhost:9000 with System Administrator credentials (login=admin, password=admin).

日志目录:

$SONARQUBE_HOME/logs:



--add-opens java.base/jdk.internal.misc=ALL-UNNAMED
-Dio.netty.tryReflectionSetAccessible=true






















