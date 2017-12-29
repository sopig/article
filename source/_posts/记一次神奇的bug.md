title: 记一次神奇的bug
photos:
  - null
date: 2015-04-26 12:25:12
tags: bug总结
---

临近发布,Xcode打出的包在不同的设备效果不同,对比下来发现:  
iPhone5s及以上完全正常,iphone5及以下部分界面数据显示不完整,非常奇怪; 


__过程__ 

首先怀疑界面约束问题,因为不同版本的Xcode支持的约束不同,特别排查了那些运行时log提示的那些界面,发现不是根本问题,用FLEX排查,发现控件都存在,真的是数据不见了; 排查中间层返回数据值,JSON一切正常;但是解析出的对象4个属性的确少了一个,定位到网 络解析库; 网络framework库内创建一个demo可运行app,在iPhone5内解析一段JSON,最终定位到一行代码:
```Objective-C
const char *propertyAttributes = property_getAttributes(property);
}
BOOL isReadWrite = YES;
isReadWrite = !(BOOL)strstr(propertyAttributes, ",R");
isReadWrite = (BOOL)strstr(propertyAttributes, ",V");
if (isReadWrite) {
//...
}
```

代码的本意是判断当前属性是否为readonly且不可赋值的属性,不对其进行KVC赋值; 但是这段代码有一个非常严重的问题就是strstr函数的返回其实是匹配字符串的地址,不匹配 返回NULL;
既然是地址,强转bool其实是风险的,最终代码修改为:
```Objective-C
const char *propertyAttributes = property_getAttributes(property);
}
BOOL isReadWrite = YES;
isReadWrite = (strstr(propertyAttributes, ",R") == NULL);
isReadWrite = (strstr(propertyAttributes, ",V") != NULL);
if (isReadWrite) {
//...
}
```

注:判断",R"是因为调试放着,主要的判断还是",V";
解析
查看BOOL在runtime内的定义:
```
/// Type to represent a boolean value.
#endif
#if !defined(OBJC_HIDE_64) && TARGET_OS_IPHONE && __LP64__
typedef bool BOOL;
#else
typedef signed char BOOL;
// BOOL is explicitly signed so @encode(BOOL) == "c" rather than "C"
// ev
```
在64位下位bool类型,其他为char;即转换十六进制时,最后2位正好位00,32位下取值为 NO,而64位,非0就是true;
这也证明了iphone5及以上正常;
