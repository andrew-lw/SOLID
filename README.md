SOLID - Object Oriented Programming
=====================================

> In object-oriented computer programming, SOLID is a mnemonic acronym for five design principles intended to make software designs more understandable, flexible, and maintainable

The principles listed below are the SOLID principles of object oriented programming (OOP). They are not an exhaustive set but many consider them the pillars of OOP. With any principle two things are present : a guideline to follow or adhere to and a consistent approach. So it is with SOLID. By adhering to the patterns laid out in each principle higher quality software can be the expected result.  

* [Single Responsibility Principle](#Single)
* [Open/Closed Principle](#Open)
* [Liskov Substitution Principle](#Liskov)
* [Interface Segregation Principle](#Interface)
* [Dependency Inversion](#Dependency)

Single Responsibility
---------------------

In Robert C. Martin's book "Clean Code" he reiterates the echo of wise developers, "Functions should do one thing. They should do it well. They should do it only.". This is the very essence of the single responsibility principle. Functions should be singular in purpose. This is fundamental to well designed systems and maintainable code. Code which adheres to the principle is easier to read, easier to extend and is more stable. Here is an example. 

```cs
public class Email_DoesALot
    {
        public static async void DoesALot(string message, string emailAddress)
        {
            //get credentials and obtain auth
            var cred = System.Environment.GetEnvironmentVariable("emailCredential");
            var base64EncodedBytes = System.Convert.FromBase64String(cred);
            var encondedString = System.Text.Encoding.UTF8.GetString(base64EncodedBytes);
            var client = new RestClient("https://someAuthEndPoint");
            var request = new RestRequest(Method.POST);
            request.AddHeader("Content-Type", "application/json");
            request.AddParameter("application/json", "...", ParameterType.RequestBody);
            IRestResponse response = client.Execute(request);
            var auth = JsonConvert.DeserializeObject<Auth>(response.Content);
            //validate email address
            //this validation is obviously just nonsense
            bool IsValid = true;
            if (string.IsNullOrWhiteSpace(emailAddress)) IsValid = false;
            if (!emailAddress.Contains("@")) IsValid = false;
            if (IsValid)
            {
                //send the email
                var apiKey = System.Environment.GetEnvironmentVariable("SENDGRID_API_KEY");
                var sgclient = new SendGridClient(apiKey);
                var from = new EmailAddress("noreply@nowhere.net", "Example User");
                var subject = "Sending an Email";
                var to = new EmailAddress(emailAddress, "Example User");
                var plainTextContent = message;
                var msg = MailHelper.CreateSingleEmail(from, to, subject, plainTextContent, null);
                await sgclient.SendEmailAsync(msg).ConfigureAwait(false);
            }
        }
    }

    public class Email_SRP
    {
        //get credentials and obtain auth
        public static Auth GetAuth()
        {
            var cred = System.Environment.GetEnvironmentVariable("emailCredential");
            var base64EncodedBytes = System.Convert.FromBase64String(cred);
            var encondedString = System.Text.Encoding.UTF8.GetString(base64EncodedBytes);
            var client = new RestClient("https://someAuthEndPoint");
            var request = new RestRequest(Method.POST);
            request.AddHeader("Content-Type", "application/json");
            request.AddParameter("application/json", "...", ParameterType.RequestBody);
            IRestResponse response = client.Execute(request);
            return JsonConvert.DeserializeObject<Auth>(response.Content);
        }

        //validate the email address
        public static bool ValidateEmail(string emailAddress)
        {
            //validate email address
            //this validation is obviously just nonsense
            if (string.IsNullOrEmpty(emailAddress)) return false;
            if (!emailAddress.Contains("@")) return false;
            return true;
        }

        //send the email
        public static async void SendEmail(string emailAddress, string message)
        {
            var apiKey = System.Environment.GetEnvironmentVariable("SENDGRID_API_KEY");
            var sgclient = new SendGridClient(apiKey);
            var from = new EmailAddress("noreply@nowhere.net", "Example User");
            var subject = "Sending an Email";
            var to = new EmailAddress(emailAddress, "Example User");
            var plainTextContent = message;
            var msg = MailHelper.CreateSingleEmail(from, to, subject, plainTextContent, null);
            await sgclient.SendEmailAsync(msg).ConfigureAwait(false);
        }
    }

    public class Auth
    {
        public string token { get; set; }
    }
```

Open / Closed
--------------

The definition you will here thrown out for this principal is, "software elements (classes, methods) should be open to extension and closed to modification".  What does this mean practically? Does this mean no code should ever be rewritten? Only certain types of code? I think most developers would agree if we could write software that never need to be changed and only extended that would be ideal. Let's look at a simple example of one way this can be accomplished, A customer with an address. The address could be expanded in the customer class but one shortcoming of this would be what happens when we need to add extra properties to the address? If the address is used inside multiple methods this can become a very big change just for simplay adding a property. If we 'close' the customer class the modification would need to take place in a different place, in this example the address class. More typically this can often be accomplished through composition (as shown) or dependency injection.  

```cs
public class Customer_NotClosed {
    public string Name { get; set; }
    public string AddressLine1 { get; set; }
    public string AddressLine2 { get; set; }
    public string City { get; set; }
    public string PostalCode { get; set; }
    public string State { get; set; }

    //now we need to add country so we have to put it here 
    public string Country { get; set; }
}

public class Customer_Closed {
    public string Name { get; set; }
    public Address CustomerAddress { get; set; }

}

public class Address {
    public string Line1 { get; set; }
    public string Line2 { get; set; }
    public string City { get; set; }
    public string PostalCode { get; set; }
    public string State { get; set; }

    //now we can add this property here, allowing the customer 
    //to adhere to the principal
    public string Country { get; set; }
}

```

Liskov Substitution
-------------------
In mathmatics there is a principle which allows for the solving of simultaneous equations via substitution. Here is an example.

```js
x + y = 3
3y + x = 5
```

The first equation could be rewritten to be _x = 3 - y_. If that is true what does that mean for the second example? It could be said here that any instance of _x_ could be replaced with _3 - y_. Let's think about inheritance and in what ways it is similar to this example. For a class S that is a subtype of class T, you should be able to say of S the following things:  

* S is a T
* S should have all the desirable properties and methods of T
* any instance of T should be replaceable with an instance of S with no deteriaration to the program

This is a great guideline for good inheritance heirarchy and as such is a backbone to OOP. Following this example can help prevent bad inheritance structures. An antipattern of this perhaps would be a cat as a subclass of airplane class. Not only does a cat need none of the core desirable properties / methods of an airplane. You also can not subsitute a cat for any airplane and get the same desired effect.  

Interface Segregation
---------------------
At the time this principle was laid out I imagine this idea would have been somewhat revolutionary in it's thinking. Today in a world full of microservices and app functions this is clearly a commonly shared idea among many developers. Perhaps this principle started us down the road we find ourselves on today. 

The interface segregation principle states 2 main concepts :
    -1 a class should not be forced to depend on methods it does not use.  
    -2 split large interfaces into smaller generalizations.  

```cs

    public class Doge : IWalker{..}

    public class Dolphin : ISwimmer {...}

    public class Turtle : IWalker, ISwimmer {...}

    public interface ISwimmer { void Swim() {...} }
    public interface IWalker { void Walk() {...} }

```

Not all animals can swim, not all animals can walk. So you could see that putting either of those methods in an IAnimal interface which be an overgeneralization if all animals inherit from IAnimal. A superior solution would be to have a ISwimmer and IWalker interfaces with the supporting methods being split. Than only swimming animals need inherit from ISwimmer and walkers from IWalker. Thus removing a dependency on an interface method which a large amount of animals would not use at all. While doing this we also accomplished the second guidline of this principle. This makes maintenance and even extension easier in the future.  

Dependency Inversion
--------------------

Other OOP Principles  
--------------------

* [Polymorphism](#Polymorphism)
* [Encapsulation](#Encapsulation)
* [Inheritance](#Inheritance)
* [Abstraction](#Abstraction)


### Polymorphism

One of the 3 pillars of object oriented programming. Allows for seperation of classes, their properties and methods from the user of that functionalilty for the purpose of different implementations. The example used in the solution is different types of transports. At compile it is not necessary to know if something is car or plane. Just that it derives from the transport base class. With that knowledge you know that no matter the specific type chosen some things are common : it has an engine of some kind, seats, it can perform some type of movement, etc. The core of the concept is captured in the below example.

```cs
public abstract class ITransport 
{
    public int FuelTankGallons {get; set;}

    public abstract Move();

    public virtual float GetRemainingFuelPct(){
        return 50.3f;
    }
}

public class Car : ITransport {
    //car is required to implement move
    public Move(){
        //vroom...
    }
    //car can override GetTransportType() but chooses not to

}

public class Plane : ITransport {
    //plane is required to implement move
    public  Move(){
        //we have lift off
    }
    //plane can override GetTransportType() and does changing it's behavior
    public override float GetRemainingFuelPct(){

        return 97.1f;
    }
}
```

### Encapsulation

Encapsulation is another pillar of OOP. It provides the means to contain consistent, controllable access to objects. A very common way to accomplish this is by making fields (properties) private and allowing access to them only via public methods. For example, a person class has a 'FirstName' property. Instead of allowing public access to this where anyone can modify the property, a private field is created (in C# generally '_firstName' would indicate it's both private and belongs to that field) and a method to modify it are created. Adding the custom setter to call that method maintains anyone attempting to modify the field would be calling that method. This allows for logic to be consistent across multiple instantiations of the class 'Person'. A simple example of why that is valuable is having the ability to ensure 'FirstName' is always properly capatilizied. You could implement this in code as shown below.  

```cs
//No Encapsulation
public class Person_NoEncapsulation {
    public string FirstName {get; set;}
}

//Has Encapsulation
public class Person_Encapsulation {
        //custom getter and setter
        public string FirstName {
            get { return _firstName; }
            set { SetFirstName(value); }
        }

        //this field is the private backing field for the public property FirstName
        private string _firstName;

        //this logic is executed everytime the FirstName value is set as indicated in the setter
        private void SetFirstName(string input)
        {         
            //example of validation 
            if (string.IsNullOrWhiteSpace(input)) throw new ArgumentException("input for FirstName can not be null or empty");

            //proper case
            CultureInfo cultureInfo = Thread.CurrentThread.CurrentCulture;
            TextInfo textInfo = cultureInfo.TextInfo;
            _firstName = textInfo.ToTitleCase(input);

        }
}

//inside a console app program.cs class
internal class Program
{
    private static void Main(string[] args)
    {
            try
            {
                var person1 = new Person_NoEncapsulation("aNdReW");
                var person2 = new Person_Encapsulation("aNdReW");
                //outputs aNdReW
                Console.WriteLine(person1.FirstName);
                //outputs Andrew
                Console.WriteLine(person2.FirstName);
                //throws exception as expected
                var person3 = new Person_Encapsulation("");
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
    }
}
```


### Inheritance

Rounding out OOP pillars is Inheritance. Inheritance allows for the creation of class heirarchies in which man common methods, properties and logic can be shared across multiple similar classes and even extended in the subclasses. A good real world example of this is people. People have a lot of "properties" and "methods" in common. Eat() could be a common method, NumberOfHearts (timelords need not apply) could be a common "property". But people also have a lot of differences. The property "Dialect" would be different if you are from New Orleans vs if you are from Ontario.  The SayHello() method is another thing that could be very different among groups of people. Inheritance allows for the shared things to keep code DRY (don't repeat yourself) while at the same time allowing for implementations that are different in each subclass. The example below demonstrates this. Each subclass shares the super class peroperties and methods in common and can even override them when they are marked for that purpose.  

```cs
public class Person {
    public string Name {get; set;}

    public string Speak(){
        // they said something
    }

    public virtual string SendEmail(){
        return "sending email from Person class";
    }
}

public class Employee : Person {
    public string JobTitle {get; set;}
    public string Department {get; set;}

    public bool ClockIn(){
        // $$$
    }

    //this class just simply overrides the super class
    public override string SendEmail(){
        return "sending email from the Employee class";
    }
}

public class Customer : Person {
    public string UserName {get; set;}

    public string Checkout(){
        // place an order
    }

    //this override will use the base class logic and implement some of it's own
    public override string SendEmail(){
        string output += base.SendEmail();
        output += "\r\n sending email from the Customer class";
        return output;

    }
}


```


### Abstraction

Where polymorphism allows for data constructs to take multiple different forms(types) and encapsulation provides consistent and controlled access to properties, abstraction removes implementation detail from the concern of the user of the abstracted object. A fantastic example of this is the computer mouse. You don't know or need to know how all the underlying hardware / software works on your mouse. You only need to know that if you plug it in and move it around it moves the cursor on your screen. The expectation is two fold. 1) The implementation of this behavior / property is hidden and 2) The implementation and the mouse still works. This allows for seperation between software which makes it more durable over time. Here is an code sample using that example.

```cs
//Mouse Class
public class Mouse {
    private bool IsPluggedIn;
    private int currXPos;
    private int currYPos;

    public Mouse() {
        IsPluggedIn = true;
    }

    public void Move(int xOffset, int yOffset){
        if (IsPluggedIn){
        //do the work to interact with the hardware
        //and screen and move the cursor
        //the user doesn't know how this works 
        //and doesn't care
            currXPos += xOffset;
            currYPos += yOffset;
        }

    }
}

//inside a console app program.cs class
internal class Program
{
    private static void Main(string[] args)
    {
            try
            {
                //user is using the mouse unaware that 
                // the private fields exist or what Move() does
                var mouse = new Mouse();
                mouse.IsPluggedIn = true;
                mouse.Move(-2,5);
                mouse.Move(3,16);
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
    }
}
```
