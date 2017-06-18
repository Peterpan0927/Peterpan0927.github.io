---
title: iOS-UI基础(七)
date: 2017-06-15 14:34:47
tags: 图片拉伸 通知 虚拟键盘
categories: TableView基础达成
author: Peterpan
---

iOS-UI基础（七）
<!--more-->

# UI基础Day8



---



## 1.实例化方法调用顺序

```objective-c
//在view的内部会首先调用initWithFrame：传递的frame全为0
TestView *test1 = [[TestView alloc] init];
//调用initWithFrame的时候，只走这一个方法
TestView *test2 = [[TestView alloc] initWithFrame:CGRectZero];
```



## 2.在完成QQ时需要注意的问题



1.在将一个类作为另一个类的属性，需要调用的时候，这个属性应该设置为强引用，而不是弱引用类型，不然在调用他的时候存放数据的内存空间已经被释放掉了，查看内存的时候会发现这个类中的属性的内存地址全是nil。

```objective-c
@property (nonatomic, strong) QQModel *qqmodel;
//如果将这个设置为weak，那么在调用frameModel.qqmodel时，qqmodel的内存地址就是0。
```



2.在设置子控件的frame的时候，注意不要把高度设置的过大，如果设置为整个屏幕的高度，会出现一些无厘头的情况，而且排错的时候可能会误认为是cell重用时的错，比较浪费时间。



3.在设置文本的时候要设置它的真实宽高。

```objective-c
CGFloat maxWidth = kScreenSize.width - 2 * margin -2 * iconWidth;
    CGSize maxSize = CGSizeMake(maxWidth, MAXFLOAT);
    
    NSDictionary *dict = @{NSFontAttributeName:[UIFont systemFontOfSize:17]} ;
    
    CGSize textRealSize = [qqmodel.text boundingRectWithSize:maxSize options:NSStringDrawingUsesLineFragmentOrigin attributes:dict context:nil].size;
```



4.图片的拉伸

在对图片进行拉伸的时候最好尽量靠近它的中心点

```objective-c
//抽取图片拉伸的方法
-(UIImage *)resizeImage:(NSString *)imageName{
    UIImage *image = [UIImage imageNamed:imageName];
    
    CGFloat halfWidth = image.size.width/2;
    
    CGFloat halfHeight = image.size.height/2;
    
    UIImage *resizeImage = [image resizableImageWithCapInsets:UIEdgeInsetsMake(halfHeight, halfWidth, halfHeight, halfWidth) resizingMode:UIImageResizingModeStretch];
    
    return resizeImage;	
  
//  UIImageResizingModeTile,  平铺
//  UIImageResizingModeStretch, 拉伸
```

图片的拉伸除了可以用代码的方法，还可以通过slicing的方式，选中Assets.xcassets中你要拉伸的那一张图，右下角有一个Show Slicing的方式可供选择，点击之后在下图中的部分修改其拉伸的方式等各种设置：

![Slicing.png](http://omunhj2f1.bkt.clouddn.com/Slicing.png)

slicing其实就是可视化的图形拉伸：![slicing1.png](http://omunhj2f1.bkt.clouddn.com/slicing1.png)

5.设置内边距

没有设置内边距：![内边距.png](http://omunhj2f1.bkt.clouddn.com/%E5%86%85%E8%BE%B9%E8%B7%9D.png)

设置好了内边距，但是没有增加宽高：![屏幕快照 2017-06-14 下午12.00.42.png](http://omunhj2f1.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-14%20%E4%B8%8B%E5%8D%8812.00.42.png)

所以，不管怎么样增加button的宽高，如果没有设置内边距，文字有时候还是会超出背景图片。

```objective-c
//设置内边距
contentButton.contentEdgeInsets = UIEdgeInsetsMake(20, 20, 20, 20);
//增加button的宽高
#define kDeltamargin 40
_textFrame.size.width += kDeltamargin;
_textFrame.size.height += kDeltamargin;
```



6.timeLabel的显示问题

当消息发出的时间和上一条消息发出的时间相同的时候，不需要显示两者，此时隐藏掉后者即可。

```objective-c
//记录timeLabel中的cell是否需要隐藏掉,将其设置为qqmodel的一个属性
@property (nonatomic, assign, getter=isHiddenTimeLabel) BOOL hiddenTimeLabel;
//在将数据写入可变数组的时候将是否隐藏传进去
 
        for(NSDictionary *dict in tempArray){
            
            QQModel *model = [QQModel qqModelWithDict:dict];
            
            QQFrameModel *lastFrameModel = self.dataArray.lastObject;
            
            if([model.time isEqualToString:lastFrameModel.qqmodel.time]){
                model.hiddenTimeLabel = YES;
            }
            
            QQFrameModel *frameModel = [[QQFrameModel alloc] init];
            
            frameModel.qqmodel = model;
            
            [_dataArray addObject:frameModel];
        }
```



7.用枚举类型让数据变的有意义

为了让用户和其他人的类型代码更好理解，可以使用枚举类型。

```objective-c
//通过QQUserType的枚举类型来代替之前的0和1，让更好理解代码
typedef NS_ENUM(NSInteger(内部的值类型),QQUsertype（枚举的名称）){
    QQUsertypeOther,//如果不声明，默认是从零开始的
    QQUsertypeMe,
};
```



8.再点击输入框的时候，输入框没有和键盘一起向上移动

可以使用平移的方式来让输入框向上移动

view在平移的时候，必须要知道键盘从出来到停止的时间

键盘平移的距离就是键盘的高度，这个时候就是用通知的方式来完成键盘和输入框的同步。

```objective-c
- (void)registerNotification{
  //添加点击键盘时候的监听者
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillAppear:) name:UIKeyboardWillShowNotification object:nil];
  //收起键盘是的监听者
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillDisappear:) name:UIKeyboardWillHideNotification object:nil];
}
//输入的时候调用的方法
- (void)keyboardWillAppear:(NSNotification *)noti{
    
    NSDictionary *dict = noti.userInfo;
    
    NSTimeInterval interval = [dict[UIKeyboardAnimationDurationUserInfoKey] doubleValue];
    
    // 键盘的高度
    // 停止后的Y值
    CGRect keyboardRect = [dict[UIKeyboardFrameEndUserInfoKey] CGRectValue];
    
    CGFloat keyboardEndY = keyboardRect.origin.y;
    
    // 没出现时的Y值
    CGRect tempRect = [dict[UIKeyboardFrameBeginUserInfoKey] CGRectValue];
    
    CGFloat keyboardBeginY = tempRect.origin.y;
    
    // 对 tableView  执行动画， 向上平移
    [UIView animateWithDuration:interval animations:^{
        
        self.view.transform = CGAffineTransformMakeTranslation(0, (keyboardEndY - keyboardBeginY));
    }];

}

```



9.在选中cell的时候，cell会变色，看起来不真实，此时在初始化cell的时候就可以设置cell的属性，使其选中不变色：

```objective-c
 cell.selectionStyle = UITableViewCellSelectionStyleNone;
```



10.一般的qq在打开的时候都是滚动到最底层，所以这里在加载的时候也需要这样，在ViewDidLoad加入下面这个方法：

```objective-c
- (void)scrollToButtom{
    NSIndexPath *indexPath = [NSIndexPath indexPathForRow:self.dataArray.count-1 inSection:0];
    
    [_tabelView scrollToRowAtIndexPath:indexPath atScrollPosition:UITableViewScrollPositionBottom animated:YES];
}
```



## textField应用

```objective-c
//设置提示文字
_textField.placeholder = @"请输入用户名";
//设置leftView的时候，采用直接赋值的方法是看不到的
	UIView *leftView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 30, 30)];
    leftView.backgroundColor = [UIColor redColor];
    test.leftView = leftView;
    test.leftViewMode = UITextFieldViewModeWhileEditing;
/*
	UITextFieldViewModeNever,   绝不显示，默认的值
    UITextFieldViewModeWhileEditing,  当编辑时显示
    UITextFieldViewModeUnlessEditing,  如果编辑，就不显示了
    UITextFieldViewModeAlways 总是显示
*/
```

##### 输入框状态介绍：

clear button同样也有上述四种状态：![屏幕快照 2017-06-14 下午5.21.11.png](http://omunhj2f1.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-14%20%E4%B8%8B%E5%8D%885.21.11.png)

如果选择显示就是出现右方的那个按钮，可以快速清空输入框

而安全文本输入则是让你的输入不回显。

```objective-c
test.clearButtonMode = YES;
test.secureTextEntry = YES;
```



## 通知简介

![通知示意图.png](http://omunhj2f1.bkt.clouddn.com/%E9%80%9A%E7%9F%A5%E7%A4%BA%E6%84%8F%E5%9B%BE.png)



- 每一个应用程序都有一个通知中心（NSNotificationCenter）实例，专门负责协助不同对象之间的消息通信

- 任何一个对象都可以向控制中心发布通知，描述自己在做什么。其他感兴趣的对象（Observer）可以申请在某个特定通知发布时（或者在某个特定对象发布通知时）收到这个通知。

  ![屏幕快照 2017-06-14 下午10.33.46.png](http://omunhj2f1.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-14%20%E4%B8%8B%E5%8D%8810.33.46.png)

  ![屏幕快照 2017-06-14 下午10.35.51.png](http://omunhj2f1.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-14%20%E4%B8%8B%E5%8D%8810.35.51.png)

使用通知的流程：

1.添加监听者

2.在适当的时候发布通知

3.当监听对象被销毁的时候，一定要从通知中心把它移除掉。

同时要注意发布消息时的属性设置问题。

![发布通知的时候obj 和name不能为空.png](http://omunhj2f1.bkt.clouddn.com/%E5%8F%91%E5%B8%83%E9%80%9A%E7%9F%A5%E7%9A%84%E6%97%B6%E5%80%99obj%20%E5%92%8Cname%E4%B8%8D%E8%83%BD%E4%B8%BA%E7%A9%BA.png)



```objective-c
//以订阅漫画为例子
#import <Foundation/Foundation.h>
#import "Person.h"
#import "Company.h"
int main(int argc, const char * argv[]) {
    @autoreleasepool {
       
        Company *souhu = [[Company alloc] init];
        souhu.companyName = @"搜狐";
        souhu.movieName = @"全职猎人";
        
        Company *tudou = [[Company alloc] init];
        tudou.companyName = @"土豆";
        tudou.movieName = @"极品家丁";
        
        Person *li = [[Person alloc] init];
        li.name = @"离";
        
        Person *zhangsan = [[Person alloc] init];
        zhangsan.name = @"张三";
        
#warning 1.注册监听
        [[NSNotificationCenter defaultCenter] addObserver:zhangsan selector:@selector(recieveNotification:) name:@"hunter" object:souhu];
        
        [[NSNotificationCenter defaultCenter] addObserver:li selector:@selector(recieveNotification:) name:@"renzhe" object:tudou];
#warning 2.发布通知
        [[NSNotificationCenter defaultCenter] postNotificationName:@"hunter" object:souhu userInfo:@{@"company":souhu}];
        [[NSNotificationCenter defaultCenter] postNotificationName:@"renzhe" object:tudou userInfo:@{@"company":tudou}];
    }
    return 0;
}
//Person.m中的部分
#import "Person.h"
#import "Company.h"
@implementation Person
-(void)recieveNotification:(NSNotification *)noti{
    //取出userInfo
    NSDictionary *dict = noti.userInfo;
    //取出公司信息
    Company *company = dict[@"company"];
    
    NSLog(@"%@,订阅了%@, %@ 更新了",self.name, company.companyName, company.movieName);
}

#warning 3.移除监听
-(void)dealloc{
    [[NSNotificationCenter defaultCenter]removeObserver:self];
}
@end
```



## 给QQ增添消息和自动回复

所有的消息显示首先都是从对象数组中读取的，所以如果要把消息显示到屏幕上需要把我们要添加的消息加入到数组之中，也就是重新声明一个新的对象；

```objective-c
//当点击键盘右下角的return按钮时就会调用这个方法
- (BOOL)textFieldShouldReturn:(UITextField *)textField {
    static NSInteger x = 1;
    // 撤销 ， textField 的第一响应者身份
    [textField resignFirstResponder];
    //发送自己的消息
    [self sendMessage:textField.text andType:QQUsertypeMe];
    //设置自动回复
    NSString *string = [NSString stringWithFormat:@"这是第%ld次,你是不是傻",x];
    [self sendMessage:string andType:QQUsertypeOther];
    //清空textField中的数据
    textField.text = @"";
    x += 1;
    return YES;
}
//发送信息调用的方法
- (void)sendMessage:(NSString *)text andType:(QQUsertype)type{
    QQModel *qqmodel = [[QQModel alloc] init];
    //取出当前时间
    NSDate *currentDate = [NSDate date];
    //设置时间格式
    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
    
    formatter.dateFormat = @"HH:mm";
    
    NSString *dateString = [formatter stringFromDate:currentDate];
    
    qqmodel.time = dateString;
    
    qqmodel.text = text;
    
    qqmodel.type = type;
    
    QQFrameModel *lastFrameModel = self.dataArray.lastObject;
    
    if([qqmodel.time isEqualToString:lastFrameModel.qqmodel.time]){
        qqmodel.hiddenTimeLabel = YES;
    }

    //把qqmodel赋值给qqFrameModel，要根据内容计算控件的frame以及cell的高度
    QQFrameModel *frameModel = [[QQFrameModel alloc] init];
    
    frameModel.qqmodel = qqmodel;
    
    [self.dataArray addObject:frameModel];
    
    NSIndexPath *indexPath = [NSIndexPath indexPathForRow:self.dataArray.count-1 inSection:0];
    //动画的方式加载新添加的数据
    [_tabelView insertRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationLeft];
    //滚动到最后一行
    [_tabelView scrollToRowAtIndexPath:indexPath atScrollPosition:UITableViewScrollPositionBottom animated:YES];
}
```

[源码仓库](https://github.com/Peterpan0927/iOS-test/tree/master/QQchat)