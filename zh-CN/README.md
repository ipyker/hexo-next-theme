[![](https://img.shields.io/badge/Hexo-brightgreen.svg?style=plastic)](https://hexo.io/)
[![](https://img.shields.io/badge/nexT-pyker-yellow.svg?style=plastic)](https://github.com/theme-next/hexo-theme-next)

[英文介绍](../README.md) | [nexT Demo](https://www.ipyker.com)
## 目录
我们主要对next主题进行了如下配置操作。

* 设置中文语言
* 设置博主文字描述
* 设置博客文章连接为year/month/day/title.html格式
* Menu增加关于、标签、分类、互动、搜索菜单
* 禁用关于、标签、分类菜单评论功能
* 添加RSS
* 设置背景图片。默认禁用，可以在themes/nexT/source/css/_custom/custon.styl文件中启用
* 设置Canvas_nest动态背景
* 图片快速加载设置
* 微信支付宝打赏功能
* 点击出现桃心效果
* 主页文章添加阴影效果
* 设置代码高亮
* 顶栏背景色
* 底栏背景色
* 在右上角实现红色的fork me on github。默认禁用，可以在themes/nexT/_config.yml文件github_banner键中启用
* 修改文章内链接文本样式
* 修改文章底部标签样式
* 在文章末尾添加“文章结束”标记
* 设置头像
* 网站底部加上访问量
* 网站底部字数统计
* 网站底部添加网站运行时间
* 网站底部添加动态桃心
* 网站底部添加备案信息
* 底部隐藏由Hexo强力驱动、主题--NexT.Mist
* 设置网站的图标Favicon
* 实现文章文字统计功能和阅读时长
* 添加来必力云跟帖功能。（需要自己注册获取ID）
* 去掉底部重复字数统计
* 修改字体大小
* 添加DaoVoice在线联系。（需要自己注册获取ID）
* 侧边栏社交小图标设置
* 添加侧栏推荐阅读
* 修改侧边栏背景图片
* 修改侧边栏文字颜色
* 在文章底部增加版权信息
* Hexo博客添加站内搜索
* 修改选中字符的颜色
* 添加aplay音乐播放
* 添加博客右下角卡通动漫
* 增加了3D库three_waves，默认关闭
* 增加了canvas页面丝带
* 增加了首页pace加载进度
* 增加图片懒加载lazyload
* 增加了fancybox
* 增加了fastclick解决延迟问题
* 增加了gulp压缩网页css js样式

## 运行步骤
1. 安装 [[node.js]](https://nodejs.org/en/)
2. 安装 [[git]](https://git-scm.com/)
3. 克隆本仓库到本地
3. 进入next-template目录运行`npm install hexo --save`加载node_modules

## 自定义配置项
next主题下载下来后，除了上述`目录`介绍的内容更新外，用户还需要更改一些自己的额外信息。
### 主站配置文件_config.yml参数
```yaml
$ vi <folder>/_config.yml
title: 派 | Pyker   #修改博客标题
description: 与其临渊羡鱼，不如退而结网。     #修改描述博主描述
url: https://www.ipyker.com    # 修改自己的网站地址
deploy:
  repository: https://github.com/ipyker/ipyker.github.io    #修改成自己github pages地址
```
### 主题配置文件_config.yml参数
```yaml
favicon:    # 修改网站图标
  small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg

beian:      # 修改备案号 
  enable: true
  icp: 粤ICP备19028706号

github_banner:    # 修改github banner地址
  enable: true
  permalink: https://github.com/ipyker

social:            # 修改社交地址和图标
  GitHub: https://github.com/ipyker || github
  E-Mail: mailto:pyker@qq.com || envelope
  Weibo: https://weibo.com/viszhang || weibo
  QQ: tencent://message/?uin=xxxxxxxxx&Site=&menu=yes || qq

links:        # 修改推荐阅读
  运维生存时间: http://www.ttlsa.com/
  爱运维: https://www.iyunw.cn
  Nginxconfig: https://nginxconfig.io/
  Linux命令手册: http://linux.51yip.com/
  echarts可视化库: https://echarts.baidu.com/index.html

avatar:
  url: /images/avatar.jpg      #修改博主头像

reward:        #修改打赏二维码
  wechatpay: /images/reward/wechatpay.png
  alipay: /images/reward/alipay.png

livere_uid: MTAyMC80NDM5Mi8yMDkyNA==   # 修改来必力评论key，否则无法管理评论
```

