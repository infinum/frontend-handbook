## Testing

_Guidelines are not rules and should not be followed blindly. Use your head and think._

You might assume that testing should be exclusively conducted by specialized testers, but that's not the case. As a developer, you should also test applications. You may utilize different tools and principles, but ensuring that your applications are covered with tests is imperative.

Tests are essentially snippets of code checking other code, ensuring that your application behaves as expected.

Just as you would inspect or test your car before a road trip or equipment before a presentation, your code should receive the same attention to detail.

Testing is a skill that requires continuous practice.

Write tests, then write more tests, and soon enough, you'll become an unstoppable developer.

Knowing whether some code is functioning correctly can be easily determined by running your tests while enjoying a sip of coffee or tea. This will instill great confidence in writing more code or refactoring existing code.

### What is testing and why is it important?

This is a seemingly simple question with a simple answer.

Testing, also known as software testing in our domain, is a method to ensure that software is free of errors, bugs, and defects by identifying and rectifying them before deployment and use by users. It also aids in identifying missing or partially implemented requirements.

Software can be tested manually by running it and checking all its parts, or you can automate this process by writing tests.

In our development domain, tests are essentially code - functions and variables - used to test other production functions and variables.

You employ various approaches, or a combination thereof, to ensure that your code doesn't crash or behave unexpectedly and that it functions as your requirements specify once in production.

There are numerous approaches to ensuring stable software, which can be categorized and grouped in various ways.

Now, why is testing important?

Because bugs can result in financial losses, such as in stock exchanges or bank accounts, and more importantly, software malfunctions can endanger lives. In 1985, several patients died due to a radiation overdose caused by a race condition in code. In 1994, an airplane crashed, resulting in the deaths of 264 people, due to a bug. That same year, a helicopter crashed, killing 29 people because of a malfunctioning flying system. The list goes on.

You might be thinking that small projects or projects that seemingly pose no danger won't have consequences, but losing users and clients is something you definitely want to avoid.

Ultimately, will your boss be pleased with you for developing faulty software?

Or perhaps you believe that testing will prolong software delivery time and slow down the process. However, searching for a single character bug that disrupts the flow of a large application is far more time-consuming, expensive, and stressful.

### Types of testing

There are lots of types, categories, groupings, and ways of testing, again we are referring to software testing.

We will mention some of them, but feel free to look on the Internet for more because the list can go long and classification can vary.

Common types you will find are:

**Unit testing**

Unit testing is testing the smallest testable part like a function or method.
You usually check if for specific input, to a function for example, you get specific output.
Or if function returns nothing, you might test if something else was done inside its body like other
function call or some variable being modified.
Unit testing is usually done by the same developers who wrote the code.

One thing to remember is that you do not test code that is not yours.
You do not test if React will call componentDidMount or not, or that JSX will be compiled correctly,
or that some external function will do whatever it should be doing.
External libraries and code should already have their tests.
This will be emphasized again later.

**Integration testing**

Since we know what are unit tests, we can now describe what are integration tests and what is
integration testing.
Basically, it is testing of units combined and tested as a group.
Purpose of these tests are checking if interaction between units is correct and that there are no
faults.

It is more complex than unit testing, and sometimes you also need to have some configuration.
For example if you test that after some „add“ button click, your list will have one item more.
These tests are usually done by the code authors as well.

**System testing**

This kind of testing goes after implementation and in our domain it is tested in a browser.

You run the application and check how it is behaving.
For example, you open application and click button that should open modal, and modal should
have some different button that should do something else and so on.
This part can be automatized by tests which does this for you: clicking on a UI, expecting components
to be shown with correct data, typing in fields etc.
This type of tests can be done by other testers.

You can come across system testing being similar with end to end testing (often written as e2e).
We will give one point of view since this is quite debatable.
Let's say we have application that has list of items on homepage and next to each item there is add
to cart action.
After adding at least 1 item, going to cart is available.
In the cart there is summary of selected items and action to purchase it which goes to payment
service and after payment is done it should return to home page.
System testing will test if we can add items, go to cart, go to purchase and go back.
End to end testing will be testing same as system testing, but additionally, our balance should be
lowered for correct amount, and correct amount should be added to seller, correct items should be
ordered and so on.
So like a real world application usage.
As you can already see, end to end testing is really difficult to do with automatic tests.
This kind of tests is usually done by QA people, since it might require a separate database and backend from staging/development one.

Above list of testing types can be also called functional and there are more than these types.
There are also non-functional testing types, like:

- Performance testing
- Stress testing
- Security testing
- Localization testing
- Acceptance testing
- And more...

Remember, what and how should software be tested and by whom depends on your organization, company or team.
For example, Quality Assurance team might do one set of tests instead of you.
Or maybe you will do all by yourself.

### How to literally write tests?

The approach varies based on your development stack.

React developers employ different tools compared to Angular or Vue developers. Similarly, Wordpress developers have their own set of testing utilities.

However, what remains consistent across these stacks is the practice of coding for tests. Here's an example:

```Javascript
// isOddNumber is your function

test('isOddNumber returns true for value 35', () => {
  expect(isOddNumber(35))
    .toEqual(true);
});
```

The specifics of utilizing tools within each stack should be covered within their respective documentation.

### TDD

We need to note that we will simplify TDD concept since it cannot be explained in single page.

_So what is TDD?_

TDD or _Test Driven Development_ is a technique of writing tests before code in a way that you write simple failing test and as smallest code possible to pass that test. After that you write another simple failing test and another smallest code to pass this second one. You continue to do that until you are finished with code and refactor in the middle.

Sounds confusing, but it is not. Let's see an example.

We will not write code, just explain what is happening. Let's say we want to develop calculator function that allows addition, subtraction, multiplication, and division. calculator function should take 3 arguments: two numbers and operation type. And it needs to be bulletproof and check for all invalid usage.

We will use TDD and start with simple failing test:

> calculator should throw error if called empty

Why is this test failing? Since we didn't write code to fulfill that test statement yet. We write some code to throw error if calculator called empty. Test passes.

Next simple failing test will be:

> calculator should throw error if called with less or more than 3 arguments

We could also have negation like: calculator should not throw error if there is exactly 3 arguments. Try to avoid negations since they can add confusion to code. Using clean code principles here. And negativity is not good.

Back to test statement. We write code to throw error if there is 0, 1, 2, 4, or more arguments. Test passes.

We see an opportunity to refactor both in code and test. There is a check for empty and for exactly three arguments. We can leave them both, or remove empty check since it is covered by this second test.

Let's remove the empty check to clean code. Next, we need to check if the first and second arguments are numbers, and the last one is a string.

> calculator should throw error if invalid types passed for arguments

We write code to check for types. Test passes.

Then

> calculator should throw error if operation type is invalid

We test that the string is the correct value which can be 'ADD', 'SUBTRACT', and so on - whatever our interface will provide. Code will be added and test passes.

Then there is that HUGE check if dividing with 0.

> calculator should not have 0 for 2nd number with 'DIV' operation type

Code for that and test passes.

You might be yelling _"Where is calculation implementation!"_. We go step by step and here we first assure that usage is correct, and then functionality should be added.

Can you see how these checks are describing usage similar to your requirements? This is very important. Imagine someone goes to your tests first. They can know what your code does, if we assume that tests pass, without going to source. This is something that is making tests cost-effective. If someone needs to know functionality or refactor, it will be easy to do. We can add a few more tests and continue to talk about TDD.

Test for:

> calculator should return 4 if 1 and 3 are passed for 'ADD' operation type

Write code. Test passes.

> calculator should return -4 if -1 and -3 are passed for 'ADD' operation type

Again. Write code. Test passes. And so on.

Many developers find TDD annoying, not useful or time-consuming. This only means they do not see its full potential. TDD is an excellent technique for catching bugs in the start. There are some rules which you have to follow.

Most popular rules are:

- You are not allowed to write any production code unless it is to make a failing unit test pass. This means you must not write code before test.
- You are not allowed to write any more of a unit test than is sufficient to fail; and compilation failures are failures. This means you must write test as small as possible and failing one.
- You are not allowed to write any more production code than is sufficient to pass the one failing unit test. This means you must not write more code than needed to pass the test.

Every TDD should stick to these rules.

With these rules, there is also something called RED-GREEN-REFACTOR phases.

Red means you write a failing test. Imagine this as a request to add new functionality by your user. You do not have implementation at the moment, just criteria for what should it be. You should not think about implementation here. Just, how it should be used, at the moment, not later. Let's get back to the calculator function and the second test. Skipping the first since we removed it. This test is checking if we have 3 arguments. Here, we might consider a few options like maybe:

- having 2 arguments - array of numbers, and operation type like [2, 3, 5], 'ADD'; so we can, for example, add multiple numbers and not just two
- having also 2 arguments - array of arrays where each array is [[2, 3, 'ADD'], [7, 6, 'SUBTRACT']] and operation type to calculate them all
- or something else.

Concentrate on current work, and not something in the future.

Green phase is where you finally can make implementation and make your test passing - green. And it is a test not tests since you should at this moment concentrate on a new one and not old tests. Write directly to pass it. Don't bother with duplication or ugly code. This can be refactored, and guess what, your tests will tell you if you refactored correctly.

And finally, the refactor phase (also known as the yellow phase) where you remove code duplication, clean a bit, make variable names better and so on. One change, one execution of all tests. If something fails, go back to where you have been before. It is easy with IDEs and Undo action.

And then you repeat all steps for a new test.

This is as we said a simplified version of TDD. TDD sounds easy but is hard to master and requires a lot of practice. TDD does not mean only unit testing, but it is mostly used for that.

Unit tests are okay, but not sufficient to be sure that the software is working properly so you also need to test it with different types of testing.

Remember, _Test it before you waste it_, and we mean code.

### Team Player

When working on a project as a team, you decide on:

- how your code will be structured,
- which tools should be used in development,
- what your and general best practices are,
- and so on...

It is no different when making these decisions (and more) for application testing.

There are some "laws" created by the community and us which need to be followed, but the rest is up to you and your team. The main point is that everyone is on the same page after the decisions are made.

#### "Laws"

![Testing](/img/testing.jpg)

1. Any testing is better than no testing.
2. Do not test 3rd party code (including yours in a different project - like in a library).
3. Tests are also code, so make them clean.
4. Pre-commit test execution is a must.
5. Testing is for you to sleep well, not your client.
6. No one is perfect at writing tests; just practice.
