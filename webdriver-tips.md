# 6 Tips for Using WebDriver with Java #

## 1. How To Highlight an Element

While debugging, sometimes it is very helpful if the element we are going to interact 
with can be highlighted. The following code will create a border around the element 
we are going to interact with:

```java
public void highlightElement (WebElement element) {
    ((Javascript Executor) driver).executeScript(
            "arguments[0].set Attribute(arguments[1], arguments[2])", 
            element, 
            "style", 
            "border: 2px solid yellow; color: yellow; font-weight: bold;");
}
```

## 2. How To Verify Text Presence ##

WebDriver is far better in many ways than Selenium RC, however if you have migrated 
away from Selenium RC, you will miss a few functions, and one of the functions is 
`isTextPresent`. It is not complicated to implement, even though it is not available 
in WebDriver:

```java
public boolean isTextPresent(String text) {
    if (driver.getPageSource().contains(text))
        return true;
    else
        return false;
}
```

## 3. Never Hard Code Waiting for an Element ##

I have seen many testers use hard coded sleep (Thread.sleep()) to wait till some element 
appears on the page. It isn’t a bad solution, as it works most of the time if you have 
given a high enough wait time; however it affects the performance of the test. 
For example, the below code will wait for 3 minutes, even if the element appears 
within 1 minute:

```java
Thread.sleep(1000*3*60);
Driver.findElement(By.Id(“username”));
```

However, if we use the explicit wait provided by WebDriver, we can avoid this 
performance bottleneck. The following code will wait a maximum of 3 minutes.
However, if within this time the element appears, the following code will 
move ahead:

```java
WebElement myDynamicElement = (new WebDriverWait(driver, 60*3))
              .until( ExpectedConditions.presenceOfElementLocated( By.id("username") ) );
```

## 4. If Click is not working – use JavaScript ##

There may be times when the click API provided by WebDriver is not working, 
or it is not returning the control . You can use JavaScript to click on the 
element at this time:

```java
public void JavascriptClick(WebElement elemen) {
    ((JavascriptExecutor)driver).executeScript( "arguments[0].click();", element) );
}
```

## 5. How To Bypass File Download Dialog Box in Firefox ##

One thing which always creates trouble is WebDrivers inability to handle dialog boxes. 
File Download Dialog box is one of them. The following trick will help you to handle 
the file download dialog box for Firefox:

```java
FirefoxProfile fp = new FirefoxProfile();
fp.setPreference("browser.download.dir","d:");
fp.setPreference("browser.download.folderList",2);
fp.setPreference("browser.helperApps.neverAsk.saveToDisk", 
                 "application/zip,application/octet-stream");
fp.setPreference("browser.download.manager.showWhenStarting",false);
```

## 6. How To Change the User Agent for Firefox ##

Sometimes you need to check how your website is going to look on android devices. 
For this, rather than running your test cases on an actual android device, you can 
change Firefox on Windows to give you the same look and feel as for an andriod. 
You can achieve this by changing the Firefox user agent. 
The following is the way you can do this:

```java
FirefoxProfile fp = new FirefoxProfile();
fp.setPreference("general.useragent.override", 
                 "Mozilla/5.0 (Android; Mobile; rv:13.0) Gecko/13.0 Firefox/13.0");
WebDriver driver = new FirefoxDriver(fp);
```

