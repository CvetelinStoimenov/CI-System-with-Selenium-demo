
# CI/CD Pipeline Selenium Demo

## Overview

This document describes the CI/CD pipeline configuration for a project that utilizes Selenium for automated UI testing. The pipeline is set up using GitHub Actions to automate the process of building, testing, and deploying code changes. The configuration ensures that tests are run in a clean environment with each code push or pull request to the `main` branch.

## Pipeline Configuration

### GitHub Actions Workflow

The GitHub Actions workflow file (`.github/workflows/selenium-ide-ci.yml`) defines the steps to automate the build and test process. Below is the configuration for the workflow:

```yaml
name: Selenium IDE CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up .NET Core
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '6.0.x'

    - name: Install Chrome
      run: |
        sudo apt-get update
        sudo apt-get install -y google-chrome-stable 
    - name: Install dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --no-restore

    - name: Set execute permissions for selenium-manager
      run: chmod +x /home/runner/work/CI-System-with-Selenium-demo/CI-System-with-Selenium-demo/SeleniumIDE/bin/Debug/net6.0/selenium-manager/linux/selenium-manager

    - name: List directory contents for debugging
      run: ls -la /home/runner/work/CI-System-with-Selenium-demo/CI-System-with-Selenium-demo/SeleniumIDE/bin/Debug/net6.0/selenium-manager/linux

    - name: Run tests
      env:
        CHROMEWEBDRIVER: /usr/bin/google-chrome
      run: dotnet test --verbosity normal` 
```
### Key Components

1.  **Trigger Events:**
    
    -   **Push to `main` Branch:** The pipeline is triggered when code is pushed to the `main` branch.
    -   **Pull Request to `main` Branch:** The pipeline runs when a pull request is opened or updated for the `main` branch.
2.  **Jobs:**
    
    -   **Build Job:** This job runs on the latest Ubuntu environment provided by GitHub Actions.
3.  **Steps:**
    
    -   **Checkout Code:** Uses `actions/checkout@v3` to clone the repository.
    -   **Set up .NET Core:** Configures the .NET Core environment with the specified version (`6.0.x`).
    -   **Install Chrome:** Installs the Google Chrome browser on the runner.
    -   **Install Dependencies:** Restores NuGet packages required for the project.
    -   **Build:** Builds the project without restoring packages again.
    -   **Set Execute Permissions:** Sets executable permissions for the Selenium Manager.
    -   **List Directory Contents:** Lists directory contents to debug and verify the existence of required files.
    -   **Run Tests:** Executes the test suite with `dotnet test` and sets the environment variable `CHROMEWEBDRIVER` to the path of Google Chrome.

### Selenium Test Code

Below is an example of a Selenium test case written in C# using NUnit:

```csharp
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
[TestFixture]
public class TC01IfUserIsInvalidTryAgainTest
{
    private IWebDriver driver;
    public IDictionary<string, object> vars { get; private set; }
    private IJavaScriptExecutor js;

    [SetUp]
    public void SetUp()
    {
        ChromeOptions options = new ChromeOptions();
        options.AddArguments("headless");
        options.AddArguments("no-sandbox");
        options.AddArguments("disable-dev-shm-usage");
        options.AddArguments("disable-gpu");
        options.AddArguments("window-size=192x1080");

        driver = new ChromeDriver(options);
        js = (IJavaScriptExecutor)driver;
        vars = new Dictionary<string, object>();
    }

    [TearDown]
    protected void TearDown()
    {
        driver.Quit();
        driver.Dispose();
    }

    [Test]
    public void tC01IfUserIsInvalidTryAgain()
    {
        // Test name: TC01 - If User Is Invalid Try Again
        driver.Navigate().GoToUrl("https://www.saucedemo.com/");
        driver.Manage().Window.Size = new System.Drawing.Size(1552, 832);
        driver.FindElement(By.CssSelector("*[data-test=\"username\"]")).Click();
        driver.FindElement(By.CssSelector("*[data-test=\"username\"]")).SendKeys("user123");
        driver.FindElement(By.CssSelector("*[data-test=\"password\"]")).Click();
        driver.FindElement(By.CssSelector("*[data-test=\"login-password\"]")).Click();
        driver.FindElement(By.CssSelector("*[data-test=\"password\"]")).Click();
        driver.FindElement(By.CssSelector("*[data-test=\"password\"]")).SendKeys("secret_sauce");
        driver.FindElement(By.CssSelector("*[data-test=\"login-button\"]")).Click();
        vars["errorMessage"] = driver.FindElement(By.CssSelector("*[data-test=\"error\"]")).Text;

        if ((Boolean)js.ExecuteScript("return (arguments[0] === 'Epic sadface: Username and password do not match any user in this service')", vars["errorMessage"]))
        {
            Console.WriteLine("Wrong username");
            driver.FindElement(By.CssSelector("*[data-test=\"username\"]")).Clear();
            driver.FindElement(By.CssSelector("*[data-test=\"username\"]")).SendKeys("standard_user");
            driver.FindElement(By.CssSelector("*[data-test=\"login-button\"]")).Click();
            Assert.That(driver.FindElement(By.CssSelector("*[data-test=\"title\"]")).Text, Is.EqualTo("Products"));
            Console.WriteLine("Successful login");
        }
    }
}
```

## Summary

This documentation provides an overview of the CI/CD pipeline setup using GitHub Actions for a project with Selenium tests. It details the workflow configuration, key components, and the Selenium test example. The pipeline ensures automated testing and integration with each code update, providing continuous feedback and improving code quality.
