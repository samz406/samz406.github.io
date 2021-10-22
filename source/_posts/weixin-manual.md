---
title: 微信公众号开发
date: 2021-10-21 20:48:07
tags: [技术]
categories: 技术
---

公众号开发记录

<!-- more -->

  最近做了个公司公众号推送功能。这里记录下开发流程。



**相关地址**

[测试公众号地址](https://mp.weixin.qq.com/debug/cgi-bin/sandboxinfo?action=showinfo&t=sandbox/index)

[公众号开发文档](https://developers.weixin.qq.com/doc/offiaccount/Getting_Started/Overview.html)







### 测试公众号开发



#### 注册测试公众号

进入[测试公众号地址](https://mp.weixin.qq.com/debug/cgi-bin/sandboxinfo?action=showinfo&t=sandbox/index)，扫码登录。



![P1.png](P1.png)



#### 接口配置信息 

接口配置信息这里配置你的回调地址。配置回调地址作用





#### 创建回调服务

可以自己创建一个springboot服务，写两个方法，一个接收GET校验，一个接收POST微信事件。

**作用一：验证消息是否来自微信服务器**



验证流程

![P3.jpg](P3.jpg)

回调服务主要用于验证填写的接口配置，微信服务器将发送GET请求到填写的服务器地址URL上，GET请求携带参数如下表所示：



| 参数      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| signature | 微信加密签名，signature结合了开发者填写的token参数和请求中的timestamp参数、nonce参数。 |
| timestamp | 时间戳                                                       |
| nonce     | 随机数                                                       |
| echostr   | 随机字符串                                                   |



开发者通过检验signature对请求进行校验（下面有校验方式）。若确认此次GET请求来自微信服务器，**会原样返回echostr参数内容**，则接入生效，成为开发者成功，否则接入失败。加密/校验流程如下：



1）将token、timestamp、nonce三个参数进行字典序排序 

2）将三个参数字符串拼接成一个字符串进行sha1加密 

3）开发者获得加密后的字符串可与signature对比，标识该请求来源于微信



具体加密实现可以参考[微信官网](https://developers.weixin.qq.com/doc/oplatform/Third-party_Platforms/2.0/api/Before_Develop/Message_encryption_and_decryption.html)，也可以点击这里[下载实例代码](https://res.wx.qq.com/op_res/-serEQ6xSDVIjfoOHcX78T1JAYX-pM_fghzfiNYoD8uHVd3fOeC0PC_pvlg4-kmP)



注意：如果回调地址调用失败,会有如下提示，说明微信调用你服务失败，是不能添加成功，需要自己检查下网络。



![P4.png](P4.png)





> 由于一般都是本地环境开发，自己可以搞一个内网穿透工具，这里推荐[natapp](https://natapp.cn/)。具体使用自行研究。





**作用二：依据回调内容实现自定义业务逻辑**



​	用户每次向公众号发送消息、或者产生自定义菜单、或产生微信支付订单等情况时，开发者填写的服务器配置URL将得到微信服务器推送过来的消息和事件，开发者可以依据自身业务逻辑进行响应，如回复消息。



>  假如服务器无法保证在五秒内处理回复，则必须回复“success”或者“”（空串），否则微信后台会发起三次重试。解释一下为何有这么奇怪的规定。发起重试是微信后台为了尽可以保证粉丝发送的内容开发者均可以收到。如果开发者不进行回复，微信后台没办法确认开发者已收到消息，只好重试。



### 业务开发



业务开发就是回调服务的第二个功能。本次业务开发实现了公众号生成带二维码参数、模板消息推送、自动回复功能。这里简要说明下。

#### 前置条件



**1、 获取access_token**

调用微信接口都需要先获取access_token。access_token是公众号的全局唯一接口调用凭据，公众号调用各接口时都需使用access_token。开发者需要进行妥善保存。access_token的存储至少要保留512个字符空间。access_token的有效期目前为2个小时，需定时刷新，重复获取将导致上次获取的access_token失效。

本次开发是把access_token放到redis缓存的。设置2个小时有效期。



请求地址如下，对应[文档](https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/Get_access_token.html)

请求方式： GET

URL:  https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET



| 参数       | 是否必填 | 描述                                  |
| ---------- | -------- | ------------------------------------- |
| grant_type | 是       | 获取access_token填写client_credential |
| appid      | 是       | 第三方用户唯一凭证                    |
| secret     | 是       | 第三方用户唯一凭证密钥，即appsecret   |

> 测试公众号里有这两个appid和secret



正常返回结果如下，access_token 就是值保存起，调用其他接口都会带上

```json
{"access_token":"ACCESS_TOKEN","expires_in":7200}
```



**2 、H5相关配置**

​	如果是H5开发，需要在公众号配置中添加H5地址。配置项目为“网页授权获取用户基本信息”；实例如下

![P6.png](P6.png)



**3 、 获取用户基本信息**

​		由于不能直接拿到用户openId,需要通过code获取用户信息。请求如下

​		请求方式：GET

​		URL: https://api.weixin.qq.com/sns/userinfo?access_token=%s&openid=%s&lang=zh_CN

​		请求对象返回如下：

​		

```java
@Data
public class UserInfoResponse {

    private String openid;

    private String nickname;

    private Integer sex;

    private String province;

    private String city;

    private String country;

    private String headimgurl;

    private Integer errcode;

}
```



**4、特别提醒**

​	接受微信相关事件需要自己写一个接口。返回参数有讲究。如果是自动回复就需要按照XML格式返回，如果不是回复文字信息，就需要返回“success”或者“”（空串）。





#### 自动回复

自动回复就是用户首次关注公众号后，发送欢迎语。处理逻辑如下

1、如果收到的是event=subscribe的事件并且eventKey是空，就推送自动回复消息。

2、 构造回复消息对象并返回。回复对象如下：

​	

```java
@Setter
@Getter
@ToString
@XmlRootElement(name="xml")
@XmlAccessorType(XmlAccessType.FIELD)
public class OutMsgEntity {
    // 发送方的账号
    @XmlElement(name="FromUserName")
    protected String FromUserName;
    // 接收方的账号(OpenID)
    @XmlElement(name="ToUserName")
    protected String ToUserName;
    // 消息创建时间
    @XmlElement(name="CreateTime")
    protected Long CreateTime;
    /**
     * 消息类型
     * text 文本消息
     * image 图片消息
     * voice 语音消息
     * video 视频消息
     * music 音乐消息
     * news 图文消息
     */
    @XmlElement(name="MsgType")
    protected String MsgType;
    // 文本内容
    @XmlElement(name="Content")
    private String Content;

}
```



> 入参的FromUserName是返回对象ToUserName，入参的ToUserName是返回对象的FromUserName。





#### 带参数二维码



目前有2种类型的二维码：

1、临时二维码，是有过期时间的，最长可以设置为在二维码生成后的30天（即2592000秒）后过期，但能够生成较多数量。临时二维码主要用于帐号绑定等不要求二维码永久保存的业务场景

 2、永久二维码，是无过期时间的，但数量较少（目前为最多10万个）。永久二维码主要用于适用于帐号绑定、用户来源统计等场景。



用户扫描带场景值二维码时，推送以下两种事件：

1） 如果用户还未关注公众号，则用户可以关注公众号，关注后微信会将带场景值关注事件(event=subscribe)推送给开发者。

2）如果用户已经关注公众号，在用户扫描后会自动进入会话，会将带场景值扫描事件(event=SCAN)推送给开发者。



获取带参数的二维码的过程包括两步，首先创建二维码ticket，然后凭借ticket到指定URL换取二维码。

**创建二维码ticket**

每次创建二维码ticket需要提供一个开发者自行设定的参数（scene_id），分别介绍临时二维码和永久二维码的创建二维码ticket过程。

请求方式：POST

URL : https://api.weixin.qq.com/cgi-bin/qrcode/create?access_token=TOKEN

**请求body:**

```json
{
	"action_name": "QR_LIMIT_STR_SCENE",
	"action_info": {
		"scene": {
			"scene_str": "test"
		}
	}
}
```



**参数说明**

| 参数        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| scene_str   | 这里就是你自己业务设置的参数。                               |
| action_name | 二维码类型，QR_SCENE为临时的整型参数值，QR_STR_SCENE为临时的字符串参数值，QR_LIMIT_SCENE为永久的整型参数值，QR_LIMIT_STR_SCENE为永久的字符串参数值；这里选择的是QR_LIMIT_STR_SCENE |
| action_info | 二维码信息                                                   |
|             |                                                              |

**返回样例：** url就是二维码地址

```json
{"ticket":"gQH47joAAAAAAAAAASxodHRwOi8vd2VpeGluLnFxLmNvbS9xL2taZ2Z3TVRtNzJXV1Brb3ZhYmJJAAIEZ23sUwMEmm
3sUw==","expire_seconds":60,"url":"http://weixin.qq.com/q/kZgfwMTm72WWPkovabbI"}
```



#### 模板消息推送

[文档地址](https://developers.weixin.qq.com/doc/offiaccount/Message_Management/Template_Message_Interface.html)

请求方式：POST

URL: https://api.weixin.qq.com/cgi-bin/message/template/send?access_token=ACCESS_TOKEN

请求Body:

```json
 {
           "touser":"OPENID",
           "template_id":"ngqIpbwh8bUfcSsECmogfXcV14J0tQlEpBO27izEYtY",
           "url":"http://weixin.qq.com/download",  
           "miniprogram":{
             "appid":"xiaochengxuappid12345",
             "pagepath":"index?foo=bar"
           },          
           "data":{
                   "first": {
                       "value":"恭喜你购买成功！",
                       "color":"#173177"
                   },
                   "keyword1":{
                       "value":"巧克力",
                       "color":"#173177"
                   },
                   "keyword2": {
                       "value":"39.8元",
                       "color":"#173177"
                   },
                   "keyword3": {
                       "value":"2014年9月22日",
                       "color":"#173177"
                   },
                   "remark":{
                       "value":"欢迎再次购买！",
                       "color":"#173177"
                   }
           }
       }
```



> 注意：first，keyword1，keyword2 这些都是替换参数，替换成你自己的业务参数。



| 参数        | 是否必填 | 说明                 |
| ----------- | -------- | -------------------- |
| touser      | 是       | 接收者openid         |
| template_id | 是       | 模板ID(需要先配置好) |
| url         | 否       | 模板跳转链接         |
| data        | 是       | 模板数据             |

测试公众号上是可以配置模板的。样例如下

![P5.png](P5.png)



### 正式公众号配置

当把微信功能开发好后，进入服务器配置时。如下截图

![P7.png](P7.png)



当你启用配置时，弹框页面会提示你自定义菜单和自动回复失效。遇到这样的情况我们处理方法如下：

​	1 、先截图自定义菜单，目的是保存样式（如果存在）。

​	![P8.png](P8.png)

​	2、 保存自动回复消息内容（如果存在）。

​	3、 进入 微信公众平台接口调试工具[页面](http://mp.weixin.qq.com/debug?token=1274810508&lang=zh_CN)

​		3.1 首先获取accessToken,如果你能获取到，就不用再做如下请求。

![P9.png](P9.png)

​		3.2  获取已经定义好的菜单列表。截图如下

![P10.png](P10.png)

​		3.3  创建菜单 截图如下,将获取的菜单json修改下然后再作为body参数请求（目的就是还原之前的自定义菜单）

![P11.png](P11.png)

   4、 自动回复需要再回调接口中返回。前面已经有说明



菜单自定义功能详情说明[点击这里](https://developers.weixin.qq.com/doc/offiaccount/Custom_Menus/Creating_Custom-Defined_Menu.html)



#### H5配置

如果是H5网页开发。还要把H5地址添加到公众号功能设置里面。详情参见截图。网页授权域名、JS接口安全域名 两项配置。

![P12.png](P12.png)





### 部分代码

以下是webflux方式实现，与spring mvc稍有不同。由于需要返回字符串。所以自动回复只能放到 ServerHttpResponse返回出去。

```java
 /**
     * 微信回调入口
     *
     * @param entity
     * @return
     */
    @RequestMapping(value = "/callback", method = RequestMethod.POST, consumes = {"text/xml;charset=UTF-8"}, produces = {"text/xml;charset=UTF-8"})
    public Mono<String> event(@RequestBody InMsgEntity entity, ServerWebExchange exchange) {

        log.warn("微信回调地址:{}", entity.toString());
        // 首次订阅公众号 推送自动回复
        if (isSubscribleEvent(entity.getEvent()) && StringUtils.isEmpty(entity.getEventKey())) {
            String replay = getReplay(entity);
            //返回参数构造，返回参数为xml格式
            ServerHttpResponse response = exchange.getResponse();
            DataBuffer buffer = response.bufferFactory().wrap(replay.getBytes(StandardCharsets.UTF_8));
            response.getHeaders().setContentType(MediaType.TEXT_XML);
            return exchange.getResponse().writeWith(Mono.just(buffer)).thenReturn("");
        }
        //这里是对应的业务消息处理
        return doEvent(entity).thenReturn("success");
    }
```





### 



