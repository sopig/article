title: '线上缺陷原因总结'
date: 2015-12-27 01:03:20
tags: 架构谈
---

## 问题
关于特殊情况下会重复弹出分享框的问题。

## 原因
* url重定向会造成此问题
* web页面js注入会造成此问题


## 代码逻辑
```shell

//这个方法是正在开始加载网页请求链接的时候的回调方法，在这个回调做的native代码逻辑如下
//1.将js执行状态设置为begin

- (void)webViewDidStartLoad:(UIWebView *)webView{
    jsstatus = JSStatusBegin;
    if (webView.isLoading) {
        return;
    }
    self.navigationItem.rightBarButtonItem = nil;
}  
```

```shell
//这个是完成网页加载请求，网页完全显示时的回调方法，代码逻辑是
//尝试调用js方法 getShareData(x,x); 
//如果活动页存在这个方法，将js执行状态设置为end，并且将活动页分享按钮添加到导航栏上
//如果活动页不存在 getShareData(x,x)这个方法，将js执行状态设置为end，活动页不会有分享按钮
//弹出分享框的代码是因为  
// 1. url重定向导致两次调用这两个回调方法
// 2. js远程注入也会导致js注入的时候再次调用这两个回调方法


- (void)webViewDidFinishLoad:(UIWebView *)webView{

    if (webView.isLoading) {
        return;
    }
    
    if (IOS7_OR_LATER) {
       
        self.context = [_webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
        self.context[@"cart"] = self;
    
        if (!self.webTitle || [self.webTitle isEqualToString:kJXWebDefaultTitle]) {
            JSValue *value = [self.context evaluateScript:@"document.title"];
            self.title = value.toString;
        }
        
        [self addShareData];
    
    }
    
    
    [[NSUserDefaults standardUserDefaults] setInteger:0 forKey:@"WebKitCacheModelPreferenceKey"];
    [[NSUserDefaults standardUserDefaults] setBool:NO forKey:@"WebKitDiskImageCacheEnabled"];
    [[NSUserDefaults standardUserDefaults] setBool:NO forKey:@"WebKitOfflineWebApplicationCacheEnabled"];
    [[NSUserDefaults standardUserDefaults] synchronize];
    

}
```


## 问题排查
* 微信群里第一时间反馈这个问题的时候，第一时间就解决了。鉴于经验和测试环境所限，代码没有考虑以下两种情况  
1.url重定向造成的问题  
2.远程js注入造成的问题

## 结果
代码已修复，问题需总结，测试环境尽可能多样性
