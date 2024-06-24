```shell
vim /etc/profile
==========
# login audit format
HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S  `who am i` "

# login script
/usr/local/bin/login.sh

# jdk env
export JAVA_HOME=/usr/local/jdk1.8.0_181
export JRE_HOME=/usr/local/jdk1.8.0_181/jre
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$JAVA_HOME/jre/lib/rt.jar

# maven env
export MAVEN_HOME=/usr/local/maven3
export PATH=$PATH:$MAVEN_HOME/bin

# nodejs env
export NODEJS_HOME=/usr/local/nodejs12
export PATH=$PATH:$NODEJS_HOME/bin
==========
```