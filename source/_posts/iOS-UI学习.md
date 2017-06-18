---
title: iOS-UI学习
date: 2017-05-12 20:04:34
tags: iOS OC 
categories: UI学习 基础
author: Peterpan

---

UI初步
<!--more-->
# UI基础(Day1-2)

iOS模拟器:当没有应用设备但是需要测试的时候使用，可以通过菜单栏来进行一些基础操作。

## 1.Main.storyBoard的作用基本设置

> 改变可视化界面的属性，在右边的菜单栏中进行修改。
>
> 在右下角进行对控件的调试
>
> 自动布局功能
>
> 如果要改变界面的大小点击上方的View Controller
>
> 复制控件可以先选中他，然后按住option拖动
>
> 当不小心删除了页面之后，可以通过command+z还原
>
> 如果想要键盘每次自动弹出，就把hardware中的连接硬件取消掉，只留下虚拟键盘

## 2.常用基本控件选择

> label:单纯显示文字
>
> Button:点击
>
> Text:输入文字

## 3.ViewController（视图控制器）

>管理视图，写代码部分
>
>ViewController里的View是由ViewController管理的
>
>UIView,UIViewController这两个类本身没有关系，UIView的作用是显示,UIViewController中有一个UIView的属性
>
>可以在这里取消自动布局

## 4.控件连线

> 右键点击控件，选择触发的事件，注意不同事件的触发效果不同
>
> 把代码中的返回值类型中改成IBAction,将它们连线,快速连线可以按住control拖拽来生成新名字的方法。
>
> 易错点：当删除方法的时候需要把按钮和方法都删除，不然会报错
>
> 如果一个方法只需要点击按钮触发，不需要在其他地方调用，那么不需要在.h中声明，只需要在.m中实现
>
> 对于只需要在控制器中使用的控件（属性），我们一般声明在类扩展中，UI控件统统使用weak,UI控件需要连线，也需要添加标识IBOutlet![](http://omg5mjb8v.bkt.clouddn.com/8E4C5C77-6CF2-4222-A0A2-C76479B055DA.png)
>
> 一个事件可以和多个控件一起连线

## 5.计算器方法示例

```objective-c
-(IBAction)doSum{
  int num1 = [self.field1.text intValue];
  int num2 = [self.field2.text intValue];
  int sum = num1 + num2;
  self.sumLabel.text = [NSString stringWithFormat:@"%d", sum];
  //取消焦点，即为在闪烁的光标，每个属性逐一取消
  [self.field1 resignFirstResponder];
  [self.field1 resignFirstResponder];
  //或者直接取消控制器的View编辑状态,推荐使用这种
  [self.view endEditing:YES];
}
```

## 6.UIKit坐标系

>坐标原点与左上角重合
>
>可以在View里面不断嵌套View子控件，使用相对坐标，坐标只是相对于它的父控件的位置

## 7.UIView

> UI界面上，所有能看到的东西都是UIView
>
> 所有的控件都是直接或者间接继承UIView
>
> UIView是一个容器，里面可以添加其他的控件
>
> 修改父控件的颜色:sender就是我们点击的那个按钮
>
> superview属性:获取父控件

```objective-c
//随机颜色获取
-(IBAction)changeFatherViewColor:(UIButton *)sender{
  UIView *fatherview = sender.superview;
  //使用随机数来生成三原色的浓淡程度
  float randomR = arc4random_uniform(255)/255.0;
  float randomG = arc4random_uniform(255)/255.0;
  float randomB = arc4random_uniform(255)/255.0;
  //使用三原色来组合随机颜色
  UIColor *randomcolor = [UIColor colorWithRed:randomR green:randomG blue:randomB alpha:1];
  fatherview.backgroundColor = randomcolor;
}
```

```objective-c
//使用代码方式设置控件的位置，不能直接修改控件的Frame属性，可以把Frame属性保存到一个临时变量上修改。
CGRect oldFrame = createview.frame;
oldFrame.origin.x = 10;
createview.frame = oldFrame
```

```objective-c
//创建动画方法一：头尾式动画
[UIView beginAnimations:nil context:nil];
//修改动画的各种属性
[UIView setAnimationsDuration:时间 单位：秒];//在多少秒内完成指令
[UIView setAnimationsDelay:时间 单位：秒];//延时的秒数
//关键代码
省略.....
//提交动画
[UIView commitAnimations];
//创建动画方法二：block动画
[UIView animateWithDuration:1 animations:^{
        createview.frame = oldFrame;
    }];
```

```objective-c
/*按钮设置的注意点
image属性可以设置按钮的图片，这样设置的图片跟文字平级
设置background图片文字是在图片的后面
按钮有很多种状态，默认是default：可以设置一套样式
点击以后是highlighted，可以设置另一套样式
selected:需要使用代码设置
disabled:禁用状态
／*
图片放置
1.放在项目中
2.放在Assets.xcassets
```

## 8.代码形式的添加button，如果想要设置不同形态的样式，需要分别创建

![](http://omg5mjb8v.bkt.clouddn.com/D9EE7EAD-673C-4E4A-95F3-43082AA158A9.png)

```objective-c
//绑定方法e.g
[btn addTarget:self action:@selector(haha) forControlEvents:UIControlEventTouchUpInside];
```

## 9.tag属性的使用

>所有直接或者间接继承来自UIView的控件都有一个tag属性
>
>这个属性，只能用来保存一个数字，对空间的外观没有任何影响
>
>可以通过属性来判断控件的类别：sender.tag，可以在main.storyboard里面定义。

## 10.transform属性的使用

> 可以使用动画效果
>
> 可以用来修改控件的位置，大小，还有空间的旋转

```objective-c
//使图片进行各种移动实例
#import "ViewController.h"

@interface ViewController ()
@property (weak, nonatomic)IBOutlet UIImageView *imageView;//连接需要操控的图片
@end

@implementation ViewController

-(IBAction)moveimage:(UIButton *)sender{
    CGRect oldframe = self.imageView.frame;
    switch (sender.tag) {
        case 1:
            oldframe.origin.x += 10;
            break;
        case 2:
            oldframe.origin.y +=10;
            break;
        case 3:
            oldframe.origin.x -= 10;
            break;
        case 4:
            oldframe.origin.y -= 10;
            break;
        case 5:
            oldframe.size.height += 10;//使图片进行整体放大，画面感更好
            oldframe.size.width += 10;
            oldframe.origin.x -= 5;
            oldframe.origin.y -= 5;
            break;
        case 6:
            oldframe.size.height -= 10;
            oldframe.size.width -= 10;
            oldframe.origin.x += 5;
            oldframe.origin.y += 5;
            break;
    }
    [UIView animateWithDuration:0.3 animations:^{
        self.imageView.frame = oldframe;
    }];
}
-(IBAction)transformimageView:(UIButton *)sender{
  self.imageView.transform = CGAffineTransformMakeRotation(M_PI);
  //M_PI是代表派，M_PI_2代表二分之派，直接创建一个固定的值的transform，如果想要每次都累加，需要使用别的方法。
  self.imageView.transform = CGAffineTransformRotate(self.imageView.transform, M_PI)
  //需要传入一个原始的transform，第二个参数是偏移量，通过正负来处理顺时针和逆时针方向。
}
@end
```

## 10.获取删除子控件

> 通过自身View的subview来获取子控件的属性，通过removeFromSuperview删除
>
> ```objective-c
> self.whiteview.subviews其实是一个数组，存放的是每一个子控件的信息，可以通过下标来访问，通过removeFromSuperview来删除，删除之后类似于栈数据结构，移除栈顶元素之后自动前补，如果为空继续删除，会进行报错。
> ```
>
> 或者通过for循环来删除
>
> ```objective-c
> //可以把子类对象放到父类类型的变量中
> for (UILabel *label in self.whiteview.subviews)
>   [label removeFromSuperview];
> ```
>
> 删除指定tag的view
>
> ```objective-c
> [self.whiteview viewWithTag:x]获取下标为x的控件
> 如果自己的tag和子控件一样，也能够获取到，优先获取父控件，而且如果删除了父控件，那么它所有的子控件都会被删除
> ```
>
> 

## 11.设置一个图片浏览器的例子

> label属性的设置：lines属性设置为0可以自动换行，高度不够，自动换行也是看不到的
>
> 按钮的启用禁用：按钮有一个属性，enabled 如果是yes，就是可以点击，反之亦然。

```objective-c
#import "ViewController.h"

@interface ViewController ()
@property(assign, nonatomic)int index;
@property (weak, nonatomic) IBOutlet UIImageView *imageview;
@property (weak, nonatomic) IBOutlet UIButton *lastbutton;
@property (weak, nonatomic) IBOutlet UIButton *nextbutton;
@property (weak, nonatomic) IBOutlet UILabel *indexLabel;
@property (weak, nonatomic) IBOutlet UILabel *infoLabel;


@end

@implementation ViewController

- (void)viewDidLoad {
    //当控制器的view和上面的子控件创建完毕之后就会执行这个方法
    [super viewDidLoad];
    self.index = 1;
    self.lastbutton.enabled = NO;
    [self changeImage];
}

- (IBAction)LastButton:(UIButton *)sender {
    self.index--;
    [self changeImage];
    if (self.index==1)
        sender.enabled = NO;
    if (self.index!=1)
        self.nextbutton.enabled = YES;
}
- (IBAction)NextButton:(UIButton *)sender {
    self.index++;
    [self changeImage];
    if (self.index==3)
        sender.enabled = NO;
    if (self.index!=3)
        self.lastbutton.enabled = YES;
}

- (void)changeImage{
    switch (self.index) {
        case 1:
            self.indexLabel.text = @"唔哈哈哈哈";
            self.infoLabel.text = @"1/3";
            self.imageview.image = [UIImage imageNamed:@"1.jpeg"];
            break;
        case 2:
            self.indexLabel.text = @"小朋友们，还记得我吗？";
            self.infoLabel.text = @"2/3";
            self.imageview.image = [UIImage imageNamed:@"2.jpg"];
            break;
        case 3:
            self.indexLabel.text = @"站住不许走";
            self.infoLabel.text = @"3/3";
            self.imageview.image = [UIImage imageNamed:@"3.jpg"];
            break;
            }
}

@end

```

## 12.storyboard控制器

> main.storyboard里面可以加载多个ViewController,但是程序启动的时候，默认加载的是箭头指向的那个。
>
> 可以拖拉箭头或者修改is initial ViewController属性来设置
>
> 在storyboard中的控制器，如果想要跟代码建立联系，需要设置CustomClass
>
> storyboard删除之后可以手动创建
>
> 如果添加了多个storyboard，默认会找到项目设置中设置的那个
>
> storyboard的实质是一个文本文件，有一定的格式的文件，格式为xml,每一个通过托拉拽的方式添加的控件信息，全部以标签的形式保存在文本中

## 13.程序启动的实质

![](http://omg5mjb8v.bkt.clouddn.com/CD2ECE37-7AD0-46A2-A03C-B4AF645ADDF9.png)

## 14.Bundle和图片放置位置的区别

> 应用程序.app就是bundle
>
> iOS程序打包好之后其实也是xxx.app
>
> 通过NSLog(NSHomeDirectory())打印的路径向前移动两个文件夹,bundle中找到相应的app
>
> 图片直接放在项目中，在bundle中可以直接看到，放在Assets.xcassets中，打包以后，会加密到Assets.car中，目前为止无法还原出来，更加安全。
>
> NSBundle这个类对应到了当前项目打包以后的xxx.app，使用mainBundle方法可以获取当前应用程序的xx.app。
>
> ```objective-c
> NSBundle *mainBundle = [NSBundle mainBundle];
> NSString *path = [mainBundle pathForResource:@"名字" ofType:@"类型"];
> NSString *path = [mainBundle pathForResource:@"名字.类型" ofType:@"nil"];//两种方法都可以，可以直接把后缀名加到名字的后面
> NSLog(path);
> ```
>
> bundle中的所有问价全路径都可以通过pathForResource方法来获取

## 15.Plist配合bundle初始化数组和字典

> 可以用来保存字典
>
> 项目中的plist文件，保存的位置是bundle中

```objective-c
//首先创建一个plist，在其中添加你需要的键值对，然后通过路径获取plist
NSBundle *mainBundle = [NSBundle mainBundle];
NSString *path = [mainBundle pathForResource:@"xxx.plist" ofType:@"nil"];
//此时通过路径来初始化字典，初始化数组也是同样的方式，也可以在数组里面嵌套字典。
NSDictionary *dict = [NSDictionary dictionaryWithContentOfFile:path];
```

## 16.使用UIView播放动画

![](http://omg5mjb8v.bkt.clouddn.com/1CE00FF4-3E56-4EBA-A9A7-EE061640E832.png)

## 17.iOS开发中内存处理的细节

> 开发iOS项目，如果app占用内存过大，iOS会发送一个警告给应用程序
>
> 程序员可以在警告中释放内存
>
> 如果发送了警告以后，app内存占用没有任何改变，就会闪退
>
> 主要的优化方法在于方法调用完之后对于内存的释放，如果是释放动画，需要延时释放 ，可以用到performSelector方法。

![](http://omg5mjb8v.bkt.clouddn.com/609AABF7-22A8-47B7-964C-E7FDF28BABB9.png)