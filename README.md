# -TableFooterView-
处理的许多细节有价值
![](http://upload-images.jianshu.io/upload_images/739863-5d1aaf3c1af7062e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/739863-4ed70f69d9ba79fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/739863-d9e2e2f3e8d0b931.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 设计实现上面的页面

**一**
- 首先第一个这样的一个界面如何设计呢？它又有哪些需要注意的细节呢
  + 分析：
    + 可以上下拖动所以是一个scrollview
    + 对于实现可以用tableview来实现
      + 分为2组，每一组的cell为1个
      + 下面的由tableview的tableFooterView来实现
- 在控制器CYTabbarController.m文件中
  - 1.分组：设置tableview的样式的为UITableViewStyleGrouped
  - 2.实现tableview数据源方法
    + 返回为2组，每组为1行
    + 给定cell标识，注册cell

```objc
static NSString * const CYMeCellId = @"me";
```
```objc
[self.tableView registerClass:[CYMeCell class] forCellReuseIdentifier:CYMeCellId];
```
- tableview定义样式为UITableViewStyleGrouped分组之后要求：
  + 第一个cell距离顶部间距为10
  + cell之间距离为10
  + 第二个cell与它的tableFooterView之间间距为10
  + 而这个10的间距很常见，所以把它抽成一个宏或者是全局变量为CYCommonMargin
- 分组样式的tableview的cell的一些结构：
![](http://upload-images.jianshu.io/upload_images/739863-4fe1fb673ad117e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 分析
  + 分组group样式第一个cell默认距离顶部（第一个cell的Y值）为35，而plain样式默认是粘着顶部的
  + 分组后的cell拥有一个sectionHeaderHeight和sectionFooterHeight
  + 所以设置每一组的头部和尾部
  + 设置内边距(-25代表：所有内容往上移动25)

```objc
    self.tableView.sectionHeaderHeight = 0;
    self.tableView.sectionFooterHeight = CYCommonMargin;
    // 设置内边距(-25代表：所有内容往上移动25)
    self.tableView.contentInset = UIEdgeInsetsMake(CYCommonMargin - 35, 0, 0, 0);
```

------------------------------------
**二**
- 自定义cell---CYMeCell
  + 初始化cell以及重写它的layoutSubviews:方法(布局cell内部子控件：让图片和文字适当排布)

```objc
- (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier
{
    if (self = [super initWithStyle:style reuseIdentifier:reuseIdentifier]) {
        self.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
        self.textLabel.textColor = [UIColor darkGrayColor];

        // 设置背景图片-图片可以设置拉伸
        self.backgroundView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"mainCellBackground"]];
    }
    return self;
}

- (void)layoutSubviews
{
    [super layoutSubviews];

    if (self.imageView.image == nil) return;

    // 调整imageView
    self.imageView.y = CYCommonMargin * 0.5;
    self.imageView.height = self.contentView.height - 2 * self.imageView.y;
    self.imageView.width = self.imageView.height;

    // 调整Label
    //    self.textLabel.x = self.imageView.x + self.imageView.width + XMGCommonMargin;
    self.textLabel.x = CGRectGetMaxX(self.imageView.frame) + CYCommonMargin;

    // CGRectGetMaxX(self.imageView.frame) == self.imageView.x + self.imageView.width
    // CGRectGetMinX(self.imageView.frame) == self.imageView.x
    // CGRectGetMidX(self.imageView.frame) == self.imageView.x + self.imageView.width * 0.5
    // CGRectGetMidX(self.imageView.frame) == self.imageView.centerX
}

```
- 实现CYMeCell的数据源方法

```objc
#pragma mark - 数据源方法
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
    return 2;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return 1;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    CYMeCell *cell = [tableView dequeueReusableCellWithIdentifier:CYMeCellId];

    if (indexPath.section == 0) {
        cell.textLabel.text = @"登录/注册";
        cell.imageView.image = [UIImage imageNamed:@"setup-head-default"];
    }else{

        cell.textLabel.text = @"离线下载";
    }

    return cell;
}
```
- 以上实现了cell内容的显示，接下来就是TableFooterView上内容数据的显示
  + 在PCH文件中定义一个宏,将数据写到桌面的plist，方便查看

----------------------------------------------
**三**
```objc
#define CYWriteToPlist(data, filename) [data writeToFile:[NSString stringWithFormat:@"/Users/gecongying/Desktop/%@.plist", filename] atomically:YES];
```
  + 自定义数据模型CYSquare
  + CYSquare.h文件中

```objc

#import <Foundation/Foundation.h>

@interface CYSquare : NSObject
/** 名字 */
@property (nonatomic, copy) NSString *name;
/** 图标 */
@property (nonatomic, copy) NSString *icon;
/** 链接 */
@property (nonatomic, copy) NSString *url;

@end
```
  + 自定义CYMeFooter

```objc
self.tableView.tableFooterView = [[CYMeFooter alloc] init];
```
  + 在CYMeFooter.m文件中

```objc
#import "CYMeFooter.h"
#import <AFNetworking.h>
#import "CYSquare.h"
#import <MJExtension.h>
//#import <UIImageView+WebCache.h>
#import <UIButton+WebCache.h>

@implementation CYMeFooter

- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        self.backgroundColor = [UIColor redColor];

        // 请求参数
        NSMutableDictionary *params = [NSMutableDictionary dictionary];
        params[@"a"] = @"square";
        params[@"c"] = @"topic";

        // 发送请求
        CYWeakSelf;
        [[AFHTTPSessionManager manager] GET:CYRequestURL parameters:params success:^(NSURLSessionDataTask *task, id responseObject) {
//            CYWriteToPlist(responseObject, @"square");
            [weakSelf createSquares:[CYSquare objectArrayWithKeyValuesArray:responseObject[@"square_list"]]];
        } failure:^(NSURLSessionDataTask *task, NSError *error) {

        }];
    }
    return self;
}

/**
 * 创建方块
 */
- (void)createSquares:(NSArray *)squares
{
    // 每行的列数
    int colsCount = 4;

    // 按钮尺寸
    CGFloat buttonW = self.width / colsCount;
    CGFloat buttonH = buttonW;

    // 遍历所有的模型
    NSUInteger count = squares.count;
    for (NSUInteger i = 0; i < count; i++) {
        CYSquare *square = squares[i];

        // 创建按钮
        UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
        [button addTarget:self action:@selector(buttonClick:) forControlEvents:UIControlEventTouchUpInside];
        [self addSubview:button];

        // frame
        CGFloat buttonX = (i % colsCount) * buttonW;
        CGFloat buttonY = (i / colsCount) * buttonH;
        button.frame = CGRectMake(buttonX, buttonY, buttonW, buttonH);

        // 数据
        [button setTitle:square.name forState:UIControlStateNormal];


    // 2.如果导入的是#import <UIButton+WebCache.h>
        // 设置按钮的image
        [button sd_setImageWithURL:[NSURL URLWithString:square.icon] forState:UIControlStateNormal];


    // 1.如果导入的是#import <UIImageView+WebCache.h>
        // 正确做法
//       [[SDWebImageManager sharedManager] downloadImageWithURL:[NSURL URLWithString:square.icon] options:0 progress:nil completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
//        [button setImage:image forState:UIControlStateNormal];
       // 错误示范
    //        [button.imageView sd_setImageWithURL:[NSURL URLWithString:square.icon]];
//                }];
    }
}

- (void)buttonClick:(CYSquareButton *)button
{
    CYLogFunc;

}
```
- 但是最后的显示是这样

![](http://upload-images.jianshu.io/upload_images/739863-e61e3c08f5f1a101.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



--------------------------------------
**四**
- 所以我们要自定义Button---CYSquareButton
  + 重写它的initWithFrame:方法和layoutSubviews:方法重新布局子控件

```objc
#import "CYSquareButton.h"

@implementation CYSquareButton

- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        self.titleLabel.textAlignment = NSTextAlignmentCenter;
        self.titleLabel.font = [UIFont systemFontOfSize:14];
        [self setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
    }
    return self;
}

- (void)layoutSubviews
{
    [super layoutSubviews];

    self.imageView.width = self.width * 0.5;
    self.imageView.height = self.imageView.width;
    self.imageView.y = self.height * 0.1;
    self.imageView.centerX = self.width * 0.5;

    self.titleLabel.width = self.width;
    self.titleLabel.y = CGRectGetMaxY(self.imageView.frame);
    self.titleLabel.x = 0;
    self.titleLabel.height = self.height - self.titleLabel.y;
}

@end
```
- 注意在CYMeFooter.m文件中导入相应的头文件CYSquareButton.h，修改Button为CYSquareButton
- 最后显示

![](http://upload-images.jianshu.io/upload_images/739863-6114153c0791a625.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 但是你会发现有问题要处理



------------------------------------------------------
**五**
- **一个控件不能响应点击事件的原因可能有：**
    + 1> **userInteractionEnabled = NO;**
    + 2> **enabled = NO;**
    + 3> **父控件的userInteractionEnabled = NO;**
    + 4> **父控件的enabled = NO;**
    + 5> **控件已经超出父控件的边框范围**
------------------------------------------------------
**六**
- 上面完成后你会发现footerView中的内容无法点击，更无法正常拖拽（bug）
  + footerView不用设置宽度，默认是填充整个TableView的，永远跟在TableView后面
  + TableView的拖拽范围是由它的contentsize决定的
    + 所以我们要让它的Button可以点击，内容可以显示，就得设置好footerView的高度
      - 我们要给它设置高度，但是拖拽是否正常又和设置高度的先后顺序有关
      + footerView的高度的设置要在创建它之前设置
      + 因为它是在你设置高度以后，根据你的高度去算出contentsize，去决定能拖拽到哪里。它是取决于你设置那一刻高度是多少
      - 所以上面的问题在于：你的高度设置是在服务器返回数据后才设置的，高度是后面设置的。所以无法影响它现在拖拽上拉的设置
      - 先设置高度，再拿到footerView
- **我们来设置footerView的高度**
- 第一种方式：先拿到footer的高度，再重新设置footerView

```objc
// 设置footer的高度
    self.height = CGRectGetMaxY(button.frame);
            // 重新设置footerView
    UITableView *tableView = (UITableView *)self.superview;
    tableView.tableFooterView = self;
```
- 第二种方式：直接改变它的contenSize（内容尺寸）---简单（推荐）

```objc
// 重新设置footerView
    UITableView *tableView = (UITableView *)self.superview;
//    tableView.tableFooterView = self;
    tableView.contentSize = CGSizeMake(0, CGRectGetMaxY(self.frame))
```

- **第三种方式：**上面两种方式算高度，我们是拿到最后一个按钮最大的Y值。还有一种方法：拿到按钮行数，再乘以按钮高度也是可以的

```objc
    // 设置footer的高度
    NSUInteger rowsCount = count / colsCount;
    if (count % colsCount) { // 不能整除，行数+1
        rowsCount++;
    self.height = rowsCount * buttonH;

    // 重新设置footerView
    UITableView *tableView = (UITableView *)self.superview;
    //    tableView.tableFooterView = self;
    tableView.contentSize = CGSizeMake(0, CGRectGetMaxY(self.frame));
    }
```
- 上面这么算，不管整不整除，都可以算出正确的行数
- 这也可以引出一个公式，将它合并为：

```objc
    // 设置footer的高度
     NSUInteger rowsCount = (count + colsCount - 1) / colsCount;
    self.height = rowsCount * buttonH;

    // 重新设置footerView
    UITableView *tableView = (UITableView *)self.superview;
    //    tableView.tableFooterView = self;
    tableView.contentSize = CGSizeMake(0, CGRectGetMaxY(self.frame));
```
- 这个公式用到一个地方很有作用：**分页**
  + 就像百度上搜索“美女”，给你返回一个结果，但是结果数据这么多，不可能一页就能显示完，所以采用分页显示数据。总共有多少条数据，每一页有固定条，总共有多少页
  + 算是万能公式，遇到相关需求，就不需要再苦逼的进行判断了

```objc
     总页数 == (总个数 + 每页的个数 - 1) / 每页的个数

     总个数：97
     每页的个数：17
     总页数 =  (97 + 17 - 1) / 17

```

![](http://upload-images.jianshu.io/upload_images/739863-ff8a262c538bf3cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



------------------------------------
**七**
- 现在按钮可以点击，界面可以正常拖拽。下面做一下分割线
  + 第一种方式：背景设为白色，添加UIView（太多不推荐）--加一个View
  + 第二种方式：在按钮原有算好的基础上，让他们的高度和宽带都减去1(记得要先算好，再减1)--空出间隙

```objc
button.frame = CGRectMake(buttonX, buttonY, buttonW-1, buttonH-1);
```
  + 第三种方式：看你在公司和美工的关系了😄
    + 直接让美工做一个有边线的按钮背景图片

```objc
[self setBackgroundImage:[UIImage imageNamed:@"mainCellBackground"] forState:UIControlStateNormal];
```

![](http://upload-images.jianshu.io/upload_images/739863-4ed70f69d9ba79fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-----------------------------------------------------
**八**
- 接下来，监听按钮的点击，拿到按钮索引对应方块中的数据
- 可以看出一个SquareButton对应一个Square模型
  + 所以在SquareButton.h文件中加一个属性

```objc
#import <UIKit/UIKit.h>

@class CYSquare;

@interface CYSquareButton : UIButton
/** 方块模型 */
@property (nonatomic, strong) CYSquare *square;
@end

```
- 在CYSquareButton.m文件中重写setSquare:方法，将数据请求封装起来

```objc
- (void)setSquare:(CYSquare *)square
{
    _square = square;

    // 数据
    [self setTitle:square.name forState:UIControlStateNormal];
    // 设置按钮的image
    [self sd_setImageWithURL:[NSURL URLWithString:square.icon] forState:UIControlStateNormal];
}
```
```objc
// 1.创建按钮
// 2.设置按钮frame
// 3.设置模型数据(拿到模型，把模型数据拆分给子控件)
button.square = squares[i];
```
- 这样的话，我拿到按钮就相当于拿到模型，一个按钮对应一个模型（更有封装性）
  - 今后当我们碰到一对一的情况的时候，就是一个数据对应一个控件的时候。你就要想到可不可以给这个控件或者是这个View绑一个模型
  - 当然一对一也可以用索引， 当我们控件的索引和我们模型的索引是一致的时候，先通过控件取出索引，再通过索引取出模型数据
  - 针对一对一我们还可以用字典，假如按钮能做一个key，通过按钮的key可以取出模型对应的value

-----------------------------------------
**九**

- 现在我们要通过url去打开一个下一个界面CYWebViewController(新建并继承于UIViewController[里面有一个WebView和工具条])
- 我们选择push过去,在CYMeFooter.m文件中

```objc
- (void)buttonClick:(CYSquareButton *)button
{
    // 拿到以http开头的URL
    if ([button.square.url hasPrefix:@"http"] == NO) return;
    CYWebViewController *webVc = [[CYWebViewController alloc] init];
    webVc.square = button.square;

    // 取出当前选中的导航控制器
    UITabBarController *rootVc = (UITabBarController *)self.window.rootViewController;
    UINavigationController *nav = (UINavigationController *)rootVc.selectedViewController;
    [nav pushViewController:webVc animated:YES];

}
```
- 上面的CYMeFooter是一个View，你是没法拿到导航控制器的，而且View里面是没有属性拿到它对应的控制器的，那我们怎么办呢？
  + 但是有一个控制器是哪里都能拿到的，就是窗口的根控制器
  + 而我们这个窗口的根控制器本质是一个CYTabBarController
  + 而“我”“精华”“新帖”“关注”对应的导航控制器其实都是CYTabBarController的子控制器
  + 但是我们要push过去，拿到导航控制器，但是这里有4个导航控制器，，所以我们要拿到对应的被选中的“我”对应导航控制器
- 所以今后得注意：不要一要push跳转就：self.NavigationController push...而是要找到对应的导航控制器，再做动作

-------------------------------------------------------
**十**

![](http://upload-images.jianshu.io/upload_images/739863-c13abaf0e05011b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 进入那个界面，显示一个网页，意味着你要把URL传给它
- 上面还得显示一个标题，你显示标题的文字最好也要和你按钮显示的文字一致
- 也就是说要传两个东西--文字和URL。既然要传两个，干脆就把模型传过去算了

- 在CYWebViewController.h文件中传入模型

```objc
#import <UIKit/UIKit.h>

@class CYSquare;

@interface CYWebViewController : UIViewController
/** 方块 */
@property (nonatomic, strong) CYSquare *square;
@end
```
- 然后再来界面布局：
- 用Xib先把底部工具条布置好，剩下的就是上面的网页界面 
  + 底部工具条可以搞个UIView，上面搞几个Button
  + 或者搞个ToolBar

![](http://upload-images.jianshu.io/upload_images/739863-3a9a8281ea42cffa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/739863-e48d37b3fd9f656a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/739863-fc2d032ec86b5374.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](http://upload-images.jianshu.io/upload_images/739863-19028c86da18e33a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](http://upload-images.jianshu.io/upload_images/739863-40ecc648e1050f0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 控制按钮的行为，在CYWebViewController.m文件中

```objc
#import "CYWebViewController.h"
#import "CYSquare.h"

@interface CYWebViewController () <UIWebViewDelegate>
@property (weak, nonatomic) IBOutlet UIWebView *webView;
@property (weak, nonatomic) IBOutlet UIBarButtonItem *backItem;
@property (weak, nonatomic) IBOutlet UIBarButtonItem *forwardItem;
@end

@implementation CYWebViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    self.title = self.square.name;
    [self.webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:self.square.url]]];

    self.webView.backgroundColor = CYCommonBgColor;
    // 为了让webView的内容能完整显示，我们可以设置它的contentInset向下挪64
    // 但是webView是继承于UIView的，是无法设置它的contentInset的。但是它能滚动，是因为它里面镶嵌了一个ScrollView的属性
    // NS_CLASS_AVAILABLE_IOS(2_0) __TVOS_PROHIBITED @interface UIWebView : UIView <NSCoding, UIScrollViewDelegate>
    self.webView.scrollView.contentInset = UIEdgeInsetsMake(64, 0, 0, 0);
}

- (IBAction)back {
    [self.webView goBack];
}

- (IBAction)forward {
    [self.webView goForward];
}

- (IBAction)refresh {
    [self.webView reload];
}

#pragma mark - <UIWebViewDelegate>
- (void)webViewDidFinishLoad:(UIWebView *)webView
{
    self.backItem.enabled = webView.canGoBack;
    self.forwardItem.enabled = webView.canGoForward;
}
@end
```
- 为了让webView的内容能完整显示，我们可以设置它的contentInset向下挪64
- 但是webView是继承于UIView的，是无法设置它的contentInset的。但是它能滚动，是因为它里面镶嵌了一个ScrollView的属性
- NS_CLASS_AVAILABLE_IOS(2_0) __TVOS_PROHIBITED @interface UIWebView : UIView <NSCoding, 


- 前进和后退按钮的设置

```objc
#pragma mark - <UIWebViewDelegate>
- (void)webViewDidFinishLoad:(UIWebView *)webView
{
    self.backItem.enabled = webView.canGoBack;
    self.forwardItem.enabled = webView.canGoForward;
}
```
![](http://upload-images.jianshu.io/upload_images/739863-6ca3e940f53b5086.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](http://upload-images.jianshu.io/upload_images/739863-d9e2e2f3e8d0b931.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **扩展：有关于webView网页加载进度条**
  + iOS中只有苹果的Safari进度条加载才是真的，其它的浏览器（UC,百度等）进度条都是假的
  + 因为苹果内部是没有提供进度条的接口给外界，就算提供了也是私有的，也就是说只有苹果官方内部才可以用，如果你用了苹果监听进度的东西的话，你的App是不能上线的
  + 那么怎么做呢？你会发现像百度和UC这样的浏览器在加载时，进度显示会不停的加载，有时候网速不好，它也加载，加载到后面，你以为要加载完了，它就是不动，加载完，瞬间就上去了
  + 有些进度条加载是做得很逼真的，我们也可以做
      + 用一个UIView，高度为1或者2，加一个定时器，给一个时间，每隔1秒钟，让它的宽度不断的加。如果网速不行，就一直加载，一直加载，加到90%的时候，先停住不动，等网速来了，webViewDidFinishLoad:网页加载完后，让它的宽度等于屏幕的宽度，给它一个动画，让它过去，最后让它消失就可以了。
      + 在github上有一个框架，可以用它，也可以根据它的思路自己写一下

![](http://upload-images.jianshu.io/upload_images/739863-65a5c4e24e2799b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/739863-c75f24cba7274c53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
-----------------------------------------------------
-----------------------------------------------------
