---
layout:     post
title:      "iOS 之 UIcollectionView学习"
subtitle:   ""
date:       2014-9-18
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
    - UIcollectionView
---

**版权声明：本文为博主原创文章，未经博主允许不得转载**

首先`UICollectionView`的确非常强大，只要完成布局，可以变化出很多种风格的界面，那么这里只介绍最常用的界面布局，也就是网格界面布局
那么直接上代码（都是纯代码完成界面布局）：

首先是`UICollectionViewFlowLayout`的初始化，作用是用来布局，在`UICollectionView`中纯代码的初始化方法是需要用到这个类来进行布局的
`UICollectionViewFlowLayout *flowLayout;`

**其有如下属性：**

```Objective_C
@property (nonatomic) CGFloat minimumLineSpacing;  //cell行间距
@property (nonatomic) CGFloat minimumInteritemSpacing;  //cell列间距
@property (nonatomic) CGSize itemSize;	//cell大小
@property (nonatomic) UICollectionViewScrollDirection scrollDirection; // 滑动方向@property (nonatomic) CGSize headerReferenceSize;	//header大小
@property (nonatomic) CGSize footerReferenceSize;	//footer大小
@property (nonatomic) UIEdgeInsets sectionInset;	//section
```
**基本上面这些就够了，初始化：**

```Objective_C
//确定是水平滚动，还是垂直滚动
flowLayout=[[UICollectionViewFlowLayout alloc] init];
[flowLayout setScrollDirection:UICollectionViewScrollDirectionVertical];
flowLayout.minimumLineSpacing = 0.1f;
flowLayout.minimumInteritemSpacing = 0.1f;
```

然后是`UIcollectionView`的初始化了，其初始化方法有两个：

```Objective_C
- (instancetype)initWithFrame:(CGRect)frame collectionViewLayout:(UICollectionViewLayout *)layout;
- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder;
```
本文只用了第一个，由于`UIcollectionView`也是继承`UIscrollView`的，所以很多属性方法都是可以用父类的。

```Objective_C
//UIcollectionView的初始化：
@property (nonatomic, retain) UICollectionView *collectionView;
self.collectionView=[[UICollectionView alloc] initWithFrame:CGRectMake(0, 0,SCREEN_WIDTH,self.view.height) collectionViewLayout:flowLayout];
self.collectionView.dataSource=self;
self.collectionView.delegate=self;
[self.collectionView setBackgroundColor:[UIColor whiteColor]];
//代码控制header和footer的显示
//	flowLayout.headerReferenceSize = CGSizeMake(SCREEN_WIDTH, 70);
[self.view addSubview:self.collectionView];
self.collectionView.alwaysBounceVertical = YES;
self.collectionView.userInteractionEnabled = YES;
self.collectionView.showsVerticalScrollIndicator = NO;
```
同样的`UIcollectionView`也需要设置`delegate`和`dataSource`，其用法也是和`UITableView`差不多
需要注意的地方是`cell`的注册方法，以及`header`和`footer`的注册方法不同。

**其注册方法主要有这些：**

```Objective_C
//cell的注册方法：
- (void)registerClass:(nullable Class)cellClass forCellWithReuseIdentifier:(NSString *)identifier;
- (void)registerNib:(nullable UINib *)nib forCellWithReuseIdentifier:(NSString *)identifier;
//header和footer的注册方法：
- (void)registerClass:(nullable Class)viewClass forSupplementaryViewOfKind:(NSString *)elementKind withReuseIdentifier:(NSString *)identifier;
- (void)registerNib:(nullable UINib *)nib forSupplementaryViewOfKind:(NSString *)kind withReuseIdentifier:(NSString *)identifier;
```

其分别是第一个为纯代码注册方法（需要代码添加class类），第二个为nib注册方法（从nib文件中加载出来）

**下面是用第一个方法初始化：**

```Objective_C
//注册Cell，必须要有
[self.collectionView registerClass:[AppCollectionViewCell class] forCellWithReuseIdentifier:@"AppCollectionViewCell"];
[self.collectionView registerClass:[HeaderCRView class] forSupplementaryViewOfKind:UICollectionElementKindSectionHeader withReuseIdentifier:@"HeaderView"];
[self.collectionView registerClass:[FooterCRView class] forSupplementaryViewOfKind:UICollectionElementKindSectionFooter withReuseIdentifier:@"FooterView"];
```
在`UIcollectionView`上注册三个类，分别是`cell`、`header`、`footer`，那么我们首先就要有这三个类，才能完成注册，不然会崩溃。

```Objective_C
//header类  需要继承自UICollectionReusableView
@interface HeaderCRView : UICollectionReusableView
@end
@implementation HeaderCRView
- (instancetype)initWithFrame:(CGRect)frame{
  self= [super initWithFrame:frame];
  if (self) {
  }
  return self;
}
@end
//footer类  同样需要继承UICollectionReusableView
@interface FooterCRView : UICollectionReusableView
@end
@implementation FooterCRView
- (instancetype)initWithFrame:(CGRect)frame{
  self= [super initWithFrame:frame];
  if (self) {
  }
  return self;
}
@end

//cell类  这个需要继承的是UICollectionViewCell
@interface AppCollectionViewCell : UICollectionViewCell
@end
@implementation AppCollectionViewCell
- (instancetype)initWithFrame:(CGRect)frame{
  self = [super initWithFrame:frame];
  if (self) {
  }
  return self;
}
@end
```
好了，class类都注册好了，接下来是怎么用了。
首先要把cell、header、footer的布局好了

```Objective_C
//定义每个Item 的大小
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath
{
  return CGSizeMake(CELLITEM_SIZE,CELLITEM_SIZE);
}
//定义每个UICollectionView 的 margin
-(UIEdgeInsets)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout insetForSectionAtIndex:(NSInteger)section
{
  return UIEdgeInsetsMake(0.1f, 0.1f, 0.1f, 0.1f);//分别为上、左、下、右
}
```
然后：UIcollectionView的dataSource方法和UItableView的差不多也有两个required

```Objective_C
//定义展示的UICollectionViewCell的个数
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section;
// 返回cell必须用-dequeueReusableCellWithReuseIdentifier:forIndexPath:来获取
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath;
```

**代码如下：**

```Objective_C
//定义展示的UICollectionViewCell的个数
-(NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
  return [arrAllApp count];
}
//每个UICollectionView展示的内容
-(UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
  static NSString * CellIdentifier = @"AppCollectionViewCell";
  AppCollectionViewCell * cell = [collectionView dequeueReusableCellWithReuseIdentifier:CellIdentifier    forIndexPath:indexPath];
  return cell;
}
```
然后是一系列的delegate方法了：

```Objective_C
//UICollectionView被选中时调用的方法
-(void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath;
//返回这个UICollectionView是否可以被选择
-(BOOL)collectionView:(UICollectionView *)collectionView shouldSelectItemAtIndexPath:(NSIndexPath *)indexPath;
//返回头headerView的大小
-(CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout referenceSizeForHeaderInSection:(NSInteger)section{
CGSize size={320,45};
return size;
}
//返回头footerView的大小
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForFooterInSection:(NSInteger)section
{
return CGSizeMake(SCREEN_WIDTH, 50);
}
//每个section中不同的行之间的行间距
- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumLineSpacingForSectionAtIndex:(NSInteger)section
{
return UNIT_PIXEL;
}
每个item之间的间距
- (CGFloat)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout minimumInteritemSpacingForSectionAtIndex:(NSInteger)section
{
    return 100;
}
//取消选择了某个cell
- (void)collectionView:(UICollectionView *)collectionView didDeselectItemAtIndexPath:(NSIndexPath *)indexPath;

```
ok，基本上就是上面这些了，当然还有一些delegate方法是可以用属性代替的，比如

```Objective_C
flowLayout.headerReferenceSize = CGSizeMake(SCREEN_WIDTH, 70);
和
//返回头footerView的大小
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout referenceSizeForFooterInSection:(NSInteger)section
{
  return CGSizeMake(SCREEN_WIDTH, 70);
}
```
其他的都累似，所以就不一一介绍了

**下面讲一下UIcollectionView遇到的坑**

1. 首先cell的大小和最小的列间距minimumInteritemSpacing一定要小于屏幕宽，不然cell会被挤到下一行
2. 由于cell是多个一起加载出来的（基本上是，如果一行有多个的话），所以初始化cell很容易耗时，尽量不要在cell初始化时对图片进行处理（圆角什么的），不然丢帧。（或者丢到线程中去处理）
3. 如果手机内存不足，可能会cell每次都初始化，这样就相当于没有复用，还是性能优化问题
4. cell是没有点击效果的，需要自己添加，所以我的方法是截获touch事件，然后添加动画效果

**代码如下：（在cell中）**

```Objective_C
//捕获cell的Touch事件，添加点击效果
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
  self.backgroundColor = [UIColor colorWithHexString:@"#000000" alpha:0.05];
  weak(self);
  [UIView animateWithDuration:0.5 animations:^{
    weakself.backgroundColor = [UIColor whiteColor];
  }];
  [super touchesBegan:touches withEvent:event];
}
- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
  self.backgroundColor = [UIColor whiteColor];
  [super touchesEnded:touches withEvent:event];
}
```
好吧，大概就这些了。。。

**补充:UIcollectionView的动画效果**

加一个动画效果，就是长按cell的时候，可以通过拖动cell交换位置，仿支付宝界面效果

**代码如下：**
首先给collectionView添加长按手势

```Objective_C
首先给collectionView添加长按手势
UILongPressGestureRecognizer *longpress = [[UILongPressGestureRecognizer alloc]initWithTarget:self action:@selector(moveCell:)];
[self.collectionView addGestureRecognizer:longpress];
NSMutableArray *arrAllApp;
UIImageView *imgView;
imgView = [[UIImageView alloc]init];
```

然后实现方法：

```Objective_C
- (void)moveCell:(UILongPressGestureRecognizer *)gest{
  static NSIndexPath *currentIndexPath = nil;
  AppCollectionViewCell *cell;
  imgView.userInteractionEnabled = YES;
  switch (gest.state) {
    case UIGestureRecognizerStateBegan:{
      NSIndexPath *indexPath = [self.collectionView indexPathForItemAtPoint:[gest locationInView:self.collectionView]];
      currentIndexPath = indexPath;
      cell = (AppCollectionViewCell *)[self.collectionView cellForItemAtIndexPath:indexPath];
      cell.transform = CGAffineTransformMakeScale(1.1, 1.1);
      cell.backgroundColor = [UIColor colorWithHexString:@"#000000" alpha:0.05];
      imgView.frame = CGRectMake(cell.originX, cell.originY, cell.width, cell.height);
      imgView.image = [self makeImageWithView:cell];
      imgView.transform = CGAffineTransformMakeScale(1.1, 1.1);
      NSLog(@"--------began:%p",imgView);
      [self.collectionView addSubview:imgView];
      cell.hidden = YES;

    }
    break;
    case UIGestureRecognizerStateChanged:{
      NSIndexPath *indexPath = [self.collectionView indexPathForItemAtPoint:[gest locationInView:self.collectionView]];
      if (indexPath && ![currentIndexPath isEqual:indexPath]) {
        [arrAllApp exchangeObjectAtIndex:indexPath.row withObjectAtIndex:currentIndexPath.row];
        [self.collectionView moveItemAtIndexPath:currentIndexPath toIndexPath:indexPath];
      }
      currentIndexPath = indexPath;
      [imgView setCenter:[gest locationInView:self.collectionView]];

    }
    break;
    case UIGestureRecognizerStateEnded:{
      NSIndexPath *indexPath = [self.collectionView indexPathForItemAtPoint:[gest locationInView:self.collectionView]];
      if (indexPath && ![currentIndexPath isEqual:indexPath]) {
        [arrAllApp exchangeObjectAtIndex:indexPath.row withObjectAtIndex:currentIndexPath.row];
        [self.collectionView moveItemAtIndexPath:currentIndexPath toIndexPath:indexPath];
      }
      cell = (AppCollectionViewCell *)[self.collectionView cellForItemAtIndexPath:indexPath];
      [UIView animateWithDuration:0.4 animations:^{
        imgView.transform = CGAffineTransformIdentity;
        [imgView removeFromSuperview];
        }completion:^(BOOL finished) {
          cell.backgroundColor = [UIColor whiteColor];
          cell.hidden = NO;
        }];
      }
        break;
    default:
        break;
    }
}
//UIView保存为图片
- (UIImage *)makeImageWithView:(UIView *)view
{
    CGSize s = view.bounds.size;
    UIGraphicsBeginImageContextWithOptions(s, NO, [UIScreen mainScreen].scale);
    [view.layer renderInContext:UIGraphicsGetCurrentContext()];
    UIImage*image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}
```
