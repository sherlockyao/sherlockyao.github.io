---
layout: post
title: "响应式编程入门（下）"
date: 2016-02-16 16:00
comments: true
categories: [iOS, Swift, Reactive, ReactiveX, RxSwift, 响应式编程, 函数式编程, MVVM]
---

书接上文，前面我们简单地介绍了响应式编程（Reactive Programming）的一些基本思想，接下来我会通过实际例子，一步步演示如何将RP应用到实际的开发中去。因为[RxSwift](https://github.com/ReactiveX/RxSwift)本身已经提供了非常好的Sample代码，所以我打算这里就通过复刻其中的GitHubSignup项目，来作为本次演示的主题。首先让我们来看看需要达到的效果：

![File Inspector]({{ root_url }} /images/20160216/signup.png")

以下是我们将实现的主要功能（我这里省掉了提交表单的逻辑，其实现方法同其他的大同小异）：

- 当用户在输入用户名时，实时去验证用户名是否可用，同时显示对应的验证结果
- 当用户在输入密码时，要验证是否大于3位，同时显示对应的验证结果
- 当用户重复密码时，要验证是否和第一次输入一致，同时显示对应的验证结果
- 当且仅当三个输入区域都验证通过时，提交按钮才可被点击，否则将禁用

因为密码的验证不需要网络请求，完全在本地完成，所以我们先从这个比较简单的入手。验证过程是在每次用户键入密码时触发的，所以我们可以把它看作是一个输入流，用图形表示就是类似这样：

```
--t----t---t-----t------>

t表示的就是一次键盘输入事件
```

不过我们不光需要知道输入事件，还需要知道输入事件产生时输入框内的文本内容，我们希望有一个这样的流：

```
--t----t---t-----t------>
  v    v   v     v
-(1)-(12)-(123)-(1234)-->
```

大家马上会想到我们可以通过一个简单的map来搞定，不过RxSwift是非常贴心的，这么常用的功能它们已经提供了封装，我们可以直接拿来用：

```swift

class ViewController: UIViewController {

    @IBOutlet weak var passwordTextField: UITextField!
    
    let password = Variable("")
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        passwordTextField.rx_text <-> password 
    }
}

infix operator <-> {
}

func <-> <T>(property: ControlProperty<T>, variable: Variable<T>) -> Disposable {
    let bindToUIDisposable = variable
        .bindTo(property)
    let bindToVariable = property
        .subscribe(onNext: { n in
            variable.value = n
            }, onCompleted:  {
                bindToUIDisposable.dispose()
        })
    
    return StableCompositeDisposable.create(bindToUIDisposable, bindToVariable)
}

```

在上面的代码中，我们申明了一个password的Variable，为的是把UI中的值绑定到它上面，方便后面的使用，所使用的```<->```绑定操作符是自定义的，可以在底部的方法中看到。可以看到，到目前为止我们其实就用了一行代码生成了需要的数据流，接下来就是验证数据，转化成我们真正需要的结果流了：

```
--t----t---t-----t------>
  v    v   v     v
-(1)-(12)-(123)-(1234)-->
  v    v   v     v
--F----F---F-----T------>
```

其实现的代码如下：

```swift

let passwordValidation = password.map { (password) -> (valid: Bool?, message: String?) in
    let numberOfCharacters = password.characters.count
    if numberOfCharacters == 0 {
        return (false, nil)
    }
    if numberOfCharacters < 4 {
        return (false, "Password must be at least 4 characters")
    }
    return (true, "Password acceptable")
}
.shareReplay(1)

```

我们的验证逻辑很简单，看一下输入字符的长度，如果大于3位则通过。最后的`shareReplay`是RxSwift提供的一个流操作函数，作用是为了保证在观察者订阅这个流的时候始终都能回播最后一个（数字1表示回播的数量）流中的值，这样能使界面上正确显示验证的状态。现在我们有了需要的流，接下来就是创建一个观察者来订阅这个流，从而能把结果显示在界面上反馈给用户，直接看代码：

```swift

func bindValidationResultToUI(source: Observable<(valid: Bool?, message: String?)>, validationErrorLabel: UILabel) {
	  source
	      .subscribeNext { v in
	          let validationColor: UIColor
	          
	          if let valid = v.valid {
	              validationColor = valid ? UIColor.greenColor() : UIColor.redColor()
	          } else {
	              validationColor = UIColor.grayColor()
	          }
	          
	          validationErrorLabel.textColor = validationColor
	          validationErrorLabel.text = v.message ?? ""
	      }
	      .addDisposableTo(disposeBag)
}

override func viewDidLoad() {
		
		// ...

    bindValidationResultToUI(
            passwordValidation,
            validationErrorLabel: self.passwordValidationLabel
        ) 
}

```

我们定义了一个帮助方法，它会订阅给定的验证流，并把结果显示在指定的Label上，怎么样是不是感觉非常简单，剩下要做的就是把刚才我们创建的密码验证流绑定到需要的标签上即可。依样画葫芦，接下来我们来创建重复密码的验证：

```swift

let repeatPasswordValidation = combineLatest(password, repeatedPassword) { (password, repeatedPassword) -> (valid: Bool?, message: String?) in
    if repeatedPassword.characters.count == 0 {
        return (false, nil)
    }
    if repeatedPassword == password {
        return (true, "Password repeated")
    }
    else {
        return (false, "Password different")
    }
}
.shareReplay(1)

```

这里我只贴出了核心的创建流的代码部分，其余的与验证密码都是雷同的。从上面的代码中可以看到，我们使用了combineLatest的函数来合并两个流，它的作用如下图：

```
X: --1----2------------3--4--|->

Y: ----A-----B---C--D--------|->
  
combineLatest(X, Y)

   ----1A-2A-2B--2C-2D-3D-4D-|->
```

可见，它正好试用我们验证密码的场景，接下来就是简单的验证逻辑，如此我们便实现了重复密码的验证过程。现在让我们来看一下稍微复杂一点的逻辑，验证用户名：

```swift

let usernameValidation = username
    .map { [unowned self] username -> Observable<(valid: Bool?, message: String?)> in
        if username.characters.count == 0 {
            return just((false, nil))
        }
        
        if username.rangeOfCharacterFromSet(NSCharacterSet.alphanumericCharacterSet().invertedSet) != nil {
            return just((false, "Username can only contain numbers or digits"))
        }
        
        let loadingValue = (valid: nil as Bool?, message: "Checking availabilty ..." as String?)
        
        return self.API.usernameAvailable(username)
            .map { available in
                if available {
                    return (true, "Username available")
                }
                else {
                    return (false, "Username already taken")
                }
            }
            .startWith(loadingValue)
    }
    .switchLatest()
    .shareReplay(1)

```

我们来具体分析一下整个过程，首先用户名的验证分为两个部分，一个本地合法性验证，另一个是远程的可用性验证，因为服务器验证涉及到异步，所以其验证结果本身就是一个流（拥有初始状态，结果状态）。然后我们来想象一下，当用户在输入用户名的时候，每当用户键入一个值，我们都需要去验证，但因为服务器验证是异步的，所以当用户键入下一字母的时候，前面的验证结果其实是没有用了，那我们应该怎么处理呢？switchLatest函数是我们的救星，下面是其官方文档的示意图：

![File Inspector]({{ root_url }} /images/20160216/switchlatest.png")

简言之，switchLatest所操作的流其原本的每个值都是一个单独的流，经过处理后，它变成了一个单一的流，其中的每个值都是原本流当中最新流中的最新值。这就完美地解决了我们的问题，每当新的验证触发后，前一个验证流的值虽然还会产生，但是我们已经不去关心它了，从而不会对我们的结果产生影响。最后让我们来处理提交按钮的禁用逻辑，当我们用流的思路去考虑问题时，将发现这个需求简直太简单了：

```swift

let signupEnabled = combineLatest(
    usernameValidation,
    passwordValidation,
    repeatPasswordValidation
    ) { username, password, repeatPassword in
        return (username.valid ?? false) &&
            (password.valid ?? false) &&
            (repeatPassword.valid ?? false)
}

signupEnabled
    .subscribeNext { [unowned self] valid  in
        self.submitButton.enabled = valid
        self.submitButton.alpha = valid ? 1.0 : 0.5
    }
    .addDisposableTo(disposeBag)

```

相信大家现在一看这个代码就已经知道它的意思了，所以就不再赘述了。响应式编程提供给了我们一种完全不同的思维方式，即使不去实际应用，去学习它本身就是一个很有意义的事情，况且在交互越来越复杂的时代，它更是提供了一种清晰的解决方案。我自己还谈不上入门，但已跃跃欲试要好好专研一番，希望此文能给想入门的同道们一些帮助。

另附上[示例代码地址](https://github.com/sherlockyao/RxSwiftPractise)