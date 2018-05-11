#准备工作

<font color=red>
注意！注意！注意！demo中的数据库存储一定要修改为mysql等持久化存储。

demo部署后,一定要确认isv.log和filecache.php两个文件有写权限。
</font>

注意：钉钉小程序应用开发在内测阶段，必须通过我们的内测邀请审核才能进行开发。申请链接：https://survey.alibaba.com/survey/4MOQXNg7m

# 当前的问题

在开发钉钉ISV应用的时候，会发现钉钉ISV应用必须部署有公网IP的服务器上，也就是在开发应用之前必须先准备购买阿里云等服务器资源。在之后的开发中，由于ISV应用的回调地址必须是公网域名或IP之下，对于大部分开发者来说，开发者无法在本地调试远程代码，对于回调URL校验不通过之类问题无法追踪，只能不断远程部署来看log日志来调试修改，本文主要讲解如何解决这些问题。

# 解决方案

基于以上问题，我们设计了一套支持ISV应用本地化调试的方案，使用这套方案可以达到以下目的：
1. ISV应用开发阶段不需要购买阿里云等公网机器资源
2. 开发调试应用可以在本地apache 或者Nginx 上进行，不需要部署到远程公网机器上

# 启动内网穿透
#### 1.下载工具
```
git clone https://github.com/open-dingtalk/pierced.git
```


![image.png | left](https://gw.alipayobjects.com/zos/skylark/public/files/59fd4d93eaf4c0323df64cedaff08fb6.png "")

启动工具，执行命令“./ding -config=./ding.cfg -subdomain=域名前缀 端口”，以mac为例：
```
cd mac_64
chmod 777 ./ding
./ding -config=./ding.cfg -subdomain=abcde 8080
```
启动后界面如下图所示：


![image.png | left](https://gw.alipayobjects.com/zos/skylark/public/files/7552c06dd9a922602d13f56652940377.png "")

参数说明：

| 参数 | 说明 |
| :--- | :--- |
| config | 钉钉提供内网穿透的配置文件，不可修改 |
| subdomain | 你需要使用的域名前缀，该前缀将会匹配到“vaiwan.com”前面，例如你的subdomain是abcde，你启动工具后将会将abcde.vaiwan.com映射到本地 |
| 端口 | 你需要代理的本地服务http-server端口，例如你本地端口为8080等 |

#### 2.启动完客户端后，你访问http://abcde.vaiwan.com/xxxxx都会映射到 [http://127.0.0.1:8080/xxxxx](http://127.0.0.1:8080/xxxxx)
注意：
> 1.你需要访问的域名是http://abcde.vaiwan.com/xxxxx 而不是http://abcde.vaiwan.com:8082/xxxxx
> 2.你启动命令的subdomain参数有可能被别人占用，尽量不要用常用字符，可以用自己公司名的拼音，例如我使用：alibaba、dingding等。
> 3.可以在本地起个http-server服务，放置一个index.html文件，然后访问http://abcde.vaiwan.com/index.html测试一下。
# 准备开发环境

搭建本地Apache+PHP+Nginx项目环境，不一一赘述。

# DEMO部署
#### 下载示例代码
```
git clone https://github.com/open-dingtalk/isv_demo_php.git
```

# 创建小程序并配置代码
登录钉钉开发者平台，创建一个小程序，如下图所示：



![image.png | left | 748x279](https://cdn.yuque.com/lark/0/2018/png/20097/1524195256423-a24d2f7d-5c76-4a36-8542-fe28d0aaa9ee.png "")


请填写应用的名称、应用LOGO、应用描述、Token、数据加密秘钥、应用类型、应用IP白名单等信息并保存。




![image.png | left | 748x619](https://cdn.yuque.com/lark/0/2018/png/20097/1524195373143-fe5133d6-38b6-46e9-af93-74640afe20bc.png "")


选择类型为“测试应用”
__注意：类型一旦选定不能进行修改，测试应用主要是用来做开发调试，不能发布商品、不能生成线下部署二维码。__

创建完成后，我们在小程序的基础信息里能拿到如下参数：

> Token: 随意填写任意字符串
> suiteKey：应用key
> suiteSecret：应用秘钥
> AESKey：数据加密密钥，点击自动生成

我们用IDE打开示例DEMO代码isv-demo-php-miniapp，配置好如上参数。
1.isv-demo-php-miniapp/config.php 里面配置好SUITE_KEY、SUITE_SECRET、TOKEN、ENCODING_AES_KEY，如下所示：

```
<?php
define('DIR_ROOT', dirname(__FILE__).'/');
define("OAPI_HOST", "https://oapi.dingtalk.com");
//Suite
define("SUITE_KEY", "XXX");
define("SUITE_SECRET", "XXX");
define("TOKEN", "XXX");
define("ENCODING_AES_KEY", "XXX");

```

| 配置项 | 配置说明 |
| :--- | :--- |
| SUITE_KEY | 创建应用拿到的suitekey |
| SUITE_SECRET | 创建应用拿到的suiteSecret |
| TOKEN | 创建应用拿到的token |
| ENCODING_AES_KEY | 创建应用拿到的aesKey |

1.__<span data-type="color" style="color: rgb(245, 34, 45);">完成上面的配置后，请将代码部署到本地Apache或者Linux下面</span>。

> 注意：我们提供的内网穿透服务可能会有延迟，如果页面刷新不出来，请稍等3~5分钟后再试。

# 验证回调
在小程序的基础信息里面点击右上角"__<span data-type="color" style="color: rgb(245, 34, 45);">修改</span>__"按钮设置回调URL，在回调URL地址上填写通过远程代理的域名服务器地址，例如：http://abcde.vaiwan.com/isv-demo-php-miniapp/receive.php  【指向自己代码路径中的receive.php】。

点击“验证有效性”验证成功并保存，如下图所示：

![image.png | left | 747x767](https://cdn.yuque.com/lark/0/2018/png/20097/1526009429213-9e17d37d-bf5f-4c19-b37a-4d9f0ff7815d.png "")

# 开发小程序
请参考使用[钉钉开发工具开发小程序](https://open-doc.dingtalk.com/microapp/isv/oq1uge)，使用钉钉提供的小程序示例DEMO DingTalkProjects来开发第一款钉钉ISV小程序应用。




![image.png | left | 747x509](https://cdn.yuque.com/lark/0/2018/png/20097/1524897102842-1cfc2582-8f55-47fb-bb00-63d04b8b4f3e.png "")



#### 获取当前企业的corpId
在​app.js里面拿到corpId 存储在globalData里面

```plain
App({
  onLaunch(options) {
    this.globalData.corpId = options.query.corpId;
  },
  onShow() {
    console.log('App Show');
  },
  onHide() {
    console.log('App Hide');
  },
  globalData: {
    corpId: "",
    hasLogin: false,
  },
});
```

#### 获取免登授权码

```plain
var app = getApp();
    var me = this;
    console.log(app.globalData.corpId);
    requestAuthCode({
      corpId: app.globalData.corpId
    }).then((res) => {
      dd.httpRequest({
        url: 'http://x.x.x.x/sendMsg.php',
        method: 'POST',
        data: {
          code: res.code,
          corpId:app.globalData.corpId,
          event:"get_userinfo"
        },
        dataType: 'json',
        success: function(res) {
          console.log(res);
          me.setData({
            userName : res.data.userid,
            deviceId : res.data.deviceId
          });
        },
        fail: function(res) {
          //my.alert({content: 'fail'});
        },
        complete: function(res) {
          //my.hideLoading();
          //my.alert({content: 'complete'});
        }
        
      });
        console.log(`auth success, authCode is ${res.code}`);
    }).catch((err) => {
        console.log(`auth fail, error message is ${JSON.stringify(err)}`);
    })
```
# 发布版本
请参考小程序开发者工具IDE的使用[发布一个版本](https://open-doc.dingtalk.com/microapp/isv/oq1uge#%E5%8F%91%E5%B8%83)，同时在开发者后台进行灰度或者发布到线上。
# 预览小程序
请参考[钉钉小程序发布预览](https://open-doc.dingtalk.com/microapp/isv/gwads2)。