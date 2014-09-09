# PAGE OBJECT MODEL IN SELENIUM #

Page Object is a Design Pattern which has become popular in test automation 
for enhancing test maintenance and reducing code duplication. A page object 
is an object-oriented class that serves as an interface to a page of your 
Application under test. The tests then use the methods of this page object 
class whenever they need to interact with that page of the UI. 
The benefit is that if the UI changes for the page, the tests themselves don’t need to change, only the code within the page object needs to change.

## Advantage of using Page Object design pattern: ##

If the page UI changes, the tests themselves don’t need to change, only the codes within the page object require change.
There is clean separation between test code and page specific code such as locators and layout.
There is single repository for the services or operations offered by the page rather than having these services scattered throughout the tests.
Let’s take an example to explain this.

First we will consider an example without using Page Object model. 
We will consider a login example containing login screen and home
page which is opened after successful login.

```java
/** Test Login to application **/
public class Login {
    ... 
    ...
    ...
    public void testLogin() {
        selenium.type("loginid", "test@test.com");
        selenium.type("password", "dummy123");
        selenium.click("submit");
        selenium.waitForPageToLoad("PageWaitPeriod");
        Assert.assertTrue(selenium.isElementPresent("Logout"), 
                                                    "Login was unsuccessful");
    }
    ...
    ...
    ...
}
```

You can see the above example; there is no separation between the test and 
the application locators like ids.

Applying Page Object model the above code can be re-written by defining 
Page Object for Sign-in page and home page and a separate class for TestCases

```java
/* Page Object for Login page. */
public class LoginPage {

    private Selenium selenium;
    public LoginPage(Selenium selenium) {
        this.selenium = selenium;
        if(!selenium.getTitle().equals("Login in page")) {
            throw new IllegalStateException(
                          "This is not sign in page, current page is: "
                          + selenium.getLocation());
        }
    }
    
    /**
     * Login as valid user
     *
     * @param userName
     * @param password
     * @return HomePage object
     */
    public HomePage loginValidUser(String userName, String password) {
        selenium.type("usernamefield", userName);
        selenium.type("passwordfield", password);
        selenium.click("sign-in");
        selenium.waitForPageToLoad("waitPeriod");
        return new HomePage(selenium)}
    }
 
    /* Page Object of the Home Page */
    public class HomePage {

    private Selenium selenium;

    public HomePage(Selenium selenium) {
        if (!selenium.getTitle().equals("Home Page of logged in user")) {
            throw new IllegalStateException(
                "This is not Home Page of logged in user, current page" +
                "is: " +selenium.getLocation());
        }
    }

    public HomePage manageProfile() {
        // Page encapsulation to manage profile functionality
        return new HomePage(selenium);
    }
    ...
    /*More methods w.r.t, the services offered by the page can be defined.*/
}
```

LoginTest will use the 2 objects defined above.

```java
/*** * Login tests */
…
…
…
public class LoginTest {
    public void testLogin() {
        LoginPage loginPage = new LoginPage (selenium);
        HomePage homePage = loginPage .loginValidUser("userName", "password");
        Assert.assertTrue(selenium.isElementPresent("compose button"),
                                                    "Login was unsuccessful");
    }
…
…
…
}
```

Your tests may use more than 1 page objects. The above example uses 2 page objects.

There is lot of flexibility in how the page object can be defined but there are few basic rule that should be followed.

Page Object should never themselves make verification or assertions. It is the part of Test and should be part of test code
No code related to what is tested should be present in page object. Page object should contain the representation of the page, and the services offered by page via methods.
There should be one verification within the page object and that is to verify the page and the elements on the page were loaded correctly.
A page object may not represent entire page. Also, you can use Page Factory for instantiating their Page objects.

