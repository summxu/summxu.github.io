---
layout:     post
title:      "Python网页自动化操作"
subtitle:   "Selenium"
date:       2018-05-30
author:     "BoomXu"
tags:
    - Python
---

> 最近几天忙忙碌碌却感到忙出来什么好结果，不过最近几天要抓紧时间学学英语。因为最近有一个表需要一直重复向网页添加信息，于是就借此机会研究了Python的网页自动化，在此是用了Selenium框架。。

# 什么是Selenium

selenium 是一套完整的web应用程序测试系统，包含了测试的录制（selenium IDE）,编写及运行（Selenium Remote Control）和测试的并行处理（Selenium Grid）。Selenium的核心Selenium Core基于JsUnit，完全由JavaScript编写，因此可以用于任何支持JavaScript的浏览器上。

selenium可以模拟真实浏览器，自动化测试工具，支持多种浏览器，爬虫中主要用来解决JavaScript渲染问题。

# selenium基本使用

在此之前先上段代码：

```
browser = webdriver.Chrome()
browser.get('http://sdxy.gov.cn:8888/auth/pub/loginerror')
wait = WebDriverWait(browser, 10, 1.0)
browser.set_page_load_timeout(8)
browser.set_script_timeout(8)
``` 

## 声明浏览器对象

首先selenium支持绝大多数主流浏览器，但是中间要有浏览器的驱动程序 在此我是调用的 **[chromedriver](http://npm.taobao.org/mirrors/chromedriver/)** 放到相同目录下就可以运行。
`browser = webdriver.Firefox()`
这是火狐的调用方式。

## 访问页面

```
browser.get("http://www.baidu.com")
print(browser.page_source)
browser.close() 
```

用**rowser.get**方法可以打开网页**page_source**是提取网页源代码

## 查找元素
### 单个元素查找
```
from selenium import webdriver

browser = webdriver.Chrome()

browser.get("http://www.taobao.com")
input_first = browser.find_element_by_id("q")
input_second = browser.find_element_by_css_selector("#q")
input_third = browser.find_element_by_xpath('//*[@id="q"]')
print(input_first)
print(input_second)
print(input_third)
browser.close()
```

这里列举一下常用的查找元素方法：

find_element_by_name
find_element_by_id
find_element_by_xpath
find_element_by_link_text
find_element_by_partial_link_text
find_element_by_tag_name
find_element_by_class_name
find_element_by_css_selector

下面这种方式是比较通用的一种方式：这里需要记住By模块所以需要导入
from selenium.webdriver.common.by import By

### 多个元素查找
```
from selenium import webdriver


browser = webdriver.Chrome()
browser.get("http://www.taobao.com")
lis = browser.find_elements_by_css_selector('.service-bd li')
print(lis)
browser.close()
```

这样获得就是一个列表:

![from zhaof](https://images2015.cnblogs.com/blog/997599/201706/997599-20170606193737497-369795287.png)

## 元素交互操作

```
from selenium import webdriver

import time

browser = webdriver.Chrome()
browser.get("http://www.taobao.com")
input_str = browser.find_element_by_id('q')
input_str.send_keys("ipad")
time.sleep(1)
input_str.clear()
input_str.send_keys("MakBook pro")
button = browser.find_element_by_class_name('btn-search')
button.click()
```
运行的结果可以看出程序会自动打开Chrome浏览器并打开淘宝输入ipad,然后删除，重新输入MakBook pro，并点击搜索

Selenium所有的api文档：<http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.common.action_chains>

### 交互动作
将动作附加到动作链中串行执行

from selenium import webdriver
from selenium.webdriver import ActionChains

browser = webdriver.Chrome()

url = "http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable"
browser.get(url)
browser.switch_to.frame('iframeResult')
source = browser.find_element_by_css_selector('#draggable')
target = browser.find_element_by_css_selector('#droppable')
actions = ActionChains(browser)
actions.drag_and_drop(source, target)
actions.perform()

## 执行JavaScript
这是一个非常有用的方法，这里就可以直接调用js方法来实现一些操作，
下面的例子是通过登录知乎然后通过js翻到页面底部，并弹框提示

```
from selenium import webdriver
browser = webdriver.Chrome()
browser.get("http://www.zhihu.com/explore")
browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
browser.execute_script('alert("To Bottom")')
```

## 元素操作

 元素操作 | 方法
---------|----------
 获取元素属性 | get_attribute('class')
 获取文本值 | text
 获取ID | id 
 位置 | location 
 标签名 | tag_name 

## 等待
当使用了隐式等待执行测试的时候，如果 WebDriver没有在 DOM中找到元素，将继续等待，超出设定时间后则抛出找不到元素的异常, 换句话说，当查找元素或元素并没有立即出现的时候，隐式等待将等待一段时间再查找 DOM，默认的时间是0
### 显式等待

指定一个等待条件，并且指定一个最长等待时间，会在这个时间内进行判断是否满足等待条件，如果成立就会立即返回，如果不成立，就会一直等待，直到等待你指定的最长等待时间，如果还是不满足，就会抛出异常，如果满足了就会正常返回

```
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

browser = webdriver.Chrome()
browser.get('https://www.taobao.com/')
wait = WebDriverWait(browser, 10)
input = wait.until(EC.presence_of_element_located((By.ID, 'q')))
button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '.btn-search')))
print(input, button)
```

### 隐式等待
到了一定的时间发现元素还没有加载，则继续等待我们指定的时间，如果超过了我们指定的时间还没有加载就会抛出异常，如果没有需要等待的时候就已经加载完毕就会立即执行
```
from selenium import webdriver

browser = webdriver.Chrome()
browser.implicitly_wait(10)
browser.get('https://www.zhihu.com/explore')
input = browser.find_element_by_class_name('zu-top-add-question')
print(input)
```
## 条件判断

1. title_is 标题是某内容
2. title_contains 标题包含某内容
3. presence_of_element_located 元素加载出，传入定位元组，如(By.ID, 'p')
4. visibility_of_element_located 元素可见，传入定位元组
5. visibility_of 可见，传入元素对象
6. presence_of_all_elements_located 所有元素加载出
7. text_to_be_present_in_element 某个元素文本包含某文字
8. text_to_be_present_in_element_value 某个元素值包含某文字
9. frame_to_be_available_and_switch_to_it frame加载并切换
10. invisibility_of_element_located 元素不可见
11. element_to_be_clickable 元素可点击
12. staleness_of 判断一个元素是否仍在DOM，可判断页面是否已经刷新
13. element_to_be_selected 元素可选择，传元素对象
14. element_located_to_be_selected 元素可选择，传入定位元组
15. element_selection_state_to_be 传入元素对象以及状态，相等返回True，否则返回False
16. element_located_selection_state_to_be 传入定位元组以及状态，相等返回True，否则返回False
17. alert_is_present 是否出现Alert

## cookie操作

get_cookies()
delete_all_cookes()
add_cookie()
```
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
print(browser.get_cookies())
browser.add_cookie({'name': 'name', 'domain': 'www.zhihu.com', 'value': 'zhaofan'})
print(browser.get_cookies())
browser.delete_all_cookies()
print(browser.get_cookies())
```

## 选项卡管理
通过执行js命令实现新开选项卡window.open()
不同的选项卡是存在列表里browser.window_handles
通过browser.window_handles[0]就可以操作第一个选项卡
```
import time
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.baidu.com')
browser.execute_script('window.open()')
print(browser.window_handles)
browser.switch_to_window(browser.window_handles[1])
browser.get('https://www.taobao.com')
time.sleep(1)
browser.switch_to_window(browser.window_handles[0])
browser.get('https://python.org')
```

还有很多强大的功能有待研究