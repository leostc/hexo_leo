---
layout: post
title: "通过description实现nslog打印对象内的变量"
date: 2015-05-04 23:23:52 +0800
comments: true
categories: iOS
---
## description
从《Effective Objective-C 2.0》知道了可以实现对象的descriptionn方法，就可以改变NSLog打印对象的结果。

通常我们用NSLog打印自定义的类时，在控制台输出的结果往往是<BaseModel : 0x7f9f306259b0>，这样的内容对于调试程序的时候，用处不大。除非我们覆写了自定义类的description方法，否则打印信息的时候就会调用NSObject类所实现的默认方法。

书中提到可以创建一个字典，把类中的所有变量放进去，在description返回这个字典，那么打印的时候，能自动将变量编排好并输出。

## runtime
虽然采用书中提到的方法，能很大缓解我们调试打印时的压力，但我要每个类的description的时候，都得创建一个字典，并把每个变量都放进去，感觉好麻烦！

于是我想到利用运行时，可以获取到自定义对象的变量表，循环遍历一遍就能将对象中的变量通通塞进字典中，就能达到自动实现自定义类的description方法了。

```objectivec
Ivar * class_copyIvarList(Class cls, unsigned int *outCount)
```
返回一个数组，里面的元素为Ivar的指针类型，指向了类中定义的变量，不过不包含父类的对象。用完之后必须释放数组free()。

```objectivec
const char * ivar_getName( Ivar ivar)
```
返回一个C的string，为Ivar的变量名。

```objectivec
id object_getIvar(id object, Ivar ivar)
```
返回ivar里的值，但是发现该方法只能获取到objective-c的对象，对于如基础数据类型（例如NSInteger, int, float, double, char等）都会报错。

```objectivec
Ivar object_getInstanceVariable(id obj, const char *name, void **outValue)
```
对于基础的数据类型，该方法中outValue可以正常返回，但需要注意的是，本方法得在为开启ARC的工程下才能实现，所以在ARC的工程下得对文件配上-fno-objc-arc的编译设置。

```objectivec
const char * ivar_getTypeEncoding( Ivar ivar)
```
返回一个C的string，为Ivar的变量的类型。通过该方法，我们就能判断出哪些变量是否为基础数据类型了。

全部代码如下：
```objectivec
-(NSString *)description{
    unsigned int numIvars = 0;
    Ivar * ivars = class_copyIvarList([self class], &numIvars);
    NSMutableDictionary *dic = [NSMutableDictionary dictionaryWithCapacity:numIvars];
    for(int i = 0; i < numIvars; i++) {
        Ivar thisIvar = ivars[i];
        NSString* key = [NSString stringWithUTF8String:ivar_getName(thisIvar)];
        const char *type = ivar_getTypeEncoding(thisIvar);
        NSString *stringType =  [[NSString stringWithCString:type encoding:NSUTF8StringEncoding] lowercaseString];
        if ([stringType hasPrefix:@"@"]) {
            id obj = object_getIvar(self, thisIvar);
            if (obj) {
                [dic setObject:obj forKey:key];
            } else {
                [dic setObject:@"nil" forKey:key];
            }
        } else if([stringType hasPrefix:@"f"]) {
            float value;
            object_getInstanceVariable(self, ivar_getName(thisIvar), (void*)&value);
            [dic setObject:@(value) forKey:key];
        } else if([stringType hasPrefix:@"d"]) {
            CGFloat value;
            object_getInstanceVariable(self, ivar_getName(thisIvar), (void*)&value);
            [dic setObject:@(value) forKey:key];
        } else if([stringType hasPrefix:@"q"]) {
            NSInteger value;
            object_getInstanceVariable(self, ivar_getName(thisIvar), (void*)&value);
            [dic setObject:@(value) forKey:key];
        } else if([stringType hasPrefix:@"i"]) {
            int value;
            object_getInstanceVariable(self, ivar_getName(thisIvar), (void*)&value);
            [dic setObject:@(value) forKey:key];
        } else if([stringType hasPrefix:@"b"]) {
            BOOL value;
            object_getInstanceVariable(self, ivar_getName(thisIvar), (void*)&value);
            [dic setObject:value?@"YES":@"NO" forKey:key];
        } else if([stringType hasPrefix:@"c"]) {
            char value;
            object_getInstanceVariable(self, ivar_getName(thisIvar), (void*)&value);
            [dic setObject:[NSString stringWithFormat:@"%c",value] forKey:key];
        } else if([stringType hasPrefix:@"*"]) {
            char* value;
            object_getInstanceVariable(self, ivar_getName(thisIvar), (void*)&value);
            [dic setObject:[NSString stringWithUTF8String:value] forKey:key];
        } else {
            [dic setObject:[NSString stringWithFormat:@"cannott recoginze %@",stringType] forKey:key];
        }
    }
    free(ivars);
    
    return [NSString stringWithFormat:@"<%@ : %p, %@>",
            [self class],
            self,
            dic];
}
```

## 使用
那么如何加入到自己的工程中呢，我想到的是，给Model的基类添加Category，覆写其description方法，这样就能在所有的Model基类的派生类中都可以用NSLog打印出我们想要的结果啦

示例工程已经上传github上了，[LHModelDescription](https://github.com/leostc/LHModelDescription)
