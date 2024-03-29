---
title: Migration BKDATA
categories: [blueking, migration]
date: 2019-10-24 09:09:19
---
# 迁移bkdata

## 停止相关服务

    ./bkcec stop bkdata

## 删除安装标记

    ssh $BKDATA_IP "source /data/install/utils.fc; sed -i '/bkdata/d' $INSTALL_PATH/.installed_module; grep bkdata $INSTALL_PATH/.installed_module"
    sed -i '/bkdata/d' /data/install/.bk_install.step; grep bkdata /data/install/.bk_install.step

    # 删除consul对应服务配置
    ssh $BKDATA_IP "source /data/install/utils.fc; rm -f $INSTALL_PATH/etc/consul.d/{dataapi,databus,monitor}.json"

## 配置install.config

- 删除原ip所在行的 bkdata
- 新增一行 $ip bkdata ip为待迁移的机器IP
- 更新env: source utils.fc

## 新机器配置好中控机的 ssh 免密登陆

    ./configure_ssh_without_pass

## 同步文件

    ./bkcec sync common
    ./bkcec sync consul
    ./bkcec sync bkdata

## 安装consul，并重启服务

    ./bkcec install consul
    
    ./bkcec stop consul
    ./bkcec start consul

## 安装bkdata

    ./bkcec install bkdata

## 给新机器授予mysql权限（非新机器时无需执行）

    ./bkcec initdata mysql

## 给新的bkdata补上初始化标记文件

    ssh $BKDATA_IP 'touch /data/bkce/.dataapi_snaphost'

## 重新渲染相关模块配置，并重启服务（注意启动顺序）

    ./bkcec render bkdata
    ./bkcec start bkdata

## 在CMDB面板中转移，迁移涉及到的机器到空闲机

转移到空闲机是为了在安装gse_agent时，能从新按照新的模块配置重新注册到CMDB。
为了能让服务器与模块的管理关系与实际保持一致，在从新安装gse_agent之前需要把
配置变动的机器置于蓝鲸业务的空闲机中。

## 重新安装gse-agent

    # 给新机器添加gse_agent，给模块配置变动的机器从新部署gse_agent.并注册到CMDB。
    ./bkcec install gse_agent

## 检查服务是否正常

- bkmonitor上是否正常采集到数据
- 节点管理上添加新节点agent
