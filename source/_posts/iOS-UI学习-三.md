---
title: iOS-UI学习(三)
date: 2017-05-18 15:30:00
tags: 较完善的App
categories: KVC字典快速赋值
---

UI初步（三）
<!--more-->

# UI基础Day4

## 1.应用图标的启动图片的设置

> 应用图标：将尺寸合适的图片拖入Assets.xcassets中的Applcon中
>
> 启动图片：在项目栏中找到Launch Images Source,点击后面的方框,删掉Launch Screen File来进行修改(iOS7及之前，iOS8以后直接在Launch Screen进行插入图片),[参考7](http://www.jianshu.com/p/a3315f6896a7)，[参考8](http://www.liuchuo.net/archives/2537),注意屏幕尺寸一定要做适配

## 2.屏幕尺寸和图片规格介绍

屏幕尺寸：

![](http://omg5mjb8v.bkt.clouddn.com/6D43B2B7-6DD9-4B71-B1D9-F012564F18AD.png)

图片规格：

![](http://omunhj2f1.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-05-16%20%E4%B8%8A%E5%8D%8810.32.56.png)

## 3.状态栏的设置[参考博客](http://www.jianshu.com/p/5aa05983b445)

加载启动图片时，如果不想显示状态栏可以在project中设置属性Hide status bar

![](http://omunhj2f1.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-05-16%20%E4%B8%8A%E5%8D%8811.32.03.png)

进入main.storyboard,在背景图片是黑色的时候，如果要显示状态栏，可以通过代码实现，[app状态栏样式设计](http://www.jianshu.com/p/9f7f3fa624e7)：

```objective-c
-(UIStatusBarStyle)preferredStatusBarStyle{
    return UIStatusBarStyleLightContent;
}
//隐藏状态栏
-(BOOL)preferredStatusBarStyle{
    return YES;
```

## 4.不同状态的按钮设置（代码）

```objective-c
[button setBackgroundImage:[UIImage imageNamed:@"btn_answer" ] forState:UIControlStateNormal];
[button setBackgroundImage:[UIImage imageNamed:@"btn_answer_highlighted" ] forState:UIControlStateHighlighted];
//记得最后初始化之后要将按钮加入到父控件当中
[_answerView addSubview:button];
```

## 5.对于self = [super init]的思考

```objective-c
- (instancetype)init
{
    if (self = [super init]) {
        // Custom initialization
    }
    return self;
}
//alloc返回一个有效的未初始化的对象实例。对于self是alloc 返回的指针，同时可以在所有的方法作用域内访问。
//如果初始化一个对象失败，就会返回nil，当返回nil的时候self = [super init]测试的主体就不会再继续执行。如果不这样做，你可能会操作一个不可用的对象，它的行为是不可预测的，最终可能会导致你的程序崩溃。
```

`self`表示当前这个类的对象，而`super`是一个编译器标示符，和`self`指向同一个消息接受者。在本例中，无论是`[self class]`还是`[super class]`，接受消息者都是`Son`对象，但`super`与`self`不同的是，`self`调用`class`方法时，是在子类`Son`中查找方法，而`super`调用`class`方法时，是在父类`Father`中查找方法。

## 6.对象初始化方式的比较

[class new]和[[class alloc]init]源码分析：

```objective-c
+ new 
{ 
    id newObject = (*_alloc)((Class)self, 0); 
    Class metaClass = self->isa; 
    if (class_getVersion(metaClass) > 1) 
    return [newObject init]; 
    else 
    return newObject; 
} 

//而 alloc/init 像这样： 
+ alloc 
{ 
    return (*_zoneAlloc)((Class)self, 0, malloc_default_zone());  
} 
- init 
{ 
    return self; 
}
//[class new]默认调用 alloc与init方法，那么我们无法使用自定义的初始化方法，多了更多的局限性。那么[class alloc] init] 会更方便， 当然[class alloc] init] 的设计也是由于历史的原因。
```

## 7.字典转模型实现懒加载的过程

> 读取文件路径
>
> 读取文件到临时数组
>
> 创建一个可变数组
>
> 遍历临时数组中的字典转为模型，存放到可变数组
>
> 把可变数组赋值给属性数组

## 8.清除子控件的方法

```objective-c
//1.遍历子控件，一个个删除
for(UIView *view in _answer.subviews){
	[view removeFromSuperiew];
}
//2.让让数组中所有的元素都执行清除方法
 [_answerView.subviews makeObjectsPerformSelector:@selector(removeFromSuperview)];
//为了内存优化，需要将上一次加载的子控件的内存释放掉
```

## 9.猜字谜代码讲解

```objective-c
//ViewController.m

#import "ViewController.h"

#import "GuessModel.h"

#define kButtonWidth 30

#define kMargin 10

#define kScreenSize [UIScreen mainScreen].bounds.size

@interface ViewController ()
@property (weak, nonatomic) IBOutlet UILabel *indexLabel;
@property (weak, nonatomic) IBOutlet UILabel *titleLabel;
@property (weak, nonatomic) IBOutlet UIButton *coinButton;
@property (weak, nonatomic) IBOutlet UIButton *imageButton;
@property (weak, nonatomic) IBOutlet UIView *answerView;
@property (weak, nonatomic) IBOutlet UIView *optionView;
@property (weak, nonatomic) IBOutlet UIButton *nextButton;
@property (strong, nonatomic) NSArray *dataArray;
@property (nonatomic, assign) NSInteger index;
@property (weak ,nonatomic) UIView *coverView;
@property (weak, nonatomic) IBOutlet UITextField *gua;
@end

@implementation ViewController
-(NSArray *)dataArray{
    if(_dataArray==nil){
        NSString *path = [[NSBundle mainBundle]pathForResource:@"questions.plist" ofType:nil];
        NSArray *tempArray = [NSArray arrayWithContentsOfFile:path];
        NSMutableArray *mutableArray = [NSMutableArray array];
        for (NSDictionary *dict in tempArray){
            GuessModel *guessModel = [GuessModel guessModelDict:dict];
            [mutableArray addObject:guessModel];
        }
        _dataArray = mutableArray;
    }
    return _dataArray;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    self.index = 1;
    [self setupUI];
    [self initCoverView];
}

- (void)initCoverView{
    UIView *coverView = [[UIView alloc]initWithFrame:self.view.bounds];
    //进行属性的关联
    self.coverView = coverView;
    coverView.alpha = 0;
    [coverView setBackgroundColor:[UIColor blackColor]];
    [self.view addSubview:coverView];
}

-(void)setupUI{
    GuessModel *guessModel = self.dataArray[_index-1];
    _indexLabel.text = [NSString stringWithFormat:@"%ld/%ld",_index, self.dataArray.count];
    _titleLabel.text = guessModel.title;
    NSString *imageName = guessModel.icon;
    [_imageButton setImage:[UIImage imageNamed:imageName] forState:UIControlStateNormal];
    //获取答案的长度
    NSString *answer = guessModel.answer;
    NSInteger length = answer.length;
    [self setupAnswerViewWithLength:length];
    [self setupOptionViewWithOptions:guessModel.options];
}

-(void)setupOptionViewWithOptions:(NSArray *)options{
    int column = 7;
    NSInteger count = options.count;
    CGFloat margin = (kScreenSize.width - column*kButtonWidth)/(column+1);
    [_optionView.subviews makeObjectsPerformSelector:@selector(removeFromSuperview)];
    for(int i = 0;i < count;i++){
        NSInteger rowIndex = i/column;
        NSInteger columnIndex = i%column;
        CGFloat buttonX = (columnIndex + 1)*margin + columnIndex*kButtonWidth;
        CGFloat buttonY = rowIndex*margin + rowIndex*kButtonWidth;
        UIButton *button = [[UIButton alloc]initWithFrame:CGRectMake(buttonX, buttonY, kButtonWidth, kButtonWidth)];
        [button setBackgroundImage:[UIImage imageNamed:@"btn_option"] forState:UIControlStateNormal];
        [button setBackgroundImage:[UIImage imageNamed:@"btn_option_highlighted" ] forState:UIControlStateHighlighted];
        [button setTitle:options[i] forState:UIControlStateNormal];
        [button setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
        [button addTarget:self action:@selector(didClickOptionButton:) forControlEvents:UIControlEventTouchUpInside];
        [_optionView addSubview:button];
    }
    
}

-(void)setupAnswerViewWithLength:(NSInteger)count{
    
    CGFloat startX = (kScreenSize.width - count*kButtonWidth - (count-1)*kMargin)/2;
    //添加本题的时候要清除上一题的Button,让数组中所有的元素都执行清除方法
    [_answerView.subviews makeObjectsPerformSelector:@selector(removeFromSuperview)];
    for(int i = 0;i < count;i++){
        CGFloat buttonX = i*kButtonWidth + i*kMargin + startX;
        UIButton *button = [[UIButton alloc]initWithFrame:CGRectMake(buttonX, kMargin, kButtonWidth, kButtonWidth)];
        [button setBackgroundImage:[UIImage imageNamed:@"btn_answer" ] forState:UIControlStateNormal];
        [button setBackgroundImage:[UIImage imageNamed:@"btn_answer_highlighted" ] forState:UIControlStateHighlighted];
        [button setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
        [button addTarget:self action:@selector(didClickAnswerButton:) forControlEvents:UIControlEventTouchUpInside];
        [_answerView addSubview:button];
    }
}

-(IBAction)nextQuestion:(UIButton *)sender {
    _index++;
    [self setupUI];
    _nextButton.enabled = (_index != self.dataArray.count);
     _optionView.userInteractionEnabled = YES;
}

-(void)didClickOptionButton:(UIButton *)optionButton{
    //取出文字
    NSString *title = optionButton.currentTitle;
    //隐藏被点击按钮
    optionButton.hidden = YES;
    //将点击的按钮文字显示到答案区域
    //obj -->数组对象
    //idx -->下标
    //stop -->判断结束
    [_answerView.subviews enumerateObjectsUsingBlock:^(__kindof UIView *_Nonnull obj,NSUInteger idx, BOOL *_Nonnull stop){
        UIButton *answerButton = obj;
        if(answerButton.currentTitle.length==0){
            [answerButton setTitle:title forState:UIControlStateNormal];
            *stop = YES;
        }
    }];
    __block BOOL isComplete = YES;
    [_answerView.subviews enumerateObjectsUsingBlock:^(__kindof UIView *_Nonnull obj,NSUInteger idx, BOOL *_Nonnull stop){
        UIButton *answerButton = obj;
        if(answerButton.currentTitle.length==0){
            isComplete = NO;
            *stop = YES;
        }
    }];
    if (isComplete) {
        //用户不能再点击选项区域中的按钮
        //userInteractionEnabled = NO,禁止任何用户交互，如果是父View设置了这个属性，那么他的子View也将不会接受用户交互
        _optionView.userInteractionEnabled = NO;
        //判断用户是否输入正确，取出用户的答案和标准答案做对比
        //取出用户答案
        NSMutableString *userAnswer = [NSMutableString string];
        [_answerView.subviews enumerateObjectsUsingBlock:^(__kindof UIView *_Nonnull obj,NSUInteger idx, BOOL *_Nonnull stop){
            UIButton *answerButton = obj;
            [userAnswer appendString:answerButton.currentTitle];
        }];
        //取出正确答案
        GuessModel *guessModel = self.dataArray[_index-1];
        NSString *rightAnswer = guessModel.answer;
        NSInteger currentCoin = _coinButton.currentTitle.integerValue;
        if ([userAnswer isEqualToString:rightAnswer]){
            //自动跳转下一题，增加一百金币
            currentCoin += 100;
            //防止最后一题回答正确后继续跳转下一题而导致报错
            if(_index != self.dataArray.count)
                [self performSelector:@selector(nextQuestion:) withObject:nil afterDelay:1];
        }else {
            //答案区域字体变红，减少500金币
            currentCoin -= 500;
            if(currentCoin < 0){
                UIAlertController *alertView = [UIAlertController alertControllerWithTitle:@"温馨提示" message:@"亲，你的钱不够了" preferredStyle:UIAlertControllerStyleAlert];
                UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil];
                UIAlertAction *sureAction = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:nil];
                [alertView addAction:cancelAction];
                [alertView addAction:sureAction];
                [self presentViewController:alertView animated:YES completion:nil];
                return;
            }
             [_answerView.subviews enumerateObjectsUsingBlock:^(__kindof UIView *_Nonnull obj,NSUInteger idx, BOOL *_Nonnull stop){
                 UIButton *answerButton = obj;
                 
                 [answerButton setTitleColor:[UIColor redColor] forState:UIControlStateNormal];
             }];
        }
        NSString *coinTitle = [NSString stringWithFormat: @"%ld",currentCoin ];
        [_coinButton setTitle:coinTitle forState:UIControlStateNormal];
    }
}

-(void)didClickAnswerButton:(UIButton *)answerButton{
    //如果没有文本就直接返回
    if(answerButton.currentTitle.length == 0){
        return;
    }
    //取出文本
    NSString *title = answerButton.currentTitle;
    //清空按钮区域的文本
    [answerButton setTitle:@"" forState:UIControlStateNormal];
    [_optionView.subviews enumerateObjectsUsingBlock:^(__kindof UIView *_Nonnull obj,NSUInteger idx, BOOL *_Nonnull stop){
        UIButton *optionButton = obj;
        //将选项中的相应文字显示出来
        if([optionButton.currentTitle isEqualToString:title]){
            optionButton.hidden = NO;
            *stop = YES;
        }
    }];
    [_answerView.subviews enumerateObjectsUsingBlock:^(__kindof UIView *_Nonnull obj,NSUInteger idx, BOOL *_Nonnull stop){
        UIButton *answerButton = obj;
        
        [answerButton setTitleColor:[UIColor redColor] forState:UIControlStateNormal];
    }];
    //打开选项区域的用户交互
    _optionView.userInteractionEnabled=YES;
}

-(UIStatusBarStyle)preferredStatusBarStyle{
    //在黑色的背景下显示状态栏
    return UIStatusBarStyleLightContent;
}

- (IBAction)didClickTipButton:(UIButton *)sender {
    
    NSInteger currentCoin = _coinButton.currentTitle.integerValue;
    currentCoin -= 1000;
    //做一个判断分数是否大于1000
    if(currentCoin < 0){
        UIAlertController *alertView = [UIAlertController alertControllerWithTitle:@"温馨提示" message:@"亲，你的钱不够了" preferredStyle:UIAlertControllerStyleAlert];
        UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil];
        UIAlertAction *sureAction = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:nil];
        [alertView addAction:cancelAction];
        [alertView addAction:sureAction];
        [self presentViewController:alertView animated:YES completion:nil];
        return;
    }
    NSString *coinstring = [NSString stringWithFormat:@"%ld",currentCoin];
    [_coinButton setTitle:coinstring forState:UIControlStateNormal];
    GuessModel *guessModel = self.dataArray[_index-1];
    NSString *rightAnswer = guessModel.answer;
    NSString *firststring = [rightAnswer substringToIndex:1];
    [_answerView.subviews enumerateObjectsUsingBlock:^(__kindof UIView *_Nonnull obj,NSUInteger idx, BOOL *_Nonnull stop){
        UIButton *answerButton = obj;
        if(idx == 0){
            [answerButton setTitle:firststring forState:UIControlStateNormal];
        }else {
            //如果不是第一个字符串，则清空
            [answerButton setTitle:@"" forState:UIControlStateNormal];
        }
        
    }];
    [_optionView.subviews enumerateObjectsUsingBlock:^(__kindof UIView *_Nonnull obj,NSUInteger idx, BOOL *_Nonnull stop){
        UIButton *optionButton = obj;
        if([optionButton.currentTitle isEqualToString:firststring]){
            //如果字符串相等，就把button隐藏
            optionButton.hidden = YES;
        }else{
            //如果不是第一个就要回到原位
            optionButton.hidden = NO;
        }
    }];
    [_answerView.subviews enumerateObjectsUsingBlock:^(__kindof UIView *_Nonnull obj,NSUInteger idx, BOOL *_Nonnull stop){
        UIButton *answerButton = obj;
        
        [answerButton setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
    }];
    _optionView.userInteractionEnabled = YES;
}
- (IBAction)didClickimageButton:(UIButton *)sender {
    [UIView animateWithDuration:0.5 animations:^{
         _coverView.alpha = 0.6;
        //将图片放大至原来的1.5倍
        _imageButton.transform = CGAffineTransformMakeScale(1.5, 1.5);
    }];
    [self.view addSubview:_coverView];
    //将图片按钮放在coverView的上面
    [self.view bringSubviewToFront:_imageButton];
}
- (IBAction)didClickBigImageButton:(id)sender {
        //通过coverView的alpha值作判断
        if(_coverView.alpha == 0)
        {
            [self didClickimageButton:nil];
        }else {
            [UIView animateWithDuration:0.5 animations:^{
        //将透明度和图片的大小复原
                _coverView.alpha = 0;
                //CGAffineTransformIdentity如果赋值回去，那么之前通过transform改变的属性都会复原
                _imageButton.transform = CGAffineTransformIdentity;
            }];
        }
}
- (IBAction)didClickHelp:(id)sender {
    UIAlertController *alertView = [UIAlertController alertControllerWithTitle:@"亲爱的玩家" message:@"你需要我们的帮助吗？" preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil];
    UIAlertAction *sureAction = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction *_Nonnull action){
        UIAlertController *alertView = [UIAlertController alertControllerWithTitle:@"通道入口" message:@"请问你是哪种类型的玩家呢？" preferredStyle:UIAlertControllerStyleActionSheet];
        UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"非RMB" style:UIAlertActionStyleCancel handler:^(UIAlertAction *_Nonnull action){
            UIAlertController *alertView = [UIAlertController alertControllerWithTitle:@"智障！！！" message:@"没钱玩个毛，滚犊子！" preferredStyle:UIAlertControllerStyleAlert];
            UIAlertAction *sureAction = [UIAlertAction actionWithTitle:@"我这就滚～" style:UIAlertActionStyleDefault handler:nil];
            [alertView addAction:sureAction];
             [self presentViewController:alertView animated:YES completion:nil];
        }];
        UIAlertAction *sureAction = [UIAlertAction actionWithTitle:@"RMB" style:UIAlertActionStyleDefault handler:^(UIAlertAction *_Nonnull action){
            UIAlertController *alertView = [UIAlertController alertControllerWithTitle:@"尊敬的玩家" message:@"我的支付宝账号是15629085780，我们加个好友慢慢聊" preferredStyle:UIAlertControllerStyleAlert];
            UIAlertAction *sureAction = [UIAlertAction actionWithTitle:@"我马上加" style:UIAlertActionStyleDefault handler:nil];
            [alertView addAction:sureAction];
            [self presentViewController:alertView animated:YES completion:nil];
        }];
        [alertView addAction:cancelAction];
        [alertView addAction:sureAction];
        [self presentViewController:alertView animated:YES completion:nil];
    }];
    [alertView addAction:cancelAction];
    [alertView addAction:sureAction];
    [self presentViewController:alertView animated:YES completion:nil];
}
- (IBAction)kaigua:(id)sender {
    if ([self.gua.text isEqualToString:@"Kana"]) {
         UIAlertController *alertView = [UIAlertController alertControllerWithTitle:@"特殊的玩家" message:@"题目的答案问问你旁边的人就知道了哟" preferredStyle:UIAlertControllerStyleAlert];
        UIAlertAction *sureAction = [UIAlertAction actionWithTitle:@"这真是个好主意" style:UIAlertActionStyleDefault handler:nil];
        [alertView addAction:sureAction];
        [self presentViewController:alertView animated:YES completion:nil];
    }else if ([self.gua.text isEqualToString:@"潘振鹏"]){
        NSInteger currentCoin = _coinButton.currentTitle.integerValue;
        currentCoin += 10000;
        NSString *coinString = [NSString stringWithFormat:@"%ld", currentCoin];
        [_coinButton setTitle:coinString forState:UIControlStateNormal];
    }
}


@end

```

```objective-c
//猜测字谜的答案部分，字典转模型
//  GuessModel.m
#import "GuessModel.h"

@implementation GuessModel
-(instancetype)initWithDict:(NSDictionary *)dict{
    if (self = [super init]) {
        [self setValuesForKeysWithDictionary:dict];
    }
    return self;
}
+(instancetype)guessModelDict:(NSDictionary *)dict{
    return [[self alloc]initWithDict:dict];
}
@end
//  GuessModel.h
#import <Foundation/Foundation.h>

@interface GuessModel : NSObject
@property (nonatomic, copy)NSString *answer;
@property (nonatomic, copy)NSString *icon;
@property (nonatomic, copy)NSString *title;
@property (nonatomic, copy)NSArray *options;

-(instancetype)initWithDict:(NSDictionary *)dict;
+(instancetype)guessModelDict:(NSDictionary *)dict;
@end

```

## 10.UIAlertController提示框

两种提示的方式：

1.中部弹出：style为UIAlertControllerStyleAlert

2.底部弹出：style为UIAlertControllerStyleActionSheet

```objective-c
//创建点击的选项并加入提示框
UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil];
        UIAlertAction *sureAction = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:nil];
        [alertView addAction:cancelAction];
        [alertView addAction:sureAction];
//在handler中可以进行回调，nil则没有触发事件
```

## 11.KVC模型

使用kvc进行快速赋值[字典快速赋值](http://www.jianshu.com/p/870eb4b4170a)：

```objective-c
test.name=dic[@"name"];
test.sex=dic[@"sex"];
test.age=dic[@"age"];
//将赋值的过程替换为一句话
[test setValuesForKeysWithDictionary:dic];
//如果model中有不存在于dict中的元素
@property (nonatomic,copy)NSString *other;
//此时如果打印一下other的值，可以看到他的值为null，显而易见,dic中的值可以完全赋值给model，而other没有被赋值，所以值是空的
NSLog(@"test.other=%@",test.other);
test.other=(null)
//如果dict中有不存在于Model中的元素
Terminating app due to uncaught exception 'NSUnknownKeyException',
reason: '[<PersonModel 0x7fd731517910> setValue:forUndefinedKey:]:
this class is not key value coding-compliant for the key age.'
//通过了编译，但是运行时报错,解决方式就是实现一个方法setValue:forUndefinedKey: 这个方法能过滤掉不存在的键值。在model.h／m文件中添加
-(void)setValue:(id)value forUndefinedKey:(NSString *)key;
-(void)setValue:(id)value forUndefinedKey:(NSString *)key{

    }
//在.m文件中不需要写任何的内容，现在就可以成功运行了
```



