#  译文: [iOS Unit Testing and UI Testing Tutorial](https://www.raywenderlich.com/709-ios-unit-testing-and-ui-testing-tutorial)

原文： [iOS Unit Testing and UI Testing Tutorial](https://www.raywenderlich.com/709-ios-unit-testing-and-ui-testing-tutorial)，作者：Audrey Tam。更新于2017年3月13日。以下为正文：



本教程讲解如何往iOS apps中添加「单元测试/unit tests」、「UI测试/UI tests」，以及如何检查「代码的覆盖率/code coverage」。

**Version：**

Swift 3，iOS 10，Xcode 8

（我在Swift4.2，iOS11，Xcode 9运行也正常）



很多开发者觉得写测试没什么卵用，但是，如果没有「测试」，你原本牛逼闪闪的app，很容易变成一坨翔，所以，「测试」是必不可少的。如果你正在看这篇教程，那么恭喜您，你是一个有追求的人，一个脱离了低级趣味的人（该处译者自由发挥），您起码知道*应该*要写测试了，只是暂时还不知道怎么写而已。

或者你有正在开发的app，但还没写测试，你希望可以在扩展app的时候，对修改的部分进行测试。也可能你已经写了一部分测试，但不确定写得对不对。又或者你随时想对正在开发的app进行测试。

这篇教程，演示了如何利用Xcode的test navigator来测试app的「模型/model」和「异步方法/asynchronous methods」；如何利用stubs、mocks模拟和library、system进行交互；如何测试UI、性能；以及如何使用「代码覆盖工具/code coverage tool」。学习过程中，会接触到一些装逼术语，学习完本宝典后，你就是大神了！



# Testing, Testing...

##  What to Test?/测什么？

开始写测试之前，有一件非常重要的事情：究竟要测什么？如果目的是扩展(修改)现有的app，那么首先要为即将要修改的部分写测试。

测试通常包括：

- 核心功能/Core functionality：模型类和方法，以及他们和控制器的交互
- 常见的UI工作流
- 边界条件/Boundary conditions
- Bug修复

## First Things FIRST: Best Practices for Testing

**FIRST**是「Fast，Independent，Repeatable，Self-validating，Timely」的缩写，描述了一套有效、简明的单元测试标准：

- **Fast/高效**：你写的测试可以很快完成——只有这样大家才不介意去跑测试代码。
- **Independent/Isolated**：测试不应该彼此依赖、拆解
- **Repeatable**：每次跑测试，得到的结果都应该一致。当然，外部数据和并发问题（concurrency issues）可能偶尔导致测试结果会不一样。
- **Self-validating**：测试应完全自动化；测试结果应该是「pass」或者「fail」，而不需要程序员从一堆日志（log）文件中推测测试结果。
- **Timely**：理想情况下，在写生产代码前，就应该写好测试。（译者：这里说的是理想情况下，所以有很多情况也是先写功能，再写测试的？）

遵循FIRST原则，可以保证写出来的测试简单、实用，不至于成为你的累赘。



# Getting Started

下载、解压、打开[starter projects](https://koenig-media.raywenderlich.com/uploads/2016/12/Starters.zip) 中的BullsEye 和 HalfTune项目。

**BullsEye**是[iOS Apprentice](https://store.raywenderlich.com)中的一个简单的app（一个游戏app——译者）；项目已经把游戏逻辑解耦到`BullsEyeGame`这个类了，并且加了另外一种游戏模式。

（运行BullsEye——译者）在右下角，可以看到，有一个Segmented Control控件，可以让使用者选择游戏风格：

- **Slide**模式，将slider滑动到尽可能靠近预先设定的目标值，

- **Type**模式，猜测slider滑动条目前的值，将结果输入顶部TextField中。

用户选择的游戏模式，app也会保存作为默认值（重启app，默认游戏模式是使用者上次选择的模式——译者）

**HalfTunes**是[NSURLsession Tutorial](https://www.raywenderlich.com/158106/urlsession-tutorial-getting-started)中的一个app，更新到Swift 3了（事实上已经更新到Swift 4了——译者）。在这个app，调用了iTunes 的API来查询歌曲，还可以下载、播放歌曲的片段。

坐稳了，开车！



# Unit Testing in Xcode

## 创建一个Unit Test Target

**Xcode Test Navigator**提供了使用测试的简便方式；下面会利用它来创建test target，并且把测试跑起来。

打开**BullsEye**，按快捷键**Command-6**，打开test navigator。

点击左下角的**+**按钮，选择菜单中的**New Unit Test Target**...

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/TestNavigator1.png)



使用默认的名字：**BullsEyeTests**。看到*test bundle*时，点击打开。如果BullsEyeTest没有出现，单击切换到其他navigators，再返回test navagator。

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/TestNavigator2.png)



可以看到模版文件导入了`XCTest`，定义了一个`XCTestCase`的子类：`BullsEyeTests`，还有`setup()`，`tearDown()`和一些example test 方法。

有三种跑测试的方法：

1. 点击菜单**Product \ Test**，或者快捷键Command-U。这种方式会将*所有*的test类都跑一边。
2. 点击test navigator中的小箭头按钮。
3. 点击gutter中的菱形按钮。（就是显示代码行数旁边的按钮——译者）

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/TestNavigator3.png)



通过点击test navigator或者gutter中的按钮，可以跑单独一个测试方法。

试一下用上面不同的方法跑一下测试，直观感受一下。因为现在这些测试什么都没做，所以很快就跑完了。

所有测试跑完之后，菱形按钮变成绿色，并呈现勾选状态。点击`testPerformanceExample()`方法下面的灰色菱形按钮，打开Performance Result：

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/TestNavigator4.png)



现在不需要`testPerformanceExample()`这个方法，暂时删掉。



## 用XCTAssert测试Models

首先，下面要用XCTAssert 来测试BullsEye model的核心功能：`BullEyeGame`对象计算的分数是否正确？

来到**BullsEyeTests.swift**，在`import`语句下，添加如下代码：

```swift
@testable import BullsEye
```

这句代码给了unit test 权限访问BullsEye中的类、方法。

在`BullsEyesTests`类的顶部，添加以下属性：

```swift
var gameUnderTest: BullsEyeGame!
```

在`setup()`方法内，`super.setUp()` 之后，创建一个新的`BullsEyeGame`对象：

```swift
gameUnderTest = BullsEyeGame()
gameUnderTest.startNewGame()
```

上面创建了一个class 层级的SUT（System Under Test）对象，所以在这个测试类里的所有测试都可以访问SUT对象里的属性和方法。

也可以调用`startNewGame`方法——这个方法创建`targetValue`。很多测试会用到`targetValue`——用来测试游戏是否正确计算「分数」。

在`tearDown()`方法内要*释放(release)* SUT对象。

```swift
gameUnderTest = nil
```

> **注意：**在`setup()`创建、在`tearDown()`释放 SUT对象，是一个好习惯。可以确保每个测试都是在干净的环境中进行。更多资料，可以查看[Jon Reid 关于此主题的帖子](https://qualitycoding.org/xctestcase-teardown/)。



现在开始写第一个测试！

用以下代码替换整个`testExample()`：

```swift
// XCTAssert to test model
func testScoreIsComputed() {
  // 1. given
  let guess = gameUnderTest.targetValue + 5
  
  // 2. when
  _ = gameUnderTest.check(guess: guess)
  
  // 3. then
  XCTAssertEqual(gameUnderTest.scoreRound, 95, "Score computed from guess is wrong")
}
```

测试方法的方法名一般都以`test`开头，后面跟的是要测什么东西。

把测试分解成**given**、**when**、**then**三部分，是一个好习惯：

1. 在**given**部分，设置所需要的值：上面的例子，创建了一个`guess`值，可以设定与targetValue的差异。
2. 在**when**部分，执行代码进行测试：调用`gameUnderTest.check(_:)`方法。
3. 在**then**部分，assert（断言）所期望的结果（在这个例子，`gameUnderTest.scoreRound`是100 - 5），如果测试结果失败，打印一条消息。

点击菱形按钮跑测试。app就会跑起来，菱形按钮也会变成绿色勾选状态。

> Note：如果要看**XCTestAssertions**的完整列表，在代码中按Command键同时点击XCTAssertEqual 打开XCTestAssertions.h，或者到这里看： [Apple’s Assertions Listed by Category](https://developer.apple.com/library/prerelease/content/documentation/DeveloperTools/Conceptual/testing_with_xcode/chapters/04-writing_tests.html#//apple_ref/doc/uid/TP40014132-CH4-SW35).

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/givenWhenThen.png)

> Note：Given-When-Then 结构起源于Behavior Driven Development（BDD/行为驱动开发），而Given-When-Then 这个名字更通俗易懂。也可以用Arrange-Act-Assert，或者Assembl-Activate-Assert。



## Debugging a Test

我们在`BullsEyeGame`故意内置了一个bug，现在就来找一下这个bug。

将`testScoreIsComputed`重命名为`testScoreIsComputedWhenGuessGTTarget`	，然后再复制-粘贴一个，创建`testScoreIsComputedWhenGuessLTTarget`。

在`testScoreIsComputedWhenGuessLTTarget()`这个测试中，**given**部分的`targetValue`*减去*5。其他保持不变。

```swift
func testScoreIsComputedWhenGuessLTTarget() {
  // 1. given
  let guess = gameUnderTest.targetValue - 5
  
  // 2. when
  _ = gameUnderTest.check(guess: guess)
  
  // 3. then
  XCTAssertEqual(gameUnderTest.scoreRound, 95, "Score computed from guess is wrong")
}
```

`guess`和`targetValue`之间相差还是5，所以score仍然是95。

打开breakpoint navigator，添加一个**Test Failure Breakpoint**；当测试方法发出失败的assertion（断言）时，测试就会停在这里。

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/AddTestFailureBreakpoint.png)

把测试跑起来：测试失败，应该会停在`XCTAssertEqual`这行。

打开debug console，检查`gameUnderTest`和`guess`的值：

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/TestFailure.png)

`guess`的值是`targetValue - 5` ，但是`scoreRound`是105，并不是期待中的95！

为了进一步找到问题点，使用平常的debug方式：在**when**语句中设置断点，在**BullsEyeGame.swift**中的`check(_:)`方法内，创建`difference`的地方也设置一个断点。然后再跑一次，逐步执行，来到`let difference `语句，查看`difference`的值：

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/DebugConsole.png)

问题出在`difference`的值是负数，所以score的值变成*100 - (-5)*；可以对diffenecne取*绝对值*来修复这个问题。在`check(_:)`方法中，取消注释正确的那行，并删除有问题的那行。

删掉两个断点，再重新跑测试，这次没有问题了。



## 用XCTestExpectation测试异步操作

上面已经学会如何测试models，如何在测试失败时debug，现在继续学习使用`XCTestExpectation`来测试网络操作（network operations）。

打开**HalfTunes**项目：这个app用`URLSession`来查询iTunes API 并下载歌曲片段。假设你要改成用[AlamoFire](https://www.raywenderlich.com/35-alamofire-tutorial-getting-started)来进行网络操作。要确认这个改写过程是否有纰漏，应该写测试来验证这些修改的代码，在修改前、修改后都要跑测试。

`URLSession`方法是*异步*的：马上返回，但要等一段时间才真正完成。要测试异步方法，可以用`XCTestExpectation`，它可以让测试等到异步操作完成。

异步测试一般比较慢，所要要和unit tests 分开。

也是在**+**号菜单中选择**New Unit Test Target**…并命名为**HalfTunesSlowTests**。在`import`下面导入HalfTunes app：

```swift
@testable import HalfTunes
```

这个类中的测试会用默认session向苹果服务器发送请求，声明`sessionUnderTest`变量，并在`setup()`中创建该对象、在`tearDown():`中释放：

```swift
var sessionUnderTest: URLSession!

override func setUp() {
  super.setUp()
  sessionUnderTest = URLSession(configuration: URLSessionConfiguration.default)
}

override func tearDown() {
  sessionUnderTest = nil
  super.tearDown()
}
```

用下面异步测试的代码替换原来的`testExample()`：

```swift
// Asynchronous test: success fast, failure slow
func testValidCallToiTunesGetsHTTPStatusCode200() {
  // given
  let url = URL(string: "https://itunes.apple.com/search?media=music&entity=song&term=abba")
  // 1
  let promise = expectation(description: "Status code: 200")
  
  // when
  let dataTask = sessionUnderTest.dataTask(with: url!) { data, response, error in
    // then
    if let error = error {
      XCTFail("Error: \(error.localizedDescription)")
      return
    } else if let statusCode = (response as? HTTPURLResponse)?.statusCode {
      if statusCode == 200 {
        // 2
        promise.fulfill()
      } else {
        XCTFail("Status code: \(statusCode)")
      }
    }
  }
  dataTask.resume()
  // 3
  waitForExpectations(timeout: 5, handler: nil)
}
```

这个测试检查发送有效查询到iTunes，返回200状态码的情况。大多数测试代码和在app中实际写的一样，下面这些是额外添加的：

1. `expectation(_:)`返回一个`XCTestExpectation`对象，并赋值保存为`promise`。通常也可以用`expectation`和`future`来命名。`description`参数描述了你期望发生的结果。
2. 为了匹配`description`，在异步方法回调成功时，调用`promise.fulfill()`。
3. `waitForExpectations(_:handler:)`确保测试一直运行，直到达成所有期望的结果（expectations），或者 `timeout` 超时结束，以先触发者为准。

把测试跑起来，如果是连着网络的，app在模拟器加载后，测试大概几秒就能完成。



## Fail Faster

测试失败大家都不愿意看到，不过不可能百分百保证每次测试都能通过。下面介绍快速识别测试是否失败，省下来的时间，就可以刷抖音刷朋友圈了:]

为了模拟测试失败，删除URL中「itunes」中的「s」：

```swift
let url = URL(string: "https://itune.apple.com/search?media=music&entity=song&term=abba")
```

再跑一次：如我们所愿，测试失败了，但是它跑完timeout的时间（5秒——译者）才提示失败！这是因为我们之前写的代码，要等到「请求」成功后，才会调用`promise.fulfill()`。不过这次的「请求」是失败的，所以只能等timeout超时后才能结束测试。

通过修改expectation，可以让「测试失败」的结果更早呈现：原来需要等到「请求」成功，现在只需等到异步方法回调即可（无论回调成功或错误——译者）。换言之，一旦app收到服务器的响应（无论是OK 或者error），就可以提示开发者了。在这之后，再进一步确认「请求」是否成功。

为了了解其中的工作原理，再创建一个测试。首先，把之前URL删除的「s」补回来，然后在类中添加如下测试：

```swift
// Asynchronous test: faster fail
func testCallToiTunesCompletes() {
  // given
  let url = URL(string: "https://itune.apple.com/search?media=music&entity=song&term=abba")
  // 1
  let promise = expectation(description: "Completion handler invoked")
  var statusCode: Int?
  var responseError: Error?
  
  // when
  let dataTask = sessionUnderTest.dataTask(with: url!) { data, response, error in
    statusCode = (response as? HTTPURLResponse)?.statusCode
    responseError = error
    // 2
    promise.fulfill()
  }
  dataTask.resume()
  // 3
  waitForExpectations(timeout: 5, handler: nil)
  
  // then
  XCTAssertNil(responseError)
  XCTAssertEqual(statusCode, 200)
}
```

这里的关键，就是在异步方法回调后，马上执行`promise.fulfill()`，这只耗费很少时间。如果「请求」失败，`then`中的assertions（断言）会抛出失败。

再跑一次测试，现在就会马上显示测试失败了，这是因为「请求（request）」失败了，而不是因为`timeout`超时导致失败。

修复`url`，重新跑测试，确认现在测试能通过。



# Faking Objects and Interactions

异步测试给了你信心——你的代码会生成正确的输入（input）给异步的API（比如AlamoFire——译者）。你可能还需要测试当接收到`URLSession的`输入时，你的代码是否可以正确工作，又或者当`UserDefaults`、CloudKit更新时，是否还能正常工作。 

很多apps和系统（system）或者库（library）的对象交互（interact）——这些对象是不在你掌控之下的——要测试这些交互会很慢而且不可重复（unrepeatable），这违反了FIRST的两条原则（第一、第三条——译者）。为了避免此类问题，可以伪造交互获得输入（input）——通过从**stubs**，或者更新**mock**对象。（就是喂假数据——译者）

当你的代码依赖到系统或库对象，就可以用这种伪造的方式——创建一个假对象喂入数据来进行这一部分的测试。[Dependency Injection by Jon Reid](https://www.objc.io/issues/15-testing/dependency-injection/) 中描述了几种可行的方法。

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/fake-433x320.png)



## 来自Stub的假数据

接下来的测试，会检查`updateSearchResults(_:)`方法是否正确地解析了下载到的数据，检查`searchResults.count`是否正确。SUT对象是view controller，这里会利用stubs和一些预先准备好的数据伪造session。

从**+**菜单中选择**New Unit Test Target…**，命名为**HalfTunesFakeTests**。在`import`语句下面，导入HalfTunes app：

```swift
@testable import HalfTunes
```

声明SUT对象，并在`setup()`中创建、在`tearDown()`中释放：

```swift
var controllerUnderTest: SearchViewController!

override func setUp() {
  super.setUp()
  controllerUnderTest = UIStoryboard(name: "Main", 
      bundle: nil).instantiateInitialViewController() as! SearchViewController!
}

override func tearDown() {
  controllerUnderTest = nil
  super.tearDown()
}
```

> Note：这里的SUT是view controller，因为HalfTunes项目有一个很大的问题——现在所有事情都在SearchViewController.swift完成。在[Moving the networking code into separate modults](http://williamboles.me/networking-with-nsoperation-as-your-wingman/) 中会解决这个问题，并且让测试工作得到简化。（应该是说要将网络请求这部分功能解耦出来——译者）

接下来，伪造的session需要一些简单的JSON数据，以喂给测试。因为只需要几组数据，所以在URL字符串后面拼接上`&limit=3`来进行限制：

```swift
https://itunes.apple.com/search?media=music&entity=song&term=abba&limit=3
```

将这个URL复制粘贴到浏览器中。会下载到一个名为**1.tx**t或类似的文件。打开确认这是一个JSON文件，然后重命名为**abbaData.json**，最后把它拖到**HalfTunesFakeTests**组中。

Supporting Files中已经有一个叫做**DHURLSessionMock.swift**的文件。这个文件定义了一个简单的协议`DHURLSession`，里面有方法（stubs）可以创建一个基于`URL`或者`URLRequest`的data task。也定义了遵守该协议的`URLSessionMock`类，可以让你基于选择的数据、response和error创建一个mock 类型的 `URLSesison`对象。

接下来设置假资料和response，并在`setup()`中创建伪造的session对象（在创建STU下面）：

```swift
let testBundle = Bundle(for: type(of: self))
let path = testBundle.path(forResource: "abbaData", ofType: "json")
let data = try? Data(contentsOf: URL(fileURLWithPath: path!), options: .alwaysMapped)

let url = URL(string: "https://itunes.apple.com/search?media=music&entity=song&term=abba")
let urlResponse = HTTPURLResponse(url: url!, statusCode: 200, httpVersion: nil, headerFields: nil)

let sessionMock = URLSessionMock(data: data, response: urlResponse, error: nil)
```

在`setup()`方法的最后，把伪造的session当作SUT的属性注入（inject）到app中：

```swift
controllerUnderTest.defaultSession = sessionMock
```

> Note：在测试中会直接使用伪造的session，这里只是展示如何注入，后续就可以调用SUT方法，使用view controller的`defalutSession`属性。



现在就可以写测试确认`updateSearchResult(_:)`方法是否能正确解析假数据。用以下代码替换`testExample()`：

```swift
// Fake URLSession with DHURLSession protocol and stubs
func test_UpdateSearchResults_ParsesData() {
  // given
  let promise = expectation(description: "Status code: 200")
  
  // when
  XCTAssertEqual(controllerUnderTest?.searchResults.count, 0, "searchResults should be empty before the data task runs")
  let url = URL(string: "https://itunes.apple.com/search?media=music&entity=song&term=abba")
  let dataTask = controllerUnderTest?.defaultSession.dataTask(with: url!) {
    data, response, error in
    // if HTTP request is successful, call updateSearchResults(_:) which parses the response data into Tracks
    if let error = error {
      print(error.localizedDescription)
    } else if let httpResponse = response as? HTTPURLResponse {
      if httpResponse.statusCode == 200 {
        promise.fulfill()
        self.controllerUnderTest?.updateSearchResults(data)
      }
    }
  }
  dataTask?.resume()
  waitForExpectations(timeout: 5, handler: nil)
  
  // then
  XCTAssertEqual(controllerUnderTest?.searchResults.count, 3, "Didn't parse 3 items from fake response")
}
```

这里还是必须写成异步测试，因为stub是假设为异步方法的。

在*when*的断言是「searchResults should be empty before the data task runs」——这是很明显的，因为我们在`setup()`中创建的是一个全新的SUT。

假数据包含了三个`Track`对象的JSON数据，所以*then*的断言是「the view controller’s `searchResults` array contains three items」。

测试跑起来。应该很快就跑完了，因为这不是真正和服务器交互。



##  Fake Update to Mock Object

上面的测试，利用**stub**从假对象中提供假资料（input）。接下来，会用**mock**对象测试你的代码是否能正确更新`UserDefaults`。

重新打开**BullsEye**项目。这个app有两种游戏模式：使用者移动slider接近目标值，或者通过slider的位置猜测目标值。右下角的segmented control用于切换游戏模式，并且更新`gameStyle`这个使用者默认选项。

下一个测试就是检查app是否正确更新了`gameStyle`这个默认值。

在test navigator，点击**New Unit Test Target…**，命名为**BullsEyeMockTests**，在`import`后添加如下代码：

```swift
@testable import BullsEye

class MockUserDefaults: UserDefaults {
  var gameStyleChanged = 0
  override func set(_ value: Int, forKey defaultName: String) {
    if defaultName == "gameStyle" {
      gameStyleChanged += 1
    }
  }
}
```

`MockUserDefaults`重写了`set(_:forKey)`方法，记录了`gameStyleChanged`更改次数。在类似的测试中也会设置一个`Bool`变量，不过这里用一个`Int`记录次数更具弹性——比如，测试可以精确地记录方法的每次调用。

在`BullsEyeMockTests`中声明SUT和mock：

```swift
var controllerUnderTest: ViewController!
var mockUserDefaults: MockUserDefaults!
```

 在`setup()`方法中，创建一个SUT和mock对象，然后注入mock对象——作为SUT的属性：

```swift
controllerUnderTest = UIStoryboard(name: "Main", bundle: nil).instantiateInitialViewController() as! ViewController!
mockUserDefaults = MockUserDefaults(suiteName: "testing")!
controllerUnderTest.defaults = mockUserDefaults
```

在`tearDown()`中释放SUT和mock对象：

```swift
controllerUnderTest = nil
mockUserDefaults = nil
```

用如下代码替换`testExample()`：

```swift
// Mock to test interaction with UserDefaults
func testGameStyleCanBeChanged() {
  // given
  let segmentedControl = UISegmentedControl()
  
  // when
  XCTAssertEqual(mockUserDefaults.gameStyleChanged, 0, "gameStyleChanged should be 0 before sendActions")
  segmentedControl.addTarget(controllerUnderTest, 
      action: #selector(ViewController.chooseGameStyle(_:)), for: .valueChanged)
  segmentedControl.sendActions(for: .valueChanged)
  
  // then
  XCTAssertEqual(mockUserDefaults.gameStyleChanged, 1, "gameStyle user default wasn't changed")
}
```

*When*的assertion（断言）是「gameStyleChanged should be 0 before sendActions」，因此在「点击」segmented control之前，`gameStyleChanged`是0。所以如果*then* assertion（断言）还是true的话，表示 `set(_:forKey:)` 方法只被调用了一次。

测试跑起来；正常来说是没问题的。



#  UI Testing in Xcode

Xcode 7开始有了UI 测试，可以创建一个「UI 测试」记录和UI的交互。「UI测试」的工作原理——查询app的UI对象、合成事件，然后将他们发送到这些对象。这个API允许开发者仔细检查UI对象的属性、状态，以便将他们与预期状态进行比较。

在**BullsEye**项目的test navigator，添加一个新的**UI Test Target**。检查确认**Target to be Tested**选择的是**BullsEye**，然后用默认的名称**BullsEyeUITests**。

在`BullsEyeUITests`类的顶部添加一个属性：

```swift
var app: XCUIApplication!
```

在`setup()`中，用如下代码替换`XCUIApplication().launch()`：

```swift
app = XCUIApplication()
app.launch()
```

把`testExample()`的名字改为`testGameStyleSwitch()`。

在`testGameStyleSwitch()`中另起一行，然后点击deitor窗口底部的红色**Record**按钮：

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/UITest.png)

当app出现在模拟器后，点击游戏模式切换开关的**Slider** segment，还有顶部的label。然后点击Xcode Record按钮停止记录。

现在`testGameStyleSwitch()`中就会有如下三行代码：（根据游戏模式的不同，Text文本内容有所不同——译者）

```swift
let app = XCUIApplication()
app.buttons["Slide"].tap()
app.staticTexts["Get as close as you can to: "].tap()
```

如果还出现了其他代码，把其他代码删掉。

第一行复制了在`setup()`中创建的属性，后面不需要点击任何东西了，所以删除第一行，还有第二行、第三行后面的`.tap()`。

打开`["Slide"]`右边的一个小下拉菜单，选择`segmentedControls.buttons["Slide"]`。

现在的代码变成这样：

```swift
app.segmentedControls.buttons["Slide"]
app.staticTexts["Get as close as you can to: "]
```

修改一下，创建一个**given**，如下：

```swift
// given
let slideButton = app.segmentedControls.buttons["Slide"]
let typeButton = app.segmentedControls.buttons["Type"]
let slideLabel = app.staticTexts["Get as close as you can to: "]
let typeLabel = app.staticTexts["Guess where the slider is: "]
```

现在两个按钮和两个顶部labels都有名称了，继续添加如下代码：

```swift
// then
if slideButton.isSelected {
  XCTAssertTrue(slideLabel.exists)
  XCTAssertFalse(typeLabel.exists)
  
  typeButton.tap()
  XCTAssertTrue(typeLabel.exists)
  XCTAssertFalse(slideLabel.exists)
} else if typeButton.isSelected {
  XCTAssertTrue(typeLabel.exists)
  XCTAssertFalse(slideLabel.exists)
  
  slideButton.tap()
  XCTAssertTrue(slideLabel.exists)
  XCTAssertFalse(typeLabel.exists)
}
```

这个测试，检查点击、选择了不同按钮之后，label是否正确存在（显示）。把测试跑起来，应该可以看到所有断言（assertions）都成功了。



#  性能测试

[苹果官方文档]()是这样定义的：*性能测试，会将需要测试的代码块运行十次，收集平均执行时间和运行的标准偏差（standard deviation for the runs）。有了这个平均值，就可以以此值为基准，进行性能评估。*

写性能测试很简单：只需要把需要测试的代码放到`measure()`方法的闭包（closure）中。

重新打开**HalfTunes**项目，在**HalfTunesFakeTests**，`testPerformanceExample()`用下面代码代替：

```swift
// Performance 
func test_StartDownload_Performance() {
  let track = Track(name: "Waterloo", artist: "ABBA", 
      previewUrl: "http://a821.phobos.apple.com/us/r30/Music/d7/ba/ce/mzm.vsyjlsff.aac.p.m4a")
  measure {
    self.controllerUnderTest?.startDownload(track)
  }
}
```

跑起来，然后点击出现在`measure()`闭包尾部的图标，查看统计信息。

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/PerformanceResult-650x228.png)

点击**Set BaseLine**，再次执行performance test——结果可能比baseline更好或者更差。**Edit**按钮可以将最新的值重设为baseline。

每台设备的configuration都保存了baseline相关信息，因此可以在不同的设备上执行相同的测试，不同设备的处理器速度、内存等各不相同，它们会维护不同的baseline。

App的每次修改，都有可能影响到性能，可以再次运行性能测试，和baseline比较一下。



# Code Coverage

Code coverage工具，可以帮忙检查哪些代码已经跑过测试，哪些代码还没测试。

> Note：当code coverage打开时，是否应该跑性能测试？[苹果官方文档](https://developer.apple.com/library/prerelease/content/documentation/DeveloperTools/Conceptual/testing_with_xcode/chapters/07-code_coverage.html#//apple_ref/doc/uid/TP40014132-CH15-SW1)时这样说的：Code coverage 数据收集会导致性能的损耗……以线性方式影响代码的执行，因此code coverage启用时，对性能影响还是可以接受的（ performance results remain comparable from test run to test run when it is enabled）。但是，当你需要精确评估性能的时候，应该考虑是否在测试中启用code coveage。

要启用code coverage，编辑scheme的**Test**，并勾选**Code Coverage**复选框（Xcode 9 是在**Options**中勾选——译者）：

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/CodeCoverageSwitch.png)

把所有测试都跑起来（Command-U），然后打开reports navigator（Command-8）。选择**By Time**，选中列表中最上面一个，再选择**Coverage**这个tab（Xcode 9 点击左边的**{} Coverage**）：

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/CoverageReport1-650x189.png)

点击**SearchViewController.swift**左边的三角形，查看方法列表：

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/CoverageReport2-650x252.png)

将鼠标悬停在`updateSearchResults(_:)`方法旁的蓝色Coverage bar上，可以看到覆盖率是71.88%。

点击方法右边的箭头按钮，打开这个方法的源文件，找到这个方法。鼠标悬停在右侧边栏的coverage annotations，这部分代码就会高亮成绿色或者红色。

![](https://raw.githubusercontent.com/Antony138/MarkdownPhotos/master/photos/2018articlesPhotos/iOSTestTutorial/CoverageReport4-650x436.png)

coverage annotations还显示了每部分代码在一次测试中的执行次数；没有被执行的部分高亮为红色。如你所愿，for循环跑了3次，而错误的分支，没有被执行。如果要提高这个方法的覆盖率，可以复制一份**abbaData.json**，修改其中的内容，就可以导致不同的错误——比如，将把key `"results"`改为`"result"`，跑测试的时候，就会执行`print("Results key not found in dictionary")`这个分支。

## 100% Coverage?

应该追求100%的代码覆盖率吗？搜索一下「100% unit test coverage」，网上有一大波争论和相反意见，以及关于「100% unit test coverage」定义本身的争论。反对派说最后的10-15%是不值得去测试的。赞成派认为正因为这部分难于测试，所以最后的10-15%是非常重要的。搜索一下「hard to unit test bad design」，可以找到有说服力的论据—— [untestable code is a sign of deeper design problems](https://www.toptal.com/qa/how-to-write-testable-code-and-why-it-matters)——难以（不能）测试的代码，往往意味着这个设计本身是有问题的。如果再深究的话，将会延伸到[Test Driven Development](http://qualitycoding.org/tdd-sample-archives/)（测试驱动开发）。



# Where to Go From Here

到此为止，我们可以利用很多有用的工具为项目进行测试了。希望看完这个关于iOS Unit Testing 和 UI Testing 的教程后，你可以胸有成竹地去测试所有东西。

这里是已经完成的[项目](https://koenig-media.raywenderlich.com/uploads/2016/12/Finished-3.zip)。

下面是一些补充教程：

- 现在你可以为项目写测试了，那么下一步当然就是*自动化*了：**Continuous Integration**（持续集成） 和 **Continuous Delivery**（持续交付）。可以从Apple Xcode Server和xcodebulid的[Automating the Test Process](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/testing_with_xcode/chapters/08-automation.html#//apple_ref/doc/uid/TP40014132-CH7-SW1)开始了解，还有[Wikipedia’s continuous delivery article](https://en.wikipedia.org/wiki/Continuous_delivery)这篇文章，借鉴了[ThoughtWorks](https://www.thoughtworks.com/continuous-delivery)一文。

- [TDD in Swift Playgrounds](http://initwithstyle.net/2015/11/tdd-in-swift-playgrounds/) 使用了`XCTestObservationCenter`来在Playgrounds中跑XCTestCase单元测试。这样可以在Playgrounds上开发和测试，然后再转到app中。

-  [CMD+U Conference](http://www.cmduconf.com/) 中的文章[Watch Apps: How Do We Test Them?](https://realm.io/news/cmduconf-boris-bugling-how-test-watch-apps/) ，演示了如何用 [PivotalCoreKit](https://github.com/pivotal/PivotalCoreKit)来测试watchOS app。

- 如果已经写好了app，但还没有写测试，可以参考 [*Working Effectively with Legacy Code* by Michael Feathers](https://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052/ref=sr_1_1?s=books&ie=UTF8&qid=1481511568&sr=1-1)一文，记住，没有通过测试的代码，不是好代码。

- Jon Reid的Quality Coding app archives，是一个学习更多关于 [Test Driven Development](http://qualitycoding.org/tdd-sample-archives/)的好地方。

如果有关于这个教程的任何疑问或建议，请加入[论坛]( https://www.raywenderlich.com)在下面进行讨论。