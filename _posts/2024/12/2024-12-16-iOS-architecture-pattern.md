---

title: iOS架构模式

date: 2024-12-16

tags: ["iOS", "Swift", "架构"] 

categories: ["iOS"] 

comments: true

---

**架构模式**是指在软件开发中，为了解决特定类型的问题而提炼出的**可复用**的、经过**验证**的设计方案。它描述了一种系统组件的**组织**方式，以及它们之间的交互关系，用于**指导**开发者如何构建高效、可靠、可维护的系统。

架构模式可以被具体化为代码结构和开发流程。通过遵循架构模式，可以减少开发过程中的不确定性，提高团队协作效率，并确保代码质量。选择合适的架构模式能够大幅提升开发效率，特别是在团队协作和代码可维护性方面。合适的架构模式不仅能帮助开发人员更高效地完成任务，也能减少后期修改和扩展的难度。

iOS 开发架构随着技术和业务需求的不断发展，从简单到复杂，经历了多个阶段的演化。以下是iOS中的各种架构模式：

## MVC

MVC（Model-View-Controller）是苹果官方推崇的设计模式，尤其是在 iOS 和 macOS 开发中。苹果在开发文档中广泛采用这种模式，特别是在 UIKit 的设计中。

### 定义

#### Model（模型）

- 表示应用程序的数据结构和业务逻辑。
- 不关心如何显示数据。
- 负责获取、存储数据，以及处理相关的业务逻辑。

#### View（视图）

- 负责显示数据。
- 与用户交互，接收用户输入并将其传递给控制器。
- 视图不包含业务逻辑，它只是展示数据和与用户交互。

#### Controller（控制器）

- 充当视图和模型之间的中介。
- 控制器从模型中获取数据，更新视图。
- 控制器响应用户的交互事件，并调用模型或更新视图。

下面是一个简单的例子：

#### Sample Code

```swift
struct UserInfo {
    let name:String
    let avatar:String
}

 class UserAPI {
        func login(userName:String, password:String, completion: @escaping (UserInfo) -> Void, fail: @escaping (String) -> Void) {
            // 模拟网络请求
            DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
                DispatchQueue.main.async{
                    completion(UserInfo(name: userName, avatar: "avatar"))
                }
            }
        }
    }

class LoginViewController: UIViewController {
    // 网络请求API
    private let userApi = UserAPI()
    
    private let userNameTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入用户名"
    }
    
    private let passwordTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入密码"
        $0.isSecureTextEntry = true
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.
        
        self.view.backgroundColor = .systemBackground
        // UI布局 
        self.view.addSubview(userNameTextField)
        userNameTextField.snp.makeConstraints { make in
            make.top.equalToSuperview().offset(100)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        
        self.view.addSubview(passwordTextField)
        passwordTextField.snp.makeConstraints { make in
            make.top.equalTo(userNameTextField.snp.bottom).offset(30)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        
        let loginButton = UIButton(type: .system).then {
            $0.setTitle("登录", for: .normal)
        }
        self.view.addSubview(loginButton)
        loginButton.snp.makeConstraints { make in
            make.top.equalTo(passwordTextField.snp.bottom).offset(16)
            make.left.right.equalToSuperview().inset(80)
            make.height.equalTo(44)
        }
        // 绑定按钮事件
        loginButton.addTarget(self, action: #selector(login(_:)), for: .touchUpInside)
    
    }
    
    // 登录按钮点击事件
    @objc func login(_ sender: UIButton) {
            
        guard let userName = userNameTextField.text, !userName.isEmpty else {
            self.view.makeToast("请输入用户名")
            return
        }
        
        guard let password = passwordTextField.text, !password.isEmpty else {
            self.view.makeToast("请输入密码")
            return
        }
        
        userApi.login(userName: userName, password: password) { [weak self]  userInfo in
            //
        } fail: { [weak self]  msg in
            self?.view.makeToast(msg)
        }
        
    }
}
```
  
### 小结

#### 特点

- Model：负责业务逻辑和数据处理。
- View：负责界面展示和用户交互。
- Controller：作为桥梁，负责协调 Model 和 View。

#### 优点

- 将业务逻辑和用户界面分离，便于维护和测试。
- 分离关注点，代码结构清晰
- UIKit 的设计天然符合 MVC 模式
  
#### 缺点

- 随着业务复杂度增加，ViewController 变得臃肿，难以维护（Massive ViewController 问题）。
- 模块间耦合度较高，单元测试困难

从这段例子中可以看出Controller承载了很多的任务，包括数据获取、数据处理、界面更新等，现在看不太复杂，但是如果业务在复杂点，会导致ViewController变得臃肿，且耦合度较高，这会导致开发效率降低，并且代码可维护性降低。对Controller进行瘦身，剥离一些业务代码。可以使用MVVM来优化。

## MVVM

单从字面上看，少了个C，多了VM。VM是View Model的缩写，把MVC中C的业务逻辑处理抽出为View Model，把Controller归类为View，将View和Model解耦，让View和Model之间通过View Model进行交互，从而实现Massive ViewController的瘦身，各个模块职责更分明。
数据流向

```
View ↔ ViewModel ↔ Model
```

为了减少MVVM代码的复杂度，可以使用RxSwift，RxSwift是一个用于处理异步操作和响应式编程的库，它提供了一种更简洁的方式来处理异步操作和响应式编程，可以轻松地实现MVVM架构。
下面是一段简单的例子：

### Sample Code

```swift
class LoginViewModel {
        
    private let bag = DisposeBag()
    
    private let userApi = UserAPI()
    
    let userNameBehavior = BehaviorRelay<String>(value: "")
    let passwordBehavior = BehaviorRelay<String>(value: "")
    let toastSubject = PublishSubject<String>()
    let loginSuccess = PublishSubject<Void>()
    
    func login() {
        
        guard !userNameBehavior.value.isEmpty else {
            toastSubject.onNext("请输入用户名")
            return
        }
        
        guard !passwordBehavior.value.isEmpty else {
            toastSubject.onNext("请输入密码")
            return
        }
        
        userApi.login(userName: userNameBehavior.value, password: passwordBehavior.value) { [weak self]  info, error in
            if error == nil {
                self?.loginSuccess.onNext(())
            } else {
                self?.toastSubject.onNext(error!.localizedDescription)
            }
        }
    }
}

class LoginViewController: UIViewController {

    private let bag = DisposeBag()
    
    private let viewModel = LoginViewModel()
    
    private let userNameTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入用户名"
    }
    
    private let passwordTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入密码"
        $0.isSecureTextEntry = true
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.
        
        self.view.backgroundColor = .systemBackground
        
        self.view.addSubview(userNameTextField)
        userNameTextField.snp.makeConstraints { make in
            make.top.equalToSuperview().offset(100)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        userNameTextField.rx.text.orEmpty.throttle(.milliseconds(300), scheduler: MainScheduler.instance).bind(to: viewModel.userNameBehavior).disposed(by: bag)
        
        self.view.addSubview(passwordTextField)
        passwordTextField.snp.makeConstraints { make in
            make.top.equalTo(userNameTextField.snp.bottom).offset(30)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        passwordTextField.rx.text.orEmpty.throttle(.milliseconds(300), scheduler: MainScheduler.instance).bind(to: viewModel.passwordBehavior).disposed(by: bag)
        
        let loginButton = UIButton(type: .system).then {
            $0.setTitle("登录", for: .normal)
        }
        loginButton.rx.tap.throttle(RxTimeInterval.milliseconds(300), scheduler: MainScheduler.instance).subscribe(onNext:{ [weak self] in
            self?.viewModel.login()
        }).disposed(by: bag)
        self.view.addSubview(loginButton)
        loginButton.snp.makeConstraints { make in
            make.top.equalTo(passwordTextField.snp.bottom).offset(16)
            make.left.right.equalToSuperview().inset(80)
            make.height.equalTo(44)
        }
        
        
        viewModel.loginSuccess.subscribe(onNext:{ [weak self] in
            // 成功回调
        }).disposed(by: bag)
        
        viewModel.toastSubject.observe(on: MainScheduler.instance).subscribe(onNext:{ [weak self] message in
            self?.view.makeToast(message)
        }).disposed(by: bag)
    }
    
}
```

在代码中借助于RxSwift，实现了收据的双向绑定，textFiled值变化会同步到ViewModel中，viewModel中值的变化也会同步到view中。
viewModel暴露的方法，在Controller中调用，viewModel内部处理后，数据变化同步更新UI。这个流程减少了Controller中过多的业务逻辑代码，只需要关注UI和用户触摸事件。相比于MVC，MVVM各个模块功能更单一，职责更明确。

### 小结

#### 特点

- Model：数据和业务逻辑。
- View：界面和用户交互。
- ViewModel：负责处理 View 的状态逻辑，将 Model 转换为 View 可用的数据。

#### 优点

- 减少 ViewController 的职责，提升代码复用性。
- 提供更好的单元测试支持。
- 数据绑定（如 RxSwift 或 Combine）能让 UI 和数据同步更高效。

#### 缺点

- 学习曲线较高，特别是对数据绑定框架不熟悉的开发者。
- 数据绑定可能增加调试难度。
  
在MVVM的例子中，虽然明确了模块的职责，但是是松散的方法和属性绑定，需要维护多个状态，可能导致 ViewModel 过于庞大，且双向绑定可能会造成数据更新时，源头不明确，增大调试难度。
既然双向绑定增加了不可控的因素，那么可以尝试考虑使用单向数据流，减少ViewModel和Model之间的数据传递状态维护。这种架构模式被称为

## MVI

把View中交互、数据的请求统称为Intent(意图)，添加State(状态)的概念，来定义业务所有的状态， 通过Intent更新State，继而更新View。

```
Intent (User Actions) → ViewModel → State → View
```

老规矩，上代码

### Sample Code

```swift
// MARK: state
struct LoginState {
    var username: String = ""
    var password: String = ""
    var isLoading: Bool = false
    var errorMessage: String?
    var isLoginSuccessful: Bool = false
}

// MARK: Intent
enum LoginIntent {
    case updateUsername(String)
    case updatePassword(String)
    case login
}

// MARK: ViewModel
class LoginViewModel {
    
    private let bag = DisposeBag()
    
    private let userApi = UserAPI()
    
    // Input: 接受 View 的意图
    let intent = PublishRelay<LoginIntent>()
    
    // Output: 暴露给 View 的状态
    private let stateRelay = BehaviorRelay<LoginState>(value: LoginState())
    var state: Observable<LoginState> { stateRelay.asObservable() }
    
    private let disposeBag = DisposeBag()
    
    init() {
        intent.subscribe(onNext: { [weak self] intent in
                    guard let self = self else { return }
                    var newState = self.stateRelay.value
                    switch intent {
                    case .updateUsername(let username):
                        newState.username = username
                    case .updatePassword(let password):
                        newState.password = password
                    case .login:
                        self.login(currentState: newState)
                        return
                    }
                    self.stateRelay.accept(newState)
                }).disposed(by: disposeBag)
    }
    
    func login(currentState: LoginState) {
        
        var newState = currentState
        
        guard !newState.username.isEmpty else {
            newState.errorMessage = "请输入用户名"
            stateRelay.accept(newState)
            return
        }
        
        guard !newState.password.isEmpty else {
            newState.errorMessage = "请输入密码"
            stateRelay.accept(newState)
            return
        }
        
        newState.isLoading = true
        stateRelay.accept(newState)
        
        // 模拟网络请求
        let resultState = currentState
        userApi.login(userName: currentState.username, password: currentState.password) { [weak self]  info, error in
            var state = resultState
            if error == nil {
                
                state.isLoading = false
                state.isLoginSuccessful = true
                
            } else {
                state.isLoading = false
                state.errorMessage = "登录失败"
            }
            self?.stateRelay.accept(state)
        }

    }
}

class LoginViewController: UIViewController {

    private let bag = DisposeBag()
    
    private let viewModel = LoginViewModel()
    
    private let userNameTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入用户名"
    }
    
    private let passwordTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入密码"
        $0.isSecureTextEntry = true
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.
        self.title = "MVI"
        self.view.backgroundColor = .systemBackground
        
        self.view.addSubview(userNameTextField)
        userNameTextField.snp.makeConstraints { make in
            make.top.equalToSuperview().offset(100)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        userNameTextField.rx.text.orEmpty
            .throttle(.milliseconds(300), scheduler: MainScheduler.instance).map { LoginIntent.updateUsername($0)}.bind(to: viewModel.intent)
            .disposed(by: bag)
        
        self.view.addSubview(passwordTextField)
        passwordTextField.snp.makeConstraints { make in
            make.top.equalTo(userNameTextField.snp.bottom).offset(30)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        passwordTextField.rx.text.orEmpty
            .throttle(.milliseconds(300), scheduler: MainScheduler.instance)
            .map { LoginIntent.updatePassword($0) }
            .bind(to: viewModel.intent)
            .disposed(by: bag)
        
        let loginButton = UIButton(type: .system).then {
            $0.setTitle("登录", for: .normal)
        }
        loginButton.rx.tap
            .throttle(RxTimeInterval.milliseconds(300), scheduler: MainScheduler.instance)
            .map {LoginIntent.login}
            .bind(to: viewModel.intent)
            .disposed(by: bag)
        self.view.addSubview(loginButton)
        loginButton.snp.makeConstraints { make in
            make.top.equalTo(passwordTextField.snp.bottom).offset(16)
            make.left.right.equalToSuperview().inset(80)
            make.height.equalTo(44)
        }
        
        viewModel.state.observe(on: MainScheduler.instance)
            .subscribe(onNext:{ [weak self] state in
                self?.updateUI(state)
            })
            .disposed(by: bag)
    }
    
    private func updateUI(_ state:LoginState) {
        // 更新UI
        if let msg = state.errorMessage, !msg.isEmpty {
            self.view.makeToast(msg)
        }
        
        if state.isLoading {
            SVProgressHUD.show()
        } else {
            SVProgressHUD.dismiss()
        }
        
        if state.isLoginSuccessful {
            self.view.makeToast("登录成功")
        }
    }
}
```

从代码中可以看到，把用户的交互定义为Intent（意图），ViewModel接受Intent更新State，之后暴露给View用以更新UI，保证数据的单向流动。ViewModel 承担了 **状态管理** 和 **业务逻辑** 的职责，是连接用户意图（Intent）和视图状态（State）的关键组件，在MVI中隶属于Model层，扩展了传统架构中 Model 的功能，并以一种更加声明式的方式与 View 交互。

### 对比MVVM

| 特点       | MVI                                                                                          | MVVM                                                                                             |
| ---------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| 核心思想   | 双向数据绑定或响应式流 <br>ViewModel 提供数据和逻辑，View 直接绑定到 ViewModel 的属性。      | 单向数据流更新视图。 <br>用户通过 Intent 表达操作意图，ViewModel 更新 State，State 再驱动 View。 |
| 数据流向   | 双向绑定  <br>可能导致状态不可控（例如复杂交互中，View 和 ViewModel 相互影响）。             | 单向数据流：用户意图 → 更新状态 → 渲染视 <br>全局状态明确可见，适合复杂交互和调试图。            |
| 适用场景   | 适合简单或中等复杂度的应用，例如表单数据处理。<br>如果状态不复杂，MVVM 更加直观和简洁。      | 适合复杂的交互逻辑和状态管理，例如聊天、游戏。<br>如果有多个状态切换场景，MVI 更容易维护。       |
| 状态管理   | 状态存储在ViewModel的属性中，由View和ViewModel共享。<br>数据绑定可能导致难以追踪的状态变化。 | 所有状态集中在一个不可变的 State 对象中。<br>单向数据流让状态清晰可见，便于调试和测试。          |
| 事件处理   | 通过 ViewModel 暴露的属性和方法处理用户事件。<br>View 和 ViewModel 的交互可能比较散乱。      | 通过Intent明确用户操作，所有事件通过统一入口进入 ViewModel。<br>Intent 使事件处理清晰统一。      |
| 代码复杂度 | 对简单场景更友好，学习成本低。<br>容易导致 ViewModel 过于复杂（God ViewModel）。             | 对复杂状态更友好，但初期实现会比较繁琐。<br>初期代码量较大，但状态清晰易维护。                   |
| 性能和调试 | 双向绑定在性能敏感场景下可能有额外开销。<br>调试较为困难，特别是当状态改变不可预测时。       | 单向数据流减少了状态更新的复杂性。<br>状态变化集中在单一入口，更易调试。                         |

### 小结

#### 特点

- MVI 是一种基于单向数据流的架构模式。
- 核心理念：
  - View 通过用户交互产生 Intent（用户意图）。
  - Intent 被处理为新的 State。
  - View 订阅并渲染 State。
- State 是单一数据源，View 仅依赖这个状态渲染界面。

#### 组成部分

- Model：表示数据和业务逻辑。
- View：展示 UI，根据 State 渲染。
- Intent：用户的交互行为，触发事件（如点击按钮、输入文字）。
- State：整个系统的单一状态，View 根据 State 渲染界面。
- ViewModel：根据 Intent 修改 State。
  
#### 缺点

- 学习曲线高：开发者需要熟悉响应式编程和单向数据流的概念。
- 复杂度：对于简单的界面可能显得过于复杂。
- 性能开销：每次状态更新可能会重新渲染整个视图，可能需要优化。
  
## 优化MVVM

经过对比，MVVM有不少缺点，借助于MVI的思想，可以通过一些优化来提升MVVM的性能和体验。
把单向数据流动的概念引入MVVM中，把暴露的方法转换为订阅信号，通过信号来驱动View。
实现如下：

```swift

protocol MVVMViewModelType {
    associatedtype Input
    associatedtype Output
    
    func transform(input: Input) -> Output
}

class LoginViewModel:MVVMViewModelType {
    
    struct Input {
        let userName: Observable<String>
        let password: Observable<String>
        let login: Observable<Void>
    }
    
    struct Output {
        let toast: Observable<String>
        let loginSuccess: Observable<Void>
    }
    
    func transform(input: Input) -> Output {
        
        input.userName.bind(to: userNameBehavior).disposed(by: bag)
        input.password.bind(to: passwordBehavior).disposed(by: bag)
        input.login.subscribe(onNext:{ [weak self] in
            self?.login()
        }).disposed(by: bag)
        
        return Output(toast: toastSubject, loginSuccess: loginSuccess)
        
    }

    private let bag = DisposeBag()
    
    private let userApi = UserAPI()
    
    // MARK: Input
    private let userNameBehavior = BehaviorRelay<String>(value: "")
    private let passwordBehavior = BehaviorRelay<String>(value: "")
    
    // MARK: Output
    private let toastSubject = PublishSubject<String>()
    private let loginSuccess = PublishSubject<Void>()
    
    private func login() {
        
        guard !userNameBehavior.value.isEmpty else {
            toastSubject.onNext("请输入用户名")
            return
        }
        
        guard !passwordBehavior.value.isEmpty else {
            toastSubject.onNext("请输入密码")
            return
        }
        
        userApi.login(userName: userNameBehavior.value, password: passwordBehavior.value) { [weak self]  info, error in
            if error == nil {
                self?.loginSuccess.onNext(())
            } else {
                self?.toastSubject.onNext(error!.localizedDescription)
            }
        }
    }
}

class LoginViewController: UIViewController {

    private let bag = DisposeBag()
    
    private let viewModel = LoginViewModel()
    
    private let userNameTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入用户名"
    }
    
    private let passwordTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入密码"
        $0.isSecureTextEntry = true
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.
        self.title = "MVVM"
        self.view.backgroundColor = .systemBackground
        
        self.view.addSubview(userNameTextField)
        userNameTextField.snp.makeConstraints { make in
            make.top.equalToSuperview().offset(100)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        
        self.view.addSubview(passwordTextField)
        passwordTextField.snp.makeConstraints { make in
            make.top.equalTo(userNameTextField.snp.bottom).offset(30)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        
        let loginButton = UIButton(type: .system).then {
            $0.setTitle("登录", for: .normal)
        }

        self.view.addSubview(loginButton)
        loginButton.snp.makeConstraints { make in
            make.top.equalTo(passwordTextField.snp.bottom).offset(16)
            make.left.right.equalToSuperview().inset(80)
            make.height.equalTo(44)
        }
    
        // bind model
        let output = viewModel.transform(
            input: MVVMImprovement.LoginViewModel.Input(
                userName: userNameTextField.rx.text.orEmpty.throttle(.milliseconds(300), scheduler: MainScheduler.instance).asObservable(),
                password: passwordTextField.rx.text.orEmpty.throttle(.milliseconds(300), scheduler: MainScheduler.instance).asObservable(),
                login: loginButton.rx.tap.map{()}
            )
        )
        
        output.toast.observe(on: MainScheduler.instance).subscribe(onNext:{ [weak self] message in
            self?.view.makeToast(message)
        }).disposed(by: bag)
        
        output.loginSuccess.subscribe(onNext:{ [weak self] in
            // 成功回调
        }).disposed(by: bag)
    }
    
}
```

在`MVVMViewModelType`的定义中，`associatedtype`使得协议能够定义一个占位符类型，类型延迟在实现中指定，它使得协议能够在多种不同类型之间复用，提高代码的灵活性和可维护性。
可以在ViewModel中定义Input和Output，通过`transform`方法将Input转换为Output，，实现数据的单向流动，view中监听Output属性，UI渲染颗粒度减小。
当业务逻辑有变动时，只需要修改Input、Output属性，在`transform`中添加转换即可，也规避了MVI中重新渲染整个视图的问题。

## MVP

MVP（Model-View-Presenter） 是另一种常见的软件架构模式，它与 MVVM 和 MVI 都有明显的不同。在 MVP 中，View、Model 和 Presenter 的职责划分更加明确，特别是在视图和业务逻辑的交互上。通过对比 MVVM 和 MVI，MVP 的设计和实现思路会更清晰。

### 核心思想

- Model：表示应用的数据层，负责数据的获取、存储和更新。它不关心如何显示数据，只提供数据。
- View：表示 UI 层，负责显示数据并接收用户输入，但不包含任何业务逻辑。View 直接与用户交互，但它不会处理逻辑。
- Presenter：负责从 Model 获取数据并根据用户交互更新 View。Presenter 充当 View 和 Model 之间的桥梁，处理视图的业务逻辑。View 会通过接口来通知 Presenter 用户的操作，Presenter 则根据业务逻辑更新 View。

### 数据流

MVP 强调 单向数据流，具体为：

- 用户操作触发 View，通过接口调用 Presenter。
- Presenter 处理业务逻辑，并从 Model 获取数据。
- Presenter 更新 View，呈现数据。
  
每一步都有Presenter参与，在MVP中，Presenter 是全知全能的，知道所有的细节。

### MVP 与 MVVM、MVI 的对比

| 特性                | MVP                                         | MVVM                                 | MVI                                 |
| ------------------- | ------------------------------------------- | ------------------------------------ | ----------------------------------- |
| View                | 通过接口与 `Presenter` 交互                 | 通过数据绑定与 `ViewModel` 绑定      | 显示 `State`，通过 `Intent` 更新    |
| ViewModel/Presenter | `State`	处理视图逻辑和业务逻辑，更新 `View` | 主要处理与视图绑定的数据             | 负责处理 `Intent` 和 `State`        |
| 数据流              | 单向流：用户操作 → `Presenter` → `View`     | 双向绑定（可能存在隐式的双向数据流） | 单向流：`Intent` → `State` → `View` |
| 适用场景            | 适合明确的视图和业务逻辑解耦，有良好的结构  | 适合响应式开发、UI更新频繁的应用     | 适合复杂的状态管理和多种状态切换    |

### Sample Code

```swift
protocol userService {
    func login(userName:String, password:String, completion: @escaping (UserInfo) -> Void, fail: @escaping (String) -> Void)
}

class UserAPI: userService {
    func login(userName:String, password:String, completion: @escaping (UserInfo) -> Void, fail: @escaping (String) -> Void) {
        // 模拟网络请求
        DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
            DispatchQueue.main.async{
                completion(UserInfo(name: userName, avatar: "avatar"))
            }
        }
    }
}

// View：视图层，界面与用户交互
protocol LoginView: AnyObject {
    func showLoading()
    func hideLoading()
    func showError(message: String)
    func navigateToHome()
}

class LoginViewController: UIViewController, LoginView {
    
    private var presenter:LoginPresenter!
    
    func showLoading() {
        SVProgressHUD.show()
    }
    
    func hideLoading() {
        SVProgressHUD.dismiss()
    }
    
    func showError(message: String) {
        self.view.makeToast(message)
    }
    
    func navigateToHome() {
        
    }
    
    private let userNameTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入用户名"
    }
    
    private let passwordTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入密码"
        $0.isSecureTextEntry = true
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.
        self.title = "MVC"
        self.view.backgroundColor = .systemBackground
        
        self.view.addSubview(userNameTextField)
        userNameTextField.snp.makeConstraints { make in
            make.top.equalToSuperview().offset(100)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        
        self.view.addSubview(passwordTextField)
        passwordTextField.snp.makeConstraints { make in
            make.top.equalTo(userNameTextField.snp.bottom).offset(30)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        
        let loginButton = UIButton(type: .system).then {
            $0.setTitle("登录", for: .normal)
        }
        self.view.addSubview(loginButton)
        loginButton.snp.makeConstraints { make in
            make.top.equalTo(passwordTextField.snp.bottom).offset(16)
            make.left.right.equalToSuperview().inset(80)
            make.height.equalTo(44)
        }
        
        loginButton.addTarget(self, action: #selector(login(_:)), for: .touchUpInside)
    
        
        presenter = LoginPresenterImpl(view: self, userService: UserAPI())
    }
    
    @objc func login(_ sender: UIButton) {
        presenter.login(username: userNameTextField.text ?? "", password: passwordTextField.text ?? "")
    }
}

protocol LoginPresenter {
    func login(username: String, password: String)
}

class LoginPresenterImpl: LoginPresenter {
    private let userService: userService
    private weak var view: LoginView?
    
    init(view: LoginView, userService: userService) {
        self.view = view
        self.userService = userService
    }
    
    func login(username: String, password: String) {
        
        guard !username.isEmpty else {

            view?.showError(message: "请输入用户名")
            return
        }
        
        guard !password.isEmpty else {
            view?.showError(message: "请输入密码")
            return
        }
        
        view?.showLoading()
        userService.login(userName: username, password: password) { [weak self] result in
            self?.view?.hideLoading()
            self?.view?.navigateToHome()
        
        } fail: { [weak self] message in
            self?.view?.hideLoading()
            self?.view?.showError(message: message)
        }
    }
}
```

从代码中可以看到，MVP 模式通过接口与 `Presenter` 交互，使得视图和业务逻辑更加分离，
适用于需要完全分离 View 和 Model 的场景，尤其是在 UI 需要与复杂业务逻辑解耦的应用中。
但是在以下情况下：

- 视图逻辑复杂：当视图更新需要处理大量的逻辑时，这些逻辑容易堆积在 Presenter 中，使其职责不够单一。
- 业务逻辑复杂：如果 Presenter 同时负责协调多个 Model 和复杂的业务规则，也会导致代码膨胀。
- 视图状态管理：当视图的状态较多且频繁变化，Presenter 需要处理各种状态的更新逻辑，进一步增加复杂性。

会让`Presenter` 变得异常臃肿。此时可以考虑优化下

- 拆分 Presenter，将 Presenter 分解为多个小的、更专注的组件
  - 主 Presenter 负责协调多个子 Presenter。
  - 每个子 Presenter 只处理一部分功能或特定视图模块
- 使用状态管理
  - 将视图的状态管理从 Presenter 中抽离到一个独立的 State 或 ViewState。
  - Presenter 只负责生成状态，视图根据状态更新自身。
- 结合响应式编程，使用 RxSwift 等响应式编程库，减少 Presenter 中手动处理事件流的代码量。
- 视图与业务分离更彻底
  - 引入更复杂的架构模式，如 MVVM 或 MVI，将视图与业务逻辑的绑定更加明确。
  - MVP 的 Presenter 可能演变为 MVVM 的 ViewModel 或 MVI 的 State Reducer，从而更自然地管理复杂场景。
  
### 小结

#### 优点

- 清晰的职责分离
  - 提供了清晰的职责分离。`View` 只负责显示和用户交互，`Presenter` 负责业务逻辑处理，Model 则负责数据的获取与管理。
  - 这种分离使得业务逻辑与视图的代码解耦，便于单元测试和维护。
- 易于单元测试
  - 由于 `Presenter` 不依赖于具体的 `View` 实现（而是通过接口来交互），这使得 `Presenter` 更易于进行单元测试。测试可以通过模拟 `View` 接口来进行，而不需要依赖 UI。
  - 例如，可以模拟 `View` 返回用户输入，并验证 `Presenter` 逻辑是否正常。
- 支持多平台和复用
  - 由于 `View` 只负责显示和用户交互，`Presenter` 与具体的 UI 渲染分离，这使得 `Presenter` 更容易复用于不同的平台和界面。例如，一个 `Presenter` 可以用于多个不同的界面，只需要提供不同的 `View` 实现。
- 简化视图更新逻辑
  - 在 MVP 中，`Presenter` 可以集中处理视图更新逻辑。当视图状态变化时，`Presenter` 可以控制 `View` 进行相应的更新，而不需要分散到多个地方，从而避免了复杂的视图更新逻辑
- 灵活的视图替换
  - 由于 `Presenter` 通过接口与 `View` 交互，替换视图实现变得简单。你只需要实现一个新的 `View` 接口，`Presenter` 不需要进行任何修改，这提高了代码的灵活性和可扩展性。
  
#### 缺点

- Presenter 可能变得臃肿
  - 在复杂的应用中，`Presenter` 可能承担过多的职责，特别是当业务逻辑比较复杂时，`Presenter` 会变得臃肿，包含大量的逻辑，难以维护。这是 MVP 架构的一个常见问题。
  - 例如，`Presenter` 可能需要协调多个 Model，管理多种状态，处理复杂的事件流，这会导致代码过于复杂。
- 较难处理复杂的视图状态
  - 如果视图的状态很多（例如，加载状态、错误状态、成功状态等），`Presenter` 需要管理这些状态并协调视图的更新，容易导致状态管理混乱，增加 `Presenter` 的复杂性。
  - 在这种情况下，`Presenter` 会变得过于复杂，无法单纯地专注于业务逻辑。
- 对 View 和 `Presenter` 的紧密耦合
  - 虽然 `Presenter` 与 `View` 之间通过接口解耦，但是仍然存在一定的耦合，特别是当 `Presenter` 对 `View` 进行大量的状态更新时，可能会导致 `Presenter` 和 `View` 紧密绑定。
  - 如果 `View` 变化频繁，`Presenter` 需要做出大量调整，增加了维护难度。
- 增加了层数
  - 在 MVP 中，视图和业务逻辑之间引入了一个额外的层（Presenter），这使得应用架构变得更加复杂。如果应用比较简单，这样的分层可能反而成为负担。
  - 对于简单的视图更新，使用 MVP 可能显得“过度设计”，增加了不必要的复杂性。
- 重复代码和冗余
  - 由于 `Presenter` 需要处理大部分的逻辑，可能导致每个页面都有重复的代码。例如，多个页面需要类似的视图更新或错误处理逻辑时，`Presenter` 中可能会出现大量相似的代码，导致冗余。
- 需要大量的接口定义
  - `View` 和 `Presenter` 之间通过接口进行通信，这要求在代码中定义大量的接口，可能会导致代码结构变得更加复杂，特别是在需要更新和扩展时，接口数量的增加可能会让代码显得更加庞大和难以管理

## VIPER

### 定义

`VIPER`是通过分离关注点（Separation of Concerns）来增强代码的可维护性、可扩展性和测试性。
是基于单一职责原则（Single Responsibility Principle）的设计模式，常用于构建复杂的应用程序.

VIPER 的名称是由五个主要组成部分首字母组成的，每个部分都有明确的职责：

View

- 负责显示用户界面（UI）。
- 获取用户输入并将其传递给 Presenter。
- 不包含任何业务逻辑或数据处理逻辑，纯粹负责渲染和用户交互。

Interactor

- 负责所有与业务逻辑和数据处理相关的操作。
- 处理来自 View 的请求，并根据需要与网络、数据库、API 等交互。
- 完成后，将数据或结果传递给 Presenter。

Presenter

- 负责将业务逻辑从 Interactor 中获取的数据转换为 View 能显示的内容。
- 处理用户交互（例如按钮点击），并向 Interactor 发送请求。
- 不包含任何界面渲染的代码，只关心如何管理 View 和 Interactor 之间的数据流。

Entity

- 用于表示应用程序中的数据模型。
- 通常是与业务逻辑密切相关的数据对象，提供给 Interactor 使用。
- 例如，在一个购物车应用中，Entity 可能是 Product 或 Cart。

Router

- 负责管理应用程序的导航逻辑（即界面间的跳转）。
- 在 VIPER 中，Router 通常负责处理推送、弹出和呈现新界面的操作。
- 通过路由（Router），View 可以决定跳转到哪里，而不直接依赖于视图控制器的导航。

上代码

### Sample Code

```swift
// Interactor 处理与登录相关的业务逻辑，比如验证用户凭据并与后端交互。
protocol LoginInteractorProtocol {
    func login(username: String, password: String)
}

class LoginInteractor: LoginInteractorProtocol {
    var presenter: LoginPresenterProtocol?
    
    func login(username: String, password: String) {
        guard !username.isEmpty else {
            presenter?.loginDidFail(error: "请输入用户名")
            return
        }
        
        guard !password.isEmpty else {
            presenter?.loginDidFail(error: "请输入秘密")
            return
        }
        // 模拟网络请求
        DispatchQueue.global().asyncAfter(deadline: .now() + 1.0) {
            if username == "admin" && password == "1234" {
                // 登录成功，返回用户数据
                let user = UserInfo(name: username, avatar: "avatar")
                self.presenter?.loginDidSucceed(user: user)
            } else {
                // 登录失败
                self.presenter?.loginDidFail(error: "Invalid username or password")
            }
        }
    }
}

// Presenter 负责处理从 Interactor 返回的数据，并通知 View 更新。
protocol LoginPresenterProtocol {
    func login(username: String, password: String)
    func loginDidSucceed(user: UserInfo)
    func loginDidFail(error: String)
}

class LoginPresenter: LoginPresenterProtocol {
    var view: LoginViewProtocol?
    var interactor: LoginInteractorProtocol?
    var router: LoginRouterProtocol?
    
    func login(username: String, password: String) {
        view?.showLoading()
        interactor?.login(username: username, password: password)
    }
    
    func loginDidSucceed(user: UserInfo) {
        view?.hideLoading()
        router?.navigateToHome(user: user)
    }
    
    func loginDidFail(error: String) {
        view?.hideLoading()
        view?.showError(message: error)
    }
}

// View 负责显示 UI 和响应用户交互。
protocol LoginViewProtocol {
    func showLoading()
    func hideLoading()
    func showError(message: String)
}

class LoginViewController: UIViewController, LoginViewProtocol {
    
    var presenter: LoginPresenterProtocol?
    
    func showLoading() {
        print("Loading...")
        SVProgressHUD.show()
    }
    
    func hideLoading() {
        print("Loading finished.")
        SVProgressHUD.dismiss()
    }
    
    func showError(message: String) {
        print("Error: \(message)")
        self.view.makeToast(message)
    }
    
    private let userNameTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入用户名"
    }
    
    private let passwordTextField = UITextField().then {
        $0.borderStyle = .roundedRect
        $0.placeholder = "请输入密码"
        $0.isSecureTextEntry = true
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Do any additional setup after loading the view.
        self.title = "VIPER"
        self.view.backgroundColor = .systemBackground
        
        self.view.addSubview(userNameTextField)
        userNameTextField.snp.makeConstraints { make in
            make.top.equalToSuperview().offset(100)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        
        self.view.addSubview(passwordTextField)
        passwordTextField.snp.makeConstraints { make in
            make.top.equalTo(userNameTextField.snp.bottom).offset(30)
            make.left.right.equalToSuperview().inset(50)
            make.height.equalTo(44)
        }
        
        let loginButton = UIButton(type: .system).then {
            $0.setTitle("登录", for: .normal)
        }
        self.view.addSubview(loginButton)
        loginButton.snp.makeConstraints { make in
            make.top.equalTo(passwordTextField.snp.bottom).offset(16)
            make.left.right.equalToSuperview().inset(80)
            make.height.equalTo(44)
        }
        
        loginButton.addTarget(self, action: #selector(login(_:)), for: .touchUpInside)
        
    }
    
    @objc func login(_ sender: UIButton) {
        presenter?.login(username: userNameTextField.text ?? "", password: passwordTextField.text ?? "")
    }
}

// Router 处理界面导航。
protocol LoginRouterProtocol {
    func navigateToHome(user: UserInfo)
}

class LoginRouter: LoginRouterProtocol {
    weak var viewController: UIViewController?
    
    func navigateToHome(user: UserInfo) {
        let homeViewController = UIViewController() // 假设这是首页
        homeViewController.view.backgroundColor = .white
        homeViewController.title = "Welcome, \(user.name)"
        viewController?.navigationController?.pushViewController(homeViewController, animated: true)
    }
}

// 组装模块 将 VIPER 的各个模块连接起来
class LoginModule {
    static func createModule() -> UIViewController {
        let view = LoginViewController()
        let presenter = LoginPresenter()
        let interactor = LoginInteractor()
        let router = LoginRouter()
        
        view.presenter = presenter
        presenter.view = view
        presenter.interactor = interactor
        presenter.router = router
        interactor.presenter = presenter
        router.viewController = view
        
        return view
    }
}

```

### 工作流程

从代码中可以看到，工作流程大致如下：

- 用户与 View 交互：用户点击某个按钮，View 会通知 Presenter。
- Presenter 处理请求：Presenter 收到事件后，调用 Interactor 中的方法来处理业务逻辑。
- Interactor 处理数据：Interactor 与后端 API 或数据库交互，获取数据或执行操作。
- Interactor 返回数据：Interactor 获取到数据后，将其传递给 Presenter。
- Presenter 更新 View：Presenter 根据从 Interactor 获取的数据，决定如何在 View 上展示信息。
- View 渲染界面：View 将更新后的信息展示给用户。

### 小结

#### 优点

- 高内聚、低耦合：各个组件之间的职责分离明确，每个模块只关心自己负责的事情，从而提高了代码的可维护性。
- 易于单元测试：由于业务逻辑（Interactor）、展示逻辑（Presenter）和视图（View）分离，单元测试变得更加容易。
- 可扩展性强：模块化的架构方便添加新功能和扩展应用程序。
- 良好的职责分离：每个模块只有单一职责，便于团队分工和协作开发。

#### 缺点

- 过于复杂：对于简单的应用，VIPER 的架构可能显得过于繁琐，增加了开发和理解的难度。
- 初期开发成本较高：设置和构建 VIPER 所需的代码量较多，适合中大型项目。
- 学习曲线：开发者需要时间来适应 VIPER 的复杂性，特别是当团队成员对这种架构不熟悉时。

#### 适用场景

- 复杂的应用程序：当应用程序的业务逻辑、界面和数据需求较为复杂时，VIPER 的模块化架构能够帮助管理复杂性。
- 需要高度可测试性和可维护性的项目：VIPER 的模块化设计使得单元测试和代码的维护变得更加容易，特别适合长期维护和持续集成的项目。
- 多团队协作的开发环境：在不同的开发团队或人员专注于不同层次（例如 UI、业务逻辑等）时，VIPER 有助于提供清晰的边界和分工。

`VIPER`是一种现代的、功能强大的架构模式，适用于复杂的 iOS 或 macOS 应用。通过清晰地分离不同的职责，VIPER 可以帮助开发者编写更加可维护、易于测试和扩展的代码。尽管对于小型项目来说可能显得有些过于复杂，但在大型应用程序中，它能提供更好的组织和模块化结构，帮助团队更有效地开发和管理项目。

## 总结

### 对比

| 架构  | 结构简洁性 | 可扩展性 | 可测试性 | 适用场景                         |
| ----- | ---------- | -------- | -------- | -------------------------------- |
| MVC   | 简单       | 低       | 低       | 小型应用，快速开发，简单交互     |
| MVVM  | 中等       | 高       | 高       | 中型应用，复杂 UI，数据绑定      |
| MVP   | 中等       | 高       | 高       | 中型应用，复杂 UI，交互频繁      |
| MVI   | 高         | 高       | 高       | 大型复杂应用，响应式编程         |
| VIPER | 复杂       | 非常高   | 非常高   | 大型应用，团队协作开发，复杂业务 |

- MVC 适合简单应用，但容易出现 "巨型视图控制器" 问题。
- MVVM 适合数据绑定和响应式编程，适合中型应用和需要复杂 UI 更新的应用。
- MVP 比 MVC 更适合具有复杂交互的应用，尤其是在 Presenter 处理业务逻辑时，UI 和业务逻辑的分离更清晰。
- MVI 提供了一个单向数据流的设计，非常适合大型应用和响应式编程，尤其是当需要处理多个状态和一致性时。
- VIPER 适合需要高度模块化和复杂功能的应用，尤其是团队协作开发的大型项目。

选择合适的架构应该根据应用的复杂性、团队的大小、业务逻辑的复杂度以及项目的可维护性需求来决定。
但是在开发中，选择合适的架构不是一蹴而就的，而是需要根据项目需求和团队情况来权衡和选择。

[Demo](https://github.com/wanghaolyj/ArchitecturalPattern)
