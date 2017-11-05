最近新项目开始尝试 Swift 混编，而我负责搭建底层库。在调研了很多开源的网络库后，最后选择了 Moya，本片文章也是对 Moya 使用过程的一个总结。

# Moya 是什么？
[Moya](https://github.com/Moya/Moya) 是一个开源的网络请求库，它底层封装了`Alamofire`，对外提供简单易用的网络请求的接口。

# 为什么选择 Moya？
大家都知道在 Swift 2.0版本发布的时候，官方当时在[Protocol-Oriented Programming in Swift](https://developer.apple.com/videos/play/wwdc2015/408/) 中就向广大小伙伴们推荐在 Swift 中面向 POP 编程。而让我选择 Moya 的最大原因也是其：**面向 POP 编程**。至于 POP、OOP 编程的优缺点可以看喵大的[这篇文章](https://onevcat.com/2016/11/pop-cocoa-1/)和[这篇文章](https://onevcat.com/2016/12/pop-cocoa-2//)，文章里已经分析的很清晰了，我在此简述一下。

### 面向 POP 解决了面向 OOP 编程的那些问题？

* 横切关注点
* 多继承的菱形依赖
* 动态派发安全性

# 如何使用 Moya ？
* 定义实现`TargetType`的接口类
* 定义 ApiManager 类
* 调用

### 定义接口

```swift
public enum Game {
    //1.
    case login(String, String)
    case userProfile
    case sticker
}

//2.
extension Game: TargetType {

    //3.
    public var baseURL: URL {
        return URL(string: "\(HttpScheme)://\(HttpHost)/v1\(ApiPrefix)")!
    }
    
    //4.
    public var path: String {
        switch self {
        case .login:
            return "/signin"
        case .sticker:
            return "/sticker"
        default:
            return "/"
        }
    }
    
    //5.
    public var method: Moya.Method {
        switch self {
        case .login:
            return .post
        default:
            return .get
        }
    }
    
    //6.
    public var task: Task {
        switch self {
        case .login(let username, let password):
            return .requestParameters(parameters: ["username" : username, "password" : password], encoding: GameURLEncoding.default)
        case .sticker:
            return .requestParameters(parameters: [:], encoding: GameURLEncoding.default)
        default:
            return .requestParameters(parameters: [:], encoding: GameURLEncoding.default)
        }
    }
    
    //7.
    public var validate: Bool {
        return false
    }
    
    //8.
    public var sampleData: Data {
        return "Hello world".data(using: String.Encoding.utf8)!
    }
    
    //9
    public var headers: [String : String]? {
        return []
    }
}
```

1：定义 api 枚举接口。

2：实现 Moya 的 `TargetType` 协议。

3：定义接口接口 Domain。

4：返回定义的接口所对应的路径 path。

5：定义该接口使用的请求方法类型。

6：添加接口所需要的额外参数，并处理其参数的对应编码方式。

7：这里可以针对特定路径来实现特定的接口验证方式。

8：这里可以针对接口返回特定的测试数据。

9：可以添加自定义的请求头参数。

### 定义 `ApiManager`

```swift
class GameAPIManager {    

    private static let `default` = GameAPIManager()

    //1.
    private let gameApiProvider = MoyaProvider<Game>
    
    //2.
    @discardableResult
    static func request(
        _ target: Game,
        callbackQueue: DispatchQueue? = nil,
        progress: Moya.ProgressBlock? = nil,
        success: @escaping (Response) -> Void,
        failure: @escaping (Error) -> Void) -> Cancellable {
        
        return GameAPIManager.default.gameApiProvider.request(
            target,
            callbackQueue: callbackQueue,
            progress: progress,
            completion: { (result) in
                switch result {
                case .success(let response):
                    success(response)
                case .failure(let error):
                    failure(error)
                }      
        })
    }
}
```

1：定义 `MoyaProvider`

2：定义请求方法.

### 调用
```swift
GameAPIManager.request(.sticker, success: { (response) in
    print("response: \(response)")
}) { (error) in
    print("error: \(error)")
}
```

没错，使用 Moya 来搭建自己的网络请求库就是这么简单。

下面我们来根据自己需求来定制一下 `ApiManager`。

###向接口的中添加全局参数

```swift

//1.
class RequestHandlingPlugin: PluginType {    
    public func prepare(_ request: URLRequest, target: TargetType) -> URLRequest {
        var mutateableRequest = request
        //2.
        return mutateableRequest.appendCommonParams();
    }
}

extension URLRequest {
    
    /// global common params
    private var commonParams: [String: Any] {
        return Enviroment.requestParams
    }
    
    /// global common header fields
    private var commonHeaderFields: [String : String] {
        return LoginManager.authorizeParams
    }
    
    mutating func appendCommonParams() -> URLRequest {
        let newHeaderFields = (allHTTPHeaderFields ?? [:]).merging(commonHeaderFields) { (current, _) in current }
        allHTTPHeaderFields = newHeaderFields
        let request = try? encoded(parameters: commonParams, parameterEncoding: URLEncoding(destination: .queryString))
        assert(request != nil, "append common params failed, please check common params value")
        return request!
    }
    
    func encoded(parameters: [String: Any], parameterEncoding: ParameterEncoding) throws -> URLRequest {
        do {
            return try parameterEncoding.encode(self, with: parameters)
        } catch {
            throw MoyaError.parameterEncoding(error)
        }
    }
}
```
1：定义`RequestHandlingPlugin`类并遵守 `PluginType`协议。

2：向最终的请求中添加额外参数。

### 验证 SSL 自签名证书
由于 Moya 底层封装的是 Alamofire, 所以验证自签名证书这里要用到 Alamofire 库。

```swift
class GameAPIManager {
    ...
    private let gameApiProvider: MoyaProvider<Game>

    //1.
    private let manager: SessionManager
    
    //2.
    private let serverTrustPolicy = ServerTrustPolicy.pinCertificates(
        certificates: ServerTrustPolicy.certificates(),
        //如果为自签名证书，这里得让 Alamofire 去验证证书，所以值为 True
        validateCertificateChain: true,
        validateHost: false
    )
    
    private let pkcs12Import = PKCS12Import(
        mainBundleResource: "client",
        resourceType: "p12",
        password: "******"
    )
    
    init() {
        let serverTrustPolicies = ["\(HttpHost)" : serverTrustPolicy]
        
        //3.
        let serverTrustPolicyManager = ServerTrustPolicyManager(policies: serverTrustPolicies)
        
        //4.
        manager = SessionManager(
            configuration: URLSessionConfiguration.default,
            delegate: SessionDelegate(),
            serverTrustPolicyManager: serverTrustPolicyManager
        )
        
        //5.
        gameApiProvider = MoyaProvider<Game>(
            manager: manager,
        )
        
        //6.
        manager.delegate.sessionDidReceiveChallengeWithCompletion = { [weak self] (session, challenge, completion) in
            guard let `self` = self else {
                completion(.performDefaultHandling, nil)
                return
            }
            
            if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust {
                guard
                    let serverTrust = challenge.protectionSpace.serverTrust,
                    self.serverTrustPolicy.evaluate(serverTrust, forHost: challenge.protectionSpace.host) else {
                        completion(.performDefaultHandling, nil)
                        return
                }
                completion(.useCredential, URLCredential(trust: serverTrust))
            }
            else if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodClientCertificate {
                guard let pkcs12Import = self.pkcs12Import else {
                    completion(.performDefaultHandling, nil)
                    return
                    
                }
                completion(.useCredential, pkcs12Import.urlCredential)
            }
            else {
                completion(.performDefaultHandling, nil)
            }
        }

    }
}
```

1：定义一个类型为 Alamofire `SesssionManager`的属性 `manager`。

2：定义自己的 `ServerTrustPolicy` 属性。 

3：指定域名的验证策略为使用特定证书。

4：初始化 `Manager` 并指定其 `Delegate`。

5：初始化 `gameApiProvider`.

6: 自定义实现Alamofire 的证书验证代理方法。

上面代码中，`PKCS12Import`是抽象出的一个管理客户端 p12 证书文件并返回 `URLCredential` 的小工具类可以在 [Demo](https://github.com/devSC/Moya-demo) 中查看其源码。

# 总结
Moya 的使用就是这么简单，就像其文档中第一句话中写的：*Moya is about working at high levels of abstraction in your application.* 是的，它尽可能的对底层进行了封装，但又不失我们对底层操作的灵活性，完美发挥了 POP 的威力。

可以在[这里](https://github.com/devSC/Moya-demo)查看示例代码。
