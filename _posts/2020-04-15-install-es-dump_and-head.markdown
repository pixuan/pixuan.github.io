---
layout: post
title:  "Install ES dump and ES head"
date:   2020-04-15 01:00:00 +0800
categories: ES es-dump es-head
---

### es-head
* 引用 <https://github.com/mobz/elasticsearch-head>
* git clone git@github.com:mobz/elasticsearch-head.git
* 根据NodeJS镜像制作es-head镜像  
修改Dockerfile如下
```
FROM node:12.16.2-alpine3.9
COPY . ./elasticsearch-head
RUN npm config set registry https://registry.npm.taobao.org \
    && cd elasticsearch-head \
    && mkdir /tmp/phantomjs \
    && mv phantomjs-2.1.1-linux-x86_64.tar.bz2 /tmp/phantomjs \
    && npm install 
WORKDIR /elasticsearch-head
CMD ["npm", "run", "start"]
EXPOSE 9100
```
* 提前下载phantomjs防止中途卡死
* 执行`docker build -t elasticsearch-head:latest .`可制作该镜像

### es-dump
* 引用 <https://github.com/taskrabbit/elasticsearch-dump>
* 拉取镜像
`docker pull taskrabbit/elasticsearch-dump:v6.25.0`  
* 编写es_dump.sh  

```sh
#!/bin/bash

date=`date "+%Y%m%d"`
date_ago=`date -d "$date -1 day" +%Y%m%d`
basedir=/home/pzx/Workspaces/elk/dumpdata
bakdir=$basedir/"$date_ago"
ftp_host=127.0.0.1
ftp_user=root
ftp_pass=root

echo "======================================"
echo "## start daily es backup on $date_ago ##"
if [ ! -d $bakdir ];then
  echo "mkdir $bakdir"
  mkdir $bakdir
else
  echo "$bakdir already exist"
fi


echo "query all es indices on $date_ago"
indices=$(curl -X GET 'localhost:9200/_cat/indices/*?v&s=index' | grep "$date_ago" | awk {'print $3'})
for index in $indices; do
echo "start backing up $index"
docker run --rm -it --network host taskrabbit/elasticsearch-dump:v6.25.0 --type=data --input=http://localhost:9200/"$index" --output=$ |gzip -c > "$bakdir"/"$index".json.gz
echo "ended backing up $index"
done

echo "# start zipping dir $bakdir  #"
cd $bakdir
tar -czvf "$date_ago".tar.gz *.json.gz
echo "# ended zipping dir $bakdir #"

echo "# start sending zipped file $bakdir/"$date_ago".tar.gz by ftp #"
cd $bakdir
ftp -niv <<- EOF
lftp-open -u $ftp_user,$ftp_pass $ftp_host
cd $ftp_dir
binary
mput $bakfile
quit
EOF
echo "# ended sending zipped file $bakdir/"$date_ago".tar.gz by ftp#"

echo "## ended daily es backup on $date_ago ##"
echo "======================================"

```  
其中如果ES开启了xpack验证, 则需要更改  
```sh
indices=$(curl -u elastic_user:elastic_password -X GET 'localhost:9200/_cat/indices/*?v&s=index' | grep "$date_ago" | awk {'print $3'})
```  

以及  
```sh
docker run --rm -it --network host taskrabbit/elasticsearch-dump:v6.25.0 --type=data --headers='{"Authorization": "Basic " + $authStr + "}' --input=http://localhost:9200/"$index" --output=$ |gzip -c > "$bakdir"/"$index".json.gz
```  
其中 
```sh 
authStr=Base64.encodeBase64("elastic_user:elastic_password".getBytes("utf-8")); # elastic_user=elastic, elastic_password=elastic => authStr=base64.b64encode("elastic:elastic".encode("utf-8"))=ZWxhc3RpYzplbGFzdGlj
```
* 编写crontab, 如每日1点执行脚本
`0 1 * * * sh es_dump.sh >> es_dump.log`
查看es_dump.log

