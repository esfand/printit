# WebDriver - Explicit and Implicit Waits

Waiting is having the automated task execution elapse a certain amount of time before continuing with the next step.

## Explicit Waits
An explicit waits is code you define to wait for a certain condition to occur before proceeding further in the code. The worst case of this is Thread.sleep(), which sets the condition to an exact time period to wait. There are some convenience methods provided that help you write code that will wait only as long as required. WebDriverWait in combination with ExpectedCondition is one way this can be accomplished.

```java
WebDriver driver = new FirefoxDriver();
driver.get("http://somedomain/url_that_delays_loading");
WebElement myDynamicElement = (new WebDriverWait(driver, 10))
  .until(ExpectedConditions.presenceOfElementLocated(
             By.id("myDynamicElement")));
```

This waits up to 10 seconds before throwing a TimeoutException or if it finds the element will return it in 0 - 10 seconds. WebDriverWait by default calls the ExpectedCondition every 500 milliseconds until it returns successfully. A successful return is for ExpectedCondition type is Boolean return true or not null return value for all other ExpectedCondition types.

This example is also functionally equivalent to the first Implicit Waits example.

### Expected Conditions

There are some common conditions that are frequently come across when automating web browsers. Listed below are Implementations of each. Java happens to have convienence methods so you don’t have to code an ExpectedCondition class yourself or create your own utility package for them.

Element is Clickable - it is Displayed and Enabled.

```java
WebDriverWait wait = new WebDriverWait(driver, 10);
WebElement element = wait.until(ExpectedConditions.elementToBeClickable(
                                    By.id("someid")));
```

The ExpectedConditions package (Java) (Python) (.NET) contains a set of predefined conditions to use with WebDriverWait.

## Implicit Waits

An implicit wait is to tell WebDriver to poll the DOM for a certain amount of time when trying to find an element or elements if they are not immediately available. The default setting is 0. Once set, the implicit wait is set for the life of the WebDriver object instance.

```java
WebDriver driver = new FirefoxDriver();
driver.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
driver.get("http://somedomain/url_that_delays_loading");
WebElement myDynamicElement = driver.findElement(
                                  By.id("myDynamicElement"));
```


# WebDriver Implicit Wait

Implicit wait is a way of telling selenium driver that it should poll DOM for certain amount of time when performing findElement or findElements. Tests will continue immediately if the element is available earlier and if not then after timeout period NoSouchElementException will be thrown. Using it requires only one time initialization and after that all findElement and findElements invocations will be aware of additional time they should wait. Although it sounds like a convenient and reliable solution it is very limiting and you may sooner or later run into one of the problems mentioned below.


## 1. Performance
Idea of setting timeout that will be used globally on each method invocation is good as long as you can change it in some specific scenarios. You may need to do this for pages with AJAX loaded components because there will be a significant difference in load times between them and static content. This means you have to set timeout to load time of the slowest component or swap timeout value between default and specific. You may think that just setting it to some big value like 30 second will solve the problem because all component will load faster than that and program will continue anyway. However this way of solving problem can be very time consuming if you just need to verify presence of element and continue test whatever the results is. For example 'if there is a pop-up window close it and continue'. In that scenario driver will have to wait for 30 seconds just to prove that there is no window.

## 2. Exceptions
Information about what went wrong is probably the most important part of the test. Using Implicit Timeout we are left with NoSuchElementException. In most cases we would like to customize error message, create a screenshot or translate thrown exception to one of our own. Of course we can just surround invocation of findElement and findElements with try and catch block but writing explicit exception handling defeat the purpose of using implicit wait.

## 3. Reliability
When using implicit wait test continue immediately after element was located in DOM. In most cases this is what we want but if we are dealing with lot of JavaScript the answer is not that obvious. For example located element can be still temporarily invisible (if we click on it we will end up with ElementNotVisibleException) or moving (if we click on it we will end up with WebDriverException and message that the element is not clickable) or was detached from DOM and attached again (we will end up with StaleElementReferenceException).

## 4. Conditions
Implicit Wait applies only to finding element and therefore it is very limiting. There are lot of other actions that would benefit from trying to execute them for some period of time. Previously mentioned exception when trying to click element that is temporarily moving or invisible are good candidates. Other nice thing that is inconvenient to do using implicit wait is conditionally finding element. If pages contains progress bar or some other indicator of processing request using AJAX we can always wait until it disappears and then perform actual search for element.

As you can see implicit wait is simple and convenient but also very limiting way of locating elements. Below is a test class that visualize some of the problems I have written about:


```java
import org.openqa.selenium.By;
import org.openqa.selenium.ElementNotVisibleException;
import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;
import java.util.concurrent.TimeUnit;

public final class ImplicitWaitTest {
    private static final int IMPLICIT_WAIT_TIMEOUT = 5000;
    private static final int IMPLICIT_WAIT_CUSTOM_TIMEOUT = 3000;
    private WebDriver driver;

    @BeforeTest
    public void initialize() {
        // requires system property: 
        //     -Dwebdriver.chrome.driver=<path to chromedriver.exe>
        driver = new ChromeDriver(); 
        driver.manage()
              .timeouts()
              .implicitlyWait(IMPLICIT_WAIT_TIMEOUT, 
                              TimeUnit.MILLISECONDS);
        driver.get("http://docs.seleniumhq.org/");
    }

    // Changing timeout - inconvenient
    @Test(groups = {"performance"}, 
          timeOut = IMPLICIT_WAIT_TIMEOUT)
    public void timeoutSwapTest() {
        driver.manage()
              .timeouts()
              .implicitlyWait(IMPLICIT_WAIT_CUSTOM_TIMEOUT, 
                              TimeUnit.MILLISECONDS);
        driver.findElement(By.xpath("//div[@id='sidebar']"));
        driver.manage()
              .timeouts()
              .implicitlyWait(IMPLICIT_WAIT_TIMEOUT, 
                              TimeUnit.MILLISECONDS);
    }

    // Verify there there is no popup expecting 
    //     NoSuchElementException - inefficient
    @Test(groups = {"performance"}, 
          expectedExceptions = {NoSuchElementException.class})
    public void verifyTest() {
        driver.findElement(By.xpath("//div[@id='popup']"));
    }

    // Translating NoSuchElementException to AssertionError 
    //     with custom message - inconvenient
    @Test(groups = {"exceptions"}, 
          expectedExceptions = {AssertionError.class})
    public void exceptionHandlingTest() {
        try {
            driver.findElement(By.xpath("//div[@id='popup']"));
        } catch (NoSuchElementException ex) {
            throw new AssertionError("Popup is not present");
        }
    }

    // Chaining actions - no way of waiting for visibility 
    // of element, we end up with ElementNotVisibleException
    // This is just to present what happens if you perform an 
    // action on 'hidden' element
    @Test(groups = {"reliability"}, 
          expectedExceptions = {ElementNotVisibleException.class})
    public void reliabilityTest() {
        driver.findElement(By.xpath("//input[@name='cof']")).click();
    }

    @AfterTest
    public void shutdown() {
        if(driver != null) {
            try {
                driver.quit();
                driver.close();
            } catch (Throwable e) {
                // ignore
            }
        }
    }
}
```





# WebDriver Explicit Wait

Explicit wait is a programmatic approach to problem of waiting for elements. 
Contrary to Implicit Wait that I have described in previous article Explicit Wait 
requires more coding but also gives much more flexibility. Any action that can be 
performed using WebDriver can be wrapped using explicit wait and performed over 
defined period of time checking custom condition. As fast as the condition is 
satisfied test will continue and if condition was not satisfied for period longer 
then defined threshold then TimeoutException or any of the not ignored exceptions 
will be thrown. I will try to address problems that I have described in article 
about Implicit Wait and show how they can be solved using explicit approach.

## 1. Performance
What we would like is to have default timeout that will be used in most cases but 
also possibility to change it if necessary. Fortunately this is what FluentWait 
class offers. Moreover we can define how frequently we should check the condition. 
Knowing that we can solve problem of verifying that given element is not present 
on the page implementing our own condition with custom timeout or just use already 
provided conditions that are defined in utility class called ExcepctedConditions.

## 2. Exceptions
Second interesting feature that we get out of the box by using FluentWait is 
ability to ignore specific exceptions and provide custom message that will be 
included in exception. No longer we have to start analyzing our tests result from 
very detailed information about internals of WebDriver or XPath. This is especially 
convenient when you are dealing with large number of tests. Basically what we are 
doing is putting another layer of user friendly information that is easier to analyze.

## 3. Reliability
This is where explicit approach really shines and probably most important reason to 
use it, especially when testing pages with lot of JavaScript. For example if we want 
to click on element that is present in DOM but temporarily invisible we can create a 
condition that will check both presence and visibility. If the element is moving we 
can try to ignore WebDriverException knowing that we would end up with 'element is 
not clickable' (this is a bit more risky because WebDriverException can be thrown 
for other reasons). Finally if we are dealing with StaleElementReferenceException 
we can try to ignore it and perform whole sequence of actions inside condition body. 
This means we will periodically try to find element and perform action on it instead 
of returning if from method and using it later (when it is more likely to end up 
with stale reference).

## 4. Conditions
Explicit wait approach is not limited to only findElement and findElements methods 
of WebDriver. We can use it to chain actions that would normally require few steps 
and verifications along the way. For example we can define a custom condition that 
will find input field, type some text and click ok button. Moreover we can easily 
implement how to conditionally find element. For example wait for progress bar to 
disappear and then look for element. We can even ask a web developer to set some 
variable to true if page was loaded or long running task had completed. Then we would 
use JavascriptExecutor or WebDriver to find that variable and be sure that page is 
fully rendered and ready to use.

As you can see explicit approach requires more coding but also solves many annoying 
problems you may run into using implicit wait. Below is a test class that visualize 
some of the issues I have written about. Please just keep in mind that I am not 
wrapping invocations of fluent wait because I want to focus on how it can be used.

```java
import com.google.common.base.Function;
import org.openqa.selenium.*;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.FluentWait;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;
import java.util.concurrent.TimeUnit;
import static org.openqa.selenium.support.ui.ExpectedConditions.*;

public final class ExplicitWaitTest {
    private static final int EXPLICIT_WAIT_POOLING = 500;
    private static final int EXPLICIT_WAIT_TIMEOUT = 7;
    private static final int EXPLICIT_WAIT_CUSTOM_TIMEOUT = 3;
    private static final int EXPLICIT_WAIT_PRESENT_TIMEOUT = 1;
    private WebDriver driver;

    @BeforeTest
    public void initialize() {
        // requires system property: 
        //     -Dwebdriver.chrome.driver=<path to chromedriver.exe>
        driver = new ChromeDriver(); 
        driver.get("http://docs.seleniumhq.org/");
    }

    @Test(groups = {"performance"}, timeOut = EXPLICIT_WAIT_TIMEOUT * 1000)
    public void customTimeoutTest() {
        // withTimeout - setting custom timeout
        new FluentWait<WebDriver>(driver)
                .pollingEvery(EXPLICIT_WAIT_POOLING, TimeUnit.MILLISECONDS)
                .withTimeout(EXPLICIT_WAIT_CUSTOM_TIMEOUT, TimeUnit.SECONDS)
                .ignoring(NoSuchElementException.class)
                .until(new Function<WebDriver, WebElement>() {
                    @Override
                    public WebElement apply(WebDriver webDriver) {
                        return webDriver.findElement(
                               By.xpath("//div[@id='sidebar']"));
                    }
                });
        // presenceOfElementLocated - defined in ExpectedConditions
        new WebDriverWait(driver, EXPLICIT_WAIT_CUSTOM_TIMEOUT)
                .until(presenceOfElementLocated(
                           By.xpath("//div[@id='sidebar']")));
    }

    @Test(groups = {"performance"}, 
                    expectedExceptions = {TimeoutException.class})
    public void verifyTest() {
        // withTimeout - using shorter timeout for checking 
        //               for non existing elements
        new FluentWait<WebDriver>(driver)
                .pollingEvery(EXPLICIT_WAIT_POOLING, TimeUnit.MILLISECONDS)
                .withTimeout(EXPLICIT_WAIT_PRESENT_TIMEOUT, TimeUnit.SECONDS)
                .ignoring(NoSuchElementException.class)
                .until(new Function<WebDriver, WebElement>() {
                    @Override
                    public WebElement apply(WebDriver webDriver) {
                        return webDriver.findElement(
                                   By.xpath("//div[@id='popup']"));
                    }
                });

        // not - defined in ExpectedConditions
        new WebDriverWait(driver, EXPLICIT_WAIT_PRESENT_TIMEOUT)
                .until(not(presenceOfElementLocated(
                    By.xpath("//div[@id='popup']"))));
    }

    @Test(groups = {"exceptions"}, expectedExceptions = 
                        {TimeoutException.class})
    public void exceptionHandlingTest() {
        // withMessage - customMessage
        // ignoring - exception we don't care about
        new FluentWait<WebDriver>(driver)
                .pollingEvery(EXPLICIT_WAIT_POOLING, TimeUnit.MILLISECONDS)
                .withTimeout(EXPLICIT_WAIT_PRESENT_TIMEOUT, TimeUnit.SECONDS)
                .ignoring(NoSuchElementException.class)
                .withMessage("Popup is not visible")
                .until(new Function<WebDriver, WebElement>() {
                    @Override
                    public WebElement apply(WebDriver webDriver) {
                        return webDriver.findElement(
                                   By.xpath("//div[@id='popup']"));
                    }
                });
    }

    @Test(groups = {"reliability"})
    public void visibilityTest() {
        // ignoring ElementNotVisibleException that will be thrown 
        // if we click() on invisible element
        new FluentWait<WebDriver>(driver)
                .pollingEvery(EXPLICIT_WAIT_POOLING, TimeUnit.MILLISECONDS)
                .withTimeout(EXPLICIT_WAIT_TIMEOUT, TimeUnit.SECONDS)
                .ignoring(NoSuchElementException.class, 
                          ElementNotVisibleException.class)
                .until(new Function<WebDriver, WebElement>() {
                    @Override
                    public WebElement apply(WebDriver webDriver) {
                        WebElement element = webDriver.findElement(
                                             By.xpath("//div[@id='sidebar']"));
                        element.click();
                        return element;
                    }
                });
        // visibilityOfElementLocated - defined in ExpectedConditions
        new WebDriverWait(driver, EXPLICIT_WAIT_TIMEOUT)
                .until(visibilityOfElementLocated(
                           By.xpath("//div[@id='sidebar']")))
                .click();
    }

    @Test(groups = {"reliability"})
    public void movingTest() {
        // ignoring WebDriverException that will be thrown 
        // if we click() on moving element
        new FluentWait<WebDriver>(driver)
                .pollingEvery(EXPLICIT_WAIT_POOLING, TimeUnit.MILLISECONDS)
                .withTimeout(EXPLICIT_WAIT_TIMEOUT, TimeUnit.SECONDS)
                .ignoring(NoSuchElementException.class)
                .ignoring(ElementNotVisibleException.class)
                .ignoring(WebDriverException.class)
                .until(new Function<WebDriver, WebElement>() {
                    @Override
                    public WebElement apply(WebDriver webDriver) {
                        WebElement element = webDriver.findElement
                                             (By.xpath("//div[@id='sidebar']"));
                        element.click();
                        return element;
                    }
                });
    }

    @Test(groups = {"conditions"})
    public void conditionTest() {
        // finding if there is a progressbar and if not 
        // searching for another element
        new FluentWait<WebDriver>(driver)
                .pollingEvery(EXPLICIT_WAIT_POOLING, TimeUnit.MILLISECONDS)
                .withTimeout(EXPLICIT_WAIT_TIMEOUT, TimeUnit.SECONDS)
                .ignoring(NoSuchElementException.class)
                .until(new Function<WebDriver, WebElement>() {
                    @Override
                    public WebElement apply(WebDriver webDriver) {
                        if(webDriver.findElements(
                                By.xpath("//div[@id='progress']")).isEmpty()) {
                            return webDriver.findElement(
                                   By.xpath("//div[@id='sidebar']"));
                        }
                        return null;
                    }
                });
    }

    @AfterTest
    public void shutdown() {
        if (driver != null) {
            try {
                driver.quit();
                driver.close();
            } catch (Throwable e) {
                // ignore
            }
        }
    }
}
```
