# 微服务消息推送

微服务消息推送目前支持，微服务5.2.1sdk的配置文件推送和微服务下线功能。

## 线上配置
```properties
#使用推送机制
app_metainfo_inotify_use=true
app_metainfo_proteus_use_push_conf=true
app_metainfo_inotify_push_server=wss://inotify-server.yonyoucloud.com/inotify
app_metainfo_inotify_pusher=https://developer.yonyoucloud.com/inotify-manager
```

## 测试环境配置
```properties
#使用推送机制
app_metainfo_inotify_use=true
app_metainfo_proteus_use_push_conf=true
app_metainfo_inotify_push_server=wss://developer-test.yonyoucloud.com/inotify-server/inotify
app_metainfo_inotify_pusher=https://developer-test.yonyoucloud.com/inotify-manager
```

## 测试用例地址

```properties
git@git.yonyou.com:isolate-support/proteus-embed-demo.git
```


## 详细操作文档

[配置中心拉取与推送](http://git.yonyou.com/install_disc/gpaas-disc/wikis/doc/debug/%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E6%8B%89%E5%8F%96%E4%B8%8E%E6%8E%A8%E9%80%81.md)
