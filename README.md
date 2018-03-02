# rangeDatepicker
A secondary developing vue component based upon the DatePicker component of iView.


## Description
If you are developing your web page through iView and seeking a different style of the DatePciker, please check it out.

This datepicker component has a different appearance compared with iView's, which comprises of two DatePicker component of iView. 

## Before use
You should install iView before using this component.
```
$ npm install iview --save
```

## How to use it
Import this component and register it to your vue instance, then you can use it easily.

### >API

**props**

|**property**|**description**|**value type**|**default value**|
|:-|:-|:-|:-|
|dateValue|The date value which is an Array contained two Date type items.|Array|An empty Array|
|dateType|Setting the methods you can use to select time. The optional value is **'date'** & **'datetime'**. <br/>**'date'**: chose date;<br/>**'datetime'**: chose date and time.|String|date|
|size|The size of the component. The optional value is **'default'** & **'large'**.|String|default|
|width|The width of the component.|Number|The width can adjust itself automatically according to the setting of 'dateType' and 'size'.|


**events**

|event|description|return value|
|:-|:-|:-|
|confirm|It would be triggered when the confirm button or the shortcut button is clicked.|An Array contained two Date type items.|


### >Sample Code

```
//supposed the component's name is 'rangeDatepicker'

<range-datepicker @confirm="function" :dateValue="dateArray"></range-datepicker>

<range-datepicker @confirm="function" :dateValue="dateArray" :width="350"></range-datepicker>

<range-datepicker size="large" @confirm="function" :dateValue="dateArray"></range-datepicker>

<range-datepicker dateType="datetime" @confirm="function" :dateValue="dateArray"></range-datepicker>
```

### >Sample Instance
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
