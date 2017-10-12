# runtime-
runtime使用场景======场景1(动态给分类添加属性);场景2(方法的交换swizzling);场景3(字典转模型);场景4(获取所有的私有属性和方法);场景5(对私有属性修改);场景6(归档:解档);场景7(动态的添加方法)






# GCD-
class ViewController: UIViewController {
    
    private static var once = ViewController() // 单例模式
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        var queue: DispatchQueue = DispatchQueue.main// 主线程
        if #available(iOS 8.0, *) {
            queue = DispatchQueue.global()
        } else {
            queue = DispatchQueue.global(priority: DispatchQueue.GlobalQueuePriority.background)// 后台执行
        }
        
        // 异步执行队列任务
        queue.async {
            print("开新线程执行")
        }
        
        // 延时执行
        var delta:Int64 = Int64(2 * NSEC_PER_SEC)// 2s后执行,可能不仅限于2s
        delta = Int64(100 * NSEC_PER_MSEC)//100毫秒后执行
        // NSEC_PER_MSEC
        let when: DispatchTime = DispatchTime.now() + Double(delta) / Double(NSEC_PER_SEC)
        queue.asyncAfter(deadline: when) {
            print("dispatch_after")
        }
        
        // 分组执行
        let group = DispatchGroup()
        queue = DispatchQueue.global(priority: DispatchQueue.GlobalQueuePriority.default)// 默认优先级执行
        for i in 0 ..< 10 {
            //异步执行队列任务
            queue.async(group: group, execute: {
                print("queue.async(group: group \(i)")
            })
        }
        // 分组队列执行完毕后执行
        group.notify(queue: queue) {
            print("dispatch_group_notify")
        }
        
        // 串行队列：只有一个线程，加入到队列中的操作按添加顺序依次执行。
        let serialQueue = DispatchQueue(label: "yangj", attributes: [])
        for i in 0 ..< 10 {
            //异步执行队列任务
            serialQueue.async {
                print("serialQueue.async \(i)")
            }
        }
        
        // 并发队列：有多个线程，操作进来之后它会将这些队列安排在可用的处理器上，同时保证先进来的任务优先处理。
        let globalQueue = DispatchQueue.global(priority: DispatchQueue.GlobalQueuePriority.default)
        for i in 0 ..< 10 {
            //异步执行队列任务
            globalQueue.async {
                print("globalQueue.async \(i)")
            }
        }
    }
}

# KVO-

/// 用户
class User: NSObject {
    /// 用户名
    dynamic var userName:String?
    
    override init() {
        userName = ""
    }
}

/// KVO测试
class KVOTests: XCTestCase {
    
    /// 用户
    var user:User!;
    
    // MARK: 开始
    override func setUp() {
        super.setUp()
        self.user = User()
        self.user.addObserver(self, forKeyPath: "userName", options: NSKeyValueObservingOptions.new, context: nil)// 监听（KVO的属性必须设置dynamic）
    }
    
    // MARK: 结束
    override func tearDown() {
        super.tearDown()
        self.user.removeObserver(self, forKeyPath: "userName")
        self.user.userName = "YangJun"
    }
    
    // MARK: 测试用例
    func testExample() {
        self.user.userName = "阳君"
    }
    
    // MARK: - 监听
    override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
        if "userName" == keyPath {
            print("\nchange:\(change ?? [:])\n")
        }
    }
}


# Parser-

/// json解析
class YJJSONParserTests: XCTestCase {
    
    var jsonString : String?
    
    override func setUp() {
        super.setUp()
        var dict = [String: String]()
        dict["name"] = "阳君"
        dict["qq"] = "937447974"
        do {
            if JSONSerialization.isValidJSONObject(dict) { // 能否转换为JSON Data
                // 转换为JSON Data
                let data  = try JSONSerialization.data(withJSONObject: dict, options: JSONSerialization.WritingOptions.prettyPrinted)
                // 转换为json串
                self.jsonString = String(data: data, encoding: String.Encoding.utf8)
                print("json生成:\(self.jsonString ?? "")")
            }
        } catch {
            print("转换出错:\(error)")
        }
    }
    
    override func tearDown() {
        super.tearDown()
        self.jsonString = nil
    }
    
    func testExample() {
        // json转data
        if let data = self.jsonString?.data(using: String.Encoding.utf8) {
            do {
                // data转JSON Object
                let jsonObject = try JSONSerialization.jsonObject(with: data, options: JSONSerialization.ReadingOptions.allowFragments)
                // JSON Object转实际对象
                if let dict = jsonObject as? Dictionary<String, AnyObject> {
                    print("json解析:\(dict)")
                }
            } catch {
                print("解析xml出错:\(error)")
            }
        }
    }
    
}

/// xml解析
class YJXMLParserTests: XCTestCase, XMLParserDelegate {
    
    override func setUp() {
        super.setUp()
        // Put setup code here. This method is called before the invocation of each test method in the class.
    }
    
    override func tearDown() {
        // Put teardown code here. This method is called after the invocation of each test method in the class.
        super.tearDown()
    }
    
    func testExample() {
        if let url = Bundle.main.url(forResource: "Main", withExtension: "xml") {
            if let parser = XMLParser(contentsOf: url) {
                parser.shouldProcessNamespaces = false;
                parser.delegate = self;
                parser.parse()
            }
        }
    }
    
    // MARK: - NSXMLParserDelegate
    // MARK: 解析开始
    func parserDidStartDocument(_ parser: XMLParser) {
        print(#function)
    }
    
    // MARK: 解析结束
    func parserDidEndDocument(_ parser: XMLParser) {
        print(#function)
    }
    
    // MARK: 解析出错
    func parser(_ parser: XMLParser, parseErrorOccurred parseError: Error) {
        print("解析错误\(parseError.localizedDescription)")
    }
    
    // MARK: 解析器每次在XML中找到新的元素时，会调用该方法
    func parser(_ parser: XMLParser, didStartElement elementName: String, namespaceURI: String?, qualifiedName qName: String?, attributes attributeDict: [String : String]) {
        print("\(elementName) - \(namespaceURI ?? "") - \(qName ?? "") - \(attributeDict)")
    }
    
}

