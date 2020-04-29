---
title: python_selenium
categories: python
tag: hide
date: 2018-12-18 22:24:31
tags:
---
Selenium简介：

支持多种语言。随着Python语言运用的越来越广，使用Python Selenium的频率逐渐变多，所以该篇文章介绍的Selenium是基于Python语言的。

支持浏览器：IE,Chrome,FireFox,Safari。支持Windows，Mac系统平台上运行

本篇文章适合有Python基础的，想尝试使用或者对Selenium有兴趣的同学们

需要Python基础教程的同学们，可参考先学习：

Python教程 - 廖雪峰的官方网站

https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000

前期准备
1.       安装Python

下载地址：https://www.python.org/downloads/release/python-364/

选择 Python 3.6.4 版本下载，根据自己的系统注意选择区分32和64安装包进行下载：


安装具体过程不在详述，可参考上述基础教程中的内容

2.       安装Python Selenium组件

利用pip install selenium命令，即可安装python中的selenium组件最新版本（当前最新版本为3.8.1），如下所示


安装完成后，会提示成功安装该组件，可以用命令pip list来验证，如下图所示：


如果要升级该组件，可用命令:pip install selenium –U进行组件版本的升级

Notes:

在安装组件的时候时常会遇到网络原因无法下载，下列方法，可进行解决:

让PIP源使用国内镜像，提升下载速度和安装成功率。 - microman - 博客园

http://www.cnblogs.com/microman/p/6107879.html

3.       IE Driver (IE浏览器驱动):

下载地址: http://docs.seleniumhq.org/download/，注意区分32位和64位系统


下载解压如下：


Notes: 建议选择32位驱动下载，在64位驱动中，会出现输入文字过慢的情况，在32位驱动中不存在该情况

4.       ChromeDriver(Chrome浏览器驱动):

下载地址：http://chromedriver.storage.googleapis.com/index.html

当前最新版本是2.33，点击该文件夹，如下图所示：


目前似乎只有32位的浏览器驱动，只有选择这个。

下载解压如下：



5.       Gecko Driver（Firefox浏览器驱动）

自Selenium进入3.0版本后，Firefox的驱动独立于Selenium组件，需要另行单独下载，不再自带

下载地址：https://github.com/mozilla/geckodriver/releases，注意区分32位和64位系统，这里依然推荐使用32位版本的驱动。


选择32位驱动下载，解压如下：


NOTES: 为了方便管理，建议将这些解压缩的驱动文件放置在一个文件夹内方便管理和后续的调用使用。

还需要注意的是，一旦浏览器版本进行了更新，那么对应的浏览器驱动也需要进行及时更新，可以关注浏览器驱动的更新日志。否则，浏览器会无法正常启动。



配置浏览器和利用框架进行测试
1.     启动三大常用浏览器

环境准备：新建一个文件夹命名为MyDriver，和上述的Drivers文件夹放在同一层级，并且在MyDriver文件夹中新建一个Getbrowser.py的Python编译文件，如下图所示：




准备好了上述环境，接下来会在Getbrowser.py中利用代码来具体实现如何启动常用的浏览器

1.1  IE浏览器

import os

from selenium import webdriver #从selenium组件包中引入webdriver

def IE():

ifos.name == 'nt': #Windows系统

IEDriver =os.path.join(os.path.abspath('..'),'Drivers','IEDriverServer.exe')

#利用相对路径去寻找Drivers文件夹的IE驱动文件夹（推荐）

#IEDriver2 ="D:\John's Code Project\Python-selenium\Drivers\IEDriverServer.exe"

#利用绝对路径去寻找Drivers文件夹的IE驱动文件夹

os.environ['webdriver.ie.driver']= IEDriver

#os.environ['webdriver.ie.driver']= IEDriver2

#将IE驱动文件路径赋给环境变量webdriver.ie.driver

driver =webdriver.Ie(IEDriver)

#启动IE浏览器

driver.maximize_window()

#浏览器最大化

driver.quit()

#退出浏览器，并杀掉其进程

else:

print ("IE cannot be ran onnon-Windows system!")

#IE浏览器无法在非Windows系统下运行

1.2  Chrome浏览器

def Chrome():

    if 'nt' in os.name:

        CDriver =os.path.join(os.path.abspath('..'),'Drivers','chromedriver.exe')

#利用相对路径去寻找Drivers文件夹的Chrome驱动文件夹（推荐）

        #CDriver2 = "D:\John's CodeProject\Python-selenium\Drivers\chromedriver.exe"

#利用绝对路径去寻找Drivers文件夹的Chrome驱动文件夹

    elif 'posix' in os.name: #Mac OS X系统

        CDriver =os.path.join(os.path.abspath('..'),'Drivers','chromedriver')

        #CDriver2 = "D:\John's Code Project\Python-selenium\Drivers\chromedriver"

    os.environ['webdriver.chrome.driver'] =CDriver

#os.environ['webdriver.chrome.driver']= CDriver2

#将Chrome驱动文件路径赋给环境变量webdriver.chrome.driver

    driver = webdriver.Chrome(CDriver)

#启动Chrome浏览器

driver.maximize_window()

#浏览器最大化

driver.quit()

#退出浏览器，并杀掉其进程

1.3  Firefox浏览器

与IE,Chrome浏览器驱动文件设置方法不同，Firefox的Gecko驱动文件需要先手动复制或者脚本方法复制（下列例子中代码会涉及）到Firefox浏览器所在的浏览器目录。默认为: C:\Program Files\Mozilla Firefox。若浏览器安装在其他目录下，则设置到自定义的目录。否则，Firefox浏览器会因为设置问题而无法启动。

def FireFox():

   if 'nt' in os.name: #Windows 系统

        FFDriver =os.path.join(os.path.abspath('..'),'Drivers','geckodriver.exe')

#定义Firefox浏览器geckodriver所在的路径

        FFBrowser = 'C:\Program Files\MozillaFirefox'

#设置Firefox浏览器所在的目录，非该驱动文件的所有路径

        if notos.path.exists(os.path.join(FFBrowser,'geckodriver.exe')):

           shutil.copy(FFDriver,os.path.join(FFBrowser,'geckodriver.exe'))

#将Drivers目录中的geckodriver.exe复制到Firefox浏览器所在目录下

        os.environ['PATH'] = FFBrowser

#将浏览器所在目录设置到系统变量PATH

driver =webdriver.Firefox(FFBrowser)

#启动Firefox浏览器

driver.maximize_window()

#浏览器最大化

driver.quit()

#退出浏览器，并杀掉其进程

2.     利用Python自带UnitTest框架进行首次测试

Unittest框架是Python自带的单元测试的框架，由三部分组成，分别是setUp,test方法和tearDown。其结构类似于Java Selenium中的TestNG框架

在MyDriver文件夹新建一个py文件，暂时命名为test.py，与Getbrowser.py同层级，如下所示：


先对Getbrowser.py进行小幅度修改,将上述driver.quit()方法更改为return driver. 目的是能够被测试用例调用不同的浏览器驱动方法。

现在进入test.py进入第一个unittest测试用例的编写:

import os

import time

import Getbrowser #引入Getbrowser，以便调用不同的浏览器

import unittest #引入unittest单元测试框架

class RunSogou(unittest.TestCase): #新建一个unittest的TestCase的类

   def setUp(self):  #运行测试用例前需要执行的方法

        self.driver = Getbrowser.Chrome()

#调用Getbrowser的Chrome浏览器,也可调用IE,Firefox浏览器

def testRunSogou(self): #测试用例主方法,必须以test开头命名,否则无法运行

        driver = self.driver

        driver.get("http://www.sogou.com")

        time.sleep(2)

        self.assertIn('搜狗搜索',driver.title)

        time.sleep(2)

#以上方法为打开搜狗主页，并验证网页标题正确与否，涉及的代码会在后续章节”常见使用方法”，详细说明，这里只需了解即可

   def tearDown(self): #运行测试用例完毕后执行的方法

        self.driver.quit() #退出浏览器

if __name__ == '__main__':

   unittest.main() #执行unittest的主方法，则会运行上述脚本

接下来运行脚本，就可以看到Chrome浏览器打开搜狗主页了


运行成功且没有发现任何问题，unittest会返回测试用例成功执行的消息:


========以上为环境配置和搭建内容，接下来都是使用的常见方法============
常见使用API的方法
1.      跳转至指定网页

driver.get('http://www.baidu.com')

2.      获取当前页面标题内容

driver.title

3.      获取当前网页地址

driver.current_url

4.      获取当前页面元素

driver.page_source

5.      回退到之前打开的页面

driver.back()

6.      前进到回退之前的页面

driver.forward()

7.      获取页面上的元素

driver.find_element_by_id('su')

#寻找id为’su’的元素

driver.find_element_by_name('wd')

#寻找name为’wd’的元素

driver.find_element_by_link_text('贴吧')

#寻找链接文本信息为’贴吧’的元素

driver.find_element_by_class_name('c-tips-container')

#寻找classname为’c-tips-container’的元素

driver.find_element_by_xpath('//*[@id="kw"]')

#寻找xpath中带有id名为kw的元素

driver.find_element_by_tag_name('div')

#寻找标签名带有’div’的元素

driver.find_element_by_partial_link_text('新')

#寻找链接文本部分带有’新’的元素

8.      设置浏览器窗口大小

driver.set_window_size(800,600)#将浏览器窗口大小设置为800*600

driver.maximize_window() #将浏览器窗口设置最大化

9.      对寻找到的元素进行一些操作

以百度搜索首页为例，进行操作：

driver.find_element_by_name('wd').send_keys(‘PythonSelenium’)

#在百度文本搜索框内输入内容Python Selenium

driver.find_element_by_name('wd').clear()

#清空文本搜索框的所有内容

driver.find_element_by_id('su').click()

#点击搜索按钮

driver.find_element_by_id('su').submit()

#提交搜索按钮，这里效果同点击搜索按钮

10.   获取元素的属性

以百度搜索首页为例，进行操作：

driver.find_element_by_id('kw').size

#获取搜索框的大小

driver.find_element_by_id('jgwab').text

#获取控件id为’jgwab’所显示的内容

driver.find_element_by_id('su').get_attribute('type')

#获取控件id为’su’的属性，例如button,radio等

driver.find_element_by_name('wd').is_displayed()

#判断搜索文本框当前是否被显示

11.   模拟鼠标的操作

以百度搜索首页为例，进行操作：

需要引入包ActionChains：

from selenium.webdriver.common.action_chainsimportActionChains

ActionChains(driver).context_click(driver.find_element_by_id('kw')).perform()

#在搜索文本框利用context_click()进行鼠标右击,perform() 是执行方法的语句

ActionChains(driver).double_click(driver.find_element_by_id('kw')).perform()

#在搜索文本框利用double_click()进行双击, perform() 是执行方法的语句

ActionChains(driver).drag_and_drop(driver.find_element_by_id('kw'),driver.find_element_by_id('su')).perform()

#利用drag_and_drop(source,target)从搜索文本框移动到点击按钮, perform() 是执行方法的语句

ActionChains(driver).move_to_element(driver.find_element_by_id('kw')).perform()

#利用move_to_element()停留在文本搜索框上, perform() 是执行方法的语句

ActionChains(driver).click_and_hold(driver.find_element_by_id('su')).perform()

#利用click_and_hold()可以点击并且停留在搜索按钮上, perform() 是执行方法的语句

12.   模拟键盘操作

以百度搜索首页为例，进行操作：

需要引入包ActionChains：

from selenium.webdriver.common.keysimportKeys

driver.find_element_by_id('kw').send_keys(Keys.BACK_SPACE)#Backspace键

driver.find_element_by_id('kw').send_keys(Keys.SPACE)#Space键

driver.find_element_by_id('kw').send_keys(Keys.DELETE)#Delete键

driver.find_element_by_id('kw').send_keys(Keys.CONTROL,'a')# CTRL + A

driver.find_element_by_id('kw').send_keys(Keys.CONTROL,'x')# CTRL + X

driver.find_element_by_id('kw').send_keys(Keys.CONTROL,'v')# CTRL + V

driver.find_element_by_id('kw').send_keys(Keys.ENTER)#ENTER键

driver.find_element_by_id('kw').send_keys(Keys.CONTROL,'c') # CTRL + C

driver.find_element_by_id('kw').send_keys(Keys.TAB)# TAB键

13.   智能等待方法

需要引入包 WebDriverWait

fromselenium.webdriver.support.uiimport WebDriverWait

WebDriverWait(driver,10).until(lambda driver :  driver.find_element_by_id('kw'))

#利用WebDriverWait方法 等待10秒，直到id为’kw’控件出现

driver.implicitly_wait(30)#在页面上隐式等待30 秒

import time #引入time包

time.sleep(5) # 等待5秒

14.   定位元素


checkboxs = driver.find_elements_by_tag_name('input')

#用find_elements 方法去寻找所有标签名’input’的元素

radios = driver.find_elements_by_css_selector('input[type=radio]')

#用find_elements 方法去寻找所有 css选择器名为input[type=radio]的元素

for mycheckbox in checkboxs: #循环遍历所有input元素控件

  if mycheckbox.get_attribute('type') == 'checkbox':

#找到’属性type’为’checkbox’

      mycheckbox.click() #点击

for myradiobutton in radios: #循环遍历所有radio元素单选控件

   myradiobutton.click()#全部点击

len(checkboxs) #计算复选框的数量

len(radios) #计算单选框的数量

checkboxs.pop(1).click()

#去掉勾选复选框的1序号的元素,pop() 为空时,默认去选最后一个元素

15.      定位层级


driver.find_element_by_link_text('Link1').click()

#点击Link1 之后会弹出下拉列表

dropdownlist = driver.find_element_by_id('dropdown1').find_element_by_link_text('Anotheraction')

#找到下拉列表后弹出的'Another action'文字

ActionChains(driver).move_to_element(dropdownlist).perform()

#光标会移动到该元素上

16.      定位框架


driver.switch_to_frame('f1')# 切换到框架f1

driver.switch_to_frame('f2') # 切换到框架f2

17.      处理对话框



driver.find_element_by_id('u1').find_element_by_name('tj_login').click()

#在百度首页找到登录按钮

driver.find_element_by_class_name('tang-content').find_element_by_id('TANGRAM__PSP_10__userName').send_keys('Selenium')

#通过二级定位找到用户名输入框，输入框在控件名为’ tang-content’之下，并输入用户名

driver.find_element_by_class_name('tang-content').find_element_by_name('password').send_keys('Selenium')

#通过二级定位找到密码框，密码框在控件名为’ tang-content’之下，并输入密码   

driver.find_element_by_class_name('tang-content').find_element_by_css_selector('input[type=submit]').click()

#通过二级定位找到登录按钮，登录按钮在控件名为’ tang-content’之下，并点击

18.      Alert框处理


driver.find_element_by_name('button').click()#点击网页中的上图按钮

Alert = driver.switch_to_alert()

#利用switch_to_alert()方法去获取当前页面中的Alert 对话框

print(Alert.text)#输出Alert对话框中的文字

Alert.accept() #.accept 点击Alert对话框的确定按钮

19.      Confirm框


driver.find_element_by_css_selector('input[type=button]').click()

#点击网页中的上图按钮

Confirm = driver.switch_to_alert()

#利用switch_to_alert()方法去获取当前页面中的 Confrim 对话框

print(Confirm.text)#输出Confirm对话框中的文字

Confirm.dismiss()#利用.dismiss() 去点击Confirm的取消按钮

20.      Prompt框


driver.find_element_by_id('button').click()#点击网页中的上图按钮

Prompt = driver.switch_to_alert()#利用switch_to_alert()方法去获取当前页面中的 Prompt 对话框

print(Prompt.text)#输出Prompt对话框中的文字

Prompt.send_keys(“Hello Selenium”)

#在输入框内输入HelloSelenium

Prompt.accept() #点击Prompt对话框的确定按钮

21.      操作下拉列表


#遍历寻找方法

dropdownlist =driver.find_elements_by_tag_name('option')

#利用find_elements去寻找标签名为option的所有元素

for mychoose in dropdownlist: #遍历整个options列表

if mychoose.get_attribute('value')== '10.69':

#找到value属性为10.69的并点击

       mychoose.click()

#利用层级定位去寻找value为7.45的选项并点击

driver.find_element_by_id('ShippingMethod').find_element_by_xpath("//option[@value='7.45']").click()

#利用两次点击，寻找到value为11.61的选项并点击

driver.find_element_by_id('ShippingMethod').click()

driver.find_element_by_xpath("//option[@value='11.61']").click()

22.      分页处理


pages =driver.find_element_by_class_name('yem').find_elements_by_tag_name('option')

#利用二层定位找到下拉列表分页元素

print("total page is : " +str(len(driver.find_element_by_id('pageE1m_a74e_ce2c').find_elements_by_tag_name('option'))))

#利用二层定位输出下拉列表分页的长度

for page in pages: #遍历整个下拉列表，且将value属性值为6的点击选中

if page.get_attribute('value')== '6':

    page.click()

23.      模拟上传文件


driver.find_element_by_name('file').send_keys(‘C:\\123.txt’)

#利用send_keys方法，输入文件路径C:\\123.txt，模拟上传文件

24.      使用JavaScript语言


driver.execute_script('$("#tooltip").fadeOut();')

#.execute_script是执行JavaScript语句的的方法,fadeout()方法可以隐藏特地元素，这里是指"#tooltip"

说明：因为fadeOut()的Jquery语言的一个方法，所以它必须遵循Jquery的语法,必须在语言头加一个$号

Button =driver.find_element_by_class_name('btn')

driver.execute_script('arguments[0].click()',Button)

#因.click()不是Jquery的语法，所以不需要加$号，该语句的作用是点击’btn’按钮，效果同Button.click()，这里用js语法实现

25.      控制浏览器滚动条

ScrollToBottom = 'window.scrollTo(0,document.body.scrollHeight)'

#将滚动条拖到最底

ScrollToTop= 'window.scrollTo(document.body.scrollHeight,0)'

#将滚动条拖到顶

ScrollUp ='window.scrollTo(document.body.scrollHeight,500)'

#将滚动条向上移动500px单位

ScrollDown= 'window.scrollTo(500,1500)'

#将滚动条从 500px的地方移动到1500px的地方

driver.execute_script(ScrollToBottom)

driver.execute_script(ScrollUp)

driver.execute_script(ScrollDown)

driver.execute_script(ScrollToTop)

#利用execute_script方法，执行JavaScript语句

26.      操作Cookie

打开有道官网 http://www.youdao.com

driver.add_cookie({'name':'name_aaaaaa', 'value':'value_bbbb'})

#使用add_cookie()去添加cookie方法.添加cookie必须在读取去完成，否则会把新添加的值给覆盖掉

cookies = driver.get_cookies()

#get_cookies()方法可以获取当前页面所有的cookie

for cookie in cookies:

print("%s-> %s" % (cookie['name'],cookie['value']))

#在当前页面获取所有name和value的cookie

driver.delete_cookie('CookieName')#删除名为CookieName的cookie

driver.delete_all_cookies()# 删除所有cookie

27.      获取其他的属性元素

Checkboxs =driver.find_elements_by_css_selector('input[type=checkbox]')

#找到checkobox复选框元素

for mycheckbox in Checkboxs:

if mycheckbox.get_attribute('data-node') =='594434498':

#找到data-node的值为'594434498'的复选框，并勾选

          mycheckbox.click()

if mycheckbox.get_attribute('data-convert')=='1':

#找到data-convert的值为'1'的复选框，并勾选

            mycheckbox.click()

if mycheckbox.get_attribute('data-type') == 'file':

#找到data-type的值为'file'的复选框，并勾选

   mycheckbox.click()

28.      退出浏览器

driver.quit()

29.      Assert验证

结合unittst框架

self.asserIn(‘百度一下’,driver.title)

self.assertEqual(‘百度一下’,driver.title)

#验证’百度一下’,是不是在百度首页标题内，格式例如:asserIn(期望值,实际值). asserIn不需要全字匹配，assertEqual需要全字匹配

30.      Demo

结合JavaScript语句，在搜狗搜索中进行搜索内容的相关验证

import unittest

import os

import time

import Getbrowser

import sys

from selenium.webdriver.support.ui importWebDriverWait from selenium.webdriver.common.action_chains import ActionChains

classUseJavaScriptOnSogou(unittest.TestCase):

   def setUp(self):

       self.driver = Getbrowser.Chrome()

       deftestUseJavaScriptOnSogou(self):

       driver = self.driver

       URL = 'http://www.sogou.com'

       driver.get(URL)

       Content = 'Hello Selenium'

       time.sleep(3)

       TitleName = driver.execute_script('return document.title')

       if '搜狗搜索引擎 - 上网从搜狗开始' == TitleName:

           print("Title name is correct!")

       else:

           print("Current title name is " + TitleName)

       SearchBox = driver.find_element_by_id('query')

       SearchButton = driver.find_element_by_id('stb')

driver.execute_script('arguments[0].value= "'+ Content + '"',SearchBox)#效果同send_keys

       time.sleep(2)

       driver.execute_script('arguments[0].click()',SearchButton)

#效果同.click()

       time.sleep(2)

       TitleName2 = driver.execute_script('return document.title')

#效果同driver.title

       if Content + ' - 搜狗搜索' == TitleName2:

           print("Title name of search is correct!")

       else:

           print("Current title name is " + TitleName2)

       time.sleep(2)

deftearDown(self):

       self.driver.quit()

if __name__ == '__main__':

  unittest.main()

要了解更多API，可以查看：http://selenium-python-zh.readthedocs.io/en/latest/api.html

其他的问题和内容
1.      关于IE的一些坑

a)      将网页缩放比例设置为100%

如果设置为非100%，会出现报错，需将浏览器网页缩放比例设置为100%才能保证浏览器能够正常运行

b)     启用保护模式

打开IE浏览器 à Internet选项 à 安全 在“安全”的标签卡中，选择所有的安全区域，将“启用保护模式”勾选，否则也会出现运行错误

c)      IE11在Windows系统无法启动

除了完成上述两点，还需要在注册表中进行设置：

32位：HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\InternetExplorer\Main\FeatureControl\FEATURE_BFCACHE

64位：

HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Internet

Explorer\Main\FeatureControl\FEATURE_BFCACHE

如果FeatureControl下没有FEATURE_BFCACHE，就以FEATURE_BFCACHE为名新建一个项。并在其下创建一个DWORD，取名为：iexplore.exe，value为0

设置完成后，重启计算机以生效，在IE11以下版本的浏览器不需要在注册表中进行设置

2.      Edge浏览器驱动下载和配置

因为我主要是在Windows 7系统下进行脚本的编写和环境的配置，而Selenium 3同样是支持最新的操作系统Windows 10中的浏览器Edge，这里就简单的说下大概怎么配置。

下载地址：https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/



下载完，可放入最初章节提到的Drivers文件夹，方便管理

曾经使用过Java Selenium中成功启动过Edge浏览器，在Python上应该是大同小异的

具体核心代码:

EdgeDriver =os.path.join(os.path.abspath('..'),'Drivers', ‘MicrosoftWebDriver.exe ')

os.environ['webdriver.edge.driver'] = EdgeDriver

driver = webdriver.Edge(EdgeDriver)

3.      智能等待

因为自动化脚本执行时间过快，往往会导致页面还未完全加载完，脚本已经运行结束，针对这个情况，可以在执行到关键语句前，可以参考上述篇章提到的第13方法：智能等待方法。以便添加一个时间值进行等待。以保证需要被测到的功能能够被执行到，从而保证结果内容的准确性。

4.      其他遇到的问题会持续更新

实例代码：https://github.com/John0731/MyPython/tree/master/src/MyItems/Selenium/BasicAPI