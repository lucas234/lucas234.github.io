---
title: Appium自动化Windows APP
date: 2019-08-15 15:46:25
tags: 
	- appium
	- App
---

###### 前提条件
- Windows 10或者更新版本
- 需要[WinAppDriver](https://github.com/Microsoft/WinAppDriver)

###### 环境搭建
- 打开Windows PC的开发者模式
- 下载[Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk)并默认安装
- 下载[Windows driver](https://github.com/microsoft/WinAppDriver/releases/tag/v1.1.1)并默认安装
- 运行`WinAppDriver.exe`（记得要用admin权限运行）， 默认路径 (`C:\Program Files (x86)\Windows Application Driver`)

可以自定义地址或端口：
  
```
  WinAppDriver.exe 4727
  WinAppDriver.exe 127.0.0.1 4725
  WinAppDriver.exe 127.0.0.1 4723/wd/hub
```
	
如下图：
![20190809134649856.png](https://i.loli.net/2019/08/19/XSpwrlh7QABNyG6.png)
	
	
###### Windows 自动化脚本
<font color="red">运行脚本前要打开 `WinAppDriver.exe`</font>
对于Windows App来说，只需要传一个`app` capabilities 即可。
对于UWP的App，`app`对应的值为Application Id（App ID）。关于如何获取APP ID，可以使用powershell命令`get-StartApps`来获取，打开powershell终端运行：`get-StartApps | select-string "计算器"`即可获取值（<font color="red">运行命令之前先打开计算器</font>）。以下是java样例代码：

```java
 DesiredCapabilities cap = new DesiredCapabilities();
 cap.setCapability("app", "LexisNexisAPAC.LexisRed_wsek3cqrhvvz2!App");
 driver = new WindowsDriver(new URL("http://127.0.0.1:4723"), cap);
 driver.findElementByAccessibilityId("CalculatorResults");
```
对于经典的Windows App，`app`对应的值为可执行的`.exe`文件路径。以下是java样例代码：

```java
// Launch Notepad
DesiredCapabilities cap= new DesiredCapabilities();
cap.SetCapability("app", "C:\\Windows\\System32\\notepad.exe");
cap.SetCapability("appArguments", "MyTestFile.txt");
cap.SetCapability("appWorkingDir", "C:\\MyTestFolder");
driver= new WindowsDriver(new URL("http://127.0.0.1:4723"), cap);
// Use the session to control the app
driver.FindElementByClassName("Edit").SendKeys("This is some text");
```

###### Windows定位元素

使用Windows SDK提供的工具`inspect.exe`（`C:\Program Files (x86)\Windows Kits\10\bin\x86`或者`C:\Program Files (x86)\Windows Kits\10\bin\10.0.18362.0\x64`根据系统查看）来定位，详情查看[inspect](https://docs.microsoft.com/en-us/windows/win32/winauto/inspect-objects)，或者使用[AccExplorer32](https://github.com/lucas234/appium-sample/tree/master/tools/Windows-UI-Inspector)、[UISpy](https://github.com/lucas234/appium-sample/tree/master/tools/Windows-UI-Inspector)定位。
支持的定位方式：

| API | 定位方法 |对应inspect.exe的属性|例子|
|:--|:--|:--|:--|
| FindElementByAccessibilityId | accessibility id | AutomationId	  | AppNameTitle |
|FindElementByClassName  |class name	  | ClassName  |TextBlock   |
| FindElementById |id  |  RuntimeId (decimal) | 42.333896.3.1 |
| FindElementByName |name  | Name  | Calculator  |
| FindElementByTagName |tag name	  |  LocalizedControlType (upper camel case) |  Text |
| FindElementByXPath |xpath  | Any  |  //Button[0] |

###### 计算器的例子
Python（[GitHub](https://github.com/lucas234/appium-sample/blob/master/samples/python/windows_calculator_test.py)）：

```python
import unittest
from appium import webdriver


class WindowsCalculatorTest(unittest.TestCase):

    @classmethod
    def setUpClass(self):
        # set up appium
        desired_caps = {}
        desired_caps["app"] = "Microsoft.WindowsCalculator_8wekyb3d8bbwe!App"
        self.driver = webdriver.Remote(command_executor='http://127.0.0.1:4723', desired_capabilities=desired_caps)

    @classmethod
    def tearDownClass(self):
        self.driver.quit()

    def getresults(self):
        displaytext = self.driver.find_element_by_accessibility_id("CalculatorResults").text
        displaytext = displaytext.strip("显示为 ")
        displaytext = displaytext.rstrip(' ')
        displaytext = displaytext.lstrip(' ')
        return displaytext

    def test_addition(self):
        self.driver.find_element_by_name("一").click()
        self.driver.find_element_by_name("加").click()
        self.driver.find_element_by_name("七").click()
        self.driver.find_element_by_name("等于").click()
        self.assertEqual(self.getresults(), "8")

    def test_combination(self):
        self.driver.find_element_by_name("七").click()
        self.driver.find_element_by_name("乘以").click()
        self.driver.find_element_by_name("九").click()
        self.driver.find_element_by_name("加").click()
        self.driver.find_element_by_name("一").click()
        self.driver.find_element_by_name("等于").click()
        self.driver.find_element_by_name("除以").click()
        self.driver.find_element_by_name("八").click()
        self.driver.find_element_by_name("等于").click()
        self.assertEqual(self.getresults(), "8")

    def test_division(self):
        self.driver.find_element_by_name("八").click()
        self.driver.find_element_by_name("八").click()
        self.driver.find_element_by_name("除以").click()
        self.driver.find_element_by_name("一").click()
        self.driver.find_element_by_name("一").click()
        self.driver.find_element_by_name("等于").click()
        self.assertEqual(self.getresults(), "8")

    def test_multiplication(self):
        self.driver.find_element_by_name("九").click()
        self.driver.find_element_by_name("乘以").click()
        self.driver.find_element_by_name("九").click()
        self.driver.find_element_by_name("等于").click()
        self.assertEqual(self.getresults(), "81")

    def test_subtraction(self):
        self.driver.find_element_by_name("九").click()
        self.driver.find_element_by_name("减").click()
        self.driver.find_element_by_name("一").click()
        self.driver.find_element_by_name("等于").click()
        self.assertEqual(self.getresults(), "8")


if __name__ == '__main__':
    unittest.main()
```
java（[GitHub](https://github.com/lucas234/appium-sample/blob/master/samples/java/src/test/java/WindowsCalculatorTest.java)）：

```java
import org.openqa.selenium.WebElement;
import org.openqa.selenium.remote.DesiredCapabilities;
import java.util.concurrent.TimeUnit;
import java.net.URL;
import io.appium.java_client.windows.WindowsDriver;

import org.testng.Assert;
import org.testng.annotations.*;

public class WindowsCalculatorTest {

    private static WindowsDriver CalculatorSession = null;
    private static WebElement CalculatorResult = null;

    @BeforeClass
    public static void setup() {
        try {
            DesiredCapabilities capabilities = new DesiredCapabilities();
            capabilities.setCapability("app", "Microsoft.WindowsCalculator_8wekyb3d8bbwe!App");
            CalculatorSession = new WindowsDriver(new URL("http://127.0.0.1:4723"), capabilities);
            CalculatorSession.manage().timeouts().implicitlyWait(2, TimeUnit.SECONDS);
            CalculatorResult = CalculatorSession.findElementByAccessibilityId("CalculatorResults");

        }catch(Exception e){
            e.printStackTrace();
        } finally {}
    }

    @AfterClass
    public static void TearDown()
    {
        CalculatorResult = null;
        if (CalculatorSession != null) {
            CalculatorSession.quit();
        }
        CalculatorSession = null;
    }

    @Test
    public void Addition()
    {
        CalculatorSession.findElementByName("七").click();
        CalculatorSession.findElementByName("七").click();
        CalculatorSession.findElementByName("加").click();
        CalculatorSession.findElementByName("八").click();
        CalculatorSession.findElementByName("等于").click();
        Assert.assertEquals("显示为 85", CalculatorResult.getText());
    }

    @Test
    public void Combination()
    {
        CalculatorSession.findElementByName("七").click();
        CalculatorSession.findElementByName("乘以").click();
        CalculatorSession.findElementByName("九").click();
        CalculatorSession.findElementByName("加").click();
        CalculatorSession.findElementByName("一").click();
        CalculatorSession.findElementByName("等于").click();
        CalculatorSession.findElementByName("除以").click();
        CalculatorSession.findElementByName("八").click();
        CalculatorSession.findElementByName("等于").click();
        Assert.assertEquals("显示为 8", CalculatorResult.getText());
    }

    @Test
    public void Division()
    {
        CalculatorSession.findElementByName("六").click();
        CalculatorSession.findElementByName("四").click();
        CalculatorSession.findElementByName("除以").click();
        CalculatorSession.findElementByName("八").click();
        CalculatorSession.findElementByName("等于").click();
        Assert.assertEquals("显示为 8", CalculatorResult.getText());
    }

    @Test
    public void Multiplication()
    {
        CalculatorSession.findElementByName("六").click();
        CalculatorSession.findElementByName("乘以").click();
        CalculatorSession.findElementByName("八").click();
        CalculatorSession.findElementByName("等于").click();
        Assert.assertEquals("显示为 48", CalculatorResult.getText());
    }

    @Test
    public void Subtraction()
    {
        CalculatorSession.findElementByName("九").click();
        CalculatorSession.findElementByName("减").click();
        CalculatorSession.findElementByName("一").click();
        CalculatorSession.findElementByName("等于").click();
        Assert.assertEquals("显示为 8", CalculatorResult.getText());
    }
}
```
###### 参考
- http://appium.io/docs/en/drivers/windows/
- https://github.com/Microsoft/WinAppDriver?WT.mc_id=-blog-scottha

###### 问题总结

> org.openqa.selenium.WebDriverException: Failed to locate opened application window with appId

遇到此问题是在启动APP时报的错，成功的启动了APP，但是抛出这个异常，导致后续无法测试，目前找了一个临时解决方案：在`new driver`时增加`try catch`机制，即可避免，例如：

```java
try {
   driver = new WindowsDriver(new URL("http://127.0.0.1:4723"), cap);
}catch (Exception e){
//            e.printStackTrace();
   driver = new WindowsDriver(new URL("http://127.0.0.1:4723"), cap);
}
```

