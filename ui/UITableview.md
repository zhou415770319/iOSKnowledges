###1.1.1重用机制(字母索引条)
```objective-c
	//重用
	static NSString *cellID = @"cell";
      //根据identify在缓存的字典中找是否有已经初始化好的cell
      UITableView *cell = [tableView dequeueReusableCellWithIdentifier:cellID];
        if (!cell) {
            cell = [[HORTransitReportCell alloc] initWithStyle:UITableViewCellStyleValue2 reuseIdentifier:cellID];
        }
      return cell;
  
 //2.注册方式
  [self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier: identifier] ;

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 重用队列中取单元格 由于上面已经注册过单元格,系统会帮我们做判断,不用再次手动判断单元格是否存在
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier: identifier forIndexPath:indexPath] ;
    return cell ;
}
```

    //  当页面拉动需要显示新数据的时候，把最后一个cell进行删除 就有可以自定义cell 此方案即可避免重复显示，又重用了cell相对内存管理来说是最好的方案 前两者相对比较消耗内存
    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
    {
        // 定义唯一标识
        static NSString *CellIdentifier = @"Cell";
        // 通过唯一标识创建cell实例
        UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier];
        
        // 判断为空进行初始化  --（当拉动页面显示超过主页面内容的时候就会重用之前的cell，而不会再次初始化）
        if (!cell) {
            cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:CellIdentifier];
        }
        else//当页面拉动的时候 当cell存在并且最后一个存在 把它进行删除就出来一个独特的cell我们在进行数据配置即可避免
        {
            while ([cell.contentView.subviews lastObject] != nil) {
                [(UIView *)[cell.contentView.subviews lastObject] removeFromSuperview];
            }
        }
        // 对cell 进行简单地数据配置
        cell.textLabel.text = @"text";
        cell.detailTextLabel.text = @"text";
        cell.imageView.image = [UIImage imageNamed:@"4.png"];
        
        return cell;
    }
```objective-c
///重用实现分析

　　查看UITableView头文件，会找到NSMutableArray*  visiableCells，和NSMutableDictnery* reusableTableCells两个结构。visiableCells内保存当前显示的cells，reusableTableCells保存可重 用的cells。

　　TableView显示之初，reusableTableCells为空，那么tableView dequeueReusableCellWithIdentifier:CellIdentifier返回nil。开始的cell都是通过 [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier]来创建，而且cellForRowAtIndexPath只是调用最大显示cell数的 次数。

　　比如：有100条数据，iPhone一屏最多显示10个cell。程序最开始显示TableView的情况是：

　　1. 用[[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier]创建10次cell，并给cell指定同样的重用标识(当然，可以为不同显示类型的 cell指定不同的标识)。并且10个cell全部都加入到visiableCells数组，reusableTableCells为空。

　　2. 向下拖动tableView，当cell1完全移出屏幕，并且cell11(它也是alloc出来的，原因同上)完全显示出来的时候。cell11加入到 visiableCells，cell1移出visiableCells，cell1加入到reusableTableCells。

　　3. 接着向下拖动tableView，因为reusableTableCells中已经有值，所以，当需要显示新的 cell，cellForRowAtIndexPath再次被调用的时候，tableView dequeueReusableCellWithIdentifier:CellIdentifier，返回cell1。cell1加入到 visiableCells，cell1移出reusableTableCells；cell2移出visiableCells，cell2加入到 reusableTableCells。之后再需要显示的Cell就可以正常重用了。

　　所以整个过程并不难理解，但需要注意正是因为这样的原因：配置Cell的时候一定要注意，对取出的重用的cell做重新赋值，不要遗留老数据。

一些情况

　　使用过程中，我注意到，并不是只有拖动超出屏幕的时候才会更新reusableTableCells表，还有：

　　1. reloadData，这种情况比较特殊。一般是部分数据发生变化，需要重新刷新cell显示的内容时调用。在 cellForRowAtIndexPath调用中，所有cell都是重用的。我估计reloadData调用后，把visiableCells中所有 cell移入reusableTableCells，visiableCells清空。cellForRowAtIndexPath调用后，再把 reuse的cell从reusableTableCells取出来，放入到visiableCells。

　　2. reloadRowsAtIndex，刷新指定的IndexPath。如果调用时reusableTableCells为空，那么 cellForRowAtIndexPath调用后，是新创建cell，新的cell加入到visiableCells。老的cell移出 visiableCells，加入到reusableTableCells。于是，之后的刷新就有cell做reuse了。
 ///
```