# UISearchController

UISearchController 是一个非常好用的搜索控件，看中的就是他的模态动画。能够快速实现想要的效果

## UISearchController 的基本属性

```objc
// 输入框内容发生变化的时候做出响应的代理
@property (nullable, nonatomic, weak) id <UISearchResultsUpdating> searchResultsUpdater;

// 待了解
@property (nonatomic, assign, getter = isActive) BOOL active;

// UISearchController 的代理对象
@property (nullable, nonatomic, weak) id <UISearchControllerDelegate> delegate;
// 设置背景灰暗，默认值为 YES
@property (nonatomic, assign) BOOL dimsBackgroundDuringPresentation 
// 设置背景模糊，默认值为 YES
@property (nonatomic, assign) BOOL obscuresBackgroundDuringPresentation 
// 模态的时候隐藏导航栏，默认值为 YES
@property (nonatomic, assign) BOOL hidesNavigationBarDuringPresentation;
// 显示搜索结果的视图控制器
@property (nullable, nonatomic, strong, readonly) UIViewController *searchResultsController;
// UISearchController内置的UISearchBar
@property (nonatomic, strong, readonly) UISearchBar *searchBar;
```

## 使用当前页面展示搜索结果

```
self.searchController = [[UISearchController alloc] initWithSearchResultsController:nil];
```
当初始化方法的resultController传值为nil的时候，表示使用当前页面作为结果展示页面

## 使用SearchResultsController 展示搜索结果

```
 ResultViewController *resultVC = [[ResultViewController alloc] initWithNibName:@"ResultViewController" bundle:nil];

self.searchController = [[UISearchController alloc] initWithSearchResultsController:resultVC];
```

当初始化方法的resultController传值为结果展示页面的视图控制器时，表示搜索结果在指定页面显示。

## 代理方法

### UISearchResultsUpdating

```
- (void)updateSearchResultsForSearchController:(UISearchController *)searchController;
```
SearchBar的输入框内容发生变化的时候会调用此代理方法，在此方法中可以**实时**处理一些对数据的获取过滤操作

### UISearchControllerDelegate

```
// UISearchController 将要模态出来的代理方法
- (void)willPresentSearchController:(UISearchController *)searchController;
// UISearchController 已经模态出来的代理方法
- (void)didPresentSearchController:(UISearchController *)searchController;
// UISearchController 将要隐藏的代理方法
- (void)willDismissSearchController:(UISearchController *)searchController;
// UISearchController 已经隐藏的代理方法
- (void)didDismissSearchController:(UISearchController *)searchController;

// Called after the search controller's search bar has agreed to begin editing or when 'active' is set to YES. If you choose not to present the controller yourself or do not implement this method, a default presentation is performed on your behalf.
- (void)presentSearchController:(UISearchController *)searchController;
```

## 注意点
* 当使用子页面展示搜索结果的时候，一定要注意使用`UISearchController`所在的页面的`UINavigationController` 去`push`出子页面. 可以使用`Block`或者`Delegate`实现将子页面的点击事件转移到父级页面去处理

* UISearchController默认的实现是点击UISearchBar的时候不显示SearchResultsController，如果要实现点击SearchBar立马就显示搜索结果页面可以使用KVO 监听`SearchResultsController.view`的`hidden`属性，保证`hidden`属性的值一直为`NO`

```
- (void)configSearchController {
    
    ResultViewController *resultVC = [[ResultViewController alloc] initWithNibName:@"ResultViewController" bundle:nil];
    
    self.searchController = [[UISearchController alloc] initWithSearchResultsController:resultVC];
    self.searchController.delegate = self;
    self.searchController.searchResultsUpdater = self;
    self.searchController.searchBar.delegate = self;
    self.searchController.dimsBackgroundDuringPresentation = YES;
    self.searchController.obscuresBackgroundDuringPresentation = YES;
    self.searchController.hidesNavigationBarDuringPresentation = YES;
    self.tableView.tableHeaderView = self.searchController.searchBar;
    
    [self.searchController.searchResultsController.view addObserver:self forKeyPath:@"hidden" options:0 context:NULL];
}

- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary *)change
                       context:(void *)context
{
    if ( object == self.searchController.searchResultsController.view &&
        [keyPath isEqualToString:@"hidden"] &&
        self.searchController.searchResultsController.view.hidden &&
        self.searchController.searchBar.isFirstResponder )
    {
        self.searchController.searchResultsController.view.hidden = NO;
    }
}

```

[Demo的地址](https://github.com/yubin-X/UISearchControllerDemo)