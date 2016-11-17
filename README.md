# iOS-MEMO
随记：遇到的iOS小问题及解决办法

## 目录
* [AFN在路径里含有中文或空格时的错误及解决](#1)
* [AFN超时设置](#2)
* [导航控制器pop到指定页面](#3)
* [关于iOS地图定位中点击设置中定位服务闪退问题](#4)
* [加入购物车红点贝塞尔曲线动画效果](#5)
* [取得当前设备启动图的确切名称](#6)
* [取消tableviewheader悬停](#7)
* [如果父视图半透明,如何使子视图不随着半透明](#8)
* [状态栏里的ActivityIndicator](#9)
* [cell的动态进入效果](#10)
* [Emoji表情上传与下载后显示转换](#11)
* [iOS系统功能调用](#12)
* [NSDictinary,NSArray,JSonString,NSData转换](#13)
* [scrollView(tableView)上下滑动渐变显示导航栏](#14)
* [webView缓存策略](#15)
* [WebView疑难杂症两则](#16)
* [xib自定义view初始化](#17)
* [IB里添加手势奔溃](#18)
* [点击webView跳转到safari](#19)


<a id="1"></a>AFN在路径里含有中文或空格时的错误及解决
```
Assertion failure in -[AFHTTPRequestSerializer requestWithMethod:URLString:parameters:error:]

path = [path stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
```
<a id="2"></a>AFN超时设置
```
AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
manager.requestSerializer.timeoutInterval = 300;
```
<a id="3"></a>导航控制器pop到指定页面
```
NSInteger index = 1;
UIViewController *destVC = self.navigationController.viewControllers[index];
[self.navigationController popToViewController:destVC animated:YES];
```
<a id="4"></a>关于iOS地图定位中点击设置中定位服务闪退问题
```
iOS8之后，如果应用中用到了地图定位，那么点击设置->隐私->定位服务 再点击该应用有时候会出现闪退问题，其原因是iOS8之后定位中添加了
NSLocationWhenInUseDescription和NSLocationAlwaysUsageDescription，需要在plist文件中添加这两个或其中一个，出现闪退是因为在plist中把这两个值设成了Boolean类型的，将其改成NSString就不会闪退了.
```
<a id="5"></a>加入购物车红点贝塞尔曲线动画效果
```
 UIImageView * action = (UIImageView *)[cell.contentView viewWithTag:10];

    CALayer *transitionLayer = [[CALayer alloc] init];

    [CATransaction begin];

    [CATransaction setValue:(id)kCFBooleanTrue forKey:kCATransactionDisableActions];

    transitionLayer.opacity = 1.0;

    transitionLayer.contents = action.layer.contents;

    transitionLayer.frame = [self.view convertRect:action.bounds fromView:action];

    [self.view.layer addSublayer:transitionLayer];

    [CATransaction commit];   

    //路径曲线

    UIBezierPath *movePath = [UIBezierPath bezierPath];

    [movePath moveToPoint:transitionLayer.position];//起点

    CGPoint toPoint = CGPointMake(30, 568-64-30); //终点

    [movePath addQuadCurveToPoint:toPoint  controlPoint:CGPointMake(26,transitionLayer.position.y-120)]; 

    //关键帧

    CAKeyframeAnimation *positionAnimation = [CAKeyframeAnimation animationWithKeyPath:@"position"];

    positionAnimation.path = movePath.CGPath;

    positionAnimation.removedOnCompletion = YES;

    CAAnimationGroup *group = [CAAnimationGroup animation];

    group.beginTime = CACurrentMediaTime();

    group.duration = 0.7;

    group.animations = [NSArray arrayWithObjects:positionAnimation,nil];

    group.timingFunction=[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut];

    group.delegate = self;

    group.fillMode = kCAFillModeForwards;

    group.removedOnCompletion = NO;

    group.autoreverses= NO;

    [transitionLayer addAnimation:group forKey:@"opacity"];
```
<a id="6"></a>取得当前设备启动图的确切名称
```
//取得当前设备启动图的确切名称

- (NSString *)launchImageName {
    
    NSString *name = nil;
    
    CGSize windowSize = [UIScreen mainScreen].bounds.size;    
    NSArray* images = [[NSBundle mainBundle] infoDictionary][@"UILaunchImages"];
    for (NSDictionary* info in images) {
        CGSize imageSize = CGSizeFromString(info[@"UILaunchImageSize"]);
        if ( CGSizeEqualToSize(imageSize, windowSize) ) {
            name = info[@"UILaunchImageName"];
            break;
        }
    }
    
    if (name == nil) {
        name = @"LetGo";
    }

    return name;
}

```
<a id="7"></a>取消tableviewheader悬停
```
改tableviewStlye为group类型，当然之后调整下显示。
```
<a id="8"></a>如果父视图半透明,如何使子视图不随着半透明
```
不用alpha，改用透明色即可
superView.backgroundColor = [[UIColor lightGrayColor] colorWithAlphaComponent:0.5];
```
<a id="9"></a>状态栏里的ActivityIndicator
```
[[UIApplication sharedApplication] setNetworkActivityIndicatorVisible:YES];
```
<a id="10"></a>cell的动态进入效果
```
在tableview的- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath这个代理里面实现:
static CGFloat initialDelay = 0.2f;
static CGFloat stutter = 0.06f;
cell.contentView.transform =  CGAffineTransformMakeTranslation(kFBaseWidth, 0);
[UIView animateWithDuration:1. delay:initialDelay + ((indexPath.row) * stutter) usingSpringWithDamping:0.6 initialSpringVelocity:0 options:0 animations:^{
                cell.contentView.transform = CGAffineTransformIdentity;
            } completion:NULL];
```
<a id="11"></a>Emoji表情上传与下载后显示转换
```
为了避免服务器奔溃，上传时转成Unicode编码
NSData *data = [emojiString dataUsingEncoding:NSNonLossyASCIIStringEncoding];
NSString *valueUnicode = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

从服务器获取后逆向来一遍
NSData *data = [string dataUsingEncoding:NSUTF8StringEncoding];
NSString *valueEmoj = [[NSString alloc] initWithData:data encoding:NSNonLossyASCIIStringEncoding];

以上方式在iOS手机间可以正常运作，但是无法和安卓统一~（处理机制不一样）——可以用GitHub上的一个库NSString＋Emojize，表情不全，勉强可用.
```
<a id="12"></a>iOS系统功能调用
```
1、调用 电话phone

[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"tel://4008008288"]];
 

2、调用自带 浏览器 safari

[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"http://www.abt.com"]];
 

3、调用 自带mail

[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"mailto://admin@abt.com"]];
 

4、调用 SMS

[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"sms://800888"]];
 

5，跳转到系统设置相关界面

1 [[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"prefs:root=WIFI"]];
其中，发短信，发Email的功能只能填写要发送的地址或号码，无法初始化发送内容，如果想实现内容的话，还需要更复杂一些，实现其各自的委托方法。

若需要传递内容可以做如下操作：

 1 //加入：MessageUI.framework
 2  #import <MessageUI/MFMessageComposeViewController.h>
 3  
 4 //实现代理：MFMessageComposeViewControllerDelegate
 5 
 6  //调用sendSMS函数  
 7 //内容，收件人列表  
 8 - (void)sendSMS:(NSString *)bodyOfMessage recipientList:(NSArray *)recipients  
 9 {   
11     MFMessageComposeViewController *controller = [[[MFMessageComposeViewController alloc] init] autorelease];   
13     if([MFMessageComposeViewController canSendText])   
15     {   
17         controller.body = bodyOfMessage;    
19         controller.recipients = recipients;   
21         controller.messageComposeDelegate = self;   
23         [self presentModalViewController:controller animated:YES];  
25     }      
27 }  
28    
29 // 处理发送完的响应结果  
30 - (void)messageComposeViewController:(MFMessageComposeViewController *)controller didFinishWithResult:(MessageComposeResult)result  
31 {  
32   [self dismissModalViewControllerAnimated:YES];   
34   if (result == MessageComposeResultCancelled)  
35     NSLog(@"Message cancelled")  
36   else if (result == MessageComposeResultSent)  
37     NSLog(@"Message sent")    
38   else   
39     NSLog(@"Message failed")    
40 }  
发送邮件的为：  

 1 //导入MFMailComposeViewController
 2 #import <MessageUI/MFMailComposeViewController.h>  
 3 //实现代理：MFMailComposeViewControllerDelegate  
 4    
 5 //发送邮件  
 6 -(void)sendMail:(NSString *)subject content:(NSString *)content{  
 7   MFMailComposeViewController *controller = [[[MFMailComposeViewController alloc] init] autorelease];  
 9     if([MFMailComposeViewController canSendMail])   
11     {   
13         [controller setSubject:subject];    
15         [controller setMessageBody:content isHTML:NO];    
17         controller.mailComposeDelegate = self;   
19         [self presentModalViewController:controller animated:YES];   
21     }      
22 }  
23    
24 //邮件完成处理  
25 -(void)mailComposeController:(MFMailComposeViewController *)controller didFinishWithResult:(MFMailComposeResult)result error:(NSError *)error{  
27     [self dismissModalViewControllerAnimated:YES];   
29     if (result == MessageComposeResultCancelled)  
30         NSLog(@"Message cancelled");  
31     else if (result == MessageComposeResultSent)  
32         NSLog(@"Message sent");   
33     else   
34         NSLog(@"Message failed");    
35    
36 }  
37    
默认发送短信的界面为英文的，解决办法为：在.xib 中的Localization添加一組chinese   
```
<a id="13"></a>NSDictinary,NSArray,JSonString,NSData转换
```
//字典、数组、JSonString - NSData
        
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
        NSDictionary *dic = @{@"key1":@"v1", @"key2":@"v2"};
        //options:NSJSONWritingPrettyPrinted,若改为0则下边的dicString是紧凑的不带空格和换行的形式
        NSData *data = [NSJSONSerialization dataWithJSONObject:dic options:NSJSONWritingPrettyPrinted error:nil];
        //options:若为0则dictionary为不可变版本
        NSDictionary *dictionary = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableContainers error:nil];
        NSLog(@"data - dictionary =%@",dictionary);
        NSString *dicString = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"dic String:  %@", dicString);
        
        NSArray *array = @[@11, @33, @59];
        //options:0，若改为NSJSONWritingPrettyPrinted则会加上回车和空格以适应漂亮打印
        NSData *arrData = [NSJSONSerialization dataWithJSONObject:array options:0 error:nil];
        NSArray *finalArray = [NSJSONSerialization JSONObjectWithData:arrData options:NSJSONReadingMutableContainers error:nil];
        NSLog(@"data - array : %@", finalArray);
        NSString *arrString = [[NSString alloc] initWithData:arrData encoding:NSUTF8StringEncoding];
        NSLog(@"arr String:  %@", arrString);
        
    }
    return 0;
}

```
<a id="14"></a>scrollView(tableView)上下滑动渐变显示导航栏
```
//scrollView(tableView)上下滑动渐变显示导航栏

- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
	CGFloat offsetY = scrollView.contentOffset.y;

        CGFloat criticalHeight = 75

        UIColor * naveColor =...;

        CGFloat alpha = offsetY / criticalHeight));

        self.navBar.backgroundColor = [naveColor colorWithAlphaComponent:alpha];

}
```
<a id="15"></a>webView缓存策略
```
NSString *stringUrl = …;
NSURL *url = [NSURL URLWithString:stringUrl];
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url cachePolicy:NSURLRequestReturnCacheDataElseLoad timeoutInterval:15];
[webView loadRequest:request];
```
<a id="16"></a>WebView疑难杂症两则
```
1.自适应内容高度

//设置内容，这里包装一层div，用来获取内容实际高度（像素），htmlcontent是html格式的字符串	
  NSString * htmlcontent = [NSString stringWithFormat:@"<div id=\"webview_content_wrapper\">%@</div>", htmlcontent];
  [_webView loadHTMLString:htmlcontent baseURL:nil];

//delegate的方法重载//
- (void)webViewDidFinishLoad:(UIWebView *)webView {
  //获取页面高度（像素）
  NSString * clientheight_str = [webView stringByEvaluatingJavaScriptFromString: @"document.body.offsetHeight"];
  float clientheight = [clientheight_str floatValue];
  //设置到WebView上
  webView.frame = CGRectMake(0, 0, self.view.frame.size.width, clientheight);
  //获取WebView最佳尺寸（点）
  CGSize frame = [webView sizeThatFits:webView.frame.size];
  //获取内容实际高度（像素）
  NSString * height_str= [webView stringByEvaluatingJavaScriptFromString: @"document.getElementById('webview_content_wrapper').offsetHeight + parseInt(window.getComputedStyle(document.getElementsByTagName('body')[0]).getPropertyValue('margin-top'))  + parseInt(window.getComputedStyle(document.getElementsByTagName('body')[0]).getPropertyValue('margin-bottom'))"];
  float height = [height_str floatValue];
  //内容实际高度（像素）* 点和像素的比
  height = height * frame.height / clientheight;
  //再次设置WebView高度（点）
  webView.frame = 。。。
}

2.去除webview底部黑线～

    webView.opaque = NO;

    webView.backgroundColor = [UIColor whiteColor];

```
<a id="17"></a>xib自定义view初始化
```
#import <UIKit/UIKit.h>

@interface SATourView : UIView

@property (strong, nonatomic) IBOutlet UIView *view;
@property (weak, nonatomic) IBOutlet UILabel *mainTitleLabel;
@property (weak, nonatomic) IBOutlet UIImageView *mainImage;

@end

#import "SATourView.h"

@implementation SATourView


- (instancetype)initWithFrame:(CGRect)frame {

    self = [super initWithFrame:frame];

    if (self) {

        // 1. Load .xib

        // 2. Adjust bounds

        // 3. Add as a subview

        [[NSBundle mainBundle] loadNibNamed:@"SATourView" owner:self options:nil];

        NSLog(@"frame = %@", NSStringFromCGRect(self.view.bounds));
        NSLog(@"bounds = %@", NSStringFromCGRect(self.bounds));

        self.bounds = self.view.bounds;

        [self addSubview:self.view];
    }
    return self;
}


- (id)initWithCoder:(NSCoder *)aDecoder {

    self = [super initWithCoder:aDecoder];

    if (self) {
        [[NSBundle mainBundle] loadNibNamed:@"SATourView" owner:self options:nil];

        [self addSubview:self.view];
    }

    return self;
}

- (IBAction)joinButtonTapped:(id)sender {

    NSLog(@"joinButtonTapped");
}


@end

SATourView *view1 = [[SATourView alloc] initWithFrame:CGRectMake(0, 0, width, height)];
    view1.bounds = self.view.bounds;
    view1.mainTitleLabel.text = @"Новый текст";

    [self.view addSubview:view1];


xib自定义view2初始化＋IBInspectable
#import <UIKit/UIKit.h>
@interface DraggableView : UIView

@property (nonatomic, assign) IBInspectable NSInteger cornerRadius;

@end

#import "DraggableView.h"

@interface DraggableView ()

@property (weak, nonatomic) IBOutlet UILabel *leftLabel;
@property (weak, nonatomic) IBOutlet UILabel *rightLabel;

@end

@implementation DraggableView

- (instancetype)initWithCoder:(NSCoder *)aDecoder {

    if (self = [super initWithCoder:aDecoder]) {

        [self load];
    }
    return self;
}

- (instancetype)initWithFrame:(CGRect)frame {

    if (self = [super initWithFrame:frame]) {

        [self load];
    }
    return self;
}

- (void)load {
    UIView *view = [[[NSBundle bundleForClass:[self class]] loadNibNamed:@"DraggableView" owner:self options:nil] firstObject];
    view.frame = self.bounds;

    [self addSubview:view];

    self.leftLabel.text = @"works";
    self.rightLabel.text = @"done :)";
}

- (void)setCornerRadius:(NSInteger)cornerRadius
{
    self.layer.cornerRadius = cornerRadius;
    self.clipsToBounds = YES;
}

@end


#import "ViewController.h"

@interface ViewController ()
@property (weak, nonatomic) IBOutlet NSLayoutConstraint *draggableViewHeightConstraint;
@property (weak, nonatomic) IBOutlet NSLayoutConstraint *draggableViewWidthConstraint;
@end

@implementation ViewController
#pragma mark - View lifecycle

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
}

- (void)viewDidLoad {
    [super viewDidLoad];

    self.draggableViewHeightConstraint.constant
    = 200;
    self.draggableViewWidthConstraint.constant = 200;
}
@end
```
<a id="18"></a>IB里添加手势奔溃
```
在xib或者storyBoard里添加单击手势识别者导致奔溃，目前是自定义tableViewCell遇到
解决：
用代码添加手势识别者。
```
<a id="19"></a>点击webView跳转到safari
```
某个页面里嵌入了webview，本来只是用来展示静态html内容，但有一天市场部的需要里边有链接。点击链接后去那里？安卓会用本地浏览器打开，所以产品经理说iOS也该用safari打开。
奇葩需求，不过实现倒是不难。在代理方法里拦截url即可。

- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    // 若用户点击了链接，用safari打开
    NSString *url = request.URL.absoluteString;
    if ([url isEqualToString:@"about:blank"]) { return YES; }
    [[UIApplication sharedApplication]openURL:[NSURL URLWithString:url]];
    return NO;
}
注意，一开始就是加载的静态html文本，所以url是"about:blank"，这个放行，让这些内容加载显示。
当用户点击了某个链接后，会再次来到这个方法，拦截url，跳转到safari打开即可。
```
