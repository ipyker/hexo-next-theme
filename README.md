[![](https://img.shields.io/badge/Hexo-brightgreen.svg?style=plastic)](https://hexo.io/)
[![](https://img.shields.io/badge/nexT-pyker-yellow.svg?style=plastic)](https://github.com/theme-next/hexo-theme-next)

[中文介绍](zh-CN/README.md) | [nexT Demo](https://www.ipyker.com)
## Catalogue
We mainly performed the following configuration operations on the next theme.

* Set Chinese language
* Set blogger text description
* Set up blog article connections format is year/month/day/title.html
* Menu add about, tags, categories, interactions, search menus
* Disable about, label, and category menu comments
* Add RSS
* Set the background image. Disabled by default，in themes/nexT/source/css/_custom/custon.styl file open
* Set Canvas_nest dynamic background
* Image quick load settings
* WeChat Alipay reward function
* Click to appear the heart effect
* Home article add shadow effect
* Set code highlighting
* header bar background color
* Background color of the footer bar
* Implement the red fork me on github in the upper right corner. Disabled by default, can be enabled in the themes/nexT/_config.yml file github_banner
* Modify the link text style within the article
* Modify the label style at the bottom of the article
* Add an "End of article" tag at the end of the article
* Set Avatar
* Add traffic to the footer of the site
* Word count at the bottom of the site
* Add website runtime at the bottom of the site
* Add a dynamic heart at the bottom of the site
* Add filing information at the bottom of the website
* Bottom hidden by Hexo powerful drive, theme - NexT.Mist
* Set the website icon Favicon
* Implement article text statistics and reading time
* Add a force to follow the cloud function. (You need to register yourself to get the ID)
* Remove the bottom count statistics
* Modify font size
* Add DaoVoice online contact. (You need to register yourself to get the ID)
* Sidebar social small icons set
* Add sidebar recommended reading
* Modify the sidebar background image
* Modify the sidebar text color
* Add copyright information at the bottom of the article
* Add a site search to the Hexo blog
* Modify the color of the selected character
* Add aplay music player
* Add a cartoon in the lower right corner of the blog
* Added 3D library three_waves, closed by default
* Added canvas page ribbon
* Added home page loading loading progress
* Increase the picture lazy loading lazyload
* Added fancybox 
* Added fastclick to resolve latency issues
* Added gulp compression web page css js style

## Operating Step
1. Install [[node.js]](https://nodejs.org/en/)
2. Install [[git]](https://git-scm.com/)
3. clone  repository to local
3. Enter next-template directory RUN `npm install hexo --save`to load node_modules

## Custom configuration items
After the next theme is downloaded, in addition to the content update described in the above `Directory`, the user also needs to change some of his own additional information.
### Master configuration file _config.yml parameter
```yaml
$ vi <folder>/_config.yml
title: 派 | Pyker   #Edit blog title
description: 与其临渊羡鱼，不如退而结网。     #Modify description blogger description
url: https://www.ipyker.com    # Modify your website address
deploy:
  repository: https://github.com/ipyker/ipyker.github.io    #Modify into your own github pages address
```
### Theme configuration file _config.yml parameter
```yaml
favicon:    # Edit website icon
  small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg

beian:      # Modify the record number 
  enable: true
  icp: 粤ICP备19028706号

github_banner:    # Modify github banner address
  enable: true
  permalink: https://github.com/ipyker

social:            # Edit social addresses and icons
  GitHub: https://github.com/ipyker || github
  E-Mail: mailto:pyker@qq.com || envelope
  Weibo: https://weibo.com/viszhang || weibo
  QQ: tencent://message/?uin=xxxxxxxxx&Site=&menu=yes || qq

links:        # Modify recommended reading
  运维生存时间: http://www.ttlsa.com/
  爱运维: https://www.iyunw.cn
  Nginxconfig: https://nginxconfig.io/
  Linux命令手册: http://linux.51yip.com/
  echarts可视化库: https://echarts.baidu.com/index.html

avatar:
  url: /images/avatar.jpg      #Modify the blog avatar

reward:        #Modify the reward QR code
  wechatpay: /images/reward/wechatpay.png
  alipay: /images/reward/alipay.png

livere_uid: MTAyMC80NDM5Mi8yMDkyNA==   # Modify to force the comment key, otherwise you can't manage the comment
```