---
layout: default
title: "CentOS挂载OSS"
date: 2020-11-19 22:10:00 +0800
categories: 运维
tags: [服务器, FTP]
---

    # 下载安装包
    wget http://gosspublic.alicdn.com/ossfs/ossfs_1.80.6_centos7.0_x86_64.rpm
    
    # 安装
    yum -y localinstall ossfs_1.80.6_centos7.0_x86_64.rpm
    
    # 设置参数，填写自己的阿里云 BucketName、AccessKey ID、Accesskey Secret
    echo BucketName:AccessKeyId:AccessKeySecret > /etc/passwd-ossfs
    
    # 设置配置文件的访问权限
    chmod 640 /etc/passwd-ossfs
    
    # 创建本地挂载的目录
    mkdir /var/www/backup
    
    # CentOS 8 需要安装，否则挂载时会报错，如果为7无需安装
    yum -y install compat-openssl10
    
    # 挂载，BucketName为自己的Backet名称、MountPath为要挂载的目录、ossEndpoint为Backet的地域节点
    ossfs BucketName MountPath -ourl=ossEndpoint

    # 卸载，可在备份完成后执行
    fusermount -u mount-path