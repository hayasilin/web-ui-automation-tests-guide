# Web UI Automation Tests Guide

[Here is Traditional Chinese version 繁體中文版本](https://github.com/hayasilin/web-ui-automation-tests-guide/blob/master/README_zh-hant.md).

This is a development guide of how to develop reliable and stable UI automation tests for web application that I conclude from my experience. I welcome your feedback in [issues](https://github.com/hayasilin/web-ui-automation-tests-guide/issues) and [pull requests](https://github.com/hayasilin/web-ui-automation-tests-guide/pulls).

If you are a mobile app developer and want to know about mobile app UI automation tests, you can see my [iOS and Android UI Automation Tests Guide](https://github.com/hayasilin/ios-android-ui-automation-tests-guide)

## Introduction

Before you start reading this guide. I assume you have basic knowledge of UI automation tests for web application and you are using frameworks like Selenium WebDriver or Cypress. In addition, you have experience in implementing UI automation tests into your CI/CD automation pipeline.

Here are some of the documents of the test frameworks. If something isn't mentioned here, it's probably covered in one of these:
- Selenium
  - [Selenium.dev](https://selenium.dev/)
  - [Selenium WebDriver](https://selenium.dev/projects/)
  - [Selenium with Python](https://selenium-python.readthedocs.io/index.html)

- Cypress
  - [Cypress.io](https://www.cypress.io/)

## Table of Contents
- [Before developing UI automation tests](#before-developing-ui-automation-tests)
  - [UI automation tests is easy to develop but expensive to maintain](#ui-automation-tests-is-easy-to-develop-but-expensive-to-maintain)
  - [What are flaky tests](#what-are-flaky-tests)
  - [Why UI automation tests make flaky tests](#why-ui-automation-tests-make-flaky-tests)
- [Best practice to make UI automation tests reliable and stable](#best-practice-to-make-ui-automation-tests-reliable-and-stable)
  - [Follow testing pyramid](#follow-testing-pyramid)
  - [The mind set of developing UI automation tests](#the-mind-set-of-developing-UI-automation-tests)
  - [Choose tests cases for UI automation tests](#choose-tests-cases-for-ui-automation-tests)
  - [Top 10 practical ways to avoid flaky tests](#top-10-practical-ways-to-avoid-flaky-tests)
- [UI tests code convention](#ui-tests-code-convention)
- [Common questions](#common-questions)

## Before developing UI automation tests

### UI automation tests are easy to develop but expensive to maintain
- UI automation tests may seem to be amazing because it simulates actual user scenario and test our web application automatically, however, although it's definitely needed by team but it also comes with some costs.
- UI automation tests are expense to maintain because our web application will keep changing in agile development world and our test code need to be changed constantly as well. Don't forgot the maintenance cost.
- Originally UI automaiton testing are developer's tool to check that they don't break anything after they change code. Nowadays UI automation testing is widely used by QA testing as well to speed up testing by reducing time consuming manual testing. However, It's easily to think that we can automate everything with UI automation tests and get rid of all your manual testing without knowing its limitation and cost. The first and foremost thing you are going to deal with is the notorious **flaky tests**.
- Flaky tests will make unreliable and unstable UI automation tests, which is meaningless for QA testing.
  - Because the uncertainty of the automation test result makes QA and the team have low confidence on UI automation tests, in other words, they won't trust the result. Eventually your team still spend times to perform manual regression testing, which is time consuming and slow your team down.
  - As a result, no one wants to write or maintain UI automation code due to lack of attention of the team members.

### What are flaky tests
- Flaky tests are when you run your automation tests, some tests passes this time but fail next time even you didn't change any code. What's worse when you check the web application by yourself, it works fine. It's unpredictable to know if the tests will pass or not.
- Flaky tests example is below, the Test A and Test C are flaky tests because pass and fail results take turns to show. On the contrary, Test B is not a flaky test because it keep passing. 
- In general, we will put UI automation tests into our CI/CD pipeline and trigger it automatically according to our setting. One of the most common tool we use is Jenkins. If your team have flaky tests, you can see the example below that your Jenkins result would be highly likely failed because Jenkins will mark the job failed even only 1 test case failed. Then, your team need to send one of the team member to check why it failed.
- Below example only has 3 tests cases. If your team has 100 more test cases and there are flaky tests. Imagine how many times your team need to check if it's really web application's issue or flaky tests.

| Test Runs   | 1st     | 2nd      | 3rd      | 4th      | 5th     | 6th      | 7th      | 8th      | 9th      | 10th     | Next run?              |
| ----------- | ------- |--------- | ---------| -------- | ------- | -------- | -------- | -------- | -------- | -------- | ---------------------- |
| Test Case A | Success | **Fail** | Success  | Success  | Success | **Fail** | Success  | Success  | **Fail** | **Fail** | **?**                  | 
| Test Case B | Success | Success  | Success  | Success  | Success | Success  | Success  | Success  | Success  | Success  | **?**                  | 
| Test Case C | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | Success  | **Fail** | **?**                  | 
| ...                                                                                                                                              |
| Jenkins     | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | **Fail** | **Fail** | **Highly likely Fail** |


- Why flaky tests are bad:
  - Make the team has no confidence on UI automation tests.
  - It's a blocker for making CI/CD pipeline if you design a series of actions depend on the result of UI auotmation tests.
  - Can't achieve the goal of fast web application delivery, let alone maintain or improve web application's quality by providing quick feedback during development

- Google's experience on handling flaky tests
  - 2015 [Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)
  - 2016 [Flaky Tests at Google and How We Mitigate Them](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
  - 2017 [Where do our flaky tests come from](https://testing.googleblog.com/2017/04/where-do-our-flaky-tests-come-from.html)
  
### Why UI automation tests make flaky tests

- Since UI automation testing is end-to-end testing that integrates the features of UI Display, API handling, cross-web behavior, and user status change, if one of the part fails then the whole test fails too. There are too many variables when running end-to-end testing and it makes UI automation has vulnerability nature and easily cause flaky tests.
- Didn't do well on **Test Isolation**.
- Native test frameworks have its limitation, sometimes your tests fail becasue of its limitation instead of your test code.

**Internal flaky factors:**
- UI or user scenario may change in every release.
- Team don't have a stable test environment or test data.
- Element's creation has strong dependency on different factors, such as framework version or API response. The creation time can be short or can be long. Don't use just time to wait (or use sleep), you should use wait until element exists before perfoming action to the element.
- Dynamic UI element design: web application's UI will change constantly according to server's config.
- Poorly written test code.

**External flaky facotors:**
- Known issues of native test frameworks (Selenium WebDriver / Cypress).
- Browser dependency (Thank god that now we only need to test on Chrome. In the past it was a disaster when we needed to test on IE, Chorme, and Firefox).
- Cross web applications testing instability.
- Prallel testing instability.
- IDE run and command line run may have different behaviors.
- and **more...**.

## Best practice to make UI automation tests reliable and stable

### Follow testing pyramid

According to Google's [Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html), testing pyramid is try to imagine a pyramid, the bulk of your tests are unit tests at the bottom of the pyramid. As you move up the pyramid, your tests gets larger, but at the same time the number of tests (the width of your pyramid) gets smaller.

Google often suggests a 70/20/10 split: 70% unit tests, 20% integration tests, and 10% end-to-end tests. The exact mix will be different for each team, but in general, it should retain that pyramid shape. Try to avoid anti-pattern like the team relies primarily on end-to-end tests, using few integration tests and even fewer unit tests. 

Just like a regular pyramid tends to be the most stable structure in real life, the testing pyramid also tends to be the most stable testing strategy.

### The mind set of developing UI automation tests
- Aware of the maintenance cost, browser dependency, test environment and data instability, and limitation of UI automation tests before start developing it.
- The testing level for UI automation testing is a **Smoke Testing**, which it to go over the happy path of your web application. UI automation should not do more.
- If there are automation tests is flaky, remove it right away with no mercy.
- Your team won't have time to keep checking or fixing flaky tests. Only a stale UI automation can actually help your team to achieve goals below:
  - Fasten web application delivery.
  - Provide quick feedback to maintain and improve web application's quality.

## Choose tests cases for UI automation tests

**Pros**
- Mature functionalities, which is stable and won't be changed in the short term.
- Most common user scenarios / Happy path.
- Suitable to automate.

**Cons**
- Features are keep changing because of business plan.
- Hard to automate or beyond test framework's limitation.
- It's costly to make it automation.

- UI automation tests can not make all test cases automation, choose the right test cases and leave others remain in manual testing.
  
### Top 10 practical ways to avoid flaky tests
1. Test isolation: test should not depend on other tests. If the test case changes status or manipulate database, please clean it up before or after the test case.
2. The UI you see now may not to be the only UI. UI may change according to user's status. Consider every factor that could cause flaky and make your tests robust to deal with all known UI changes.
3. Need to run all the tests around 10 times after complete new test code to see it won't become flaky tests.
4. After develop test code, run your new tests on the devices that your team used for UI automation tests to check there are no device dependency issues.
5. Less navigation of screen is better. The automation test flow should be as simple as possible. One of the best ways is to open the page (URL) you want to start the test. It not only can shorten testing time, but also avoid flaky tests.
6. Choose the mature functionalities to develop UI automation carefully.
7. Don't over design your test code or make it too complex, keep it simple and use design pattern like page object pattern to make it easy to maintain.
8. Stability is more important than completion. Don't try to cover as many as tests you can, just automate the tests that is easy to automate, remember UI automation testing is just a smoke testing. Unstable and unreliable UI automation tests are meaningless to your team. Because it's waste of time to investigate flaky tests.
9. Pick just one UI element to assert in every test case. The name of UI testing easily confuse us and make we think that we need to test the every UI element on the screen. However, it will only create flaky tests. In practical, please think UI testing as a tool to test user behavior or user scenario instead of testing UI element's position or layout.
10. Don't use too many browser to run parallel testing. Test result from several browsers should provide your team enough confidence to decide release your web appliction or not.

## UI tests code convention

**Open browser**

```java
WebDriver driver = new ChromeDriver();
```

**Open URL**

```java
driver.get("http://www.google.com");
```

**Find element by name attribute**

```java
WebElement element = driver.findElement(By.name("name"));
```

**Type text in input tag**

```java
element.sendKeys("text");
```

**Submit the form**

```java
element.submit();
```

**Get current page title**

```java
driver.getTitle();
```

**Wait 10 seconds until the page is loaded**

```java
(new WebDriverWait(driver, 10)).until(new ExpectedCondition<Boolean>()
{
    public Boolean apply(WebDriver d)
    {
        return d.getTitle().toLowerCase().startsWith("text");
    }
});

```

**Close browser**

```java
driver.quit();
```

## Common questions
1. What frequency should we trigger our UI automation tests in CI/CD pipeline to test our app is functioning properly?
  - My recommendation is **once a day** at early morning or late night.
    - Because usually your team, including app developers or server side developers, are modifying code or environemnt's config for new features during working hours. So it's better to avoid running UI automation tests during working hour to avoid flaky factors.
    - In theory, we may imagine that we can run UI tests when developers send their pull request, just like unit tests. After all tests are passed, the developer can merge the code into branch. However, in reality it's infeasible. Because when you have more UI test cases, you need more time to run. As you already know, unit tests run fast but UI tests take longer time to run, it will make developers and code need to wait long time before they can merge their code, which cause efficiency issue. Furthermore, as I mentioned above, UI tests have fragile nature, it would fail for various reasons. If your UI tests keep failing, it will block your CI/CD pipeline. Hence practically I suggest we don't need to make UI tests into pull request process.

2. What test frameworks or tools should we use?
  - Selenium WebDriver is still a popular framework, however, Cypress is also recommended.
    - Selenium WebDriver's history is mostly equals to the web UI automation testing history and it is still popular now. If you are using it, it is still a useful tool.
    - New comer like Cypress provides another option. Unlike Selenium WebDriver, Cypress is built on top of Mocha and Chai and by bundling Chrome, it provides a more native way to test your web application and gives you more control. If you just start to develop web UI automation tests, Cypress is worth a try.