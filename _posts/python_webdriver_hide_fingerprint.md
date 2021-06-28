---
title: Python selenium 调用 隐藏指纹特征
date: 2021-06-28 16:40:00
categories: Python
toc: true
tags:
- Python
- Selenium
- WebDriver
---
<Excerpt in index | 首页摘要> 
> Python selenium 调用 隐藏指纹特征
>
<!-- more -->
<The rest of contents | 余下全文>  

### 为什么要隐藏?

很多网址做了大量的反扒行为 为了方便绕过这个检测 需要做指纹特征的隐藏

[检测当前浏览器指纹特征 https://bot.sannysoft.com/](https://bot.sannysoft.com/)

### 如何隐藏

核心就是隐藏浏览器navigator.webdriver 这个对象

然后加上User-Agent等尽可能像是普通用户

再加上一个小技巧, 如果目标网站提供Mobile版本 通常爬Mobile站会容易许多..

上代码

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

chrome_options = Options()
chrome_options.add_argument("incognito")
chrome_options.add_argument("disable-extensions")
chrome_options.add_argument("disable-infobars")
chrome_options.add_argument(
    "user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) "
    "Chrome/91.0.4472.124 Safari/537.36")
chrome_options.add_argument('--disable-blink-features=AutomationControlled')
chrome_options.add_experimental_option("excludeSwitches", ["enable-automation"])
chrome_options.add_experimental_option('useAutomationExtension', False)

browser = webdriver.Chrome(executable_path="C:\\Users\\gloomy\\Downloads\\chromedriver_win32\\chromedriver.exe",
                           options=chrome_options)
browser.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
    "source": """
        Object.defineProperty(navigator, 'webdriver', {
            get: () => undefined
        })
    """
})
browser.set_window_size(400, 785)
titleEl = browser.find_element_by_class_name("title-content")
print(titleEl.text)
print(titleEl.text)
browser.close()
```