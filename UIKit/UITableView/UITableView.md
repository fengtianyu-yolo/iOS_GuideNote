# UITableView

## 属性和方法

### `contentOffSet`属性

这个指的时内容起始位置的偏移量，就是第一个cell和当前tableivew顶点的距离

### `contentInset`属性

inset的意思就是留白的区域，也就是默认的间距。和html中的padding一样

`contentInset`就是tableView中的内容区域和tableview控件的间距。它的值是UIEdgeInsets类型，表示可以和四个边设置距离。

比如设置`contentInset = UIEdgeInsetsMake(50,0,0,0)`，就表示第一个cell的位置是在tableview顶部下面的50。这50的部分就是空着的。

## 常见问题

### 为什么tableView不从视图的(0,0)点开始显示

[UIScrollView的`UIScrollViewContentInsetAdjustmentBehavior`属性导致](/adjustinset.md)

### `[cell addSubview]` 和 `[cell.contentView addsubview]`的区别

在默认状态下显示的时候没什么区别

在tableview编辑状态下的时候，`[cell addSubview]`的子视图不会跟着进行左滑。

