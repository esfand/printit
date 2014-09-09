# The 5 Minute Getting Started Guide #

## First Step

WebDriver is a tool for automating testing web applications, and in particular to 
verify that they work as expected. It aims to provide a friendly API that's easy 
to explore and understand, which will help make your tests easier to read and maintain. 
It's not tied to any particular test framework, so it can be used equally well 
with JUnit, TestNG or from a plain old "main" method. This "Getting Started" guide 
introduces you to WebDriver's Java API and helps get you started becoming familiar with it.

Start by downloading the latest binaries and unpack them into a directory. 
A safe choice is the latest version of selenium-server-standalone-x.y.z.jar x, y and z 
will be digits e.g. 2.4.0 at the time of writing. From now on, we'll refer to this 
directory as $WEBDRIVER_HOME. Now, open your favourite IDE and:

* Start a new Java project in your favourite IDE
* Add all the JAR files under $WEBDRIVER_HOME to the CLASSPATH 

You can see that WebDriver acts just as a normal Java library does: it's entirely 
self-contained, and you don't need to remember to start any additional processes or 
run any installers before using it.

You're now ready to write some code. An easy way to get started is this example, 
which searches for the term "Cheese" on Google and then outputs the result page's 
title to the console. You'll start by using the HtmlUnitDriver. 
This is a pure Java driver that runs entirely in-memory. Because of this, 
you won't see a new browser window open.

```java
package org.openqa.selenium.example;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.htmlunit.HtmlUnitDriver;

public class Example  {
    public static void main(String[] args) {
        // Create a new instance of the html unit driver
        // Notice that the remainder of the code relies on the interface, 
        // not the implementation.
        WebDriver driver = new HtmlUnitDriver();

        // And now use this to visit Google
        driver.get("http://www.google.com");

        // Find the text input element by its name
        WebElement element = driver.findElement(By.name("q"));

        // Enter something to search for
        element.sendKeys("Cheese!");

        // Now submit the form. WebDriver will find the form for us from the element
        element.submit();

        // Check the title of the page
        System.out.println("Page title is: " + driver.getTitle());

        driver.quit();
    }
}
```

Compile and run it. You should see a line with the title of the Google search 
results as output. Congratulations, you've managed to get started with WebDriver!

In this next example, let's use a page that requires Javascript to work properly, 
such as Google Suggest. You will also be using the FirefoxDriver. 
Make sure that Firefox is installed on your machine and is in the normal location for your OS.

Once that's done, create a new class called GoogleSuggest, which looks like:

```java
package org.openqa.selenium.example;

import java.util.List;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.firefox.FirefoxDriver;

public class GoogleSuggest {
    public static void main(String[] args) throws Exception {
        // The Firefox driver supports javascript 
        WebDriver driver = new FirefoxDriver();
        
        // Go to the Google Suggest home page
        driver.get("http://www.google.com/webhp?complete=1&hl=en");
        
        // Enter the query string "Cheese"
        WebElement query = driver.findElement(By.name("q"));
        query.sendKeys("Cheese");

        // Sleep until the div we want is visible or 5 seconds is over
        long end = System.currentTimeMillis() + 5000;
        while (System.currentTimeMillis() < end) {
            WebElement resultsDiv = driver.findElement(By.className("gssb_e"));

            // If results have been returned, the results are displayed in a drop down.
            if (resultsDiv.isDisplayed()) {
              break;
            }
        }

        // And now list the suggestions
        List<WebElement> allSuggestions = driver.findElements(
                                                 By.xpath("//td[@class='gssb_a gbqfsf']"));
        
        for (WebElement suggestion : allSuggestions) {
            System.out.println(suggestion.getText());
        }

        driver.quit();
    }
}
```

When you run this program, you'll see the list of suggestions being printed to the console. 
That's all there is to using WebDriver!

Hopefully, this will have whet your appetite for more. In the NextSteps page, you will learn 
how more about how to use WebDriver. It covers aspects such as navigating forward and backward 
in your browser's history, how to use frames and windows and it provides a more complete 
discussion of the examples than has been done as you've been GettingStarted. If you're ready, 
let's take the NextSteps! 



## Next Steps For Using WebDriver##

Once you've completed the GettingStarted tutorial, you may want to know more. 
This is the right place to look!

## Which Implementation of WebDriver Should I Use? ##

WebDriver is the name of the key interface against which tests should be written, 
but there are several implementations. These are:

You can find out more information about each of these by following the links in 
the table. Which you use depends on what you want to do. For sheer speed, 
the HtmlUnitDriver is great, but it's not graphical, which means that you can't 
watch what's happening. As a developer, you may be comfortable with this, 
but sometimes it's good to be able to test using a real browser, especially 
when you're showing a demo of your application (or running the tests) for an audience. 
Often, this idea is referred to as "safety", and it falls into two parts. 
Firstly, there's "actual safety", which refers to whether or not the tests works 
as they should. This can be measured and quantified. Secondly, there's "perceived safety", 
which refers to whether or not an observer believes the tests work as they should. 
This varies from person to person, and will depend on their familiarity with the application under test, WebDriver and your testing framework.

To support higher "perceived safety", you may wish to choose a driver such as the FirefoxDriver. This has the added advantage that this driver actually renders content to a screen, and so can be used to detect information such as the position of an element on a page, or the CSS properties that apply to it. However, this additional flexibility comes at the cost of slower overall speed. By writing your tests against the WebDriver interface, it is possible to pick the most appropriate driver for a given test.

To keep things simple, let's start with the HtmlUnitDriver:

WebDriver driver = new HtmlUnitDriver();

Navigating

The first thing you'll want to do with WebDriver is navigate to a page. The normal way to do this is by calling "get":

```java
driver.get("http://www.google.com");
```

WebDriver will wait until the page has fully loaded (that is, the "onload" event has fired) before returning control to your test or script.
Interacting With the Page

Just being able to go to places isn't terribly useful. What we'd really like to do is to interact with the pages, or, more specifically, the HTML elements within a page. First of all, we need to find one. WebDriver offers a number of ways of finding elements. For example, given an element defined as:

```html
<input type="text" name="passwd" id="passwd-id" />
```

we could use any of:

```java
WebElement element;
element = driver.findElement(By.id("passwd-id"));
element = driver.findElement(By.name("passwd"));
element = driver.findElement(By.xpath("//input[@id='passwd-id']"));
```

You can also look for a link by its text, but be careful! The text must be an exact match! You should also be careful when using XpathInWebDriver. If there's more than one element that matches the query, then only the first will be returned. If nothing can be found, a NoSuchElementException will be thrown.

WebDriver has an "Object-based" API; we represent all types of elements using the same interface: WebElement. This means that although you may see a lot of possible methods you could invoke when you hit your IDE's auto-complete key combination, not all of them will make sense or be valid. Don't worry! WebDriver will attempt to do the Right Thing, so if you call a method that makes no sense ("setSelected()" on a "meta" tag, for example) an exception will be thrown.

So, we've got an element. What can we do with it? First of all, you may want to enter some text into a text field:

```java
element.sendKeys("some text");
```

You can simulate pressing the arrow keys by using the "Keys" class:

```java
element.sendKeys(" and some", Keys.ARROW_DOWN);
```

It is possible to call sendKeys on any element, which makes it possible to test keyboard shortcuts such as those used on GMail. A side-effect of this is that typing something into a text field won't automatically clear it. Instead, what you type will be appended to what's already there. You can easily clear the contents of a text field or textarea:

```
element.clear();
```

## Filling In Forms ##

We've already seen how to enter text into a textarea or text field, but what about the other elements? You can "toggle" the state of checkboxes, and you can use "setSelected" to set something like an OPTION tag selected. Dealing with SELECT tags isn't too bad:

```java
WebElement select = driver.findElement(By.xpath("//select"));
List<WebElement> allOptions = select.findElements(By.tagName("option"));
for (WebElement option : allOptions) {
    System.out.println(String.format("Value is: %s", option.getAttribute("value")));
    option.click();
}
```

This will find the first "SELECT" element on the page, and cycle through each of it's OPTIONs in turn, printing out their values, and selecting each in turn. As you can see, this isn't the most efficient way of dealing with SELECT elements. WebDriver's support classes come with one called "Select", which provides useful methods for interacting with these.

Once you've finished filling out the form, you probably want to submit it. One way to do this would be to find the "submit" button and click it:

```java
driver.findElement(By.id("submit")).click();  // Assume the button has the ID "submit" :)
```

Alternatively, WebDriver has the convenience method "submit" on every element. If you call this on an element within a form, WebDriver will walk up the DOM until it finds the enclosing form and then calls submit on that. If the element isn't in a form, then the "NoSuchElementException" will be thrown:

```java
element.submit();
```

### Drag And Drop ###

You can use drag and drop, either moving an element by a certain amount, or on to another element:

```java
WebElement element = driver.findElement(By.name("source"));
WebElement target = driver.findElement(By.name("target"));

(new Actions(driver)).dragAndDrop(element, target).perform();
```

### Moving Between Windows and Frames ###

It's rare for a modern web application not to have any frames or to be constrained to a single window. WebDriver supports moving between named windows using the "switchTo" method:

```java
driver.switchTo().window("windowName");
```

All calls to driver will now be interpreted as being directed to the particular window. But how do you know the window's name? Take a look at the javascript or link that opened it:

```html
<a href="somewhere.html" target="windowName">Click here to open a new window</a>
```

Alternatively, you can pass a "window handle" to the "switchTo().window()" method. Knowing this, it's possible to iterate over every open window like so:

```java
for (String handle : driver.getWindowHandles()) {
  driver.switchTo().window(handle);
}
```

You can also swing from frame to frame (or into iframes):

```java
driver.switchTo().frame("frameName");
```

It's possible to access subframes by chaining switchTo() calls, and you can specify the frame by its index too. That is:

```java
driver.switchTo().frame("frameName")
      .switchTo().frame(0)
      .switchTo().frame("child");
```

would go to the frame named "child" of the first subframe of the frame called "frameName". All frames are evaluated as if from currently switched to frame. To get back to the top level, call:

```java
driver.switchTo().defaultContent();
```

### Navigation: History and Location ###

Earlier, we covered navigating to a page using the "get" command 
`(driver.get("http://www.example.com"))` As you've seen, 
WebDriver has a number of smaller, task-focused interfaces, and navigation is a useful task. 
Because loading a page is such a fundamental requirement, the method to do this lives on 
the main WebDriver interface, but it's simply a synonym to:

```java
driver.navigate().to("http://www.example.com");
```

To reiterate: `navigate().to()` and `get()` do exactly the same thing. 
One's just a lot easier to type than the other!

The "navigate" interface also exposes the ability to move backwards and forwards in your browser's history:

```java
driver.navigate().forward();
driver.navigate().back();
```

Please be aware that this functionality depends entirely on the underlying browser. 
It's just possible that something unexpected may happen when you call these methods 
if you're used to the behaviour of one browser over another.

### Cookies ###

Before we leave these next steps, you may be interested in understanding how to use cookies. 
First of all, you need to be on the domain that the cookie will be valid for:

```java
// Go to the correct domain
driver.get("http://www.example.com");

// Now set the cookie. This one's valid for the entire domain
Cookie cookie = new Cookie("key", "value");
driver.manage().addCookie(cookie);

// And now output all the available cookies for the current URL
Set<Cookie> allCookies = driver.manage().getCookies();
for (Cookie loadedCookie : allCookies) {
    System.out.println(String.format("%s -> %s", 
                                     loadedCookie.getName(), 
                                     loadedCookie.getValue()));
}
```

### Next, Next Steps! ###

This has been a high level walkthrough of WebDriver and some of its key capabilities. 
You may want to look at the DesignPatterns to get some ideas about how you can reduce the 
pain of maintaining your tests and how to make your code more modular. 


## Design Patterns and Development Strategies ##

Over time, projects tend to accumulate large numbers of tests. 
As the total number of tests increases, it becomes harder to make changes 
to the codebase --- a single "simple" change may cause numerous tests to fail, 
even though the application still works properly. Sometimes these problems 
are unavoidable, but when they do occur you want to be up and running again 
as quickly as possible. The following design patterns and strategies have been 
used before with WebDriver to help making tests easier to write and maintain. 
They may help you too.

* DomainDrivenDesign: Express your tests in the language of the end-user of the app.
* PageObjects: A simple abstraction of the UI of your web app.
* LoadableComponent: Modeling PageObjects as components.
* BotStyleTests: Using a command-based approach to automating tests, rather than the object-based approach that PageObjects encourage
* AcceptanceTests: Use coarse-grained UI tests to help structure development work.
* RegressionTests: Collect the actions of multiple AcceptanceTests into one place for ease of maintenance. 

If your favourite pattern has been left out, please leave a comment on this page, 
and we'll try and include it. Also, please remember that these are just suggestions: 
in order to use WebDriver you don't need to follow these patterns and strategies, 
and it's likely that not all of them will be useful to you. Just use the ones that you like! 


### Domain Driven Design ###

The best advise you can be given is to read Eric Evan's excellent "Domain Driven Design" book. However, in a nutshell there are some key points.

* Tests should be written using a "Ubiquitous Language"
* Make use of the "Repository" pattern to make it easier to create objects 

#### Ubiquitous Language ####

This refers to the idea that everyone on a project should attempt to speak the same language. That is, rather than referring to "load data into this table" or "click on the third column" it should be possible to use language such as "create a new account" or "order displayed results by name"



### Page Objects ###

Within your web app's UI there are areas that your tests interact with. A Page Object simply models these as objects within the test code. This reduces the amount of duplicated code and means that if the UI changes, the fix need only be applied in one place.

#### Implementation Notes ####

PageObjects can be thought of as facing in two directions simultaneously. Facing towards the developer of a test, they represent the services offered by a particular page. Facing away from the developer, they should be the only thing that has a deep knowledge of the structure of the HTML of a page (or part of a page) It's simplest to think of the methods on a Page Object as offering the "services" that a page offers rather than exposing the details and mechanics of the page. As an example, think of the inbox of any web-based email system. Amongst the services that it offers are typically the ability to compose a new email, to choose to read a single email, and to list the subject lines of the emails in the inbox. How these are implemented shouldn't matter to the test.

Because we're encouraging the developer of a test to try and think about the services that they're interacting with rather than the implementation, PageObjects should seldom expose the underlying WebDriver instance. To facilitate this, methods on the PageObject should return other PageObjects. This means that we can effectively model the user's journey through our application. It also means that should the way that pages relate to one another change (like when the login page asks the user to change their password the first time they log into a service, when it previously didn't do that) simply changing the appropriate method's signature will cause the tests to fail to compile. Put another way, we can tell which tests would fail without needing to run them when we change the relationship between pages and reflect this in the PageObjects.

One consequence of this approach is that it may be necessary to model (for example) both a successful and unsuccessful login, or a click could have a different result depending on the state of the app. When this happens, it is common to have multiple methods on the PageObject:

```java
public class LoginPage {
    public HomePage loginAs(String username, String password) {
        // ... clever magic happens here
    }
    
    public LoginPage loginAsExpectingError(String username, String password) {
        //  ... failed login here, maybe because one or both of the username 
        //      and password are wrong
    }
    
    public String getErrorMessage() {
        // So we can verify that the correct error is shown
    }
}
```

The code presented above shows an important point: the tests, not the PageObjects, should be responsible for making assertions about the state of a page. For example:

```java
public void testMessagesAreReadOrUnread() {
    Inbox inbox = new Inbox(driver);
    inbox.assertMessageWithSubjectIsUnread("I like cheese");
    inbox.assertMessageWithSubjectIsNotUnread("I'm not fond of tofu");
}
```

could be re-written as:

```
public void testMessagesAreReadOrUnread() {
    Inbox inbox = new Inbox(driver);
    assertTrue(inbox.isMessageWithSubjectIsUnread("I like cheese"));
    assertFalse(inbox.isMessageWithSubjectIsUnread("I'm not fond of tofu"));
}
```

Of course, as with every guideline there are exceptions, and one that is commonly seen with PageObjects is to check that the WebDriver is on the correct page when we instantiate the PageObject. This is done in the example below.

Finally, a PageObject need not represent an entire page. It may represent a section that appears many times within a site or page, such as site navigation. The essential principle is that there is only one place in your test suite with knowledge of the structure of the HTML of a particular (part of a) page.

#### Summary ####

* The public methods represent the services that the page offers
* Try not to expose the internals of the page
* Generally don't make assertions
* Methods return other PageObjects
* Need not represent an entire page
* Different results for the same action are modelled as different methods 

#### Example ####

```java
public class LoginPage {
    private final WebDriver driver;

    public LoginPage(WebDriver driver) {
        this.driver = driver;

        // Check that we're on the right page.
        if (!"Login".equals(driver.getTitle())) {
            // Alternatively, we could navigate to the login page, 
            //perhaps logging out first
            throw new IllegalStateException("This is not the login page");
        }
    }

    // The login page contains several HTML elements that will 
    // be represented as WebElements.
    // The locators for these elements should only be defined once.
        By usernameLocator = By.id("username");
        By passwordLocator = By.id("passwd");
        By loginButtonLocator = By.id("login");

    // The login page allows the user to type their username 
    // into the username field
    public LoginPage typeUsername(String username) {
        // This is the only place that "knows" how to enter a username
        driver.findElement(usernameLocator).sendKeys(username);

        // Return the current page object as this action doesn't 
        //navigate to a page represented by another PageObject
        return this;    
    }

    // The login page allows the user to type their password 
    // into the password field
    public LoginPage typePassword(String password) {
        // This is the only place that "knows" how to enter a password
        driver.findElement(passwordLocator).sendKeys(password);

        // Return the current page object as this action doesn't navigate 
        // to a page represented by another PageObject
        return this;    
    }

    // The login page allows the user to submit the login form
    public HomePage submitLogin() {
        // This is the only place that submits the login form and 
        // expects the destination to be the home page.
        // A seperate method should be created for the instance of 
        // clicking login whilst expecting a login failure. 
        driver.findElement(loginButtonLocator).submit();

        // Return a new page object representing the destination. 
        // Should the login page ever
        // go somewhere else (for example, a legal disclaimer) then 
        // changing the method signature for this method will mean that all 
        // tests that rely on this behaviour won't compile.
        return new HomePage(driver);    
    }

    // The login page allows the user to submit the login form knowing that 
    // an invalid username and / or password were entered
    public LoginPage submitLoginExpectingFailure() {
        // This is the only place that submits the login form and expects the 
        // destination to be the login page due to login failure.
        driver.findElement(loginButtonLocator).submit();

        // Return a new page object representing the destination. Should the user 
        // ever be navigated to the home page after submiting a login with 
        // credentials expected to fail login, the script will fail when it 
        // attempts to instantiate the LoginPage PageObject.
        return new LoginPage(driver);   
    }

    // Conceptually, the login page offers the user the service of being able to 
    // "log into" the application using a user name and password. 
    public HomePage loginAs(String username, String password) {
        // The PageObject methods that enter username, password & submit login 
        // have already defined and should not be repeated here.
        typeUsername(username);
        typePassword(password);
        return submitLogin();
    }
}
```

#### Support in WebDriver ####

There is a PageFactory in the support package that provides support for this pattern, and helps to remove some boiler-plate code from your Page Objects at the same time. 


### Using the LoadableComponent ###

#### What Is It? ###

The LoadableComponent is a base class that aims to make writing PageObjects less painful. It does this by providing a standard way of ensuring that pages are loaded and providing hooks to make debugging the failure of a page to load easier. You can use it to help reduce the amount of boilerplate code in your tests, which in turn make maintaining your tests less tiresome.

There is currently an implementation in Java that ships as part of Selenium 2, but the approach used is simple enough to be implemented in any language.

#### Simple Usage ####

As an example of a UI that we'd like to model, take a look at the new issue page. From the point of view of a test author, this offers the service of being able to file a new issue. A basic Page Object would look like:

```java
package com.example.webdriver;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;

public class EditIssue {

  private final WebDriver driver;

  public EditIssue(WebDriver driver) {
    this.driver = driver;
  }

  public void setSummary(String summary) {
    WebElement field = driver.findElement(By.name("summary"));
    clearAndType(field, summary);
  }

  public void enterDescription(String description) {
    WebElement field = driver.findElement(By.name("comment"));
    clearAndType(field, description);
  }

  public IssueList submit() {
    driver.findElement(By.id("submit")).click();
    return new IssueList(driver);
  }

  private void clearAndType(WebElement field, String text) {
    field.clear();
    field.sendKeys(text);
  }
}
```

In order to turn this into a LoadableComponent, all we need to do is to set that as the base type:

```java
public class EditIssue extends LoadableComponent<EditIssue> {
  // rest of class ignored for now
}
```

This signature looks a little unusual, but it all it means is that this class represents 
a LoadableComponent that loads the EditIssue page.

By extending this base class, we need to implement two new methods:

```java
  @Override
  protected void load() {
    driver.get("http://code.google.com/p/selenium/issues/entry");
  }

  @Override
  protected void isLoaded() throws Error {
    String url = driver.getCurrentUrl();
    assertTrue("Not on the issue entry page: " + url, url.endsWith("/entry"));
  }
```

The load method is used to navigate to the page, whilst the isLoaded method is used to determine whether we are on the right page. Although the method looks like it should return a boolean, instead it performs a series of assertions using JUnit's Assert class. There can be as few or as many assertions as you like. By using these assertions it's possible to give users of the class clear information that can be used to debug tests.

With a little rework, our PageObject looks like:

```java
package com.example.webdriver;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

import static junit.framework.Assert.assertTrue;

public class EditIssue extends LoadableComponent<EditIssue> {

  private final WebDriver driver;
  
  // By default the PageFactory will locate elements with the same name or id
  // as the field. Since the summary element has a name attribute of "summary"
  // we don't need any additional annotations.
  private WebElement summary;
  
  // Same with the submit element, which has the ID "submit"
  private WebElement submit;
  
  // But we'd prefer a different name in our code than "comment", so we use the
  // FindBy annotation to tell the PageFactory how to locate the element.
  @FindBy(name = "comment") private WebElement description;
  
  public EditIssue(WebDriver driver) {
    this.driver = driver;
    
    // This call sets the WebElement fields.
    PageFactory.initElements(driver, this);
  }

  @Override
  protected void load() {
    driver.get("http://code.google.com/p/selenium/issues/entry");
  }

  @Override
  protected void isLoaded() throws Error {
    String url = driver.getCurrentUrl();
    assertTrue("Not on the issue entry page: " + url, 
               url.endsWith("/entry"));
  }
  
  public void setSummary(String issueSummary) {
    clearAndType(summary, issueSummary);
  }

  public void enterDescription(String issueDescription) {
    clearAndType(description, issueDescription);
  }

  public IssueList submit() {
    submit.click();
    return new IssueList(driver);
  }

  private void clearAndType(WebElement field, String text) {
    field.clear();
    field.sendKeys(text);
  }
}
```

That doesn't seem to have bought us much, right? One thing it has done is encapsulate the information about how to navigate to the page into the page itself, meaning that this information's not scattered through the code base. It also means that we can do this in our tests:

```java
EditIssue page = new EditIssue(driver).get();
```

This call will cause the driver to navigate to the page if that's necessary.
Advanced Usage: Nested Components

LoadableComponents start to become more useful when they are used in conjunction with other LoadableComponents. Using our example, we could view the "edit issue" page as a component within a project's website (after all, we access it via a tab on that site). You also need to be logged in to file an issue. We could model this as a tree of nested components:

```text
 + ProjectPage
 +---+ SecuredPage
     +---+ EditIssue
```

What would this look like in code? For a start, each logical component would have its own class. The "load" method in each of them would "get" the parent. The end result, in addition to the EditIssue class above is:

```java
ProjectPage.java:

package com.example.webdriver;

import org.openqa.selenium.WebDriver;

import static org.junit.Assert.assertTrue;

public class ProjectPage extends LoadableComponent<ProjectPage> {

  private final WebDriver driver;
  private final String projectName;

  public ProjectPage(WebDriver driver, String projectName) {
    this.driver = driver;
    this.projectName = projectName;
  }

  @Override
  protected void load() {
    driver.get("http://" + projectName + ".googlecode.com/");
  }

  @Override
  protected void isLoaded() throws Error {
    String url = driver.getCurrentUrl();

    assertTrue(url.contains(projectName));
  }
}
```

and SecuredPage.java:

```java
package com.example.webdriver;

import org.openqa.selenium.By;
import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;

import static org.junit.Assert.fail;

public class SecuredPage extends LoadableComponent<SecuredPage> {

  private final WebDriver driver;
  private final LoadableComponent<?> parent;
  private final String username;
  private final String password;

  public SecuredPage(WebDriver driver, 
                     LoadableComponent<?> parent, 
                     String username, 
                     String password) {
    this.driver = driver;
    this.parent = parent;
    this.username = username;
    this.password = password;
  }

  @Override
  protected void load() {
    parent.get();

    String originalUrl = driver.getCurrentUrl();

    // Sign in
    driver.get("https://www.google.com/accounts/ServiceLogin?service=code");
    driver.findElement(By.name("Email")).sendKeys(username);
    WebElement passwordField = driver.findElement(By.name("Passwd"));
    passwordField.sendKeys(password);
    passwordField.submit();

    // Now return to the original URL
    driver.get(originalUrl);
  }

  @Override
  protected void isLoaded() throws Error {
    // If you're signed in, you have the option of picking a different login.
    // Let's check for the presence of that.

    try {
      WebElement div = driver.findElement(By.id("multilogin-dropdown"));
    } catch (NoSuchElementException e) {
      fail("Cannot locate user name link");
    }
  }
}
```

The "load" method in EditIssue now looks like:

```java
  @Override
  protected void load() {
    securedPage.get();

    driver.get("http://code.google.com/p/selenium/issues/entry");
  }
```

This shows that the components are all "nested" within each other. 
A call to get() in EditIssue will cause all its dependencies to load too. 
The example usage:

```java
public class FooTest {
  private EditIssue editIssue;

  @Before
  public void prepareComponents() {
    WebDriver driver = new FirefoxDriver();

    ProjectPage project = new ProjectPage(driver, "selenium");
    SecuredPage securedPage = new SecuredPage(driver, project, "example", "top secret");
    editIssue = new EditIssue(driver, securedPage);
  }

  @Test
  public void demonstrateNestedLoadableComponents() {
    editIssue.get();

    editIssue.setSummary("Summary");
    editIssue.enterDescription("This is an example");
  }
}
```

If you're using a library such as Guiceberry in your tests, the preamble of setting up the PageObjects can be omitted leading to nice, clear, readable tests. 

### Bot Style Tests ###

Although PageObjects are a useful way of reducing duplication in your tests, it's not always a pattern that teams feel comfortable following. An alternative approach is to follow a more "command-like" style of testing.

#### Example ####

A "bot" is an action-oriented abstraction over the raw Selenium APIs. This means that if you find that commands aren't doing the Right Thing for your app, it's easy to change them. As an example:

```java
public class ActionBot {
  private final WebDriver driver;

  public ActionBot(WebDriver driver) {
    this.driver = driver;
  }

  public void click(By locator) {
    driver.findElement(locator).click();
  }

  public void submit(By locator) {
    driver.findElement(locator).submit();
  }

  /** 
   * Type something into an input field. WebDriver doesn't normally clear these
   * before typing, so this method does that first. It also sends a return key
   * to move the focus out of the element.
   */
  public void type(By locator, String text) { 
    WebElement element = driver.findElement(locator);
    element.clear();
    element.sendKeys(text + "\n");
  }
}
```

Once these abstractions have been built and duplication in your tests identified, it's possible to layer PageObjects on top of bots.
