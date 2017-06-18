---
title: iOS-UI学习(二)
date: 2017-05-12 20:12:01
tags: iOS UI
categories: MVC初步 xib与storyboard

---
UI初步（二）
<!--more-->
# UI基础Day3

## 1.代码生成多个图片的方法（多重循环）

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    CGFloat Width = 50;
    CGFloat Height = 25;
    CGFloat margin = (CGRectGetMaxX(self.view.frame)-5*Width)/6;
    for (int i = 0; i < 15; i++) {
        NSInteger x = i/5;
        NSInteger y = i%5;
        CGFloat viewx = (y+1)*margin + y*Width;
        CGFloat viewy = (x+1)*margin + x*Height;
        UIView *imageview = [[UIView alloc] initWithFrame:CGRectMake(viewx, viewy, Width, Height)];
        imageview.tag = i;
        [imageview setBackgroundColor:[UIColor blueColor]];
        [self.view addSubview:imageview];
    }
}
```

## 2.在View中嵌套图片和文字,以下代码均写在for循环中

![](http://omg5mjb8v.bkt.clouddn.com/154562D5-9767-486C-98B2-F24030E30966.png)

```objective-c
nameLabel.text = @"xxxx";
//设置文本内容
nameLabel.textAlignment = NSTextAlignmentCenter;
//设置对齐的模式
nameLabel.font = [UIFont boldSystemFontOfSize:15];
//设置字体
/*
boldSystemFontOfSize  :   加粗体
systemFontOfSize      :   默认体
italicSystemFontOfSize:   斜体
*/
```

## 3.button属性（在按钮up的时候button的highlighted状态会被clear）

设置background image ,image,title要分状态设置

> 默认
>
> 高亮 —> 按钮被点击的时候，自动切换成高亮状态
>
> 被选中—>设置button的selected 属性为Yes
>
> 被禁用—>设置button的enabled 属性为NO 不可用

如果想点击之后一直保持在高亮状态的话，需要进行属性的设置：

```objective-c
- (void)highlightButton:(UIButton *)b {   
    [b setHighlighted:YES];  
}  
  
 - (IBAction)onTouchup:(UIButton *)sender {  
    [self performSelector:@selector(highlightButton:) withObject:sender afterDelay:0.0];  
}  
//最后将按钮和方法连线
```



## 4.不可变数组和可变数组

1.不可变数组：内部元素不可修改，防止被不正当修改

2.可变数组：相对而言，更消耗性能

## 5.注意，如果想使用懒加载加载数据的话，必须要用self.重写的getter方法，不然直接访问的是这个对象，而不是通过getter方法。

## 6.可以通过plist文件来优化

当需要遍历多个内部某些属性不同的模块时，可以通过在plist数组或字典来实现，只需要在代码中读取plist文件中的内容即可。

## 7.循环时算法的优化

```objective-c
for(int i = 0;i < self.dataArray.count;i++){
  //其中行索引和列索引分别由（i/列数）和（i%列数）来替代
} 
```

![](http://omg5mjb8v.bkt.clouddn.com/5E87432E-75AB-4A46-95F2-EBEA63384130.png)

## 8.字典转模型

![示例图](http://omg5mjb8v.bkt.clouddn.com/0B00C653-FEFA-49C9-96C5-0EBC9FC5E82A.png)

![](http://omg5mjb8v.bkt.clouddn.com/688A2E58-81CA-4887-9BC8-F67579FCF3B7.png)

```objective-c
//Appmodel.h中的代码
@property(nonatomic, copy)NSString *name;
@property(nonatomic, copy)NSString *icon;
-(instancetype)initWithDict:(NSDictionary *)dict;
+(instancetype)appModelWithDict:(NSDictionary *)dict;
//Appmodel.m中的代码
-(instancetype)initWithDict:(NSDictionary *)dict{
  if(self = [super init]){
    self.name = dict[@"name"];
    self.icon = dict[@"icon"];
  }
  return self;
}
+(instancetype)appModelWithDict:(NSDictionary *)dict{
  return [[self alloc] initWithDict:dict];
}
}
/*
注意在写接口的时候，
创建实例方法的同时也要创建一个类方法，
是一个约定俗成的规矩,
同时在写类方法的时候也要考虑到之后继承和多态的问题。
同时，self在哪个类里就表示这个类自身，
谁调用的方法，这个self就表示谁。
图片中的代码也可以优化一下了
*／
```

## 9.id和instancetype的区别

1.id和instancetype都可以作为返回值

2.instancetype不可以作为参数类型，但是它可以检测返回类型。

3.在ARC(Auto Reference Count)环境下:instancetype用来在编译期确定实例的类型,而使用id的话,编译器不检查类型, 运行时检查类型.

4.在MRC(Manual Reference Count)环境下:instancetype和id一样,不做具体类型检查

## 10.xib的使用

1.在添加新文件中找到View旁边的Empty添加进项目目录中。

2.在项目文件中xib文件是以.xib结尾，安装之后，就自动变成了以.nib

3.可以在xib中布置你想要的Label,Button，再加载进函数中，可以缩减代码部分。

![](http://omg5mjb8v.bkt.clouddn.com/7873B694-004F-4A7D-A163-645AED229C02.png)

4.xib是一个空的view,加载xib的方法：

```objective-c
UIView *yellowview = [[[NSBundle mainBundle] loadNibNamed:@"xib的文件名" owner:nil options:nil] lastObject];
```

frame: 是以父控件的左上角为坐标原点

bounds:是以自身的左上角为坐标原点

## 11.xib和class关联以及简单mvc

1.xib里的文件也可以通过连线的方式来和控制器关联，可以重新创建一个xx.m的控制器，将其中的class改为xib文件的名字即可，这样就可以将其模块化，避免主控制器中代码过于繁杂。

2.如果涉及到代码和逻辑操作的时候，把xib和一个class进行关联

```objective-c
//xxx.h代码
#import <UIKit/UIKit.h>
@class AppModel;

@interface YellowView : UIView

// 属性如果非必要不会暴露在.h文件里
// 不安全, 扩展性不好, 代码量

@property (nonatomic, strong) AppModel *appModel;

@end
//xxx.m代码
#import "xxx.h"
#import "AppModel.h"
@interface YellowView()

@property (weak, nonatomic) IBOutlet UIImageView *iconImageView;
@property (weak, nonatomic) IBOutlet UILabel *nameLabel;
@property (weak, nonatomic) IBOutlet UIButton *downloadButton;

@end

//重写appmodel的setter方法
-(void)setAppmodel:(AppModel *)appmodel{
  //对属性赋值
  _appModel = appModel;
  //对子控件进行赋值
  _iconImageView.image = [UIImage imageNamed:appModel.icon];
  _nameLabel.text = appModel.name;
}
```

MVC：

> M—> Model   描述数据，处理数据
>
> V—> View   只是用来显示（UI控件）
>
> C—> Controller  管理view的生命周期，处理用户交互（代码逻辑）

## 12.xib和storyboard的异同点

共同点：可以设置UI界面

不同点：storyboard（描述整个界面）进行界面跳转，是重量级的，xib(描述局部界面)是轻量级的

## 13.按钮的设置

点击下载按钮时候需要重新写一个view覆盖原来的界面,并用动画的效果来显示。

```objective-c
- (IBAction)didClickDownloadButton:(id)sender {
    
    // 转为UIButton类型
    UIButton *downButton = sender;
    
    // 拿到控制器的view
    UIView *controllerView = self.superview;
    
    // 创建一个遮罩view
    UIView *coverView = [[UIView alloc] initWithFrame:controllerView.bounds];
    
    // 设置透明度
    // alpha 0(完全透明) <--> 1(完全不透明)
    coverView.alpha = 0;
    
    // 设置颜色
    [coverView setBackgroundColor:[UIColor blackColor]];
    
    // 添加到控制器的view上
    [controllerView addSubview:coverView];
    
    // superView的size
    CGSize superSize = controllerView.frame.size;
    
    // 添加coverView上的文本
    UILabel *noticeLabel = [[UILabel alloc] initWithFrame:CGRectMake(0, 0, controllerView.bounds.size.width, 20)];
    
    // 设置对齐方式
    noticeLabel.textAlignment = NSTextAlignmentCenter;
    
    // 设置文字
    noticeLabel.text = @"下载中...";
    
    noticeLabel.textColor = [UIColor whiteColor];
    
    noticeLabel.center = CGPointMake(superSize.width/2, superSize.height/2);
    
    // 添加到coverView上
    [coverView addSubview:noticeLabel];
```



![](http://omg5mjb8v.bkt.clouddn.com/24E07267-E3F5-49BC-A897-99368D74543C.png)

## 14.插件安装

> 通过xcode的info.plist获得他的UUID
>
> 切换到插件安装的路径
>
> 找到那个目录下的info.plist，将xcode的UUID给增添进去。

## 15.九宫格完整代码展示

```objective-c
#ViewController.m
#import "ViewController.h"
#import "AppModel.h"
#import "YellowView.h"

// 定义了列数
#define kColumn 3

// imageView的y值
#define kTopY 5

// 定义imageView的宽高
#define kImageViewWidth 45

@interface ViewController ()
/**
 防止数据被修改:
 不可变数组: 内部元素的数据是不可修改
 可变数组更消耗性能
 */
@property (nonatomic, strong) NSArray *dataArray;

@end

@implementation ViewController

/**
 懒加载数据 , 重写get方法
 必须通过self.dataArray --> 调用 dataArray的get方法

 */
- (NSArray *)dataArray {
    if (nil == _dataArray) {
        // 1. 读取文件的路径
        NSString *path = [[NSBundle mainBundle] pathForResource:@"app.plist" ofType:nil];
        
        // 2. 读取文件内容到临时数组
//        _dataArray = [NSArray arrayWithContentsOfFile:path];
        NSArray *tempArray = [NSArray arrayWithContentsOfFile:path];
        
        // 3. 遍历tempArray数组 把字典转为 appModel对象
        NSMutableArray *muta = [NSMutableArray array];
        
        for (NSDictionary *dict in tempArray) {
            // 实例化一个 appModel 对象
            
            AppModel *appModel = [AppModel appModelWithDict:dict];
      
            // 添加到可变数组
            [muta addObject:appModel];
        }
        
        // 4. 把可变数组赋值给  _dataArray
        _dataArray = muta;
        
    }
    return _dataArray;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    /**
     self 在哪里类里就表示这个类自身
     谁调用的方法, 这个self 就表示谁
     */

    // 定义间距
//    CGFloat margin = 20;    
    // 定义view的宽高
    CGFloat yellowViewWidth = 80;
    CGFloat yellowViewHeight = 95;
    
    // 计算间距 (屏幕的宽度 - column * yellowViewWidth) / (column + 1)
    CGFloat margin = (self.view.frame.size.width - kColumn * yellowViewWidth) / (kColumn + 1);
    
    for (int i = 0; i < self.dataArray.count; i++) {
        
        // 计算行索引
        NSInteger rowIndex = i / kColumn;
        
        // 列索引
        NSInteger columnIndex = i % kColumn;
    
        // 计算x值 , 和列索引有关
        // 计算公式: x = (i + 1) * margin + i * yellowViewWidth
        CGFloat yellowViewX = (columnIndex + 1) * margin + columnIndex * yellowViewWidth;
    
        // 计算y值的时候是和 行索引有关
        CGFloat yellowViewY = (rowIndex + 1) * margin + rowIndex * yellowViewHeight;
        
        // 实例化一个view并设置了frame
        
        // 加载xib , 通过方法, 接收返回的view对象
       YellowView *yellowView =  [[[NSBundle mainBundle] loadNibNamed:@"YellowView" owner:nil options:nil] lastObject];
        
        [yellowView setFrame:CGRectMake(yellowViewX, yellowViewY, yellowViewWidth, yellowViewHeight)];
        
        // 添加到控制器的view上
        [self.view addSubview:yellowView];
        
        // 取出对应的model, 设置yelloView子控件上的数据
        AppModel *appModel  = self.dataArray[i];
        
        // 把appModel赋值给 yellowView的appModel属性
        yellowView.appModel = appModel;
        

    }

}
```

```objective-c
#YellowView.h
#import <UIKit/UIKit.h>
@class AppModel;

@interface YellowView : UIView

// 属性如果非必要不会暴露在.h文件里
// 不安全, 扩展性不好, 代码量

@property (nonatomic, strong) AppModel *appModel;


@end

  
#YellowView.m
#import "YellowView.h"
#import "AppModel.h"

@interface YellowView()

/**
 显示图片的ImageView
 */
@property (weak, nonatomic) IBOutlet UIImageView *iconImageView;
@property (weak, nonatomic) IBOutlet UILabel *nameLabel;
@property (weak, nonatomic) IBOutlet UIButton *downloadButton;

@end

@implementation YellowView


// 重写 appModel的set方法

- (void)setAppModel:(AppModel *)appModel {
    
    // 1. 先对属性进行赋值
    _appModel = appModel;

    
    // 2. 对子控件进行赋值
    _iconImageView.image = [UIImage imageNamed:appModel.icon];
    _nameLabel.text = appModel.name;
}


// 点击下载按钮调用的方法
- (IBAction)didClickDownloadButton:(id)sender {
    
    // 转为UIButton类型
    UIButton *downButton = sender;
    
    // 拿到控制器的view
    UIView *controllerView = self.superview;
    
    // 创建一个遮罩view
    UIView *coverView = [[UIView alloc] initWithFrame:controllerView.bounds];
    
    // 设置透明度
    // alpha 0(完全透明) <--> 1(完全不透明)
    coverView.alpha = 0;
    
    // 设置颜色
    [coverView setBackgroundColor:[UIColor blackColor]];
    
    // 添加到控制器的view上
    [controllerView addSubview:coverView];
    
    // superView的size
    CGSize superSize = controllerView.frame.size;
    
    // 添加coverView上的文本
    UILabel *noticeLabel = [[UILabel alloc] initWithFrame:CGRectMake(0, 0, controllerView.bounds.size.width, 20)];
    
    NSLog(@"%@", NSStringFromCGPoint(noticeLabel.center));
    
    // 设置对齐方式
    noticeLabel.textAlignment = NSTextAlignmentCenter;
    
    // 设置文字
    noticeLabel.text = @"下载中...";
    
    noticeLabel.textColor = [UIColor whiteColor];
    
    noticeLabel.center = CGPointMake(superSize.width/2, superSize.height/2);
    
    NSLog(@"设置后: %@", NSStringFromCGPoint(noticeLabel.center));
    
    // 添加到coverView上
    [coverView addSubview:noticeLabel];
    
    // 通过动画来显示 coverView
    [UIView animateWithDuration:1 animations:^{
        
        coverView.alpha = 0.6;
        
    } completion:^(BOOL finished) {
        
        [UIView animateWithDuration:1   // 动画执行的时间
                              delay:1   // 延迟时间执行动画
                            options:UIViewAnimationOptionCurveEaseInOut // 动画效果
                         animations:^{  // 要执行动画的操作
                             
                             coverView.alpha = 0;
            
        } completion:^(BOOL finished) { // 执行完动画之后的回调
            
            // 下载完成之后又, 要把button的状态修改为 被禁用状态
            downButton.enabled = NO;
            
            // 把coverView 从父控件上移除
            [coverView removeFromSuperview];
            
        }];
    }];
    
}


@end

```

```objective-c
#AppModel.h
#import <Foundation/Foundation.h>

@interface AppModel : NSObject

/**
 这两个属性名称一定要和 字典里的key值保持一致
 属性的类型  -- > 字典里key对应的value 类型保持一致
 */
@property (nonatomic, copy) NSString *name;

@property (nonatomic, copy) NSString *icon;

/**
 id 和instancetype 都可以做为返回值
 instancetype :  不可以作为参数类型 , 但 它可以检测返回类型
 */
- (instancetype)initWithDict:(NSDictionary *)dict;


+ (instancetype)appModelWithDict:(NSDictionary *)dict;

@end
#AppModel.m
#import "AppModel.h"

@implementation AppModel

- (instancetype)initWithDict:(NSDictionary *)dict {
    // 调用父类的实例化方法
    if (self = [super init]) {
        self.name = dict[@"name"];
        self.icon = dict[@"icon"];
    }
    return self;
}

+ (instancetype)appModelWithDict:(NSDictionary *)dict {
    // 这里在进行alloc 的时候, 最好使用 self
    return [[self alloc] initWithDict:dict];
}

@end

```
 

