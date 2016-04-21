---
layout: post
title: "UISegmentedControl的使用方法"
date: 2014-11-20
comments: false
categories: UI
---

第一次做项目的时候用到了UISegmentedControl这个控件，表示做项目之前从来没见过这东西，更别说会用了，研究了半天才研究差不多，特地拿来和大家分享

UISegmentedControl类似于UIButton,它可以提供多个选择操作,响应事件,但具有很大的局限性,我们更多的是使用自定义的,不过在这里还是介绍下它的基本用法.

一切都在代码中，废话就不多说了，小伙伴们慢慢研究吧

```
NSArray *segmentedArray = [[NSArrayalloc]initWithObjects:@"1",@"2",@"3",@"4",nil]; 
   //初始化UISegmentedControl 
   UISegmentedControl *segmentedControl = [[UISegmentedControlalloc]initWithItems:segmentedArray]; 
   segmentedControl.frame = CGRectMake(20.0,……)
   // 设置默认选择项索引 
   segmentedControl.selectedSegmentIndex = 2;
   segmentedControl.tintColor = [UIColor redColor]; 
   // 有基本四种样式
   segmentedControl.segmentedControlStyle = UISegmentedControlStylePlain;
   segmentedControl.segmentedControlStyle = UISegmentedControlStyleBordered;
   segmentedControl.segmentedControlStyle = UISegmentedControlStyleBar;
   segmentedControl.segmentedControlStyle = UISegmentedControlStyleBezeled;


   // 设置在点击后是否恢复原样	
  segmentedControl.momentary = YES;
  [segmentedControl setTitle:@"two" forSegmentAtIndex:1];//设置指定索引的题目 
  [segmentedControl setImage:[UIImage imageNamed:@"btn_jyy.png"] forSegmentAtIndex:3];//设置指定索引的图片 
   // 在指定索引插入一个选项并设置图片
  [segmentedControl insertSegmentWithImage:[UIImage imageNamed:@"mei.png"] atIndex:2 animated:NO];
   // 在指定索引插入一个选项并设置题目 
  [segmentedControl insertSegmentWithTitle:@"insert" atIndex:3 animated:NO];

   // 移除指定索引的选项 
  [segmentedControl removeSegmentAtIndex:0 animated:NO];
   // 设置指定索引选项的宽度 
  [segmentedControl setWidth:70.0 forSegmentAtIndex:2];
   // 设置选项中图片等的左上角的位置 
  [segmentedControl setContentOffset:CGSizeMake(10.0,10.0) forSegmentAtIndex:4];
   
  //获取指定索引选项的图片imageForSegmentAtIndex： 
  UIImageView *imageForSegmentAtIndex = [[UIImageViewalloc]initWithImage:[segmentedControl imageForSegmentAtIndex:1]]; 
  imageForSegmentAtIndex.frame = CGRectMake(60.0, 120.0, 30.0, 30.0);  ;
   
  //获取指定索引选项的标题titleForSegmentAtIndex 
  UILabel *titleForSegmentAtIndex = [[UILabel alloc]initWithFrame:CGRectMake(100.0, 160.0, 30.0, 30.0)]; 
  titleForSegmentAtIndex.text = [segmentedControl titleForSegmentAtIndex:0]; 
   
  //获取总选项数segmentedControl.numberOfSegments 
  UILabel *numberOfSegments = [[UILabel alloc]initWithFrame:CGRectMake(140.0, 170.0, 30.0, 30.0)]; 
  numberOfSegments.text = [NSString stringWithFormat:@"%d",segmentedControl.numberOfSegments];
 
  //获取指定索引选项的宽度widthForSegmentAtIndex： 
  UILabel *widthForSegmentAtIndex = [[UILabel alloc]initWithFrame:CGRectMake(180.0, 210.0, 70.0, 30.0)]; 
  widthForSegmentAtIndex.text = [NSString stringWithFormat:@"%f",[segmentedControl widthForSegmentAtIndex:2]]; 
  
   // [segmentedControl setEnabled:NO forSegmentAtIndex:4];//设置指定索引选项不可选 
   // BOOL enableFlag = [segmentedControl isEnabledForSegmentAtIndex:4];//判断指定索引选项是否可选 
    [mySegmentedControladdTarget:selfaction:@selector(didClicksegmentedControlAction:)forControlEvents:UIControlEventValueChanged];  

-(void)didClicksegmentedControlAction:(UISegmentedControl *)Seg{
  NSInteger Index = Seg.selectedSegmentIndex;
  NSLog(@"Index %i", Index);
  switch (Index) {
    case 0:
      [self selectmyView1];
      break;
    case 1:
      [self selectmyView2];
      break;
    case 2:
      [self selectmyView3];
      break;
    case 3:
      [self selectmyView4];
      break;
…………………………………….
    default:
      break;
  }
}
```