---
layout: post
author: zhao
title: Linux：常用命令总结
modified: 2015-08-17
tags: [Linux]
image:
  feature: pic-14.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

##常用命令总结
<table>
<tbody>
<tr>
<td><em>命令名</em></td>
<td><em>命令简介</em></td>
<td><em>命令参数</em></td>
<td><em>示例</em></td>
</tr>

<tr>
<td>mkdir</td>
<td>make directorys<p>创建目录</td>
<td> -p（parents)：如果需要父目录，则创建</td>
<td></td>
</tr>

<tr>
<td>touch</td>
<td>创建文件或者修改文件的时间戳</td>
<td></td>
<td></td>
</tr>

<tr>
<td>ls</td>
<td>list<p>列出文件</td>
<td><p>-a ：列出目录下所有的文件，包括以“.”开头的隐藏文件<p>-l ：列出文件的详细信息，如创建者，创建时间，文件的读写权限列表等等</td>
<td></td>
</tr>

<tr>
<td>cd</td>
<td>change directory<p>切换目录层次</td>
<td></td>
<td><p>cd - ：切换至上一个工作目录<p>cd ~ ：切换至用户home目录</td>
</tr>

<tr>
<td>pwd</td>
<td><p>print working directory<p>查看当前工作目录</td>
<td></td>
<td></td>
</tr>

<tr> 
<td>echo</td>  
<td>打印输出内容</td>  
<td></td>
<td></td>
</tr>

<tr> 
<td>cat</td>  
<td>查看文件内容</td>  
<td></td>
<td></td>
</tr>

<tr> 
<td>mv</td>  
<td><p>move<p>移动文件或者目录<p>**移动目录时，源目录结尾不要多余斜线，结尾目录最后最好加上斜杠。**</td>  
<td></td>
<td></td>
</tr>

<tr> 
<td>rm</td>  
<td><p>remove<p>删除操作</td>  
<td><p>-r（recursive）：递归删除目录 <p>-f（force）：强制</td>
<td></td>
</tr>

<tr> 
<td>rmdir</td>  
<td>删除空目录，比较鸡肋</td>  
<td></td>
<td></td>
</tr>

<tr> 
<td>head</td>  
<td>头部，显示文件头部，默认是10行</td>  
<td>-n ：指定输出头n行</td>
<td></td>
</tr>

<tr> 
<td>tail</td>  
<td>尾部，显示文件尾部部，默认是10行</td>  
<td>-n ：指定输出尾n行</td>
<td></td>
</tr>

<tr> 
<td>grep</td>  
<td>过滤核心命令之一</td>  
<td><p>-v（--invert-match）： 排除 <p> -E：以|分开，可以过滤多个 <p>-Ei：不区分大小写</td>
<td></td>
</tr>

<tr> 
<td>egrep</td>  
<td>相当于grep -E</td>  
<td></td>
<td></td>
</tr>

<tr> 
<td>sed</td>  
<td>取各种内容</td>  
<td> -n（--quiet, --silent） 取消默认输出 <p> -i（--in-place）： 编辑文件</td>
<td><p>sed -n /内容/p 文件    p（print）<p>sed /内容/d 文件    d（delete）</td>
</tr>

<tr> 
<td>which</td>  
<td>后跟命令名，查看命令所在目录</td>  
<td></td>
<td></td>
</tr>

<tr> 
<td>alias</td>  
<td><p>查看系统别名<p>通过给危险命令加保护参数，可以防止误操作。<p>把很多复杂的字符串变成一个简单的字符串。<p>可以把别名放在.bashrc /etc/profile</td>  
<td></td>
<td>alias rm='echo you cannot use it' 给rm起别名'，每当执行rm的时候会提示的后面的提示内容而不执行rm操作</td>
</tr>

<tr> 
<td>unalias</td>  
<td>后跟命令名，作用是取消别名</td>  
<td></td>
<td></td>
</tr>

</tbody>
</table>