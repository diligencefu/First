# First

两年前写的一个古诗软件里面的选择器，闲着没什么事，就给传上去玩玩。

Controller类里面没什么东西，就是根据view选择的内容，加载不同的数据，然后就是改变需要显示列表的View的frame的变化。
    _headView=[[SXSearchHeadView alloc]initWithFrame:CGRectMake(0, 0,ScreenWidth, 40*almightyHeight) andDataSource:Arr];
    __weak SXMainViewController *vc=self;
    
    //接受点击button时的下标
    _headView.titleBlock=^(NSInteger index){
        [vc showOrHiddenWhenTheValueChanged:index];
    };
    
    //接收点击cell时的消息
    //当headView的cell被点击时，收回tableView
    __weak SXSearchHeadView *view=_headView;
    
    _headView.selectBlock=^(NSString *str){
        [UIView animateWithDuration:.6 animations:^{
            float height=40*almightyHeight;
            
            CGRect rect=view.frame;
            
            rect.size.height=height;
            
            view.frame=rect;
            
        }];
        
        [vc showInfoWithDynasty:view.dynastyButton.currentTitle andPeomType:view.dataButton.currentTitle andForm:view.formButton.currentTitle];
    };
    
    [self.view addSubview:_headView];
}

//选择完成之后调用
- (void)showInfoWithDynasty:(NSString *)dynasty andPeomType:(NSString *)type andForm:(NSString *)form{

    self.navigationItem.title = [NSString stringWithFormat:@"%@%@的%@",dynasty,type,form];
    
}


主要的东西还是在view类里面。
//开始选择
-(void)beginToSearch:(UIButton *)sender{
    
//    因为有的三个字有的两个字，要调整一下内间距，有一个根据字体大小获得文本长度的方法，有兴趣的可以找一下，我比较懒
    
    [self changeFrameOfButton:sender];
    _currentTag=sender.tag;
    
    //如果两次点击是同一次button
    if (_selectIndex==sender.tag) {
        _selectIndex=0;
        
        //并且这个tableView‘是打开状态的
        if (_isForming||_isDybasting||_isDataing) {
            //封装方法，让其关闭
            [self restorationForButtonImageView];
            
        }else{
            //如果不是开启状态，打开tableView并显示你所点击的button对应的内容
            [self change:sender.tag];
        }
        
    }else{
        //如果两次点击的不是同一个button，打开tableView并显示你所点击的button对应的内容
        [self change:sender.tag];
        
    }
    //最后，将你所点击的button的下标block传给控制器
    if (_titleBlock) {
        _titleBlock(self.selectIndex);
    }
    //点击之后刷新tableView 以刷新button对应的内容
    [self.searchTableView reloadData];
    
    
//    [self.searchTableView selectRowAtIndexPath:[NSIndexPath indexPathForRow:0 inSection:0] animated:NO scrollPosition:UITableViewScrollPositionTop];
}

//如果两次点击的不是同一个Button或者点击的是同一个button‘ 但是两次点击的时候tableView都处于关闭状态的时候------调用此方法 。
//有点难讲，这个状态比较少触发，可以打个断点多玩玩
-(void)change:(NSInteger)num{
    
    //既然是要打开，点击的button的下标赋给_selectIndex 以便后面block传给控制器
    _selectIndex=num;
    
    //根据点击不同的button，将其设置为选中状态，打开状态开启，图片旋转，其他的都取消掉
    if (num==100) {
        
        [UIView animateWithDuration:.25 animations:^{
            
            _dataButton.selected=YES;
            _dynastyButton.selected=NO;
            _formButton.selected=NO;
            
//            点击到的按钮图片旋转，其他的都返回原样
            _dataButton.imageView.transform=CGAffineTransformMakeRotation(M_PI);
            _dynastyButton.imageView.transform=CGAffineTransformIdentity;
            _formButton.imageView.transform=CGAffineTransformIdentity;
            
            //设置button的image
            [_dataButton setImage:[UIImage imageNamed:@"upblack"] forState:(UIControlStateNormal)];
            [_dynastyButton setImage:[UIImage imageNamed:@"upwhite"] forState:(UIControlStateNormal)];
            [_formButton setImage:[UIImage imageNamed:@"upwhite"] forState:(UIControlStateNormal)];
            
        }];
        
        _isDataing=YES;
        _isForming=NO;
        _isDybasting=NO;
        
    }else if (_currentTag==101){
        
        [UIView animateWithDuration:.25 animations:^{
            _dynastyButton.selected=YES;
            _dataButton.selected=NO;
            _formButton.selected=NO;
            
            _dataButton.imageView.transform=CGAffineTransformIdentity;
            _dynastyButton.imageView.transform=CGAffineTransformMakeRotation(M_PI);
            _formButton.imageView.transform=CGAffineTransformIdentity;
            
            //设置button的image
            [_dataButton setImage:[UIImage imageNamed:@"upwhite"] forState:(UIControlStateNormal)];
            [_dynastyButton setImage:[UIImage imageNamed:@"upblack"] forState:(UIControlStateNormal)];
            [_formButton setImage:[UIImage imageNamed:@"upwhite"] forState:(UIControlStateNormal)];
            
        }];
        
        _isDataing=NO;
        _isForming=NO;
        _isDybasting=YES;
        
    }else if (_currentTag==102){
        
        [UIView animateWithDuration:.25 animations:^{
            
            _dynastyButton.imageView.transform=CGAffineTransformIdentity;
            _formButton.imageView.transform=CGAffineTransformMakeRotation(M_PI);
            
            _dataButton.imageView.transform=CGAffineTransformIdentity;
            
            _formButton.selected=YES;
            _dynastyButton.selected=NO;
            _dataButton.selected=NO;
        
            //设置button的image
            [_dataButton setImage:[UIImage imageNamed:@"upwhite"] forState:(UIControlStateNormal)];
            [_dynastyButton setImage:[UIImage imageNamed:@"upwhite"] forState:(UIControlStateNormal)];
            [_formButton setImage:[UIImage imageNamed:@"upblack"] forState:(UIControlStateNormal)];
            
        }];
        
        _isDataing=NO;
        _isForming=YES;
        _isDybasting=NO;
        
    }
}


- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    //1.根据分区获取该分区对应的数组
    
    if (_currentTag==100) {
        return self.dataArr.count;
    }else if (_currentTag==101){
        return self.dynastyArr.count;
    }
    return self.formArr.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    
    SXShowConditionCell *cell=[tableView dequeueReusableCellWithIdentifier:identify forIndexPath:indexPath];
    
//    看看是否需要显示后面的选中标记
    BOOL isSame = NO;
    
    //根据点击不同的button将不同的数据array中的string置为其标题
    NSString *name;
    
    
    if (_currentTag==100) {
        
//        在赋值的过程中，因为选择过的内容会显示在顶部的按钮上，如果数组里面的字符和按钮相同，就证明那个是上次选择的对象，加个标记
        if ([self.dataArr[indexPath.row] isEqualToString:_dataButton.titleLabel.text]) {
            isSame = YES;
        }
        
        name = self.dataArr[indexPath.row];
    }else if (_currentTag==101){
        if ([self.dynastyArr[indexPath.row] isEqualToString:_dynastyButton.titleLabel.text]) {
            isSame = YES;
        }
        name = self.dynastyArr[indexPath.row];
    }else if (_currentTag==102){
        
        if ([self.formArr[indexPath.row] isEqualToString:_formButton.titleLabel.text]) {
            isSame = YES;
        }
        name = self.formArr[indexPath.row];
    }
    
    cell.backgroundColor = [UIColor clearColor];
    
//    isSame取反的原因可以去cell类里面看一眼
    [cell setName:name andIsSelected:!isSame];

    return cell;
}

-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
    
    //与点击button原理一样
    if (_currentTag==100) {
        if ([_dataButton.titleLabel.text isEqualToString:self.dataArr[indexPath.row]]) {
            
        }else{
            
            [_dataButton setTitle:self.dataArr[indexPath.row] forState:UIControlStateNormal];
            
        }
    }else if (_currentTag==101){
        
        [_dynastyButton setTitle:self.dynastyArr[indexPath.row] forState:UIControlStateNormal];
    [self changeFrameOfButton:_dynastyButton];
        
    }else if (_currentTag==102){
        
        [_formButton setTitle:self.formArr[indexPath.row] forState:UIControlStateNormal];
        [self changeFrameOfButton:_formButton];
    }
    
//    选择完毕，恢复状态
    [self restorationForButtonImageView];
    
    //每当点击cell 说明类型变换了，告诉控制器，需要重新根据标题更新新数据了。
    if (_selectBlock) {
        _selectBlock(@"点击了");
    }
    
    [self.searchTableView reloadData];
}

//    当点击了任何一个cell，就说明选择过了，除了按钮的标题要变，其他的都恢复初始状态
-(void)restorationForButtonImageView{
    
    [UIView animateWithDuration:.25 animations:^{
        _dataButton.imageView.transform=CGAffineTransformIdentity;
        _dynastyButton.imageView.transform=CGAffineTransformIdentity;
        _formButton.imageView.transform=CGAffineTransformIdentity;
        
        _isDataing=NO;
        _isForming=NO;
        _isDybasting=NO;
        
        _dataButton.selected=NO;
        _dynastyButton.selected=NO;
        _formButton.selected=NO;
        
        [_dataButton setImage:[UIImage imageNamed:@"upwhite"] forState:(UIControlStateNormal)];
        [_dynastyButton setImage:[UIImage imageNamed:@"upwhite"] forState:(UIControlStateNormal)];
        [_formButton setImage:[UIImage imageNamed:@"upwhite"] forState:(UIControlStateNormal)];
        
    }];
}

