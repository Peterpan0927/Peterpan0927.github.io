---
title: iOS-UI学习(五)
date: 2017-06-03 00:48:11
tags: TableView cell
categories: 多种代理方法 cell重用 数据模型嵌套
author: Peterpan
---

UI初步（五）
<!--more-->

# UI基础Day6

## 1.tableView基本使用

仅仅在main.storyboard中加入一个UITableView的控件，可以显示如下的效果：

![11A23983-B207-46FA-BD95-D941390BFF4A](http://omunhj2f1.bkt.clouddn.com/11A23983-B207-46FA-BD95-D941390BFF4A.png)

其中每一个小横行被称作cell，我们可以使用代理的方式来添加每一行中的信息和cell的数量，注意，这两个方法在协议中是用@required来修饰的，表示必须实现，否则就会报错：

```objective-c
#import "ViewController.h"

@interface ViewController ()<UITableViewDataSource>

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
}
//创建cell的数量
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    //根据自定义返回每一节的cell数量和内容
  	 NSInteger row = 0;
    switch (section) {
        case 0:
            row = 3;
            break;
        case 1:
            row = 2;
            break;
        case 2:
            row = 1;
            break;
        default:
            break;
    }
    return row;
}
//编辑cell中的文本内容
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(nonnull NSIndexPath *)indexPath{
    UITableViewCell *cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:nil];
    
   switch (indexPath.section) {
        case 0:
            switch (indexPath.row) {
                case 0:
                    cell.textLabel.text = @"Linux";
                    break;
                case 1:
                    cell.textLabel.text = @"Mac";
                    break;
                case 2:
                    cell.textLabel.text = @"Windows";
                    break;
            }
            break;
        case 1:
            switch (indexPath.row) {
                case 0:
                    cell.textLabel.text = @"QQ";
                    break;
                case 1:
                    cell.textLabel.text = @"WeChat";
                    break;
            }
            break;
        case 2:
            switch (indexPath.row) {
                case 0:
                    cell.textLabel.text = @"Baidu";
                    break;
            }
            break;
   }
    return cell;
}

//创建cell的节数
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView{
    return 3;
}

//添加头部和尾部的信息
- (NSString *)tableView:(UITableView *)tableView titleForFooterInSection:(NSInteger)section{
    return @"fuck you";
}

-(NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section{
    return @"我是Van";
}

@end
```

### plain:

​	如果设置了section的footer 和header上的title，在滚动的时候就会产生悬浮效果

### group:

​	section的头部和底部会默认的有一段间距。

### 数据源调用方法顺序：

1.numberOfSectionsInTableView

2.numberOfRowsInSection 调用多遍，上面的方法返回多少组久调用多少次

3.cellForRowAtIndexPath 总共有多少行就调用多少遍

## 2.headerView的使用

tableView和section都可以设置headerView:

```objective-c
//tableView
UIView *headerView = [[UIView alloc]initWithFrame:CGRectMake(0,0,100,100)];
[headerView setBackgroundColor:[UIColor orangeColor]];
_tableView.tableHeaderView = headerView;
//section
- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section{
    UIView *headerView = [[UIView alloc]init];
    [headerView setBackgroundColor:[UIColor redColor]];
    return headerView;
}

```

效果图：![屏幕快照 2017-06-01 下午7.52.18.png](http://omunhj2f1.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-06-01%20%E4%B8%8B%E5%8D%887.52.18.png) 

## 3.demo展示（英雄属性）

```objective-c
#import "ViewController.h"
#import "HeroModel.h"

@interface ViewController ()<UITableViewDataSource,UITableViewDelegate>

@property (nonatomic, strong) NSArray *dataArray;
@property (weak, nonatomic) IBOutlet UITableView *tableView;

@end

@implementation ViewController

#pragma mark -
#pragma mark -  懒加载
- (NSArray *)dataArray {
    if (nil == _dataArray) {
        // 1. 路径
        NSString *path = [[NSBundle mainBundle] pathForResource:@"heros.plist" ofType:nil];
        
        // 2. 读取内容
        NSArray *tempArray = [NSArray arrayWithContentsOfFile:path];
        
        // 3. 可变数组
        NSMutableArray *mutable = [NSMutableArray array];
        
        // 4. 字典转模型
        for (NSDictionary *dict in tempArray) {
            HeroModel *model = [HeroModel heroModelWithDict:dict];
            
            [mutable addObject:model];
        }
        
        _dataArray = mutable;
        
    }
    return _dataArray;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 设置控制器成为tableView的代理
    _tableView.delegate = self;
    
    // 设置分割线颜色的
    _tableView.separatorColor = [UIColor redColor];
    
    // 侵蚀 , 分割线样式
    _tableView.separatorStyle = UITableViewCellSeparatorStyleSingleLine;
    
    // top, left, bottom, right , 上， 下 是没有效果的
    _tableView.separatorInset = UIEdgeInsetsMake(0, 0, 0, 0);
    
    // 允许多选
    _tableView.allowsMultipleSelection = YES;
    
    // 设置行高 ,（静态设置） 如果每个cell的高度都一样， 推荐这种设置
//    _tableView.rowHeight = 100;
    
    
//    UIView *headerView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 100, 100)];
//    [headerView setBackgroundColor:[UIColor orangeColor]];
//    
//    _tableView.tableHeaderView = headerView;
    
    UIView *footerView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 100, 100)];
    [footerView setBackgroundColor:[UIColor yellowColor]];
    
    _tableView.tableFooterView = footerView;
    
}

// indexPath  可以根据具体的某行显示不同的高度
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    // 反回不同的高度
    return 55;
}


// 多少组
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    return 1;
}

// 多少行

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return self.dataArray.count;
}

// 每一行显示的内容
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    // 实例化tableViewcell
    UITableViewCell *cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:nil];
    /**
     UITableViewCellStyleDefault : 不显示detailTextLabel
     UITableViewCellStyleValue1 ： detailLabel 显示在 textLabel 右侧
     UITableViewCellStyleValue2 ： imageView不再显示， textLabel居左 变蓝色
     UITableViewCellStyleSubtitle ：都显示， detailLabel 在 textLabel下侧
     */
    
    // 设置cell上控件的内容
    HeroModel *model = self.dataArray[indexPath.row];
    
    // 设置imageView
    cell.imageView.image = [UIImage imageNamed:model.icon];
    
    // 设置文本
    cell.textLabel.text = model.name;
    
    // 设置detailTextLabel
    cell.detailTextLabel.text = model.intro;
    
    // 设置右侧箭头
    // accessory : 配件
    cell.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
    
    // 设置选择样式
    /**
     UITableViewCellSelectionStyleNone,
     UITableViewCellSelectionStyleBlue,  用灰色来代替了
     UITableViewCellSelectionStyleGray,
     UITableViewCellSelectionStyleDefault
     cell.selectionStyle = UITableViewCellSelectionStyleBlue;
     */
    
    /**
     设置选中的背景view
     UIView *tempView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 100, 30)];
     tempView.backgroundColor = [UIColor yellowColor];
     
     cell.selectedBackgroundView = tempView;
     */
    
    // cell.backgroundColor = [UIColor yellowColor];
    
    /**
     accessoryView 自定义控件
     自定义 accessoryView 的时候， frame中的 坐标（x,y)修改后无效
     UIView *tempView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 30, 30)];
     tempView.backgroundColor = [UIColor redColor];
     
     cell.accessoryView = tempView;
     */
    
    return cell;
}


- (NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section {
    
    return @"英雄";
}

// 可以对 section的header 和 footer 设置view
- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section {
    
    
    UIView *headerView = [[UIView alloc] init];
    [headerView setBackgroundColor:[UIColor redColor]];
    
    return headerView;
}


- (BOOL)prefersStatusBarHidden {
    return YES;
}


@end
```

上述代码虽然能实现基本的功能，但是有一个很大的弊端，就是在拖动tableView的时候，每一次都在不停的创建新的cell，意思就是，当最上方的cell超出屏幕的时候，再一次往上拖动的时候就会重新实例化一个对象。

cell重用机制：

1.通过重用标识符在缓存池中去寻找对应的的cell

2.进行判断：如果找到，直接拿过来复用，如果找不到就直接实例化一个对象

3.把旧的数据覆盖掉

优化后的代码：

```objective-c
//避免多次分配内存，先定义重用标识符
static NSString *identifier = "heroCell"; 
//到缓存池中去寻找cell，将标识符设置为"heroCell"
UITableView *cell = [tableView dequeueReusableCellWithIdentifier:identifier];
//判断是否找到
if (nil == cell) {
 cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:indentifier];
}
//如果找到之后就进行数据的复写，没有找到就是直接赋值
```

## 4.实现数据模型的嵌套和索引

```objective-c
#import "ViewController.h"
#import "CarModel.h"
#import "InnerCarModel.h"

@interface ViewController ()<UITableViewDataSource>

@property (nonatomic, strong) NSArray *dataArray;

@property (nonatomic, strong) NSArray *indexArray;

@end

@implementation ViewController

#pragma mark -
#pragma mark -  懒加载
- (NSArray *)dataArray {
    if (nil == _dataArray) {
        
        // 1. 路径
        NSString *path = [[NSBundle mainBundle] pathForResource:@"cars_total.plist" ofType:nil];
        
        // 2. 读取
        NSArray *tempArray = [NSArray arrayWithContentsOfFile:path];
        
        // 3. 临时可变数组
        NSMutableArray *mutal = [NSMutableArray array];
        
        // 4. 转模型
        for (NSDictionary *dict in tempArray) {
            CarModel *model = [CarModel carModelWithDict: dict];
            
            [mutal addObject:model];
        }
        
        // 把carModel中title 放入数组， 用作后面的索引返回值
        _indexArray = [mutal valueForKeyPath:@"title"];
        
        _dataArray = mutal;

    return _dataArray;
}


- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
}


// 多少组
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    return self.dataArray.count;
}

// 多少行
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    CarModel *carModel = self.dataArray[section];
    
    return carModel.cars.count;
}

// 每一行显示的内容
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    // 1. 定义一个重用标识符
    static NSString *identifier = @"settingCell";
    
    // 2. 到缓存池中去找对应的cell， 根据重用标识符
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:identifier];
    
    // 3. 判断是否找到， 如果找不到， 就重新实例化cell
    if (nil == cell) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:identifier];
    }
    
    CGRect
    // 4. 设置数据
    // 先取出该组对应的model
    CarModel *carModel = self.dataArray[indexPath.section];
    
    // 再取出该组中行对应的model
    InnerCarModel *innerCarModel = carModel.cars[indexPath.row];
    
    // 对cell的控件进行赋值
    cell.imageView.image = [UIImage imageNamed:innerCarModel.icon];
    
    cell.textLabel.text = innerCarModel.name;
    
    return cell;
}

- (NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section {
    // 取出carModel
    CarModel *carModel = self.dataArray[section];
    
    return carModel.title;
}

- (NSArray<NSString *> *)sectionIndexTitlesForTableView:(UITableView *)tableView {
    
    return _indexArray;
}



- (BOOL)prefersStatusBarHidden {
    return YES;
}


@end
```

```objective-c
//嵌套在CarModel中的InnerCarModel
#import "CarModel.h"
#import "InnerCarModel.h"

@implementation CarModel

- (instancetype)initWithDict:(NSDictionary *)dict {
    if (self = [super init]) {
        
        [self setValuesForKeysWithDictionary:dict];
        // 经过kvc赋值之后， 现在 cars 这个数组中有值， 而且存放的是 字典
        // 1. 定义一个临时可变数组
        NSMutableArray *mutable = [NSMutableArray array];
        
        // 2. 转
        for (NSDictionary *dict in self.cars) {
            InnerCarModel *innerModel = [InnerCarModel innerCarModelWithDict:dict];
            
            // 添加到可变数组中
            [mutable addObject:innerModel];
        }
        
        // 把可变数组赋值给 self.cars , mutable 数组中装的是 InnerCarModel对象
        self.cars = mutable;
    }
    return self;
}

+ (instancetype)carModelWithDict:(NSDictionary *)dict {
    return [[self alloc] initWithDict:dict];
}

@end
```

## 5.对cell的编辑（修改处）

```objective-c
@interface ViewController ()<UITableViewDataSource,UITableViewDelegate,UIAlertViewDelegate>
//因为要对数据源进行修改，所以要用一个可变数组，同时还要加入新的协议
- (NSMutableArray *)dataArray {
    if (nil == _dataArray) {
        
        // 实例化 dataArray
#warning 不要忘记实例化 dataArray数组
        _dataArray = [NSMutableArray array];
        
        // 1. 路径
        NSString *path = [[NSBundle mainBundle] pathForResource:@"heros.plist" ofType:nil];
        
        // 2. 读取内容
        NSArray *tempArray = [NSArray arrayWithContentsOfFile:path];
        
        // 3. 可变数组
//        NSMutableArray *mutable = [NSMutableArray array];
        
        // 4. 字典转模型
        for (NSDictionary *dict in tempArray) {
            HeroModel *model = [HeroModel heroModelWithDict:dict];
            
            [_dataArray addObject:model];
        }
        
//        _dataArray = mutable;
        
    }
    return _dataArray;
}

// 当cell 被选中的时候 ， 就会调用
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    
    // 取出数组中对应的model
    HeroModel *heroModel = self.dataArray[indexPath.row];
   
    // 弹出提示框
    UIAlertView *alertView = [[UIAlertView alloc]
                              initWithTitle:@"编辑"                                                  message:nil
                                                       delegate:self
                                              cancelButtonTitle:@"取消"
                                              otherButtonTitles:@"确定", nil];
    
    // 显示输入框
    /**
     UIAlertViewStyleDefault = 0,
     UIAlertViewStyleSecureTextInput,
     UIAlertViewStylePlainTextInput,
     UIAlertViewStyleLoginAndPasswordInput
     */
    alertView.alertViewStyle = UIAlertViewStylePlainTextInput;
    
    
    // 把英雄的名字赋值给 alertView中textField
    // 取出aletView的 textField
    UITextField *textField = [alertView textFieldAtIndex:0];
    
    textField.text = heroModel.name;
    
    // 点击cell的时候， 把row 设置给alertView的tag属性
    alertView.tag = indexPath.row;
    
    
    // 展示alertView
    [alertView show];
   
}

#pragma mark -
#pragma mark -  alertView的代理方法
- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex {
    
    if (buttonIndex == 1) { // 点击了确定按钮
        /**
         修改数据显示， 修改英雄的名字
         */
        
        // 取出 alertView中的文本输入框
        UITextField *textField = [alertView textFieldAtIndex:0];
        
        // 修改过之后的名字
        NSString *newName = textField.text;
        
        // 修改数据源中 的 英雄名字
        // 取出alertView 的tag , 点击的某个行
        NSInteger index = alertView.tag;
        
        // 取出点击行所对应的 数据源中的模型
        HeroModel *model = self.dataArray[index];
        
        // 修改model中的 name属性
        model.name = newName;
        
        
        // 刷新tableView中的数据, 对所有数据进行刷新
//        [_tableView reloadData];
        
        
        // 对某一行cell的数据进行刷新
        NSIndexPath *indexPath = [NSIndexPath indexPathForRow:index inSection:0];
        [_tableView reloadRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationLeft];
        
    }
}
```

## 6实现插入和删除

```objective-c
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath {
    
    // 可以对传入的编辑模式进行判断
    if (editingStyle == UITableViewCellEditingStyleDelete) {
        
        // 1. 在数据源中把row 对应到 dataArray 下标中的对象 给移除掉
        [self.dataArray removeObjectAtIndex:indexPath.row];
        
        // 2. 刷新数据
//        [_tableView reloadData];
//        [_tableView reloadRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationLeft];
        
        [_tableView deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationLeft];
        
    } else if (editingStyle == UITableViewCellEditingStyleInsert) {
        
        // 1. 根据点击的indexPath.row 在数据源中插入heroModel 对象
        HeroModel *model = [[HeroModel alloc] init];
        model.name = @"千珏";
        
        [self.dataArray insertObject:model atIndex:indexPath.row];
        
        // 2. 刷新数据
        
        [_tableView insertRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationLeft ];
    }
}
- (UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath {
    //可以通过按钮来改变编辑的模式
    return UITableViewCellEditingStyleInsert;
}
//自定义删除按钮上显示的文本
- (nullable NSString *)tableView:(UITableView *)tableView titleForDeleteConfirmationButtonForRowAtIndexPath:(NSIndexPath *)indexPath {
    return @"咔嚓掉";
}
//当一行cell被取消选中的时候待用
- (void)tableView:(UITableView *)tableView didDeselectRowAtIndexPath:(NSIndexPath *)indexPath {
    NSLog(@"调用了--- %ld", indexPath.row);
}
//当一行cell被选中的时候调用
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    
}
```
