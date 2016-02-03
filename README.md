# 自动遍历工具

# 为什么做这个工具
* 各大云市场上自动遍历功能也都是限制了时长,并且便利并不支持定制
* 降低边边角角的漏测问题
* 结合接口测试发现后台问题
* 发现深层次的布局问题. 通过新老版本的diff可以发现每个版本的变化和变动
* 解决monkey等工具太弱智不可控的缺点

# 设计目标
* 自动爬取加上规则引导(完成)
* 支持定制化, 可以自己设定遍历深度(完成)
* 支持插件化, 允许别人改造和增强(TODO)
* 支持滑动等更多动作(TODO)
* 支持自动截获接口请求(Doing)
* 支持新老版本的界面对比(Doing)

# 安装环境
### mac下安装appium
<pre>
brew install node
npm install -g appium
</pre>

### 准备环境
真机或者模拟器均可. 确保adb devices可以看到就行
### 下载appcrawler.
可以使用打包好的工具, 不需要安装scala和sbt.只要有java即可  
运行示例
<pre>
appcrawler conf/xueqiu.conf
</pre>

### 配置文件定制化
通过修改配置文件. 可以实现细节的控制.
<pre>
{
  "saveScreen" : true,
  "currentDriver": "android",
  "capability" : {
    "deviceName" : "",
    "platformVersion" : "",
    "platformName" : "",
    "autoWebview" : "false",
    "autoLaunch" : "false",
    "noReset" : "true",
    "app" : "/Users/seveniruby/Downloads/bb38d2a653301fe816de0a3aa7eb6520.apk"
  },
  "androidCapability" : {
    "appPackage" : "com.xxx.xxx",
    "appActivity" : "LaunchActivity",
    "appium" : "http://127.0.0.1:4730/wd/hub"
  },
  "iosCapability" : {
    "deviceName" : "iPhone 6",
    "bundleId" : "",
    "platformVersion" : "9.2",
    "appium" : "http://127.0.0.1:4723/wd/hub",
    "autoAcceptAlerts" : "true"
  },
  "defineUrl" : "",
  "baseUrl" : "",
  "maxDepth" : 6,
  "blackUrlList" : [ "RoomList.*", "Chat.*"],
  "backButton" : [ ],
  "firstList" : [ ],
  "selectedList" : [
    "//*[@enabled='true' and @resource-id!='' and not(contains(name(), 'Layout'))]",
    "//*[@enabled='true' and @content-desc!='' and not(contains(name(), 'Layout'))]",
    "//android.widget.TextView[@enabled='true']",
    "//android.widget.ImageView[@enabled='true' and @clickable='true']"
  ],
  "lastList" : [ ],
  "blackList" : [ "消息", "聊天室" ],
  "elementActions" : [ ]
}
</pre>

# 设计理念
## 定义url
界面唯一性:每个screen都有一个唯一的id, 这样可以类比为普通的接口测试中的url.  
android的url默认为当前的activity名字.  
iOS没有activity概念, 默认使用当前页面dom的md5值的后五位作为标记. 如果页面不变. 那么这个md5值也不会变.  
也可以自己指定某些特征作为url, 比如title或者某些关键控件的文本

控件的唯一性取决于这个url和控件自身的id name tag text loc等属性.  
比如一个输入框id=input, 在多个页面中都出现了.  
如果url为空, 那么它只会被点击一次. 
如果url设置为当前activiy的名字, 那么有多少页面包含它他就会被点击多少次.  

url的定义是一门艺术, 可以决定如何优雅的遍历.  

## 设定引导规则
遇到什么控件触发什么操作, 用来做引导输入, 是一种触发机制.    
rule方法有三个参数. 元素的id或者name属性.第二个参数为输入. "click"会执行点击操作. 其他都会被当成文本输入.  
第三个参数为这个规则被应用多少次. 默认是无限. 比如登录时的输入,可以设置为1次. 大部分情况默认即可.  
<pre>
  "elementActions":[
    {
      "action":"click",
      "idOrName":"已有帐号？立即登录",
      "times":0
    },
    {
      "action":"click",
      "idOrName":"登录",
      "times":0
    },
    {
      "action":"156005347XX",
      "idOrName":"account",
      "times":0
    },
    {
      "action":"xxxxxxxx",
      "idOrName":"password",
      "times":0
    }
  ]
</pre>
## 后退标记back
android默认是back键,不需要设定.  
iOS上没有back键, 需要自己指定, 通过xpath定位方式指定遍历完所有控件应该点击什么控件返回. 
<pre>
  "backButton":[
    "//*[@name='nav_icon_back']",
    "//UIAButton[@name='取消']",
    "//UIAButton[@name='Cancel']",
    "//UIAButton[@name='关闭']",
    "//*[@value='首页']",
    "//UIAButton[@name='首页']"
  ],
</pre>
## 黑名单black
控件黑名单为black方法. 他会绕过id name或者text中包含特定关键词的控件.  
url黑名单可以绕过特定的activity或者window  
<pre>
  "blackList" : [ "消息", "聊天室" ]
</pre>
## 遍历顺序控制
适用于在一些列表页或者tab页中精确的控制点击顺序  
selectedList表示默认要遍历的元素特征  
first表示优先遍历元素特征  
last表示最后应该遍历的元素特征  
统一使用XPath来表示  
<pre>
"firstList":[
    "//android.widget.ListView//android.widget.TextView",
    "//android.widget.ListView//android.widget.Button"
  ],
  "selectedList":[
    "//*[@enabled='true' and @resource-id!='' and not(contains(name(), 'Layout'))]",
    "//*[@enabled='true' and @content-desc!='' and not(contains(name(), 'Layout'))]",
    "//android.widget.TextView[@enabled='true' and @clickable='true']",
    "//android.widget.ImageView[@clickable='true']",
    "//android.widget.ImageView[@enabled='true' and @clickable='true']"
  ],
  "lastList":[
    "//*[contains(@resource-id,'group_header_view')]//android.widget.TextView"
  ],
</pre>

## url控制
支持黑名单, 最大遍历深度, 和重命名url, 重新设定初始url  
<pre>
  "defineUrl":"//*[contains(@resource-id, '_title')]",
  "baseUrl":".*Main.*",
  "maxDepth":3,
  "blackUrlList":[
    "StockMoreInfoActivity",
    "StockDetailActivity",
    "UserProfileActivity"
  ],
</pre>


# 测试用例方式
除了命令行用法外, 也可以使用标准的测试用例方式去指定. 从而跟自动化测试进行结合.  
<pre>
class Demo extends FunSuite{
  test("xueqiu"){
    val futu=new AndroidCrawler
    futu.conf.capability++=Map("app"->"http://xqfile.imedao.com/android-release/xueqiu_730_01191600.apk")
    futu.conf.androidCapability++=Map("appPackage"->"com.xueqiu.android")
    futu.conf.androidCapability++=Map("appActivity"->".view.WelcomeActivityAlias")
    futu.setupApp()
    futu.start()
  }
</pre>