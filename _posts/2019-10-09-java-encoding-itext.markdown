---
layout: post
title:  "Problems with Chinese Fonts in iText-PDF"
date:   2019-10-09 17:00:00 +0800
categories: java encoding itext
---

一、服务器不再单独维护字体，添加simsun.ttf文件到项目中  
无法获取到resources中的ttf文件，查询得知需要修改pom文件  
<https://stackoverflow.com/questions/44572557/flying-saucer-pdf-table-name-does-not-exist-in-exception-on-ubuntu>
```
<resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
                <excludes>
                    <exclude>fonts/*</exclude>
                </excludes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>false</filtering>
                <includes>
                    <include>fonts/*</include>
                </includes>
            </resource>
        </resources>
```
然后直接引用即可  
`addFont("/fonts/samplefont.ttf", BaseFont.IDENTITY_H,BaseFont.EMBEDDED);`


二、修改simsun字体为思源宋体
* 下载思源宋体文件，Language-specific OTFs -> Simplified Chinese  
<https://github.com/adobe-fonts/source-han-serif/tree/release/>

* 添加文件到/fonts目录，修改字体引用为/fonts/SourceHan.otf，结果中文显示无效
* 修改为ttf后缀，依然无效
* 求助网上<https://stackoverflow.com/questions/23958538/problems-with-chinese-fonts-in-itext-pdf-on-windows-machines>  
免费字体网： <https://www.freechinesefont.com/>  
使用回答中的hxb-meixinti.ttf依然无效；hxb-meixinti.ttf用例确实有效，但是结合html文件无效，此处应该怀疑html可能是bug点
* 查看itext使用版本为2.0.8，搜索iText 2.0.8 html to pdf  
<https://blog.csdn.net/lxz946786639/article/details/51184502>， 文中提示： 
>为了解决中文问题，请在body中加入样式 font-family: SimSun;  
其中 "c://windows//fonts//simsun.ttc,1" 使用Windows系统字体，和html页面中font-family: SimSun; 对应。 

故知需修改html文件中的font-family
<https://www.w3school.com.cn/css/pr_font_font-family.asp>
* 查看思源宋体font-family  
<https://www.jianshu.com/p/44ef95b2c86f> 

>思源黑体 :  Source Han Sans CN

修改ccs：`font-family: 'Source Han Serif SC';`  
Bingo～
* 解决table在pdf中显示不全问题  
<https://www.iteye.com/blog/skyfar666-2001353>  
>模版中在table 加样式 style="margin-top: 60px;table-layout:fixed; word-break:break-strict;"

思源宋体协议文件搞定
