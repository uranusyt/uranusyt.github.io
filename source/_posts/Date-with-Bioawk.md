---
title: Date with Bioawk
date: 2019-03-16 16:39:23
tags: bioawk bioinformatics
toc: true
---

# 废话
这是生平第一篇blog，因此难免要说点废话，所以前戏，啊不对，前言就叫废话吧。git+hexo搭建的平台杠杠的，感谢师弟MingLian小大佬成功带入写blog的坑。第一篇就拿生信李恒大神的神作bioawk镇楼吧！
# bioawk简介
除了鼎鼎大名的BWA、samtools等，李恒大神应群众呼声又做了出于awk又胜于awk的bioawk供广大生信猿们玩耍。bioawk是用C写的，用法和awk基本一样。
## 安装
### 分步安装

    $ sudo apt-get install byacc git    #安装git
    $ git clone git://github.com/lh3/bioawk.git  #默认安装在当前目录
    git clone git://github.com/lh3/bioawk.git ${path}/bioawk #若加path则必须为空，因此可在想要安装的目录下起一个新名称，git会clone到其下
    $ cd bioawk
    $ make
    $ echo "export PATH=${path}/bioawk/:\${PATH}" >> ~/.bashrc  
    $ .  ~/.bashrc 

### 一步安装脚本
    #!/bin/bash
    read -p  'Input installed  path: ' tmp_path
    path=${tmp_path/\//}
    if [ ! -d "${path}" ];then echo "No such file:${path}" && exit ;fi
    git clone git://github.com/lh3/bioawk.git ${path}/bioawk && \
    cd ${path}/bioawk && make && \
    echo "export PATH=${path}/bioawk/:\${PATH}" >> ~/.bashrc  && .  ~/.bashrc && \
    echo 'Successfully install bioawk!' || echo 'Failed install bioawk'

## 使用
bioawk基本思想是把组成不同类型的文件（sam、bam、fasta、fastq、vcf etc）的基本元素封装成变量，直接调用即可。
### 语法

    usage: bioawk [-F fs] [-v var=value] [-c fmt] [-tH] [-f progfile | 'prog'] [file ...]

-c 支持输入文件格式，查看帮助：

    bioawk -c -h
    
bed:
        1:chrom 2:start 3:end 4:name 5:score 6:strand 7:thickstart 8:thickend 9:rgb 10:blockcount 11:blocksizes 12:blockstarts
sam:
        1:qname 2:flag 3:rname 4:pos 5:mapq 6:cigar 7:rnext 8:pnext 9:tlen 10:seq 11:qual
vcf:
        1:chrom 2:pos 3:id 4:ref 5:alt 6:qual 7:filter 8:info
gff:
        1:seqname 2:source 3:feature 4:start 5:end 6:score 7:strand 8:frame 9:attribute
fastx:
        1:name 2:seq 3:qual 4:comment
**上面出现的名称即可引用其变量**

### 实例

* 打印fasta序列ID、序列、长度、GC含量

       bioawk -cfastx '{print "ID: "$name"\tlength: "length($seq)"\tGC: "gc($seq)"\t"$seq}' demo.fa

结果:
    ID: g1  length: 18      GC: 0.722222    atcccccccccccccttt
    ID: g2  length: 26      GC: 0.0769231   cattatatcttatttttttttttttt
    ID: g3  length: 48      GC: 0.229167    acccccccccctttttttttttttcatttttttttttttttttttttt
原始文件：
    >g1
    atcccccccccccccttt
    >g2 enzyme
    cattatatcttatttttttttttttt
    >g3
    accccccccccttttttttttttt
    catttttttttttttttttttttt
注意：
**可以看到bioawk在提取ID的时候只提取了空格分隔后的第一个字段，默认输出域分割符是\t**

* 输出反向互补序列
    
      bioawk -cfastx '{print $name,$seq,revcomp($seq)}' demo.fa
 结果：
     g1      atcccccccccccccttt      aaagggggggggggggat
    g2      cattatatcttatttttttttttttt      aaaaaaaaaaaaaataagatataatg
    g3      acccccccccctttttttttttttcatttttttttttttttttttttt        aaaaaaaaaaaaaaaaaaaaaatgaaaaaaaaaaaaaggggggggggt

* 根据序列ID提取序列，这个是最爱的功能

        bioawk -cfastx 'BEGIN{while((getline k <"id.txt")>0)i[k]=1}{if(i[$name])print ">"$name"\n"$seq}' demo.fa  
      
id.txt:
     g1
    g2
    g2 enzyme
    >g3

结果：
    >g1
    atcccccccccccccttt
    >g2
    cattatatcttatttttttttttttt
注意：id.txt 需要去掉>
* 当然还有其他的花样，大家自己去探索吧

# Reference
https://github.com/lh3/bioawk

