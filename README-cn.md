# react-native-largelist

For English docs [click here](./README.md)

**react-native-largelist** 是一个为React Native准备的高性能的列表组件，它比官方的SectionList给人以更好的性能体验及更完善的功能（兼容iOS和Android）.

## 特点
* react-native-largelist 比官网的SectionList性能表现更好,在最坏的情况下（比如从第一行直接用代码滑动到第1000行），即使出现白板，也是瞬间消失。
* 支持超大数据源，支持无限列表，支持超快速度滑动。
* 跨平台，兼容iOS和Android。
* 支持分组，支持每组头视图自动吸顶，新的Section挂在列表顶部时，支持回调。
* 行组件进入或离开安全区域时可配置回调事件。
* 支持单独的头部和尾部组件。
* 支持下拉刷新和上拉加载更多。
* 支持上拉加载视图自定义配置，上拉加载完成自定义视图配置。
* 支持获取当前的动态属性，比如视图大小、当前偏移、当前Section、当前滑动视图总大小、头部或尾部组件高度、当前可视行等。
* 支持滑动到指定位置或指定行。
* 支持数据更新。
* 支持自定义优化属性，可根据实际情况修改优化参数提升性能。
* 如需要未提供的其他属性或回调事件可以通过提交issue提醒作者添加。

## 预览
![Preview](./readme_resources/sample1.gif)
![Preview](./readme_resources/sample2.gif)
![Preview](./readme_resources/sample3.gif)
![Preview](./readme_resources/sample4.gif)
![Preview](./readme_resources/sample5.gif)
![Preview](./readme_resources/sample6.gif)
![Preview](./readme_resources/sample7.gif)
![Preview](./readme_resources/sample8.gif)

## 性能展示
查看LargeList的性能表现：[优酷](http://v.youku.com/v_show/id_XMzI0ODc4ODkyOA==.html) 或者 [youtube](https://youtu.be/k95G3_QGYHE)

## 接入步骤

* 确认您的项目是React Native项目

* 使用下面的命令安装:

```
npm install react-native-largelist --save
```

* 像下面这样使用它:

```
import { LargeList } from "react-native-largelist";

//other code
...
<LargeList
  style={{ flex: 1 }}
  bounces={true}
  refreshing={this.state.refreshing}
  onRefresh={() => {
    this.setState({ refreshing: true });
    setTimeout(() => this.setState({ refreshing: false }), 2000);
  }}
  safeMargin={600}
  numberOfRowsInSection={section => this.props.numberOfEachSection}
  numberOfSections={()=>this.props.numberOfSections}
  heightForCell={(section, row) => row % 2 ? this.minCellHeight : this.maxCellHeight}
  renderCell={this.renderItem.bind(this)}
  heightForSection={section =>section % 2 ? this.minSectionHeight : this.maxSectionHeight}
  renderHeader={this.renderHeader.bind(this)}
  renderFooter={this.renderFooter.bind(this)}
  renderSection={section => {
    return (
      <View
        style={{
          flex: 1,
          backgroundColor: section % 2 ? "grey" : "yellow",
          justifyContent: "center",
          alignItems: "center"
        }}
      >
        <Text>
           I am section {section}
        </Text>
      </View>
    );
  }}
/>
...
```

## 基本用法

```
import { LargeList } from "react-native-largelist"
```

属性:

属性  |  类型  |  默认  |  作用  
------ | ------ | --------- | --------
（ViewPropTypes） | （ViewPropTypes） |  | View的所有属性
numberOfSections | ()=>number | ()=>1 | 总的Section数量
numberOfRowsInSection | (section:number) => number | section=>0 | 函数：根据section索引返回当前section的Cell数量
renderCell | (section:number,row:number) => React.Element | 必须 | 函数:根据当前Section和Row，返回当前Cell的render
heightForCell | (section:number,row:number) => number | 必须 | 函数：根据Section和row index，返回当前Cell的高度
renderSection | (section:number) => React.Element | section=>null | 函数：当前Section的render函数
heightForSection | (section:number) => number | ()=>0 | 函数：返回当前section的高度
renderHeader | () => React.Element | ()=>null | 函数：列表的头部组件的render函数
renderFooter | () => React.Element | ()=>null | 函数：列表的尾部组件的render函数
bounces | boolean | true | 组件滑动到边缘是否可以继续滑动，松开后弹回
refreshing | boolean | undefined | 是否正在刷新
onRefresh | () => any | undefined | 下拉刷新的回调,如果用户设置了此属性，则添加一个刷新控件
onScroll | ({nativeEvent:{contentOffset:{x:number,y:number}}})=> any |  | 滑动的回调，同官方ScrollView

## 原理
在了解高级用法之前，我们先要了解下基本原理：

和UITableView/RecyclerView基本原理一样，每一行的Cell/Item是重用的，当上面的Cell/Item滑离屏幕的时候，它对于我们来说已经不用再展示了，所以，把它挪到底部，用新的数据渲染。这样就不会出现由于数据量过大，造成View的浪费导致加载卡顿。

但是，这又和原生的UITableView/RecyclerView不一样，因为UI线程和我们的JavaScript线程不是同一个线程，并且他们是异步的，他们之间通讯还会花费较多的时间。所以，我在可视区域之外的上下各自额外渲染了一段距离（safeMargin），用以缓冲，避免用户突然的滑动造成视觉上看到上下边缘的加载是闪烁的。

下面的图可以直观地看到react-native-largelist的设计：

![](./readme_resources/largelist_advanced_usage.png)

当然，在之后的版本中，我会把iOS原生优化加入其中的选项中，这样，不论你滑动的速度有多快，用户都不会看到白板的情况了。

## 高级属性

### safeMargin
* type: number
* default: 600
* 上下额外的预渲染高度，值越大在快速滑动过程中越不容易看到空白，但是第一次加载的时间越长

### dynamicMargin
* type: number
* default: 500
* 在快速滑动过程中，允许上下端相互借用的预渲染高度。比如假设safeMargin=600，dynamicMargin=500，在快速下滑过程中，上端只额外渲染100高度，下端额外渲染1100高度，反之亦然。 在慢速滑动过程中该属性无作用。请注意：该属性不能大于safeMargin

### scrollEventThrottle
* type: number
* default: ios:16
* 同ScrollView的scrollEventThrottle，滑动过程中，onScroll回调时间最小间隔，值越小，相同的滑动追踪次数越多，越不容易出现空白，性能越差。

### onIndexPathDidEnterSafeArea
* type: (indexPath:IndexPath)=>any
* default: ()=>null
* 当一个indexPath进入SafeArea时回调

### onIndexPathDidLeaveSafeArea
* type: (indexPath:IndexPath)=>any
* default: ()=>null
* 当一个indexPath离开SafeArea时回调

### showsVerticalScrollIndicator
* type: bool
* default: true
* 显示垂直滚动指示器

### onSectionDidHangOnTop
* type: section=>any
* default: ()=>null
* 当一个新的Section被挂在LargeList顶部时的回调

### speedLevel1
* type: number
* default: 4
* 当滑动的速度超过这个speedLevel1，LargeList则不会rerender，仅仅只是移动位置。  单位是  每毫秒移动的逻辑像素量。

### speedLevel2
* type: number
* default: 10
* 当前版本无意义

### nativeOptimize
* type: bool
* default: false
* 启用原生优化，iOS专用。这是一个实验性的功能，使用该功能以后，safeArea就没有意义了。要使用改功能，请将"${YourProject}/node_modules/react-native-largelist/ios/STTVTableView.xcodeproj"拖入您的iOS项目，并且保证链接了它

### onLoadMore
* type: ()=>any
* default: null
* 上拉加载更多的回调，如果设置了此属性，则在下面会有一个上拉的控件

### heightForLoadMore
* type: ()=>number
* default: ()=>70
* 上拉控件的高度

### allLoadCompleted
* type: bool
* default: false
* 数据源是否全部加载完成

### renderLoadingMore
* type: ()=>React.Element
* default: ()=> < ActivityIndicator style={{ marginTop: 10, alignSelf: "center" }} size={"large"}/ >
* 用户自定义上拉加载更多的动画视图，目前暂不支持事件驱动动画

### renderLoadCompleted
* type: ()=>React.Element
* default: ()=> < Text style={{ marginTop: 20, alignSelf: "center", fontSize: 16 }}>No more data< /Text >
* 用户自定义上拉时，加载完成的动画视图。

## 方法
### scrollTo(offset:Offset, animated:boolean=true)
滑动到目标偏移Offset:{x:number,y:number},目前x值只支持0
### scrollToIndexPath(indexPath:IndexPath, animated:boolean = true)
滑动到一个指定的IndexPath:{section:number,row:number}
### scrollToEnd(animated:boolean=true)
滑动到最底部
### visibleIndexPaths():IndexPath[]
获取当前可见的所有IndexPaths
### renderedIndexPaths():IndexPath[]
获取当前已经渲染好的所有IndexPaths
### freeCount(): number
获取当前空闲的Cell数量
### reloadIndexPath(indexPath: IndexPath)
局部重新加载指定indexPath.
### reloadIndexPaths(indexPaths: IndexPath[])
局部重新加载一些指定的indexPaths
### reloadAll()
局部重新加载所有indexPaths

注意:

1. reloadIndexPath, reloadIndexPaths, reloadAll 都是局部重新加载，因此影响列表的全局属性，比如numberOfSections ,numberOfRowsInSection,heightForSection,heightForCell属性改变，则会导致错乱，请使用reloadData。

### reloadData()
全局重新加载所有数据

注意：

1. 如果影响列表的全局属性，比如numberOfSections ,numberOfRowsInSection,heightForSection,heightForCell属性改变，必须使用此方法更新
2. 不要过度使用它，因为它的效率是很低的。

## 动态变量
### size:Size
获取LargeList的当前可视Size：{width:number,height:number}
### contentOffset:Offset
获取LargeList当前偏移Offset：{x:number,y:number}
### safeArea: Range
获取LargeList当前的已经渲染好的区域Range:{top:number,bottom:number}
### topIndexPath: IndexPath
获取LargeList当前SafeArea顶部的IndexPath:{section:number,row:number}
### bottomIndexPath: IndexPath
获取LargeList当前SafeArea底部的IndexPath:{section:number,row:number}
### contentSize:Size
获取LargeList当前总大小Size:{width:number, height:number}
### currentSection:number
获取放置在顶部的Section
### headerHeight:number
获取LargeList当前的Header高度
### footerHeight:number
获取LargeList当前的Footer高度

## 目标计划
1. 修正细节问题
2. 提供左右滑动编辑功能
3. 代码优化，支持TypeScript类型检查


## 更新日志

### 版本 1.1.0
* 添加上拉加载更多
* 修复reloadData有时候出现问题
* 修复scrollTo几个方法，如果动画为false，会出现bug的问题
* 将 "visiableIndexPaths" 修改为 "visibleIndexPaths", "visiableIndexPaths" 将在2.0.0版本后完全不支持
* "numberOfSections"的类型由number改变为function, number 将在2.0.0版本后完全不支持

### Version 1.0.0
* release


