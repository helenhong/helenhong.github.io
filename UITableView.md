#About UITableView
##Initial
```
- (instancetype)initWithFrame:(CGRect)frame style:(UITableViewStyle)style NS_DESIGNATED_INITIALIZER; // must specify style at creation. -initWithFrame: calls this with UITableViewStylePlain

- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder NS_DESIGNATED_INITIALIZER;
 
typedef NS_ENUM(NSInteger, UITableViewStyle) 
{
   	 UITableViewStylePlain,          // regular table view
    UITableViewStyleGrouped         // preferences style table view
};
```
这是UItableView.h文件里介绍的两种初始化方法,其中UITableVIewStyle有两个枚举值。

UITableViewCell 有默认的样式，也可以自定义样式：
默认样式有四种

```

typedef NS_ENUM(NSInteger, UITableViewCellStyle) {
    UITableViewCellStyleDefault,	// Simple cell with text label and optional image view (behavior of UITableViewCell in iPhoneOS 2.x)
    UITableViewCellStyleValue1,		// Left aligned label on left and right aligned label on right with blue text (Used in Settings)
    UITableViewCellStyleValue2,		// Right aligned label on left with blue text and left aligned label on right (Used in Phone/Contacts)
    UITableViewCellStyleSubtitle	// Left aligned label on top and left aligned label on bottom with gray text (Used in iPod).
};

```

如果要使用默认样式，则要使用如下方法初始化，后续只需要利用UITableViewCell的重用即可。

```
UITableViewCell *cell=[tableView dequeueReusableCellWithIdentifier:cellIdentifier];
if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIdentifier];
    }
    
```
如果要使用自定义的UITableViewCell，则必须在使用前把Nib或者是Class与cellIdentifier绑定。如下：

```
- (void)registerNib:(nullable UINib *)nib forCellReuseIdentifier:(NSString *)identifier NS_AVAILABLE_IOS(5_0);
- (void)registerClass:(nullable Class)cellClass forCellReuseIdentifier:(NSString *)identifier NS_AVAILABLE_IOS(6_0);

```

UITableViewCell的重用机制是指UITableView只创建在屏幕范围内可见个数的cell。当cell滑出屏幕之后，会进入到缓冲池当中。当有新的数据需要展示时，系统会先从缓冲池里寻找可以重用的cell。这样做，可以节约系统的内存空间，提高性能。

##Display Information
###dataSource
UITableViewDataSource协议必须实现的两个代理方法是：

```

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;

// Row display. Implementers should *always* try to reuse cells by setting each cell's reuseIdentifier and querying for available reusable cells with dequeueReusableCellWithIdentifier:
// Cell gets various attributes set automatically based on table (separators) and data source (accessory views, editing controls)

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;

```
就如注释里解释的一样，重用cell的时候必须绑定reuseIdentifier。

###delegate
UITableDelegate同时页遵循UIScrollViewDelegate协议。

在初始化UITableView的时候一定要注意给delegate和datasource赋值对象，delegate和datasource可以是tableview所属的控制器来实现，也可以定义其他Class来实现。

###header and footer

每一个section都有一个header和一个footer，也有默认样式和可实现自定义样式。

###section
##Interaction
##edit
##select
