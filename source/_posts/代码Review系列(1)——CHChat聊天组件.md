---
title: 代码Review系列(1)——CSChat聊天组件
categories: 组件 #分类
date: 2016-06-28 16:05:08
#tags: [博客] #代码Review
#description: 
---

## 背景介绍
CSChat聊天组件是因为公司业务需要集成XMPP即时通信功能，当时仿照微信做了一个聊天的控制器，想到这个Controller会在多处以及多个项目中集成，所以按照MVVM加命令模式把收发消息的功能和UI展示的效果进行解耦，并想对其中每一部分的功能单独复用。
以下博文是根据第二个版本功能（增加群聊功能）代码的Review以及重构建议。关于CSChat的具体设计思路在以下链接:[CSChat聊天组件设计思路](http://chausson.github.io/)

## 文件夹结构
<img src="http://chausson.github.io/img/fileMeum.png"  title="工程文件夹目录">

* 问题:
1.当前文件的结构混乱，作为一个展示的Demo没有达到直观的效果,第三方的类库也没有进行归档和整理。
2.文件以及类的过多臃肿。
3.文件夹以及类名不够明确，不能直观的明白这些文件夹和类具体是存放什么的，它的职责是什么。

* 建议:
1.文件夹的目录划分层级关系应当清晰，可以把所以都放置在一个文件夹下，里面根据功能以及层级关系进行不同的文件夹的拆分。
2.对外引用或者暴露的类越少越好，可以将第三方依赖库大部分支持利用cocoapod管理，而将其他的工具类可以作为一个私有方法类，在内部使用,比如可以新建一个CSChatPrivate的类，里面集成所有工具类提供的方法。
3.可以利用一些简短或者易懂的命名直观的了解这个类以及该文件夹存在的模块内容，以及职责。

## 主要控制器CSChatViewController
* CSChatViewController.h：
``` obj-c
#import <UIKit/UIKit.h>
#import "CSChatVIewModel.h"
#import "CSChatToolView.h"
#import "CSChatTableView.h"
@interface CSChatViewController : UIViewController<CSChatToolViewKeyboardProtcol, UITableViewDataSource, UITableViewDelegate>{
    NSMutableArray *messageList;
    
    NSMutableDictionary *sizeList;
}
- (instancetype)init __unavailable;
- (instancetype)initWithViewModel:(CSChatViewModel *)viewModel;

@property (strong ,nonatomic) CSChatToolView *chatView;
@property (strong ,nonatomic) CSChatTableView *chatTableView;
@property (strong ,nonatomic) CSChatViewModel *viewModel;
@end

```
h文件中只暴露公开的属性，并且禁用其他初始化方法，在新增的初始化方法中增加注释。

* CSChatViewController.m：
``` obj-c
- (instancetype)initWithViewModel:(CSChatViewModel *)viewModel{
    self = [super init];
    if (self) {
        _viewModel = viewModel;
        self.title = viewModel.chatControllerTitle;
        self.view.backgroundColor = [UIColor whiteColor];
         _chatView = [[CSChatToolView alloc]initWithObserver:self];
        _chatTableView = [[CSChatTableView alloc] init];
        _chatTableView.delegate = self;
        _chatTableView.dataSource = self;
        [self layOutsubviews];
        @weakify(self);
        [RACObserve(self.viewModel, cellViewModels) subscribeNext:^(NSArray *cells) {
            @strongify(self);
            [self.chatTableView reloadData];
            if (cells.count >5) {
                [self.chatTableView scrollToRowAtIndexPath:
                 [NSIndexPath indexPathForRow:[cells count]-1 inSection:0]
                                          atScrollPosition: UITableViewScrollPositionBottom
                                                  animated:NO];
                
                [self.chatTableView reloadData];
            }
        }];
      
    }
    return self;
}
#pragma mark activity
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
}

```
1.可以在didload之后增加两个方法一个是ui初始化以及布局的方法(可以拆分成ui初始化配置以及增加到父容器和增加约束的方法),还有一个就是绑定VM属性相关的方法.
2.在initWithViewModel方法中，尽量不要初始化ui相关的对象,只保持数据的初始化,将UI布局方法和数据初始化方法分离。

## CSChatViewModel(主要Controller的ViewModel)
* CSChatViewModel.h：
``` obj-c
#import "CSChatCellViewModel.h"
#import <Foundation/Foundation.h>
#import "CSChatModel.h"

typedef void(^chatBlock)(CSChatModel* list);

@interface CSChatViewModel : NSObject
- (instancetype)init __unavailable;
- (instancetype)initWithMessageList:(CSChatModel *)list;
/** 聊天列表VM*/
@property (nonatomic ,strong ) NSArray <CSChatCellViewModel *>*cellViewModels;
/** 自己用户图标*/
@property (nonatomic ,copy ) NSString *userIcon;
/** 接收图标*/
@property (nonatomic ,copy ) NSString *receiverIcon;
/** 显示标题*/
@property (nonatomic ,copy ) NSString *chatControllerTitle;

- (void)postMessageWithText:(NSString *)text;
- (void)sendSoundWithVoice:(NSString *)path;

@end

```
1.没有对增加的Block进行注释，并且通过变量名和看不出是做什么的。
2.- (instancetype)initWithMessageList:(CSChatModel *)list; 这个初始化方法可以废弃,因为聊天列表需要的是CSChatCellViewModel的对象,并不关心CSChatModel，可以尽量不暴露和Model业务有关的方法，甚至有必要可以去除.

* CSChatViewModel.m：
``` obj-c
#define FACE_NAME_HEAD  @"/s"
// 表情转义字符的长度（ /s占2个长度，xxx占3个长度，共5个长度 ）
#define FACE_NAME_LEN   5


NSString * swiftDateToStr(NSDate *date){
    
    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
    formatter.dateFormat = @"HH:mm";

    return   [formatter stringFromDate:date];
}
static const CGFloat kDefaultPlaySoundInterval = 3.0;

//用来判断是否可以收发消息
BOOL isLayout = YES;

@interface CSChatViewModel()<EMChatManagerDelegate>

@end


@implementation CSChatViewModel{
    
    NSDate *_lastPlaySoundDate;
}
- (instancetype)initWithMessageList:(CSChatModel *)list{
    
    self = [super init];
    if (self) {
        if (isLayout) {
           __weak  typeof(self) weakSelf = self;
            [[CSChatBusinessCommnd standardChatDefaults]setReceiverBlock:^(NSString *message) {
                __strong typeof(self) strongSelf = weakSelf;

                
                
                [strongSelf receiverMessageWithText:message];
            }];
  
        }else{
            UIAlertView *alert = [[UIAlertView alloc]initWithTitle:@"TIPS" message:@"NON BUINESS MODEL" delegate:nil cancelButtonTitle:@"OK" otherButtonTitles:nil, nil];
            [alert show];
        }
        NSMutableArray *cellTempArray = [[NSMutableArray alloc ]initWithCapacity:list.chatContent.count];
        for (int i = 0; i < list.chatContent.count; i++) {
            CSChatCellViewModel *cellViewModel = [[CSChatCellViewModel alloc]initWithModel:list.chatContent[i]];
            if (i != 0) {
            CSChatViewItemModel *last = list.chatContent[i-1];

                
            [cellViewModel sortOutWithTime:last.time];
            }
            [cellTempArray addObject:cellViewModel];
        }
         _cellViewModels = [NSArray arrayWithArray:cellTempArray];
    }
    
//    //注册环信消息回调
//    [[EMClient sharedClient].chatManager addDelegate:self delegateQueue:nil];
    
    return self;
}


@end

```
1.初始化方法整体较为混乱,并且ViewModel中不应该有和UI控件直接相关的东西,去除UIAlertView或者通过其他类去关联，而不应该直接使用。
2.EMChatManagerDelegate是环信SDK中的类，不应该出现在ViewModel中。
3.变量名在OC的语法中应该能直观的看出这个变量的作用，建议在斟酌一下。

``` obj-c
- (void)postMessageWithText:(NSString *)text{
    CSChatViewItemModel *model = [[CSChatViewItemModel alloc] init];
    model.content = text;
    model.icon = self.userIcon;
    model.type = CSMessageText;
    model.time = swiftDateToStr([NSDate date]);
    CSChatCellViewModel *cellViewModel = [[CSChatCellViewModel alloc]initWithModel:model];
    [cellViewModel sortOutWithTime:[_cellViewModels lastObject]?[_cellViewModels lastObject].time:nil];
    NSMutableArray *cellTempArray = [NSMutableArray arrayWithArray:[_cellViewModels copy]];
    [cellTempArray addObject:cellViewModel];
    self.cellViewModels = [NSArray arrayWithArray:cellTempArray];
    if (isLayout) {
        
        
        NSMutableDictionary *dic = [NSMutableDictionary dictionary];
        
        [dic setObject:[text stringByReplacingEmojiUnicodeWithCheatCodes] forKey:@"msgContent"];
        
        
        //判断是发单聊消息还是群聊消息给服务器
        if ([CSChatGroupSet sharedGroupSet].chatType == CSChatCellChatTypeChat) {
            
            
            [[CSChatBusinessCommnd standardChatDefaults] postMessageWithDic:dic andUrl:nil];
        }else{
            
            [[CSChatBusinessCommnd standardChatDefaults] postGroupMessageWithDic:dic andUrl:nil];
        }

        
    }

}
```
1.CSChatGroupSet不应该利用该单例去判断群聊还是单聊，没有充分利用ViewModel自身类的作用，有点冗余。
2.对发送消息而言不应该传dic这个变量,没有把发送和接收的方法完全解耦开，有点和业务相关了，不了解业务的情况下，不知道如何拼装dic。

* CSChatConfiguration.h：
``` obj-c
#import <Foundation/Foundation.h>

@interface CSChatConfiguration : NSObject


//环信注册
+(void)registerWithUsername;
//环信登录
+(void)loginWithUsername;
//创建群组
+(NSString*)bulidGroup;

//发送消息到环信
+(void)sendToHyphenate:(NSString*)text;
//发送群聊消息到环信
+(void)sendToHyphenateGroup:(NSString*)text;

@end
```
1.既然是一个配置类,那所有的配置信息都应该在该类中展现,包括刚刚提到的群聊单聊，如果有必要可以开放一些获取全局配置信息的方法。
2.不应该出现发送消息这个功能,配置类不会去做具体功能的实现。
3.所有的配置方法都没有传入参数,那是如何进行配置，如果是静态的，那这个类的作用完全没有体现。
4.可以为一些拓展的功能预留一些配置的接口，比如是否消息进行本地存储,是否支持拍照和上传图片,是否支持定位。

* CSChatBusinessCommnd.h：
``` obj-c
#import <Foundation/Foundation.h>
#import "CSChatModel.h"

typedef void(^ReceiverBlock)(NSString *);
//返回数据的block
typedef void(^listBlock)(CSChatModel *listModel);

@class CSChatBusinessCommnd;

@protocol CSChatBusinessCommndDelegate <NSObject>
- (void)receiveMessageWithText:(NSString *)message;

@end


@interface CSChatBusinessCommnd : NSObject
+ (instancetype)standardChatDefaults;

// 发送单聊消息
- (void)postMessageWithDic:(NSDictionary *)dic andUrl:(NSString *)url;
// 发送群聊消息
- (void)postGroupMessageWithDic:(NSDictionary *)dic andUrl:(NSString *)url;

- (void)postSoundWithData:(NSString *)path;
- (void)setReceiverBlock:(ReceiverBlock )block;

//接收消息的回调
-(void)receiveMessage:(listBlock)dicBlock;

@end
```
1.该类的目的应该是对执行发送和接收消息的操作执行的人和发事件的人解耦,所有的具体收发事件必须通过该类去完成。
2.其中post的方法可以不必带入url等参数,如果需要可以利用之前建立的configuration类进行处理,可以对外部增加一些发送以及接收的接口,或者传入不同的参数。
3.对发送成功和失败应该有一些处理。

* CSChatToolView.h：
``` obj-c
#import <UIKit/UIKit.h>
@class CSChatToolView;
@protocol CSChatToolViewKeyboardProtcol <NSObject>
@optional
- (void)chatKeyboardWillShow;
- (void)chatKeyboardDidShow;
- (void)chatKeyboardWillHide;
- (void)chatKeyboardDidHide;
- (void)chatInputView;

- (void)sendMessageWithText:(NSString *)text;
- (void)sendSoundWithDataPath:(NSString *)voice;
@end
@interface CSChatToolView : UIView

- (instancetype)init __unavailable;
- (instancetype)initWithFrame:(CGRect)frame __unavailable;
/**
 * @brief 初始化toolView并设计观察对象
 * @return toolview实例对象
 */
- (instancetype)initWithObserver:(NSObject<CSChatToolViewKeyboardProtcol>*)object;
/**
 * @brief 是否隐藏键盘
 */
- (void)setKeyboardHidden:(BOOL)hidden;
/**
 * @brief 拓展事件调用
 */
- (void)assistanceActionWithIndex:(NSInteger )index
                         andBlock:(void (^)())block;
/**
 * @brief 在视图添加到父视图之后调用 约束布局
 */
- (void)autoLayoutView __attribute((deprecated("这个接口等实现约束以后再启用")));
@end
```
1.该类是键盘输入框的视图,如果支持单独复用，可以为该类增加一些必要的属性提高自定义性，比如该输入框当前的状态,当前输入的文字,或者内容等。

* CSChatToolView.ma
1.该类的内容太多就不贴代码了,可以适当的把一些方法提取出来放入我们之前提到CSChatPrivate私有类中。
2.最好与第三方HUD解耦,自己实现录音的时间效果等。
``` obj-c
//发送消息到环信
-(void)sendToHyphenate:(NSString*)text{
    
    
    //判断是发单聊消息还是群聊消息给环信
    if ([CSChatGroupSet sharedGroupSet].chatType == CSChatCellChatTypeChat) {
        
         [CSChatConfiguration sendToHyphenate:text];
    }else{
        
        [CSChatConfiguration sendToHyphenateGroup:text];
    }

}
```
3.其中不应该出现与环信有关的方法,所有的发送和接收应该由刚刚提到的Command类去处理。


