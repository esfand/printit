# Web Driver Design Patterns#

## Design Patterns and Development Strategies ##

Over time, projects tend to accumulate large numbers of tests. As the total number of tests increases, it becomes harder to make changes to the codebase --- a single "simple" change may cause numerous tests to fail, even though the application still works properly. Sometimes these problems are unavoidable, but when they do occur you want to be up and running again as quickly as possible. The following design patterns and strategies have been used before with WebDriver to help making tests easier to write and maintain. They may help you too.

* **DomainDrivenDesign:** Express your tests in the language of the end-user of the app.
* **PageObjects:** A simple abstraction of the UI of your web app.
* **LoadableComponent:** Modeling PageObjects as components.
* **BotStyleTests:** Using a command-based approach to automating tests, rather than the object-based 
approach that PageObjects encourage
* **AcceptanceTests:** Use coarse-grained UI tests to help structure development work.
* **RegressionTests:** Collect the actions of multiple AcceptanceTests into one place for ease of maintenance.

If your favourite pattern has been left out, please leave a comment on this page, and we'll try and include it. Also, please remember that these are just suggestions: in order to use WebDriver you don't need to follow these patterns and strategies, and it's likely that not all of them will be useful to you. Just use the ones that you like!


## Page Objects ##

Within your web app's UI there are areas that your tests interact with. A Page Object simply models these as objects within the test code. This reduces the amount of duplicated code and means that if the UI changes, the fix need only be applied in one place.

###Implementation Notes###

PageObjects can be thought of as facing in two directions simultaneously. Facing towards the developer of a test, they represent the services offered by a particular page. Facing away from the developer, they should be the only thing that has a deep knowledge of the structure of the HTML of a page (or part of a page) It's simplest to think of the methods on a Page Object as offering the "services" that a page offers rather than exposing the details and mechanics of the page. As an example, think of the inbox of any web-based email system. Amongst the services that it offers are typically the ability to compose a new email, to choose to read a single email, and to list the subject lines of the emails in the inbox. How these are implemented shouldn't matter to the test.

Because we're encouraging the developer of a test to try and think about the services that they're interacting with rather than the implementation, PageObjects should seldom expose the underlying WebDriver instance. To facilitate this, methods on the PageObject should return other PageObjects. This means that we can effectively model the user's journey through our application. It also means that should the way that pages relate to one another change (like when the login page asks the user to change their password the first time they log into a service, when it previously didn't do that) simply changing the appropriate method's signature will cause the tests to fail to compile. Put another way, we can tell which tests would fail without needing to run them when we change the relationship between pages and reflect this in the PageObjects.

One consequence of this approach is that it may be necessary to model (for example) both a successful and unsuccessful login, or a click could have a different result depending on the state of the app. When this happens, it is common to have multiple methods on the PageObject:

```java
public class LoginPage {
    public HomePage loginAs(String username, String password) {
        // ... clever magic happens here
    }
    
    public LoginPage loginAsExpectingError(String username, String password) {
        //  ... failed login here, maybe because one or both of the username and password are wrong
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
could be re-written as:

public void testMessagesAreReadOrUnread() {
    Inbox inbox = new Inbox(driver);
    assertTrue(inbox.isMessageWithSubjectIsUnread("I like cheese"));
    assertFalse(inbox.isMessageWithSubjectIsUnread("I'm not fond of tofu"));
}
```

Of course, as with every guideline there are exceptions, and one that is commonly seen with PageObjects is to check that the WebDriver is on the correct page when we instantiate the PageObject. This is done in the example below.

Finally, a PageObject need not represent an entire page. It may represent a section that appears many times within a site or page, such as site navigation. The essential principle is that there is only one place in your test suite with knowledge of the structure of the HTML of a particular (part of a) page.

### Summary ###

* The public methods represent the services that the page offers
* Try not to expose the internals of the page
* Generally don't make assertions
* Methods return other PageObjects
* Need not represent an entire page
* Different results for the same action are modelled as different methods

### Example ###

```java
public class LoginPage {
    private final WebDriver driver;

    public LoginPage(WebDriver driver) {
        this.driver = driver;

        // Check that we're on the right page.
        if (!"Login".equals(driver.getTitle())) {
            // Alternatively, we could navigate to the login page, 
            // perhaps logging out first
            throw new IllegalStateException("This is not the login page");
        }
    }

    // The login page contains several HTML elements that will be 
    // represented as WebElements.
    // The locators for these elements should only be defined once.
        By usernameLocator = By.id("username");
        By passwordLocator = By.id("passwd");
        By loginButtonLocator = By.id("login");

    // The login page allows the user to type their username into 
    // the username field
    public LoginPage typeUsername(String username) {
        // This is the only place that "knows" how to enter a username
        driver.findElement(usernameLocator).sendKeys(username);

        // Return the current page object as this action doesn't navigate 
        // to a page represented by another PageObject
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
        // This is the only place that submits the login form and expects 
        // the destination to be the home page.
        // A seperate method should be created for the instance of clicking 
        // login whilst expecting a login failure. 
        driver.findElement(loginButtonLocator).submit();

        // Return a new page object representing the destination. Should the 
        // login page ever go somewhere else (for example, a legal disclaimer) 
        // then changing the method signature for this method will mean that 
        // all tests that rely on this behaviour won't compile.
        return new HomePage(driver);    
    }

    // The login page allows the user to submit the login form knowing that 
    // an invalid username and / or password were entered
    public LoginPage submitLoginExpectingFailure() {
        // This is the only place that submits the login form and expects 
        // the destination to be the login page due to login failure.
        driver.findElement(loginButtonLocator).submit();

        // Return a new page object representing the destination. 
        // Should the user ever be navigated to the home page after submiting 
        // a login with credentials expected to fail login, the script will 
        // fail when it attempts to instantiate the LoginPage PageObject.
        return new LoginPage(driver);   
    }

    // Conceptually, the login page offers the user the service of being able
    // to "log into" the application using a user name and password. 
    public HomePage loginAs(String username, String password) {
        // The PageObject methods that enter username, password & submit 
        // login have already defined and should not be repeated here.
        typeUsername(username);
        typePassword(password);
        return submitLogin();
    }
}
```


## The PageFactory ##

In order to support the PageObject pattern, WebDriver's support library contains a factory class.

###A Simple Example###

In order to use the PageFactory, first declare some fields on a PageObject that are `WebElements` or 
`List<WebElement>`, for example:

```java
package org.openqa.selenium.example;

import org.openqa.selenium.WebElement;

public class GoogleSearchPage {
    // Here's the element
    private WebElement q;

    public void searchFor(String text) {
        // And here we use it. Note that it looks like we've
        // not properly instantiated it yet....
        q.sendKeys(text);
        q.submit();
    }
} 
```

In order for this code to work and not throw a NullPointerException because the "q" field isn't instantiated, we need to initialise the PageObject:

```java
package org.openqa.selenium.example;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.htmlunit.HtmlUnitDriver;
import org.openqa.selenium.support.PageFactory;

public class UsingGoogleSearchPage {
    public static void main(String[] args) {
        // Create a new instance of a driver
        WebDriver driver = new HtmlUnitDriver();

        // Navigate to the right place
        driver.get("http://www.google.com/");

        // Create a new instance of the search page class
        // and initialise any WebElement fields in it.
        GoogleSearchPage page = PageFactory.initElements(driver, GoogleSearchPage.class);

        // And now do the search.
        page.searchFor("Cheese");
    }
}
```

### Explanation ###

The PageFactory relies on using sensible defaults: the name of the field in the Java class is assumed to be the "id" or "name" of the element on the HTML page. That is, in the example above, the line:

```java
    q.sendKeys(text);
 ```
 
is equivalent to:

```java
    driver.findElement(By.id("q")).sendKeys(text);
```    
The driver instance that's used is the one that's passed to the PageFactory's initElements method.

In the example given, we rely on the PageFactory to instantiate the instance of the PageObject. It does this by first looking for a constructor that takes "WebDriver" as its sole argument (public SomePage(WebDriver driver) {). If this is not present, then the default constructor is called. Sometimes, however, the PageObject depends on more than just an instance of the WebDriver interface. Should this be the case, it is possible to get the PageFactory to initialise the elements of an already constructed object:

```java
ComplexPageObject page = new ComplexPageObject("expected title", driver);

// Note, we still need to pass in an instance of driver for the 
// initialised elements to use
PageFactory.initElements(driver, page);
```

### Making the Example Work Using Annotations ###

When we run the example, the PageFactory will search for an element on the page that matches the field name of the WebElement in the class. It does this by first looking for an element with a matching ID attribute. If this fails, the PageFactory falls back to searching for an element by the value of its "name" attribute.

Although the code works, someone who's not familiar with the source of the Google 
home page may not know that the name of the field is "q". Fortunately, we can pick 
a meaningful name and change the strategy used to look the element up using an 
annotation:

```java
package org.openqa.selenium.example;

import org.openqa.selenium.By;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.How;
import org.openqa.selenium.WebElement;

public class GoogleSearchPage {
    // The element is now looked up using the name attribute
    @FindBy(how = How.NAME, using = "q")
    private WebElement searchBox;

    public void searchFor(String text) {
        // We continue using the element just as before
        searchBox.sendKeys(text);
        searchBox.submit();
    }
} 
```

One wrinkle that remains is that every time we call a method on the WebElement, the driver will go and find it on the current page again. In an AJAX-heavy application this is what you would like to happen, but in the case of the Google search page we know that the element is always going to be there and won't change. We also know that we won't be navigating away from the page and returning (which would mean that a different element with the same name would be present) It would be handy if we could "cache" the element once we'd looked it up:

```java
package org.openqa.selenium.example;

import org.openqa.selenium.By;
import org.openqa.selenium.support.CacheLookup;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.How;
import org.openqa.selenium.WebElement;

public class GoogleSearchPage {
    // The element is now looked up using the name attribute,
    // and we never look it up once it has been used the first time
    @FindBy(how = How.NAME, using = "q")
    @CacheLookup
    private WebElement searchBox;

    public void searchFor(String text) {
        // We continue using the element just as before
        searchBox.sendKeys(text);
        searchBox.submit();
    }
}
```

### Reducing Verbosity ###

The example above is still a little verbose. A slightly cleaner way of annotating the field would be:

```js
public class GoogleSearchPage {
  @FindBy(name = "q")
  private WebElement searchBox;

  // The rest of the class is unchanged.
}
```

### Notes ###

* If you use the PageFactory, you can assume that the fields are initialised. If you don't use the PageFactory, then NullPointerExceptions will be thrown if you make the assumption that the fields are already initialised.
* List<WebElement> fields are decorated if and only if they have @FindBy or @FindBys annotation. Default search strategy "by id or name" that works for WebElement fields is hardly suitable for lists because it is rare to have several elements with the same id or name on a page.
* WebElements are evaluated lazily. That is, if you never use a WebElement field in a PageObject, there will never be a call to "findElement" for it.
* The functionality works using dynamic proxies. This means that you shouldn't expect a WebElement 
  to be a particular subclass, even if you know the type of the driver.  For example, if you are 
  using the HtmlUnitDriver, you shouldn't expect the WebElement field to be initialised with an 
  instance of HtmlUnitWebElement.


## Bot Style Tests ##

Although PageObjects are a useful way of reducing duplication in your tests, it's not always a pattern that teams feel comfortable following. An alternative approach is to follow a more "command-like" style of testing.

### Example ###

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

Once these abstractions have been built and duplication in your tests identified, 
it's possible to layer PageObjects on top of bots.

