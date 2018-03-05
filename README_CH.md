[English Version Document](https://github.com/trevorHsu/rangeDatepicker/blob/master/README.md)

# rangeDatepicker
基于iView的DatePicker组件进行的二次开发。

## Description
如果你正在使用iView作为你的UI库，并且希望使用一种不同样式的DatePicker组件，请试试这个组件。

这个时间选择组件的样式不同于iView的样式，它实际上是由两个iView的DatePicker组件组成的。

## 使用之前
使用该组件之前，你需要先安装iView。
```
$ npm install iview --save
```

## 如何使用
导入该组件，并将其注册到你的vue实例中，接着就可以轻松使用它了。

### >API

**属性**

|**属性**|**说明**|**类型**|**默认值**|
|:-|:-|:-|:-|
|dateValue|时间值，是一个数组，其中包含两个Date类型的值。|Array|空数组|
|dateType|配置选择时间的方式。可选值为 **“date”** 和 **“datetime”**。 <br/>**“date”**: 选择日期;<br/>**“datetime”**: 选择日期与时间。|String|date|
|size|组件的尺寸。 可选值为 **“default”** 和 **“large”**。|String|default|
|width|组件的宽度。|Number|组件宽度可以根据“dateType”和“size”的设置自动调整。|


**事件**

|**事件**|**说明**|**返回值**|
|:-|:-|:-|
|confirm|当点击确认按钮或者左侧快捷方式的按钮时，事件将被触发。|一个数组，包含两个Date类型的值。|


### >示例代码

```
//假设组件名叫做“rangeDatepicker”

<range-datepicker @confirm="function" :dateValue="dateArray"></range-datepicker>

<range-datepicker @confirm="function" :dateValue="dateArray" :width="350"></range-datepicker>

<range-datepicker size="large" @confirm="function" :dateValue="dateArray"></range-datepicker>

<range-datepicker dateType="datetime" @confirm="function" :dateValue="dateArray"></range-datepicker>
```

### >样例
**date**

![image][date-pic]
<br/>
<br/>

**datetime**

![image][datetime-pic-1]
![image][datetime-pic-2]



[date-pic]: https://raw.githubusercontent.com/trevorHsu/rangeDatepicker/master/sample%20instance%20picture/d1.png

[datetime-pic-1]: https://raw.githubusercontent.com/trevorHsu/rangeDatepicker/master/sample%20instance%20picture/d2.png

[datetime-pic-2]: https://raw.githubusercontent.com/trevorHsu/rangeDatepicker/master/sample%20instance%20picture/d3.png
