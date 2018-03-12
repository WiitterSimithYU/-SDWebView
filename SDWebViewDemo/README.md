# SDWebView
针对WKWebView进行的封装、支持和H5交互、包括调用js方法等。挺全面的！真的。其他针对WKWebView封装的就不要看了。
```
//给webView添加监听title和进度条
[self.webView addObserver:self forKeyPath:@"title" options:NSKeyValueObservingOptionNew context:nil];
[self.webView addObserver:self forKeyPath:@"estimatedProgress" options:NSKeyValueObservingOptionNew context:nil];

//kvo监听进度条
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    if ([object isKindOfClass:[SDWebView class]]) {
        if ([keyPath isEqualToString:@"estimatedProgress"]) {
            CGFloat newprogress = [[change objectForKey:NSKeyValueChangeNewKey] doubleValue];
            if (newprogress == 1) {
                self.webView.progressView.hidden = YES;
                [self.webView.progressView setProgress:0 animated:NO];
            }else {
                self.webView.progressView.hidden = NO;
                [self.webView.progressView setProgress:newprogress animated:YES];
            }
        }
        if ([keyPath isEqualToString:@"title"]) {
            if (self.webView.title.length > 10) {
                self.navigationItem.title = [self.webView.title substringToIndex:14];
            } else {
                self.navigationItem.title = self.webView.title;
            }
        }
    } else {
        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
    }
}
//移除监听
[self.webView removeObserver:self forKeyPath:@"estimatedProgress"];
[self.webView removeObserver:self forKeyPath:@"title"];


//js调OC时，需要让前端人员写如下js代码   
                              //👇 AppModel是我们所注入的对象   也就是SDWebView的jsHandlers
window.webkit.messageHandlers.AppModel.postMessage({body: response});

//对象可以注入多个，所以jsHandlers是个数组  如下代码：注入三个对象到页面中
self.webView.jsHandlers = @[TOLOGIN,TOPAY,TOYATI];

//如果注入了对象 要调用如下方法，注销handle 不然会creash
[self.webView removejsHandlers];


//OC调用js时，可以调用如下方法:
- (void)callJS:(nonnull NSString *)jsMethodName;

//可能使用的属性及方法如下:


/**
 需要加载的urlStr
 */
@property (nonatomic, copy) NSString *URLString;

/**
 进度条
 */
@property (strong, nonatomic) UIProgressView *progressView;

/**
 webView的标题、如果navigationItemTitle需要和webView保持一致、直接getter方法即可
 */
@property (nonatomic, copy) NSString *webViewtitle;

/**
 注入H5页面的交互模型
 */
@property (nonatomic, strong) NSArray<NSString *> *jsHandlers;


/**
 是否显示加载的HTML页面源码 default NO
 */
@property (nonatomic, assign) BOOL displayHTML;

/**
 是否显示加载的HTML页面中的cookie default NO
 */
@property (nonatomic, assign) BOOL displayCookies;

/**
 每次跳转是否需要打印跳转的URL default YES
 */
@property (nonatomic, assign) BOOL displayURL;

/**
 获取交互的参数代理
 */
@property (nonatomic, weak) id<SDWebViewDelegate> delegate;

/**
 根据URL初始化
 @param urlString URLString
 @return WebviewVc实例
 */
- (instancetype)initWithURLString:(NSString *)urlString;

/**
 *  加载本地HTML页面
 *
 *  @param htmlName html页面文件名称
 */
- (void)loadLocalHTMLWithFileName:(nonnull NSString *)htmlName;

/**
 移除jsHandler
 */
- (void)removejsHandlers;

/**
 清除所有cookie
 */
- (void)removeCookies;

/**
 清除指定域名中的cookie
 
 @param cookieName 域名
 */
- (void)removeCookieWithHostName:(NSString *)hostName;

/**
 *  调用JS方法（无返回值）
 *
 *  @param jsMethod JS方法名称
 */
- (void)callJS:(nonnull NSString *)jsMethodName;

/**
 *  调用JS方法（可处理返回值）
 *
 *  @param jsMethod JS方法名称
 *  @param handler  回调block
 */
- (void)callJavaScript:(nonnull NSString *)jsMethodName handler:(nullable void(^)(__nullable id response))handler;

//如果前端人员不知道OC调用js 的方法怎么写 可参考如下代码 😝
function configer() {
    reture {'c':"100"}
}
在callJS方法中直接让前端人员返回一个json 或者 其他返回值 OC即可在handler这个blcok 中获取 这个返回值
```
