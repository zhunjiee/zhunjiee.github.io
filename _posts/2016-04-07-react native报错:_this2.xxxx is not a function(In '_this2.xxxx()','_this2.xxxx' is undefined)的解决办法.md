---
layout: post
title: "react native报错:_this2.xxxx is not a function(In '_this2.xxxx()','_this2.xxxx' is undefined)的解决办法"
date: 2016-04-07
comments: false
categories: React Native
---

### react native报错:_this2.xxxx is not a function(In '_this2.xxxx()','_this2.xxxx' is undefined)的解决办法

直入主题,最近做react native的navigator跳转,点击进行跳转的时候,就会出现如图所示的错误,我是用ES6写的

![](https://dn-zhunjiee.qbox.me/Snip20160921_1.png)

百度了好久也没解决,自己尝试了好久终于解决了,下面说一下如何会出现这个问题?

这是出问题的源码:

```
class HomeView extends Component {
    render() {
        return (
            <ListView
                dataSource={this.state.dataSource}
                renderRow={this.renderRow}
            />
        );
    }

    // 返回一行的数据
    renderRow(rowData) {
        return(
            <TouchableOpacity
                activeOpacity={0.5}
                onPress={()=>this.pushToNewDetail()}
            >
                <View>
					<Text>{rowData.data}</Text>
                </View>
            </TouchableOpacity>
        );
    }

    // 跳转到新闻详情页
    pushToNewDetail(docid) {
        // navigator跳转
        this.props.navigator.push({
            component: NewDetail,
            title: '返回',
            passProps:{docid},
        });
    }
}

```

报这种错误肯定就是this的问题,可能是哪两个this指的不是一个对象的原因.


有人说`onPress={()=>this.pushToNewDetail()}`写成`onPress={()=>this.pushToNewDetail().bind(this)}`

我试了试,还是报一样的错误,我就在考虑是不是写的位置不对,在尝试了很久以后,终于找到了原因

```
render() {
        return (
            <ListView
                dataSource={this.state.dataSource}
                renderRow={this.renderRow}
            />
        );
    }
```

原因在上面的代码中,在重写`renderRow={this.renderRow}`方法的时候,这里的`this`不再是最外层的`this`,应该改成以下的形式,错误即可解决


```
render() {
        return (
            <ListView
                dataSource={this.state.dataSource}
                renderRow={this.renderRow.bind(this)}
            />
        );
    }
```

ES6的写法真的是有太多需要注意的问题啊