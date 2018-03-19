# getNetworking
iOS如何获取手机当前的网络状态
获取iOS网络状态，我目前知道的有两种办法。

方法一：Reachability。

相信大家使用最多的方法就是使用Reachability

这是苹果的官方演示demo中使用到的方法。

1、首先你需要下载并导入Reachability。这是苹果官方演示demo，把里面的Reachability文件拷贝到自己的工程。下载地址：https://developer.apple.com/library/ios/samplecode/Reachability/Introduction/Intro.html

2、导入SystemConfiguration.framework框架。

3、分析reachability中的代码含义，可以看到以下三种网络状态：无网络，wifi和蜂窝网。

[html] view plain copy
typedef enum : NSInteger {  
    NotReachable = 0,//没有网络  
    ReachableViaWiFi,//当前使用Wifi网络  
    ReachableViaWWAN//使用的蜂窝网络  
} NetworkStatus;  
4、获取网络状态的代码

[html] view plain copy
#pragma mark - 获取网络状态  
+(NSString *)internetStatus {  
      
    Reachability *reachability   = [Reachability reachabilityWithHostName:@"www.apple.com"];  
    NetworkStatus internetStatus = [reachability currentReachabilityStatus];  
    NSString *net = @"wifi";  
    switch (internetStatus) {  
        case ReachableViaWiFi:  
            net = @"wifi";  
            break;  
              
        case ReachableViaWWAN:  
            net = @"wwan";  
            break;  
              
        case NotReachable:  
            net = @"notReachable";  
              
        default:  
            break;  
    }  
      
    return net;  
}  

值得一提的是HostName改成"www.baidu.com"或者其他中国网站时经常会获取网络状态错误，不能得到正确的网络状态。所以最好使用苹果的网站“www.apple.com
”

这种方法是目前最普遍的使用方式，由于是苹果官方demo，所以比较权威。但是这种方法的缺点是不能知道用户使用的手机网络是2G、3G还是4G。

这样就有了第二种获取网络状态的方法。


方法二：

这种方法通过监听手机的statusbar的状态还获取用户的网络状态。可以通过苹果的审核在Appstore上架。代码量非常小，简单易懂，最重要的是能区分2G、3G、4G、LTE。话不多说，直接上代码。

[html] view plain copy
+ (NSString *)networkingStatesFromStatebar {  
    // 状态栏是由当前app控制的，首先获取当前app  
    UIApplication *app = [UIApplication sharedApplication];  
      
    NSArray *children = [[[app valueForKeyPath:@"statusBar"] valueForKeyPath:@"foregroundView"] subviews];  
      
    int type = 0;  
    for (id child in children) {  
        if ([child isKindOfClass:[NSClassFromString(@"UIStatusBarDataNetworkItemView") class]]) {  
            type = [[child valueForKeyPath:@"dataNetworkType"] intValue];  
        }  
    }  
      
    NSString *stateString = @"wifi";  
      
    switch (type) {  
        case 0:  
            stateString = @"notReachable";  
            break;  
              
        case 1:  
            stateString = @"2G";  
            break;  
              
        case 2:  
            stateString = @"3G";  
            break;  
              
        case 3:  
            stateString = @"4G";  
            break;  
              
        case 4:  
            stateString = @"LTE";  
            break;  
              
        case 5:  
            stateString = @"wifi";  
            break;  
              
        default:  
            break;  
    }  
      
    return stateString;  
}  

不过需要注意的是，使用这种方法时一定要保证statusbar没有隐藏。如果你的App隐藏了statusbar，那么你也就不能通过这种方法获得网络状态。
