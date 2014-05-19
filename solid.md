# Writing C# Code Using SOLID Principles #

www.codeguru.com/csharp/csharp/writing-c-code-using-solid-principles.htm

## Introduction ##

Most of the modern programming languages including C# support objected oriented programming. 
Features such as encapsulation, inheritance, overloading and polymorphism are code level 
features. Using these features is just one part of the story. Equally important is to apply 
some object oriented design principles while writing your C# code. SOLID principles is a set 
of five such principles--namely Single Responsibility Principle, Open/Closed Principle, 
Liskov Substitution Principle, Interface Segregation Principle and Dependency Inversion Principle. 
Applying these time proven principles make your code structured, neat and easy to maintain. 
This article discusses SOLID principles and also illustrates how they can be applied to your 
Java Code.

## Single Responsibility Principle (SRP) ##

The Single Responsibility Principle (SRP) states that:

> A class should have only a single responsibility. 
> Only one potential change in the software's specification should be able to affect the 
> specification of the class.

SRP indicates that a class should have one and only one responsibility. 
Consider an example of a hypothetical class named Customer that is supposed to contain 
information about a real life customer. To deal with this requirement you added properties 
and methods to the Customer class that select, insert, update and delete customer data. 
Additionally, you added a method that exports the customer data in CSV format. 
That means the Customer class has two responsibilities - represent and manipulate customers 
and export customer data in CSV format. Now imagine a situation where you wish to change 
the data export from CSV to tab delimited data. Due to this change you will need to change 
the code of the customer class. Changing the code of the Customer class also means that 
you need to redeploy the Customer class. The root cause is that Customer class has been 
assigned multiple responsibilities. A change in any of these responsibilities will change 
the Customer class. If Customer class would have been given just the responsibility of 
managing customers only, a change in the customer manipulation logic would have caused 
the Customer class to change. Since data export logic is no longer part of the Customer 
class any change in that logic would not have any impact on the Customer class.

Let's try to see SRP in action. Consider the following C# class.

```java
public class CustomerHelper {
    NorthwindEntities db = new NorthwindEntities();
 
    public List<Customer> GetAllCustomers() {
        return db.Customers.ToList();
    }
 
    public Customer GetCustomerByID(string customerid) {
        return db.Customers.Find(customerid);
    }
 
    public string ExportCustomerData() {
        List<Customer> data = GetAllCustomers();
        StringBuilder sb = new System.Text.StringBuilder();
        foreach(Customer item in data) {
            sb.Append(item.CustomerID);
            sb.Append(",");
            sb.Append(item.CompanyName);
            sb.Append(",");
            sb.Append(item.ContactName);
            sb.Append(",");
            sb.AppendLine(item.Country);
        }
        return sb.ToString();
    }
}
```

The CustomerHelper class has three methods; `GetAllCustomers()`, `GetCustomerByID()` and 
`ExportCustomerData()`. As explained earlier the first two are fine here but adding 
`ExportCustomerData()` method violates SRP. A solution is to isolate this method in its 
own class as shown in the following code:

```java
public class DataExporter {
    public string ExportCustomerData(List<Customer> data) {
        StringBuilder sb = new System.Text.StringBuilder();
        foreach (Customer item in data) {
            sb.Append(item.CustomerID);
            sb.Append(",");
            sb.Append(item.CompanyName);
            sb.Append(",");
            sb.Append(item.ContactName);
            sb.Append(",");
            sb.AppendLine(item.Country);
        }
        return sb.ToString();
    }
}
```

`ExportCustomerData()` method is now inside DataExporter class. If there is any change 
in the data export format or logic only, DataExporter class will undergo a change whereas 
`CustomerHelper` class will remain unchanged.

## Open/Closed Principle (OCP) ##

The Open / Closed Principle or OCP states that:

Software should be open for extension, but closed for modification.

Let's say you created a class that provides certain functionality. 
A few months later some need arises such that the class is supposed to provide some 
additional functionality. In such cases you shouldn't touch the source code of the 
class that is already built. Instead, you should be able to extend it so as to add 
the extra functionality.

Let's try to understand this with an example. Suppose that you are building a class 
that is responsible for calculating taxes on a given income. At the time of 
development you were told that this class will be used for tax calculation in three 
countries, say USA, UK and India. The following code shows a simplistic 
representation of such a class.

```java
public class TaxCalculator {
    public decimal CalculateTax(decimal amount,string country) {
        decimal taxAmount = 0;
        switch(country) {
            case "USA":
                //calculate tax as per USA rules
                break;
            case "UK":
                //calculate tax as per UK rules
                break;
            case "IN":
                //calculate tax as per India rules
                break;
        }
        return taxAmount;
    }
}
```

The TaxCalculator class has a method CalculateTax() that accepts the amount on which the tax is to be calculated and the country whose tax calculation rules are to be applied. Inside, a switch statement checks the country and accordingly the tax is calculated and returned to the caller.

Now, suppose that after a few months you also need to calculate tax in a few more countries. How will you take care of this additional functionality? You will change the CalculateTax() method and add additional cases inside the switch statement. That means any change in the functionality is forcing you to change the core class - TaxCalculator. This is violation of OCP.

Now, let's rewrite our code so that it follows OCP.

```java
public abstract class TaxCalculatorBase {
    public decimal TotalAmount { get; set; }
    public abstract decimal CalculateTax();
}
 
public class USATax:TaxCalculatorBase {
    public override decimal CalculateTax() {
        //calculate tax as per USA rules
        return 0;
    }
}
 
public class UKTax : TaxCalculatorBase {
    public override decimal CalculateTax()
    {
        //calculate tax as per UK rules
        return 0;
    }
}
 
public class IndiaTax : TaxCalculatorBase {
    public override decimal CalculateTax()
    {
        //calculate tax as per India rules
        return 0;
    }
}
```

The above code creates an abstract class TaxCalculatorBase. This class defines a public property - TotalAmount - and an abstract method CalculateTax(). The CalculateTax() method doesn't have any code since it is an abstract method. The actual tax calculation happens in the derived classes - USATax, UKTax and IndiaTax. These classes provide the concrete implementation of the CalculateTax() method. Tomorrow if tax calculation is needed for a few more countries you need not modify any of the existing code. All you need to do is create another class that inherits from TaxCalculatorBase class and implement the required tax calculation logic inside its CalculateTax() method.

## Liskov Substitution Principle (LSP) ##

Liskov Substitution Principle or LSP states that:

Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.

You must be aware that you can use an object of a derived class anywhere where base class is expected. However, when such a substitution occurs the functionality and correctness of the code shouldn't change. Let's understand this with an example. Suppose you have created a class named SpecialCustomers that maintains a list of special Customers (may be they are special because they are frequent buyers, or they purchase high value items). This class is shown below:

```java
public class SpecialCustomers {

    List<Customer> list = new List<Customer>();
 
    public /*virtual*/ void addCustomer(Customer obj) {
        list.Add(obj);
    }
 
    public int count() {
        return list.Count;
    }
}
```

The SpecialCustomers class maintains a List of Customer instances (Customer class is not shown in the code for the sake of simplicity). The AddCustomer() method accepts an instance of Customer and adds to the generic List. The Count property returns the number of Customer elements in the List.

Now, let's assume that you create another class - TopNCustomers - that inherits from the SpecialCustomers class. This class is shown below:

```java
public class TopNCustomers:SpecialCustomers
{
    private int maxCount = 5;
 
    public override void AddCustomer(Customer obj)
    {
        if (Count < maxCount)
        {
            AddCustomer(obj);
        }
        else
        {
            throw new Exception("Only " + maxCount + " customers can be added.");
        }
    }
}
```

The `TopNCustomers` class overrides the `addCustomer()` method of the SpecialCustomers base class. The new implementation checks whether the customer count is less than maxCount (5 in this case). If so, the Customer is added to the List else an exception is thrown.

Now, have a look at the following code that uses both of these classes.

```java
SpecialCustomers sc = null;
sc = new TopNCustomers();
for (int i = 0; i < 10; i++) {
    Customer obj = new Customer();
    sc.AddCustomer(obj);
}
```

The code declares a variable of type SpecialCustomers and then points it to an instance of TopNCustomers. This assignment is perfectly valid since TopNCustomers is derived from SpecialCustomers. The problem comes in the for loop. The for loop that follows attempts to add 10 Customer instances to the TopNCustomers. But TopNCustomers allows only 5 instances and hence throws an error. If sc would have been of type SpecialCustomers the for loop would have successfully added 10 instances into the List. However, since the code substitutes TopNCustomers instance in place of SpecialCustomers the code produces an exception. Thus LSP is violated in this example.

To rectify the problem we will rewrite the code like this:

```java
public abstract class CustomerCollection {
    public abstract void AddCustomer(Customer obj);
    public abstract int Count { get; }
}
 
public class SpecialCustomers:CustomerCollection {
    List<Customer> list = new List<Customer>();
 
    public override void AddCustomer(Customer obj)
    {
        list.Add(obj);
    }
 
    public override int Count
    {
        get
        {
            return list.Count;
        }
    }
}
 
public class TopNCustomers : CustomerCollection
{
    private int count=0;
    Customer[] list = new Customer[5];
 
    public override void AddCustomer(Customer obj)
    {
        if(count<5)
        {
            list[count] = obj;
            count++;
        }
        else
        {
            throw new Exception("Only " + count + " customers can be added.");
        }
    }
 
    public override int Count
    {
        get
        {
            return list.Length;
        }
    }
}
```

Now, the code has CustomerCollection abstract class with one property (Count) and one method (AddCustomer). The SpecialCustomers and TopNCustomers classes inherit from this abstract class and provide the concrete implementation for the Count and AddCustomer(). Notice that in this case TopNCustomers doesn't inherit from SpecialCustomers.

The new set of classes can then be used as follows:

```java
Customer c = new Customer() { CustomerID = "ALFKI" };
CustomerCollection collection = null;
collection = new SpecialCustomers();
collection.AddCustomer(c);
collection = new TopNCustomers();
collection.AddCustomer(c);
```

The above code declares a variable of CustomerCollection type. Once it points to an instance of SpecialCustomers and then to TopNCustomers. In this case, however, their base class is CustomerCollection and the instances are perfectly substitutable for it without producing any inaccuracies.

## Interface Segregation Principle (ISP) ##

Interface Segregation Principle or ISP states that:

No client should be forced to depend on methods it does not use. 
Many client-specific interfaces are better than one general-purpose interface.

What does that mean? Let's take an example. Suppose you are building an order processing system that accepts orders from the customers and places them in the system for dispatch and further processing. The orders can be accepted through various channels such as an ecommerce website, telephone and cash on delivery. To deal with this kind of processing you created the following abstract class:

```java
public abstract class OrderProcessor
{
    public abstract bool ValidatePaymentInfo();
    public abstract bool ValidateShippingAddress();
    public abstract void ProcessOrder();
}
```

The OrderProcessor class has three methods ValidatePaymentInfo(), ValidateShippingAddress() and ProcessOrder(). The ValidatePaymentInfo() method is supposed to validate the credit card information and return true if the information is correct. The ValidateShippingAddress() method is supposed to check whether the address is reachable by the available means of transport and if so return true. Finally, the ProcessOrder() will place an order into the system.

Now, assume that you created two concrete implementations of OrderProcessor as shown below:

```java
public class OnlineOrder extends OrderProcessor {
 
    public override bool ValidatePaymentInfo() {
        return true;
    }
 
    public override bool ValidateShippingAddress() {
        return true;
    }
 
    public override void ProcessOrder() {
        //place order here if everything is ok.
    }
}
 
public class CashOnDeliveryOrder:OrderProcessor {
 
    public override bool ValidatePaymentInfo() {
        throw new NotImplementedException();
    }
 
    public override bool ValidateShippingAddress() {
        throw new NotImplementedException();
    }
 
    public override void ProcessOrder() {
        //place order here if everything is ok.
    }
}
```

The OnlineOrder class is supposed to be used by some ecommerce application that accepts payments through credit cards. The CashOnDeliveryOrder class is supposed to serve cash on delivery orders. These orders won't accept any credit card payments. They accept only cash payments. Can you see the problem there? The two methods ValidatePaymentInfo() and ValidateShippingAddress() throw NotImplementedException because CashOnDeliverOrder doesn't need these methods. However, since the OrderProcessor class provides a general purpose interface the CashOnDeliveryOrder class is forced to write these methods. Thus ISP is being violated in this example.

Now let's fix the problem. The following code shows the corrected version of the classes:

```java
public abstract class OrderProcessor {
    public abstract void ProcessOrder();
}
 
public abstract class OnlineOrderProcessor {
    public abstract bool ValidatePaymentInfo();
    public abstract bool ValidateShippingAddress();
    public abstract void ProcessOrder();
}

public class ECommerceOrder extends OnlineOrderProcessor {
 
    public override bool ValidatePaymentInfo() {
        return true;
    }
 
    public override bool ValidateShippingAddress() {
        return true;
    }
 
    public override void ProcessOrder() {
        //place order here if everything is ok.
    }
}
 
public class CashOnDeliveryOrder : OrderProcessor
{
    public override void ProcessOrder()
    {
        //place order here if everything is ok.
    }
}
```

The above code defines two abstract classes OrderProcessor and OnlineOrderProcessor. The OrderProcessor has only one method, ProcessOrder() whereas OnlineOrderProcessor has three methods - ValidatePaymentInfo(), ValidateShippingAddress() and ProcessOrder(). The ECommerceOrder class inherits from OnlineOrderProcessor and the CashOnDeliveryOrder class inherits from OrderProcessor. Since OrderProcessor doesn't contain the ValidatePaymentInfo() and ValidateShippingAddress() CashOnDelivery need not write them at all. Thus we created multiple client specific interfaces (OrderProcessor and OnlineOrderProcessor) instead of a generic one.

## Dependency Inversion Principle (DIP) ##

Dependency Inversion Principle or DIP states that:

High-level modules should not depend on low-level modules. Both should depend on abstractions. 
Abstractions should not depend on details. Details should depend on abstractions.

Consider that you are developing an application that deals with user management (creating a user, managing their passwords, etc.). After every user management task you wish to send a notification to the user as well as to the administrator about the activity performed. The following code shows a simplified implementation of this system.

```java
public class EmailNotifier
{
    public void Notify(string email, string message)
    {
        //send email here
    }
}
 
public class UserManager
{
    EmailNotifier notifier = new EmailNotifier();
 
    public void CreateUser(string userid,string password,string email)
    {
        //create user here
        notifier.Notify(email, "User created successfully!");
    }
 
    public void ChangePassword(string userid, string oldpassword,string newpassword)
    {
        //change password here
        notifier.Notify(email, "Password changed successfully");
    }
}
```

The EmailNotifier class has just one method - Notify() that accepts an email address and a 
notification message. Inside, it sends an email to the specified address notifying the user 
of the change. The UserManager class declares an instance of EmailNotifier class and uses 
it in methods such as CreateUser() and ChangePassword(). So far so good. Now assume that 
instead of email notification you want to send SMS based notifications. To cater to this 
change you not only need to write another class (say SMSNotifier) but also need to change 
UserManager class because UserManager uses EmailNotifier. This problem arises due to the 
fact that high level module UserManager depends on a concrete low level module EmailNotifier.

Let's change our classes to follow DIP.

```java
public abstract class NotifierBase
{
    public abstract void Notify(string message);
}
 
public class EmailNotifier:NotifierBase
{
    public string EmailAddress { get; set; }
    public override void Notify(string message)
    {
        //send email here
    }
}
 
public class SMSNotifier : NotifierBase
{
    public string MobileNumber { get; set; }
    public override void Notify(string message)
    {
        //send SMS here
    }
}
 
public class UserManager
{
    public NotifierBase Notifier { get; set; } 
 
    public void CreateUser(string userid,string password,string email)
    {
        //create user here
        Notifier.Notify("User created successfully!");
    }
 
    public void ChangePassword(string userid, string oldpassword,string newpassword)
    {
        //change password here
        Notifier.Notify("Password changed successfully");
    }
}
```

Now, the code defines an abstract class NotifierBase that defines the Notify() method. The two classes EmailNotifier and SMSNotifier inherit from the NotifierBase class and provide the necessary implementation details. More importantly the UserManager class no longer uses a specific concrete implementation. It uses a NotifierBase abstraction in the form of the Notifier public property. This public property can be set by the consumer of the UserManager class either to an instance of EmailNotifier class or to an instance of SMSNotifier class or any other class that inherits from NotifierBase class. Thus our code now depends on abstractions and not on the concrete implementation.

## Summary ##

SOLID principles of object oriented programming allow you to write structured and neat code that is easy to extend and maintain. SOLID principles include Single Responsibility Principle (SRP), Open/Closed Principle (OCP), Liskov Substitution Principle (LSP), Interface Segregation Principle (ISP) and Dependency Inversion Principle (DIP).
