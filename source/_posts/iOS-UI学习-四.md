---
title: iOS-UI学习(四)
date: 2017-05-23 23:28:30
tags: 代理设计的理解
categories: 代理 图片轮播器 模块化
---

UI初步（四）
<!--more-->

# UI基础Day5

## 1.自定义一个coverView

> 首先要创建一个coverView的类
>
> 在这个类当中重写初始化他的方法
>
> 在ViewController里面进行对象的实例化并加入其父控件(注意，最后一定要赋值回去：coverView.model = _model)

```objective-c
#import "CoverView.h"

@implementation CoverView

-(instancetype)initWithFrame:(CGRect)frame{
  if (self == [super initWithFrame:frame]){
  	self.backgroundColor = [UIColor blackColor];
  	self.alpha = 0.6;
    CGSize *viewSize = [UIScreen mainScreen].bounds.size;
    UILabel *nameLabel = [UILabel alloc]initWithFrame:CGRectMake(0,0,viewSize.width,20)
      //以下省略对nameLabel的其他设置
      [self addSubview:nameLabel];
  }
  return self;
}
//注意，在main.storyboard里面，越靠近view离底层越近
```

## 2.UIScrollView控件（滚动，缩放）

可以滚动的控件的父类UIScrollView或者继承自UIScrollView，首先添加一个scrollView，再向其中添加子view，并设置相关属性。

1.contentSize属性：

```objective-c
//限定滚动的范围，在设置的时候，一定要比scrollerView的Size大，否则不会产生滚动的效果，如果宽度设置为0，就表示在横向中不能滚动
_scrollView.contentSize = CGSizeMake(1024, 768);
```

2.滚动指示器属性

```objective-c
_scrollView.showsVerticalScrollIndicator=YES;//垂直滚动指示器
_scrollView.showsHorizontalScrollIndicator=YES;//水平滚动指示器
```

3.弹簧效果

```objective-c
_scrollView.bounces = YES;//弹簧效果，默认是YES，一般不关闭不设置contentSize的时候弹簧效果开启，但是看不到
//设置下面属性后,就算不设置contentSize，也会有弹簧效果,前提是_scrollView.bounces的属性是YES
_scrollView.alwaysBounceHorizon = YES;
_scrollView.alwaysBounceVertical = YES;
```

4.内边界

```objective-c
//没有设置contentInset的时候，imageView默认会停留在初始位置，contentInset会停留在设置的内边距位置，
_scrollView.contentInset = UIEdgeInsetsMake(10,10,10,10);
```

5.偏移量

![C275B81E-725D-4560-92C8-2FC5C7D40AE2](http://omunhj2f1.bkt.clouddn.com/C275B81E-725D-4560-92C8-2FC5C7D40AE2.png)

```objective-c
_scrollView.contentOffset = CGPointMake(100, 100);
```

6.不能滚动的原因小结

> contentsize比scrollView的size小
>
> scrollView.userInteractionEnabled = NO;禁止了用户交互（限制更大）
>
> _scrollView.scrollEnabled = NO;禁止滚动

7.手动修改offset

```objective-c
//通过动画和按钮实现手动的流畅滚动效果
-(IBAction)didClick:(id)sender{
  CGPoint offset = _scrollView.contentOffset;
  offset.x += 20;
  offset.y += 20;
  //动画效果实现(方法1)
  [UIView animateWithDuration:0.5 animations:^{
    _scrollView.contentOffset = offset;
  }];
  //设置contentoffset的时候有动画效果（方法2）
  [_scrollView setContentOffset:offset animated:YES];
}
```

## 3.代理的使用[代理设计模式](http://www.jianshu.com/p/2113ffe54b30)

使用代理的情况：自己不能做或者不想做

1.制定协议以及代理方法，定义一个代理的属性

2.遵守协议，实现对应的代理方法

3.制定协议者在适当的时候调用代理方法

**协议中的内容一般都是方法列表，当然也可以定义属性，协议是公共的定义，如果只是某个类使用，我们常做的就是写在某个类中。如果是多个类都是用同一个协议，建议创建一个Protocol文件，在这个文件中定义协议。遵循的协议可以被继承，协议只能定义公用的一套接口，类似于一个约束代理双方的作用。但不能提供具体的实现方法，实现方法需要代理对象去实现。协议可以继承其他协议，并且可以继承多个协议，在iOS中对象是不支持多继承的，而协议可以多继承。**

![6234F898-5C33-4E75-8056-84BB36E12A6A](http://omunhj2f1.bkt.clouddn.com/6234F898-5C33-4E75-8056-84BB36E12A6A.png)

```objective-c
//doublehui.h
#import <UIKit/UIKit.h>

#warning 1. 指定协议及代理方法，当前协议继承了一个协议，所以只继承了来自这一个协议的方法列表
@protocol DoubleHuiDelegate <NSObject>

- (void)needPigMeet;

@end

@interface DoubleHui : UIView

#warning 2. 创建一个代理属性
@property (nonatomic, weak) id<DoubleHuiDelegate> delegate;

@end
//doublehui.m 
#import "DoubleHui.h"

@implementation DoubleHui

- (instancetype)initWithFrame:(CGRect)frame {
    if (self = [super initWithFrame:frame]) {
        
        UIButton *button = [[UIButton alloc] initWithFrame:CGRectMake(0, 0, 50, 50)];
        [button setBackgroundColor:[UIColor redColor]];
        
        // 监听方法
        [button addTarget:self
                   action:@selector(clickButton)
         forControlEvents:UIControlEventTouchUpInside];
        
        [self addSubview:button];
    }
    return self;
}

- (void)clickButton {
    // 点击按钮的时候就表示， 缺肉了
    NSLog(@"缺猪肉， 赶紧送");    
    // 通知代理， 调用代理方法
    //[self.delegate needPigMeet];
    // respondsToSelector:@selector 返回一个bool ，如果该对象实现了这个方法， 就返回yes ，如果没有返回NO
    
    #warning 3. 调用代理方法
    if ([self.delegate respondsToSelector:@selector(needPigMeet)]) {
        [self.delegate needPigMeet];
    }
}
@end
```

```objective-c
//ViewController.m
#warning  1. 遵守协议
@interface ViewController ()<DoubleHuiDelegate>

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    DoubleHui *huiView = [[DoubleHui alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
    
    [huiView setBackgroundColor:[UIColor orangeColor]];
     // 设置代理
    #warning  2. 设置代理，赋值回去
    huiView.delegate = self;
    
    
    [self.view addSubview:huiView];
}

#warning  3. 实现代理方法
-(void)needPigMeet {
    NSLog(@"肉来了");
}

@end
```

代理使用的原理：![44EFD2BE-DB86-4B5D-9F7E-51485E8D816B](http://omunhj2f1.bkt.clouddn.com/44EFD2BE-DB86-4B5D-9F7E-51485E8D816B.png)

我对于代理的形象化理解：

小明要上厕所，告诉老师，老师同意，小明去上厕所，这个事件中，小明作为委托方，上厕所的方法在小明的身上，但是老师如果不同意，小明就无法上厕所，当老师同意的时候，小明才会调用自身的上厕所这个方法。

## 4.监听ScrollView的滚动

可以使用拖拽的方式让scrollView和ViewController关联，然后设置代理

```objective-c
#import "ViewController.h"

@interface ViewController ()<UIScrollViewDelegate>
@property (weak, nonatomic) IBOutlet UIScrollView *scrollView;
@property (weak, nonatomic) IBOutlet UIImageView *imageView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    _scrollView.contentSize = _imageView.frame.size;
    
}
//当scrollView滚动的时候就会调用
- (void)scrollViewDidScroll:(UIScrollView *)scrollView{
    NSLog(@"滚动时调用");
}
//开始拖拽的时候调用
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView{
    NSLog(@"开始拖拽");
}
//停止拖拽的时候调用
- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate{
    NSLog(@"停止拖拽");
}
@end
```

scrollView代理的使用：

> 遵守协议
>
> 设置成为代理
>
> 实现对应的方法

## 5.实现图片的缩放

放大缩小倍数属性：

```objective-c
_scrollView.minimumZoomScale  最大的缩小倍数

_scrollView.maximumZoomScale  最大的放大倍数
```

同时需要设置代理

```objective-c
#import "ViewController.h"

@interface ViewController ()<UIScrollViewDelegate>
@property (weak, nonatomic) IBOutlet UIScrollView *scrollView;
@property (weak, nonatomic) IBOutlet UIImageView *imageView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    _scrollView.contentSize = _imageView.frame.size;
    _scrollView.maximumZoomScale = 3;
    _scrollView.minimumZoomScale = 0.3;
    //设置成为代理,如果没有设置代理属性则不会调用代理方法
    _scrollView.delegate = self;
    
}
//实现代理，告诉scrollView哪个图片需要被拉伸或者缩放
-(UIView *)viewForZoomingInScrollView:(UIScrollView *)scrollView{
    return _imageView;
}
//只有图片在放大或者缩小的过程中调用
- (void)scrollViewDidZoom:(UIScrollView *)scrollView{
    NSLog(@"Zooming");
}
//在开始放大或缩小图片的时候调用
- (void)scrollViewWillBeginZooming:(UIScrollView *)scrollView withView:(UIView *)view{
    NSLog(@"begin zooming");
}
//在结束放大或者缩小图片的时候调用
- (void)scrollViewDidEndZooming:(UIScrollView *)scrollView withView:(UIView *)view atScale:(CGFloat)scale{
    NSLog(@"end zooming");
}

```

## 6.图片轮播器

图片的设置：

```objective-c
#define kImageCount 5
- (void)setupScrollView {
    // 取出scrollView的size
    CGSize scrollViewSize = _scrollView.frame.size;
    
    for (int i = 0; i < kImageCount; i++) {
        // 计算imageView的x值
        CGFloat imageViewX = i * scrollViewSize.width;
        
        UIImageView *imageView = [[UIImageView alloc] initWithFrame:CGRectMake(imageViewX, 0, scrollViewSize.width, scrollViewSize.height)];
        
        // 设置图片
//        imageView.image =[UIImage imageNamed:@"img_01"];
        // 拼接图片的名称
        NSString *imageName = [NSString stringWithFormat:@"img_%02d",i + 1];
        
        imageView.image = [UIImage imageNamed:imageName];
        
        // 添加到scrollView
        [_scrollView addSubview:imageView];
    }
    
    // 设置 scrollView的contentSize
    _scrollView.contentSize = CGSizeMake(kImageCount * scrollViewSize.width, 0);
    
    // 隐藏滚动指示器
    _scrollView.showsHorizontalScrollIndicator = NO;
    
}
```

设置分页的模式：

```objective-c
//开启分页效果，根据scrollView的宽度进行分页
_scrollView.pagingEnabled = YES;
```

pageControll设置,拖动一个pageControll按钮到main.storyboard中

```objective-c
- (void)setupPageControll {
  //根据具体图片的数量来修改
  _pageControl.numberOfPages = kImageCount;
     
    // 设置指示器的颜色
    // 非当前的指示器
  _pageControl.pageIndicatorTintColor = [UIColor grayColor];
// 设置当前指示器的颜色
  _pageControl.currentPageIndicatorTintColor = [UIColor redColor];
// 设置当前在第几个点,默认为0，如果超出范围则在最后一个显示
  _pageControl.currentPage = 1;
}
```

 将pageControl和scrollView进行关联

![4809ED67-267E-490E-B679-69AB2C3A8884](http://omunhj2f1.bkt.clouddn.com/4809ED67-267E-490E-B679-69AB2C3A8884.png)

```objective-c
//拖动图片的时候移动pageControl
#define kScrollViewSize (_scrollView.frame.size)
#pragma mark -  当scrollView停止减速的时候调用
// Decelerating 减速
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView {
    // currentPage = scrollView.contentOffset.x / kScrollViewSize.width
    
    _pageControl.currentPage = scrollView.contentOffset.x / kScrollViewSize.width;
}
```

点击按钮实现滚动的方法

```objective-c
- (IBAction)didClickButton {
  CGPoint offset = _scrollView.contentOffset;
  NSInteger currentPage = _pageControl.currentPage;
  if (currentPage == 4){
    currentPage = 0;
    offset.x = 0;   //或者写为offset = CGPointZero;
  }else{
  currentPage += 1;
  offset.x += kScrollViewSize.width;
  }
  _pageControl.currentPage = currentPage;
  [_scrollView setContentOffset:offset animated:YES];
}
```

通过定时器的实现自动滚动

```objective-c
  - (void)initImageTimer {
    /**
     scheduled 计划，安排
     interval : 间隔
     target :  一般指控制器
     selector: 方法
     userInfo : 用户自定义的参数
     repeats: 重复
     
     每隔1秒钟 调用 控制器的  didClickButton： 方法， 传递的参数为nil
     
     一旦创建就会立即生效
     
     在使用timer的时候， 如果调用了 invalidate方法， 那么这个计时器就不会再次生效
     重新创建新的timer
     */
    _timer = [NSTimer scheduledTimerWithTimeInterval:2
                                              target:self
                                            selector:@selector(didClickButton:)
                                            userInfo:nil
                                             repeats:YES];
    
   // [_timer fire];  调用fire ， 这个计时器会立即执行， 不会等待 interval 设置的时间
   // 设置优先级，使其优先级和UITextView相同，滚动UITextView的同时也执行定时器的方法
    NSRunLoop *mainLoop = [NSRunLoop mainRunLoop];
    
    
    [mainLoop addTimer:_timer forMode:NSRunLoopCommonModes];
    
}
/**
 在开始拖拽的时候， 把计时器停止
 
 invalidate 无效的意思
 */
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView {
    // 让计时器无效
    [_timer invalidate];
}

/**
 当停止拖拽的时候， 让计时器开始工作
 手指离开scrollView的时候
 */
- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate {
   [_timer fire];
    
    [self initImageTimer];
}
```

UITextView是可以滚动的文章控件，可以配合轮播器一起使用

补：[关于NSRunLoop和NSTimer的深入理解](http://www.superqq.com/blog/2016/05/05/ios-nsrunllop-nstimer/)

NSRunLoop是消息机制的处理模式

NSRunLoop的作用在于有事情做的时候使的当前NSRunLoop的线程工作，没有事情做让当前NSRunLoop的线程休眠

NSTimer默认添加到当前NSRunLoop中，也可以手动制定添加到自己新建的NSRunLoop

- NSRunLoopCommonModes 这是一组可配置的通用模式。将input sources与该模式关联则同时也将input sources与该组中的其它模式进行了关联。
- 每次运行一个run loop，你指定（显式或隐式）run loop的运行模式。当相应的模式传递给run loop时，只有与该模式对应的input sources才被监控并允许run loop对事件进行处理（与此类似，也只有与该模式对应的observers才会被通知）

## 7.代理使用weak修饰

设置代理属性的时候，使用weak修饰符，是为了防止循环引用

addSubView方法其实是对加入的子控件有一个强引用，所以有两个对象互相饮用的时候，其中一个必须是weak，否则无法释放掉。

![%E4%BB%A3%E7%90%86%E7%9A%84weak%E4%BF%AE%E9%A5%B0%E7%AC%A6](http://omunhj2f1.bkt.clouddn.com/%E4%BB%A3%E7%90%86%E7%9A%84weak%E4%BF%AE%E9%A5%B0%E7%AC%A6.png)

## 8.使用代理优化九宫格代码思路

1.在yellowView.h里面创造一个代理的方法和代理的属性

```objective-c
- (void)yellowView:(YellowView *)yellowView  didClickButton:(AppModel *)model;
```

2.在yellowView.m点击按钮的方法中中调用代理的方法

```objective-c
//在传递参数的时候，不管别人用不用，首先要把自己传递出去
if([self.delegate respondsToSelector:@selector(yellowView:didClickButton:)]){
 [self.delegate yellowView:self didClickButton:_appModel]; 
}
```

3.在ViewController.m的ViewDidload中创建coverView,并在类扩展中创建一个coverView属性

```objective-c
//在这之前在循环生成YellowView过程中把self赋值给代理属性
UIView *coverView = [[UIView alloc]initWithFrame:CGRentMake:self.view.size.bounds];
self.coverView = coverView;
coverView.backgroundColor = [UIColor blackColor];
coverView.alpha = 0;
[self.view addSubView:coverView];
```



4.在ViewController.m中构造代理方法

```objective-c
- (void)yellowView:(YellowView *)yellowView  didClickButton:(AppModel *)model{
  _coverView.alpha = 0.6;
  _coverView.appModel = model;
}
```

