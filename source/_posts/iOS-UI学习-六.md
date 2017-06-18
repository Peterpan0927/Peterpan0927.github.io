---
title: iOS-UI学习(六)
date: 2017-06-11 18:13:00
tags: 简单微博 MVC实现
categories: TableView的进阶使用
author: Peterpan
---
iOS-UI学习(六)
<!--more-->

# UI基础Day7

## 1.Demo优化

```objective-c
// 使用alertController新版的方式
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    // 弹出alertController
    UIAlertController *controller = [UIAlertController alertControllerWithTitle:@"编辑" message:nil preferredStyle:UIAlertControllerStyleAlert];
    
    // 添加action-动作(按钮)
    UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil];
    
    // 添加到 alerController
    [controller addAction:cancelAction];
    
    // 添加确定按钮
    UIAlertAction *sureAction = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        
        // 1. 取出新的英雄名字
        // 1.1 取出alertController中 textField
        UITextField *textField = [controller textFields][0];
        
        NSString *newName = textField.text;
        
        // 2. 修改模型中的数据
        HeroModel *heroModel = self.dataArray[indexPath.row];
        heroModel.name = newName;
        
        // 3. 刷新数据源
//        [_tableView reloadData];
        [_tableView reloadRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationLeft];
        
    }];
    
    // 取出点击cell对应的模型
    HeroModel *heroModel = self.dataArray[indexPath.row];
    
    // 添加输入框
    [controller addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
        
        // 给textField 设置文本
        textField.text = heroModel.name;
    }];
    
    // 添加确定按钮到 alertController
    [controller addAction:sureAction];
    
    [self presentViewController:controller animated:YES completion:nil];
   
}
```

## 2.通过xib自定义cell

```objective-c
//使用这个方法返回的是自定义的子cell
cellForRowAtIndexPath
//加载自定义的xib
cell = [[[NSBundle mainBundle]loadNibNamed:@"GrouponsCell" owner:nil options:nil] lastObject];
```

注意⚠️：在通过xib自定义cell的时候一定要设置xib的重用标识符

![屏幕快照 2017-06-05 下午5.52.40.png](http://omunhj2f1.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-05%20%E4%B8%8B%E5%8D%885.52.40.png)

如果将一个类和xib进行关联，那么实例化之后调用的方法就是：

```objective-c
-(void)awakeFromNib{
  NSLog(@"调用此方法")；
}
```

## 3.在headerView封装一个scrollView

1.遵守UIScrollViewDelegate协议。

2.重写headerView的初始化方法

```objective-c
- (instancetype)initWithFrame:(CGRect)frame{
    
    if(self = [super initWithFrame:frame]){
        
        [self setupScrollView];
        
        [self setupPageController];
   
    }    
    return self;
}
```

3.设置ScrollView的属性

```objective-c
- (void)setupScrollView{
    UIScrollView *scrollView = [[UIScrollView alloc]initWithFrame:self.bounds];
    //赋值给属性
    self.scrollView = scrollView;
    //设置允许分页
    self.pagingEnabed c                                                                                                                                                                                                                                                                                                          = YES;
    //设置代理属性
    scrollView.delegate = self;
    //隐藏滚动条
    scrollView.showsHorizontalScrollIndicator = NO;
    
    [self addSubview:scrollView];
}
```

4.设置页面控制的属性

```objective-c
- (void)setupPageController{
    CGSize bannerViewSize = self.bounds.size;
    
    UIPageControl *pageControl = [[UIPageControl alloc]initWithFrame:CGRectMake(0, 0, bannerViewSize.width/2, 20)];
    
    self.pageControl = pageControl;
    
    pageControl.center = CGPointMake(bannerViewSize.width/2, bannerViewSize.height*0.9) ;
    
    pageControl.currentPageIndicatorTintColor = [UIColor  redColor];
    
    pageControl.pageIndicatorTintColor = [UIColor grayColor];
    //设置分页的初始下标
    pageControl.currentPage = 0;
    
    [self addSubview:pageControl];
    
}

```

5.设置传入的图片数组，重写setter方法

```objective-c
- (void)setImageArray:(NSArray *)imageArray{
    CGSize bannerViewSize = self.bounds.size;
    
    NSInteger count = imageArray.count;
    
    for( int i = 0 ; i < count ; i++ ){
        
        NSString *imageName = imageArray[i];
        
        UIImageView *imageView = [[UIImageView alloc]initWithFrame:CGRectMake(i*bannerViewSize.width, 0, bannerViewSize.width, bannerViewSize.height)];
        
        imageView.image = [UIImage imageNamed:imageName];
        
        [_scrollView addSubview:imageView];
        
        _scrollView.contentSize = CGSizeMake(count*bannerViewSize.width, 0);
        //设置分页数
        _pageControl.numberOfPages = count;
        
    }
}

```

## 4.自定义footerView

临时隐藏应用的时候，可能会用到三种方法：

```objective-c
view.alpha = 0; //view透明
view.backgroundColor = [UIColor clearColor]; //view的背景透明
view.hidden = YES; //整个view隐藏
//从性能上看隐藏要好些，因为透明的话 GPU需要计算两层view的内容再显示
```

```objective-c
//定义一个新的类设置footerView
#import <UIKit/UIKit.h>
@class FooterView;

@protocol FooterViewDelegate <NSObject>

- (void)footerView:(FooterView *)footerView;

@end


@interface FooterView : UIView
//设置代理属性
@property (nonatomic, weak) id<FooterViewDelegate> delegate;

- (void)showLoadViewWith:(BOOL)isShow;

@end
  
#import "FooterView.h"
@interface FooterView()

@property (nonatomic, weak) IBOutlet UIView *loadMoreView;
@property (weak, nonatomic) IBOutlet UIActivityIndicatorView *activityView;

@end

@implementation FooterView

- (IBAction)didClickButton:(id)sender{
  //判断是否可以使用代理的方法
    if([self.delegate respondsToSelector:@selector(footerView:)]){
        [self.delegate footerView:self];
    }
}

- (void)showLoadViewWith:(BOOL)isShow{
    _loadMoreView.alpha = isShow;
    if(isShow)
        [_activityView startAnimating];
    else
        [_activityView stopAnimating];
}
@end
```

```objective-c
//记得要将self赋值给footerView的代理属性，才能调用代理的方法
//ViewController中的方法
- (void)footerView:(FooterView *)footerView{
    
    [footerView showLoadViewWith:YES];
    //通过GCD延迟执行，再多线程中会细讲
  dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [footerView showLoadViewWith:NO];
        GroupsonsModel *model = [[GroupsonsModel alloc]init];
        model.title = [NSString stringWithFormat:@"哲学%d",arc4random_uniform(5)];
        model.price = [NSString stringWithFormat:@"%d",arc4random_uniform(34) ];
        model.buyCount = [NSString stringWithFormat:@"%d",arc4random_uniform(678) ];
        NSIndexPath *indexPath = [NSIndexPath indexPathForRow:self.dataArray.count inSection:0];
        
        [self.dataArray addObject:model];
        //添加cell，设置动画方式
        [_tableView insertRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationLeft];
        //使添加一个cell的时候滚动到底部
        [_tableView  scrollToRowAtIndexPath:indexPath atScrollPosition:UITableViewScrollPositionBottom animated:YES];

    });
}
```

注意⚠️：通过在footerView可以直接拿到控制器来进行插入cell，但是十分不建议这种写法，要用代理的模式，来保证整个代码的过程分工明确，而不是逻辑混乱，每个子view只管好自己的事情，不要随便去调用控制器，善用代理的设计模式。以下这种方式是不可取的。

```objective-c
//在FooterView.h中加入一个控制器
#import <UIKit/UIKit.h>
@class FooterView, ViewController;

@protocol FooterViewDelegate <NSObject>

- (void)footerView:(FooterView *)footerView;

@end

@interface FooterView : UIView


@property (nonatomic, weak) id<FooterViewDelegate> delegate;

// 决定是否要显示 loadView
- (void)showLoadViewWith:(BOOL)isShow;

@property (nonatomic, strong) ViewController *viewController;

@end
//点击按钮的时候直接将新创建的cell加入到控制器
- (IBAction)didClickLoadButton:(id)sender {
    
    // 1. 让loadMoreView 显示出来
    _loadMoreView.alpha = 1;
    // 2. 转菊花，也就是那个UIActivityIndicatorView
    [_activityView startAnimating];
#warning  以后不要这么做，使用代理通知到控制器来做
    // 3. 插入数据
    GrouponsModel *model = [[GrouponsModel alloc] init];
    model.title = @"0000000";
    
    // 放到数组
    [self.viewController.dataArray addObject:model];
    
    // 4. 刷新tableView
    [self.viewController.tableView reloadData];
    
}
```

## 5.实现简单的微博

```objective-c
//为了避免使用代理，将main.storyboard中的View替换为TableView
//并将ViewController继承的父类改成UITableView,将二者关联起来。
//自定义微博的cell
#import <UIKit/UIKit.h>
@class WeiboModel;

@interface WeiboViewCell : UITableViewCell

@property (nonatomic, strong) WeiboModel *weiboModel;

@end

  
#import "WeiboViewCell.h"
#import "WeiboModel.h"
@interface WeiboViewCell()

@property (nonatomic, weak) UIImageView *userImageView;

@property (nonatomic, weak) UIImageView *vipImageView;

@property (nonatomic, weak) UIImageView *pictureImageView;

@property  (nonatomic, weak) UILabel *nameLabel;

@property (nonatomic, weak) UILabel *contentLabel;

@end


@implementation WeiboViewCell
- (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier{
    if(self = [super initWithStyle:style reuseIdentifier:reuseIdentifier]){
        [self setupUI];
    }
    return self;
}

- (void)setupUI{
    
    UIImageView *userImageView = [[UIImageView alloc] init];
    
    self.userImageView = userImageView;
    
    [self.contentView addSubview:userImageView];
    
    UIImageView *vipImageView = [[UIImageView alloc] init];
    
    self.vipImageView = vipImageView;
    
    [self.contentView addSubview:vipImageView];
    
    UIImageView *pictureImageView = [[UIImageView alloc] init];
    
    self.pictureImageView = pictureImageView;
    
    [self.contentView addSubview:pictureImageView];
    
    UILabel *nameLabel = [[UILabel alloc] init];
    
    self.nameLabel = nameLabel;
    
    [self.contentView addSubview:nameLabel];
    
    UILabel *contentLabel = [[UILabel alloc] init];
    
    contentLabel.numberOfLines = 0;
    
    // 设置字体为15
    contentLabel.font = [UIFont systemFontOfSize:15];
    
    self.contentLabel = contentLabel;
    
    [self.contentView addSubview:contentLabel];
    
    
}

- (void)setWeiboModel:(WeiboModel *)weiboModel{
    self.userImageView.image = [UIImage imageNamed:weiboModel.icon];
    
    self.contentLabel.text = weiboModel.text;
    
    self.vipImageView.image = [UIImage imageNamed:@"vip"];
    
    self.nameLabel.text = weiboModel.name;
    
    if(weiboModel.picture){
        self.pictureImageView.image = [UIImage imageNamed: weiboModel.picture];
    }
    
    CGFloat margin = 10;
    CGFloat userImageWidth = 50;
    
    _userImageView.frame = CGRectMake(margin, margin, userImageWidth, userImageWidth);
    
    // 计算用户名称的frame
    CGSize nameLabelMaxSize = CGSizeMake(MAXFLOAT, MAXFLOAT);
    //返回真实文本的宽高
    CGSize nameLabelRealSize = [weiboModel.name boundingRectWithSize:nameLabelMaxSize options:NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName:[UIFont systemFontOfSize:17]} context:nil].size;
    
    
    CGFloat nameLabelX = CGRectGetMaxX(_userImageView.frame) + margin;
    //    CGFloat nameLabelHeight = 20;
    //    CGFloat nameLabelWidth = 150;
    CGFloat nameLabelY = (userImageWidth - nameLabelRealSize.height)/ 2 + margin;
    
    _nameLabel.frame = CGRectMake(nameLabelX, nameLabelY, nameLabelRealSize.width, nameLabelRealSize.height);
    
    // 计算vip imageView的 frame
    CGFloat vipWidth = 20;
    CGFloat vipX = CGRectGetMaxX(_nameLabel.frame) + margin;
    CGFloat vipY = nameLabelY;
    
    _vipImageView.frame = CGRectMake(vipX, vipY, vipWidth, vipWidth);
    
    

    // 计算文本的frame
    CGFloat contentLabelWidth = self.contentView.frame.size.width - 2 * margin;
    
    // 给 显示的文本一个区域
    CGSize contentMaxSize = CGSizeMake(contentLabelWidth, MAXFLOAT);
    // NSFontAttributeName 字体的大小
    NSDictionary *attributesDict = @{NSFontAttributeName:[UIFont systemFontOfSize:17]};
#warning 计算文本实际宽高的时候， 计算的字体大小要和label中设置的字体大小保持一致
    // 根据限定的条件， 来计算text 真实的宽高，给文本一个矩形空间
    CGSize contentRealSize =  [weiboModel.text boundingRectWithSize:contentMaxSize options:NSStringDrawingUsesLineFragmentOrigin attributes:attributesDict context:nil].size;
    
    
    CGFloat contentLabelX = margin;
    
    CGFloat contentLabelY = CGRectGetMaxY(_userImageView.frame) + margin;
    
    //    CGFloat contentLabelHeight = 300;
    
    _contentLabel.frame = CGRectMake(contentLabelX, contentLabelY, contentRealSize.width, contentRealSize.height);
    
    // 配图的frame
    CGFloat pictureWidth = 2 * userImageWidth;
    CGFloat pictureX = margin;
    CGFloat pictureY = CGRectGetMaxY(_contentLabel.frame) + margin;
    
    CGFloat cellHeight = 0;
    
    if (weiboModel.picture) {
        _pictureImageView.frame = CGRectMake(pictureX, pictureY, pictureWidth, pictureWidth);
        
        // cell的高度
        cellHeight = CGRectGetMaxY(_pictureImageView.frame) + margin;
        
    } else {
        _pictureImageView.frame = CGRectZero;
        
        // 没有配图， 就要按照文本的最大Y值
        cellHeight = CGRectGetMaxY(_contentLabel.frame) + margin;   
    }   
}
@end
```

tableView的调用顺序：

首先有多少条数据就会调用多少次 heightForRowIndexPath:

之后就会调用cellForRowIndexPath:返回cell

但是每次调用的时候cellForRowIndexPath的时候都会重新调用一次heightForRowIndexPath:

所以如果需要将每一个微博的实际行高传给控制器显示出来的时候我们就需要一个frameModel，专门来设定控件的位置。

[改良版代码](https://github.com/Peterpan0927/iOS-test)

展示效果：

![0C39B15B-2AB5-4AC7-8929-2245F72376C4.png](http://omunhj2f1.bkt.clouddn.com/0C39B15B-2AB5-4AC7-8929-2245F72376C4.png)

---

小结：在定义一个类的时候要尽量遵从单一原则，是这个类的功能的尽量的简单化，分而治之，在出现问题的时候可以快速的定位。