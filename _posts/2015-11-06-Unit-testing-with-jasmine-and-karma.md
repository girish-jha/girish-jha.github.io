---
layout: post
title:  "Unit testing with jasmine and karma"
categories: TDD
---

### Why?

Probably because your manager asked you to do so. But we need to see beyond that. We as developers, want the code we have written to work, and nothing can be more rewarding than being confident about the code we have just written.
Unit tests are just the tool to give a quick feedback of the same. In a setup where we don't write unit tests, we have to run the application to find if it is working or we have broken something totally unrelated. Unit tests can come in handy there.
Please don't get me wrong here, I am not saying that we can skip the functional testing of the application. That's completely unavoidable. But with proper unit tests in place, number of times we test the application functionally(manually) reduces. We do not have to test(manually) the application for each line of code we have written. That part is taken care of by the unit tests we have written. Moreover as the unit tests are run by the computer, we can configure it to run the unit tests against the whole codeBase on change of each line. This will make sure that any change we are making has not broken anything else.

### How?

For unit testing javascript we will be discussing jasmine as the test framework and karma as the test runner.
To understand what it is, we can think of jasmine as a library which provides us several functions to make unit testing easier. Karma is an app, which runs the jasmine tests. Karma is a Node.Js app.

### Setup

From the above you must have got the hint that we need to have Node.Js, karma and jasmine for our tests to run. We have a few more things to start. So lets install all the things first.

<ol>
    <li>Install Node.js. Once installed, to check the installation you can use follwing command in your command prompt:
<code>node -v</code>and it must return you the version of nodejs installed. At the time of writing this blog, the recommended version is</li>
    <li>Install some node packages</li>
</ol>

```bash
npm install -g karma
npm install -g karma-cli
npm install -g karma-coverage
npm install -g phantomjs
npm install -g karma-jasmine
npm install -g karma-phantomjs-launcher
npm install -g karma-ng-html2js-preprocessor
npm install -g karma-firefox-launcher
npm install –g karma-ie-launcher
```
Alternatively you can run one single command to install all the packages.

```
npm install -g karma-ie-launcher karma-chrome-launcher karma-firefox-launcher karma-ng-html2js-preprocessor karma-phantomjs-launcher karma-jasmine phantomjs karma-coverage karma-cli karma
```
And your machine is good to go.

#### Boot it up

Once all the stuffs installed, navigate to your project directory(where you want to put your karma config files) in command prompt and type following command:

`karma init karma.conf.js`

This will ask you some questions and you can select deferent options by pressing tab key. Select as mentioned below. We will see in more details once the file is generated.

<ol>
    <li>Which testing framework do you want to use ? <strong>Select jasmine</strong></li>
    <li>Do you want to use Require.js ? <strong>Select No</strong></li>
    <li>Do you want to capture any browsers automatically ? <strong>Tab through and select PhantomJs</strong></li>
    <li>What is the location of your source and test files ? <strong>Leave it empty for now</strong></li>
    <li>Should any of the files included by the previous patterns be excluded ? <strong>Leave it empty for now</strong></li>
    <li>Do you want Karma to watch all the files and run the tests on change ? <strong>Select No for now</strong></li>
</ol>

With this your karma conf file is generated in your current working directory. The karma.conf looks like this:

```javascript
// Karma configuration
// Generated on Mon Nov 09 2015 08:00:00 GMT+0000 (GMT Standard Time)
module.exports = function(config) {
config.set({
// base path that will be used to resolve all patterns (eg. files, exclude)
basePath: '',
// frameworks to use
// available frameworks: https://npmjs.org/browse/keyword/karma-adapter
frameworks: ['jasmine'],
// list of files / patterns to load in the browser
files: [
],
// list of files to exclude
exclude: [
],
// preprocess matching files before serving them to the browser
// available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
preprocessors: {
},
// test results reporter to use
// possible values: 'dots', 'progress'
// available reporters: https://npmjs.org/browse/keyword/karma-reporter
reporters: ['progress'],
// web server port
port: 9876,
// enable / disable colors in the output (reporters and logs)
colors: true,
// level of logging
// possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
logLevel: config.LOG_INFO,
// enable / disable watching file and executing tests whenever any file changes
autoWatch: false,
// start these browsers
// available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
browsers: ['PhantomJS'],
// Continuous Integration mode
// if true, Karma captures browsers, runs the tests and exits
singleRun: false
})
}
```
They have put nice comments above each configuration, But i would like to reiterate what they are saying.

<ol>
</ol>

<ol>
    <li>The first field is
<code>basePath</code>, this is the path of the directory from where all the other files are looked for. We will see the usage of it in while discussing files and exclude in point 3 & 4. For now, we can leave it empty.</li>
    <li>Then comes the
<code>frameworks</code>. I believe it's quite self explanatory. We are using Jasmine as test framework.</li>
    <li><code>files</code>is an important config. Whatever filename or paths you put in this will be fetched while testing and be loaded in the browser. Think of it like putting tag in browser. The most important 3 files that has to go (and in order )as a library is as follows:</li>
</ol>

```javascript
files: [
"lib/AngularJS/angular.js",
"lib/AngularJS/angular-mocks.js"
]
```

 

<ol>
</ol>

<ol>After these two we can put any other library we require in our application. For example if you are using underscorejs you will add the following.</ol>

 

```javascript
files: [
"lib/AngularJS/angular.js",
"lib/AngularJS/angular-mocks.js",
"lib/Underscore/underscore.js"
]
```

 

<ol>
</ol>

<ol>And then you add all your script files in the order you require in your application. For example, if you have put all your angular modules in the folder js/apps/ and rest in each corresponding folders put the following.</ol>

 

```javascript
files: [
"lib/AngularJS/angular.js",
"lib/AngularJS/angular-mocks.js",
"lib/Underscore/underscore.js",
"js/app/myapp.js",
"js/services/**/*.js"
"js/controllers/**/*.js"
"js/directives/**/*.js",
"tests/**/*.js"
]
```

 

<ol>The double asterisk ** tells karma to look inside all the child folders as well. Please note that I have put services before controllers, the reason for that is we generally keep our controllers dependent on services. So it's important to have services instantiated before controllers. Any path you are putting in there, should be in the similar order. It's important to remember that all the paths are relative to the basePath you have set as the first config option. basePath becomes important if you are keeping your karma.conf file in a location such that all the paths have a common starting string. We can extract the common part in the paths to basePath
    <li><code>exclude</code>is for any files or pattern you want to exclude from the paths you mentioned in the files section.</li>
    <li><code>preprocessors</code>certain apps which will run before the files are served from karma. One of the use-case is applying certain kind of transforms that angularJS applies and you want to mock it</li>
    <li><code>browsers</code>are the browsers under which the test will run. Currently we have selected phantomJs. It is a headless browser which runs without opening a window. It is very useful to run the tests in CI servers. If you need to debug the tests, you can add Chrome, FireFox or IE as well.</li>
    <li>Another Important config which is not generated by default is
<code>plugins</code>The The plugins in the current app i am working on looks like this:
<code>plugins: ['karma-jasmine', 'karma-coverage', 'karma-chrome-launcher', 'karma-phantomjs-launcher', 'karma-ng-html2js-preprocessor'],</code>You can include more plugins if required.</li>
    <li><code>autoWatch</code> and
<code>singleRun</code> are other two important configs. if it is set like this:
<code>autoWatch: true,
singleRun: false,</code>it will keep watching all the file patterns defined in
<code>files</code>part and will re-run all the tests if any of those files are changed.</li>
</ol>

### Time for action

Now we are ready to write some javaScript in TDD fashion.
According to great [Robert Martin](https://en.wikipedia.org/wiki/Robert_Cecil_Martin) there are [3 rules of TDD](http://butunclebob.com/ArticleS.UncleBob.TheThreeRulesOfTdd) :

1. You are not allowed to write any production code unless it is to make a failing unit test pass.
2. You are not allowed to write any more of a unit test than is sufficient to fail; and compilation failures are failures.
3. You are not allowed to write any more production code than is sufficient to pass the one failing unit test.

So, to wite the test first we need to know about the requirements. Suppose our requirement is to write a little function which will filter a list of objects called person based on their age.
The structure of person is like this:

```javascript
person = {
"name": "Adam",
"age" : 25,
"gender": "male"
}
```

We will have a function which filters the list of persons coming from server and return if their age is more than 18

### Starting to code

#### Setting up environment
1. Install all the required tools as mentioned in above.
2. I would suggest you to install one more npm package called `bower`. It helps in installing client side libraries for your app `npm install -g bower`.
3. Open the console and go to directory of your choice where you want to keep the source.
4. Crete a directory called "jasmineDemo" and inside that 2 more folders called `js` & `tests`. We will keep all our application js files in folder `js` and all the tests in folder `tests`

```
mkdir jasmineDemo
cd jasmineDemo
mkdir js tests
```
5. Install some client side libraries
```
bower install angular angular-mocks
```
This will install two libraries in your folder namely `angular` & `angular-mocks`. `angular-mocks` is a library created by angular team to help us mocking any dependencies to our controller, services etc.
6. Run the karma init so that karma makes a config file for you.
```
karma init karma.conf.js
```
- It will ask you some questions, press tabs to change options and enter to select.
- Let the framework be `jasmine`
- Select no for requireJs(you can select requireJs if you use it in your project)
- For browsers press tab untill you see `phantomJs`
- type following 4 in when it asks for location of souce files
```
bower_components/angular/angular.js
bower_components/angular-mocks/angular-mocks.js
js/**/*.js
tests/**/*.js
```
- Leave empty when it asks for any patterns being excluded.
- Select `yes` when it asks if you want karma to watch all the files.
With these options karma will keep watching any changes to your tests or js directory and run alll the tests again if any of the files changed. You can edit any of these configuration. For more info please refer you [previous post](https://fromcaffeinetocode.wordpress.com/2015/11/09/unit-testing-js-code-with-jasmine/) of this tutorial.


#### Starting to code

We will be creating a test first as it is mentioned above by Robert Martin

1. You are not allowed to write any production code unless it is to make a failing unit test pass.

Create a file in the tests directory `tests/demotests.js` and write following:

```javascript
    describe('[demoControllerTests] ', function() {
        var controller, scope;

        beforeEach(module('demoModule'));
        beforeEach(function(){
            inject(function($rootScope, $controller){
                scope = $rootScope.$new();
                controller = $controller('democontroller', {
                $scope : scope,
            });
            });
        });
    });
```

`describe` is a function defined in `jasmine` which is used to create a testFixture.
You can think of it as a testClass. The first parameter to the functiion is a string which acts like the name of the testFixture(or testClass if you prefer that way).
`beforeEach` is a function again part of `jasmine` framework which works like setup to your tests. Any code written inside a `beforeEach` will run before each test.
`inject` and `module` are functions defined in `angular-mocks` which helps us injecting the dependencies, and instantiating the modules respectively.

As you might have guessed that we are trying to instantiate the module named `demoModule` and we have created an instance of `scope` and `controller` and assigned it to class level variables so that other methods can access them.

We are all set-up. Let's add our first test.
in the same file `tests/demotests.js` let's add our first test of this series of posts

```javascript
describe('[demoControllerTests] ', function() {
    var controller, scope;

    beforeEach(module('demoModule'));

    beforeEach(function(){
        inject(function($rootScope, $controller){
            scope = $rootScope.$new();
            controller = $controller('demoController', {
            $scope : scope,
        });
    });
});

it('should filter people more than 18 years', function() {
//ARRANGE
        var adam = {
        'name': 'Adam',
        'age' : 25,
        'gender': 'male'
        };
        var james = {
        'name': 'James',
        'age' : 18,
        'gender': 'male'
        };
        var john = {
        'name': 'John',
        'age' : 16,
        'gender': 'male'
        };
        var hank = {
        'name': 'Hank',
        'age' : 12,
        'gender': 'male'
        };
        var sherlock = {
        'name': 'Sherlock',
        'age' : 39,
        'gender': 'male'
        };
        var people = [adam, james, john, hank, sherlock];

        //ACT
        var result = scope.getAdults(people);

        //ASSERT
        expect(result.length).toBe(3);
        expect(result[0]).toBe(adam);
        expect(result[1]).toBe(james);
        expect(result[2]).toBe(sherlock);

    });
});
```
In the line 16 in the above snippet we have created our first test. `it` is a function in `jasmine` framework which takes two parameters. First parameter is a string which is the name of the test and the second parameter is a function which isour test.
Any test has 3 parts
1. Arrange - Where we setup the parameters or any other pre-conditions to our tests.
2. Act is the actual call to the "system under test". ie. call to the method we want to test.
3. Assert - Assertions on the result.

As you might have guessed this test will fail. As we havn't created the method nor the module. You can see this test failing in the console. Do not get scared with the long error message you see.
The message is just telling you it could not find the module.

So now lets create our controller and moduleto make the test pass.

Create a file in `js/demoController.js`

```javascript
angular.module('demoModule', []).controller('demoController', ['$scope', function($scope){
    $scope.getAdults = function(people){
        return people.filter(function(person){ return person.age >= 18; });

    };
}]);
```
and just by doing this will make your test pass. You will see your tests failing if you change anything in the code or in the test if uit does not match the requirement.