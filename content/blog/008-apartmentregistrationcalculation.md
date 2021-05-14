---
title: "统计购房登记者的核验情况脚本"
date: 2021-05-14T10:12:33+08:00
draft: false
weight: 8
---

购房登记流程为：登记-->核验-->摇号，核验截止时间比登记时间晚一天，此时间段内会实时更新登记人的核验情况，但并不会出整体的统计数据。且网站并未做前后端分离，数据又分页返回。所以需要通过脚本实时统计登记人的核验情况。

<!--more-->

## 脚本内容

```bash
#! /bin/bash

tatalcount=0
gangxucount=0

for((i=1;i<=30;i++));
do
	cmd="curl ************************"
	`$cmd > tmp.txt`

	sed -n '/<tr>/,/<\/tr>/p' tmp.txt | sed 's/<\/tr>/!/g' > trtmp.txt
	gangxu=`awk 'BEGIN{FS="\n";RS="!";ORS=""}{for(x=1;x<=NF;x++){print $x} print "\n"}' trtmp.txt | grep "刚需" | grep "核验通过" | wc -l`
	total=`awk 'BEGIN{FS="\n";RS="!";ORS=""}{for(x=1;x<=NF;x++){print $x} print "\n"}' trtmp.txt | grep "核验通过" | wc -l`
	gangxucount=$[$gangxucount+$gangxu]
	totalcount=$[$totalcount+$total]
done

echo "总共核验通过: $totalcount"
echo "刚需核验通过: $gangxucount"

rm -rf tmp.txt trtmp.txt
```

## 参考链接

1. https://www.cnblogs.com/leezhxing/p/4694323.html
