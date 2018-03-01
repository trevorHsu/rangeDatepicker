<template>
    <div :class="size_inner == 'large' ? 'datepicker-range large-datepicker-range' : 'datepicker-range'" :style="{width: width+'px'}">
        <!--时间显示栏 begin-->
        <div class="date-display" @click="selectPartShowHandler('show')" @mouseover="inputBarMouseoverHandler" @mouseout="inputBarMouseoutHandler">
            <Input ref="dateDisplayBar" :size="size_inner" placeholder="请选择时间" :icon="dateDisplayBarIcon" readonly :value="dateText" @on-click.stop="clearDateDisplay" @on-blur="dateDisplayBarBlur"></Input>
        </div>
        <!--时间显示栏 end-->
        <!--时间选择面板 begin-->
        <transition name="vertical-toggle">
        <div v-show="selectPartShow" class="date-select" @mousedown="dateDisplayBarFocus">
            <div class="date-control clearfix">  
                <div class="date-shortcut dateselect-part">
                    <div class="title">时间范围</div>
                    <div class="content clearfix" @click="dateShortcutClickHandler">
                        <p alias="yesterday">昨日</p>
                        <p alias='today'>今日</p>
                        <p alias='lastWeek'>上周</p>
                        <p alias='thisWeek'>本周</p>
                        <p alias='lastMonth'>上月</p>
                        <p alias='thisMonth'>本月</p>
                        <p alias='lastYear'>去年</p>
                        <p alias='thisYear'>本年</p>
                        <p alias='last7Days'>过去7天</p>
                        <p alias='last30Days'>过去30天</p>
                        <p alias='onlineUntilNow'>上线之今</p>
                    </div>
                </div>
                <div class="date-datepicker dateselect-part">
                    <div class="title">起始时间</div>
                    <div class="content">
                        <DatePicker ref="sDatepicker" :type="dateType_inner" v-model="value[0]" :open="true" :options="sOptions" class="datepicker-style">
                            <a href="javascript:void(0)">
                                <template></template>
                            </a>
                        </DatePicker>
                    </div>
                </div>
                <div class="date-datepicker dateselect-part">
                    <div class="title">终止时间</div>
                    <div class="content">
                        <DatePicker ref="eDatepicker" :type="dateType_inner" v-model="value[1]" :open="true" :options="eOptions" class="datepicker-style">
                            <a href="javascript:void(0)">
                                <template></template>
                            </a>
                        </DatePicker>
                    </div>
                </div>
            </div>
            <div :class="dateBtnClass">
                <div class="timepicker-btn" v-if="showTimePickerBtn" ref="timepickerBtn" @click.stop="togglePickerPanel">
                    <span>{{!this.isTime ? '选择时间' : '选择日期'}}</span>
                </div>
                <ButtonGroup>
                    <Button type="default" size="small" @click="timeClearHandler">清空</Button>
                    <Button type="primary" size="small" @click="timeConfirmHanlder">确认</Button>
                </ButtonGroup>
            </div>
        </div>
        </transition>
        <!--时间选择面板 end-->
    </div>
</template>


<script>
/*
    event: 
        confirm(确认):        返回时间数组 时间的值为Date类型

    props: 
        dateValue(时间值):    传两个Date类型 或者 空数组    eg: [new Date(), new Date()]
        dateType(时间类型):   传入字符串，默认为date   date 日期   datetime 日期及时间
        width(宽度):          传入数字  默认自动调整 date为240  datetime为300
        size(尺寸类型):       传入字符串  默认default  （default，large）

    eg(假设引入的组件命名为rangeDatepicker): 
        <range-datepicker @confirm="function" :dateValue="dateArray"></range-datepicker>
        <range-datepicker @confirm="function" :dateValue="dateArray" :width="350"></range-datepicker>
        <range-datepicker size="large" @confirm="function" :dateValue="dateArray"></range-datepicker>
        <range-datepicker dateType="datetime" @confirm="function" :dateValue="dateArray"></range-datepicker>

    注意：当利用左侧时间范围中的时间快捷方式来选择时间时，在该组件返回的时间数组中，起始时间为00:00:00，终止时间为23:59:59或当前时间
*/
import {startservicetime} from '@/vendors/getstarttime.js';

export default {
    data(){
        return {
            dateText: '',
            dateType_inner: 'date',
            size_inner: 'default',
            value: [],
            selectPartShow: false,
            dateDisplayBarIcon: 'ios-calendar-outline',
            sOptions: {
                disabledDate: null,
            },
            eOptions: {
                disabledDate: null
            },
            enableClearRangeClass_1: true,              //允许执行删除range类的操作
            enableClearRangeClassSwitchTimer_1: null,
            enableClearRangeClass_2: true,              //允许执行删除range类的操作
            enableClearRangeClassSwitchTimer_2: null,
            sDatepickerCellsDOM: null,
            eDatepickerCellsDOM: null,
            showTimePickerBtn: false,
            isTime: false,
            enableDateDisplayBarBlur: true,             //判断是否使DateDisplayBar失去焦点
            enableDateDisplayBarBlurTimer: null,
        };
    },
    created(){
        this.initDatePicker();
    },
    mounted(){
        this.dateRangeControl();
        setTimeout(()=>{
            this.datepickerAppearanceHandler();
        }, 0);
    },
    computed: {
        dateBtnClass: function(){
            return this.showTimePickerBtn ? 'date-btn-datetime' : 'date-btn-date';
        }
    },
    methods: {
        initDatePicker: function(){
            this.importUserInput();
            this.setDatepcikerConfig();
        },
        importUserInput(){
            //判断dateType传值
            let typeArr = ['date', 'datetime'];
            if(typeArr.indexOf(this.dateType) === -1){
                this.dateType_inner = 'date';
                console.error('Component: rangeDatepicker --- incorrect value for the component prop "dateType"');
            } else{
                this.dateType_inner = this.dateType;
            }
            
            //判断size传值
            let sizeArr = ['default', 'large'];
            if(sizeArr.indexOf(this.size) === -1){
                this.size_inner = 'default';
                console.error('Component: rangeDatepicker --- incorrect value for the component prop "size"');                
            } else{
                this.size_inner = this.size;
            }

            //判断时间传值是否合法
            let dateValue = this.dateValue;   //用户传入值
            let value = this.value;           //组件内部值
            if(this.dateValue instanceof Array){
                let res = [];
                for(let i=0; i<2; i++){
                    if(!dateValue[i] || !(dateValue[i] instanceof Date)){
                        res[i] = undefined;
                    } else{
                        res[i] = dateValue[i];
                    }
                    this.value = res;
                }
            } else{
                console.error('Component: rangeDatepicker --- type of the property "dateValue" should be "Array"');
                this.value = [];
            }
        },
        setDatepcikerConfig: function(){
            switch(this.dateType_inner){
                case 'date':
                default:
                    this.showTimePickerBtn = false;
                    break;
                case 'datetime':
                    this.showTimePickerBtn = true;
                    break;
            }
        },
        inputBarMouseoverHandler: function(){
            if(this.dateText !== ''){
                this.dateDisplayBarIcon = 'ios-close';
            }
        },
        inputBarMouseoutHandler: function(){
            this.dateDisplayBarIcon = 'ios-calendar-outline';  
        },
        dateDisplayBarFocus: function(){
            this.enableDateDisplayBarBlur = false;
            let inputbar = this.$refs.dateDisplayBar.$el.querySelector('.ivu-input');
            setTimeout(()=>{
                inputbar.focus();
                inputbar = null;
            },0);
            clearTimeout(this.enableDateDisplayBarBlurTimer);
            this.enableDateDisplayBarBlurTimer = setTimeout(()=>{
                this.enableDateDisplayBarBlur = true;
            }, 1);
        },
        dateDisplayBarBlur: function(){
            if(this.enableDateDisplayBarBlur){
                this.selectPartShowHandler('hide');
            }
        },
        clearDateDisplay: function(e){
            if(this.dateDisplayBarIcon === 'ios-close'){
                this.timeClearHandler();
                if(this.selectPartShow) this.selectPartShowHandler('hide');
            }
        },
        datepickerAppearanceHandler: function(){
            if(this.dateType_inner === 'date') return;
            let refs = this.$refs,
                sPickerPanelBody = refs.sDatepicker.$el.querySelector('.ivu-picker-panel-body'),
                sDatepickerConfirm = sPickerPanelBody.querySelector('.ivu-picker-confirm'),
                ePickerPanelBody = refs.eDatepicker.$el.querySelector('.ivu-picker-panel-body'),
                eDatepickerConfirm = ePickerPanelBody.querySelector('.ivu-picker-confirm');
            refs = null;

            sPickerPanelBody.removeChild(sDatepickerConfirm);
            ePickerPanelBody.removeChild(eDatepickerConfirm);

            sPickerPanelBody = null;
            sDatepickerConfirm = null;
            ePickerPanelBody = null;
            eDatepickerConfirm = null;
        },
        dateShortcutClickHandler: function(e){
            if(!e.target.attributes.alias) return;
            let shortcutType = e.target.attributes.alias.value;
            this.dateShortcutHandler(shortcutType);
        },
        dateShortcutHandler: function(type){
            let dateValue = this.value,
                today = new Date((new Date).getFullYear(), (new Date).getMonth(), (new Date).getDate()),
                oneDay = 24*60*60*1000,
                thatDayTime = oneDay - 1,        //一天持续的时间
                sDate = new Date(today),
                eDate = new Date(),
                sYear = null,
                eYear = null,
                dayNumber = null;

            switch(type){
                case 'yesterday':
                    sDate = new Date(today - oneDay);
                    eDate = new Date(sDate.valueOf() + thatDayTime);
                    break;
                case 'today':
                    break;
                case 'lastWeek':
                    dayNumber = today.getDay() !== 0 ? today.getDay() : 7;
                    sDate = new Date(today- oneDay * (dayNumber + 6));
                    eDate = new Date(today- oneDay * dayNumber + thatDayTime);
                    break;
                case 'thisWeek':
                    dayNumber = today.getDay() !== 0 ? today.getDay() : 7;
                    sDate = new Date(today - oneDay * (dayNumber - 1));
                    break;
                case 'lastMonth':
                    eDate = new Date(today - oneDay * today.getDate());
                    sDate = new Date(eDate - oneDay * (eDate.getDate() - 1));
                    eDate = new Date(today - oneDay * today.getDate() + thatDayTime);
                    break;
                case 'thisMonth':
                    sDate = new Date(today - oneDay * (today.getDate()-1));
                    break;
                case 'lastYear':
                    sYear = new Date(today);
                    sYear.setFullYear(today.getFullYear() - 1);
                    sYear.setMonth(0);
                    sYear.setDate(1);
                    eYear = new Date(sYear);
                    eYear.setFullYear(eYear.getFullYear() + 1 );
                    eYear.setTime(eYear.getTime()-1);
                    sDate = sYear;
                    eDate = eYear;
                    break;
                case 'thisYear':
                    sYear = new Date(today);
                    sYear.setFullYear(today.getFullYear());
                    sYear.setMonth(0);
                    sYear.setDate(1);
                    sDate = sYear;
                    break;
                case 'last7Days':
                    sDate = new Date(today - oneDay*6);
                    break;
                case 'last30Days':
                    sDate = new Date(today - oneDay*29);
                    break;
                case 'onlineUntilNow':
                    sDate = new Date(startservicetime);
                    break;
            }
            this.value = [sDate, eDate];
            setTimeout(()=>{this.timeConfirmHanlder();},0);
        },
        selectPartShowHandler: function(showOrNot){     //是否显示选择面板   show  hide
            let selectPartShowOrNot = showOrNot === 'show' ? true : false,
                currentShowStatus = this.selectPartShow;
            if(currentShowStatus.toString() === selectPartShowOrNot.toString()) return;

            this.showDatePanelOrTimePanel('date');
            this.$set(this, 'selectPartShow', selectPartShowOrNot);
        },
        timeClearHandler: function(){
            this.showDatePanelOrTimePanel('date');
            let val = this.value;
            if((val[0] == undefined || (val[0] && val[0].toString() === 'Invalid Date')) && (val[1] == undefined || (val[1] && val[1].toString() === 'Invalid Date'))) return;

            let refs = this.$refs,
                sDatepickerCellsDOM = refs.sDatepicker.$el.querySelector('.ivu-select-dropdown .ivu-date-picker-cells'),
                eDatepickerCellsDOM = refs.eDatepicker.$el.querySelector('.ivu-select-dropdown .ivu-date-picker-cells'),
                currentMonthDatesDOM_s = sDatepickerCellsDOM.querySelectorAll('.ivu-date-picker-cells-cell:not(.ivu-date-picker-cells-cell-prev-month)'),
                currentMonthDatesDOM_e = eDatepickerCellsDOM.querySelectorAll('.ivu-date-picker-cells-cell:not(.ivu-date-picker-cells-cell-prev-month)');
            refs = null;
            sDatepickerCellsDOM = null;
            eDatepickerCellsDOM = null;
            this.value.splice(0, 2);
            //清除range样式
            this.clearDateRangeStyleCore(currentMonthDatesDOM_s);
            currentMonthDatesDOM_s = null;
            this.clearDateRangeStyleCore(currentMonthDatesDOM_e);
            currentMonthDatesDOM_e = null;
        },
        timeConfirmHanlder: function(){
            let val = this.value;
            if((val[0] && val[0].toString() === 'Invalid Date') || (val[1] && val[1].toString() === 'Invalid Date') || val[0] == undefined || val[1] == undefined || val[0] === '' || val[1] === ''){
                this.$Message.warning('请选择时间');
                return;
            }
            this.selectPartShowHandler('hide');
            this.$emit('confirm', this.value);        
        },
        showDatePanelOrTimePanel: function(panelType){  //date: 显示日期面板   time: 显示时间面板
            if(this.dateType_inner === 'datetime'){
                let refs = this.$refs,
                    sDatepicker = refs.sDatepicker,
                    eDatepicker = refs.eDatepicker,
                    picker_1 = sDatepicker.picker,
                    picker_2 = eDatepicker.picker;
                
                switch(panelType){
                    case 'date':
                        this.$set(picker_1, 'currentView', 'date');
                        this.$set(picker_1, 'isTime', false);
                        this.$set(picker_2, 'currentView', 'date');
                        this.$set(picker_2, 'isTime', false);
                        this.isTime = false;
                        break;
                    case 'time':
                        this.$set(picker_1, 'currentView', 'time');
                        this.$set(picker_1, 'isTime', true);
                        this.$set(picker_2, 'currentView', 'time');
                        this.$set(picker_2, 'isTime', true);
                        this.isTime = true;                        
                        break;
                    default:
                        console.error('[Parameter Error]: function ---- showDatePanelOrTimePanel');
                }
                
                refs = null;
                sDatepicker = null;
                eDatepicker = null;
                picker_1 = null;
                picker_2 = null;
            }
        },
        togglePickerPanel: function(){
            if(!this.isTime){
                this.showDatePanelOrTimePanel('time');
            } else{
                this.showDatePanelOrTimePanel('date');
            }
        },
        disabledDateHandler_1: function(date){
            let rawDate = this.value[1];
            if(rawDate && rawDate instanceof Date && rawDate.toString() !== 'Invalid Date'){
                setTimeout(()=>{        //延迟执行，避免回调地狱，减少短时间内的内存占用
                    this.dateRangeStyleHandler_1(date);
                }, 0);
                let computedDate = new Date(rawDate.getFullYear(), rawDate.getMonth(), rawDate.getDate());    //时分秒设为0
                return date && date.valueOf() > computedDate.valueOf() + 86400000/2;
            } else{
                return false;
            }
        },
        disabledDateHandler_2: function(date){
            let rawDate = this.value[0];
            if(rawDate && rawDate instanceof Date && rawDate.toString() !== 'Invalid Date'){
                setTimeout(()=>{        //延迟执行，避免回调地狱，减少短时间内的内存占用
                    this.dateRangeStyleHandler_2(date);
                }, 0);
                let computedDate = new Date(rawDate.getFullYear(), rawDate.getMonth(), rawDate.getDate());    //时分秒设为0
                return date && date.valueOf() < computedDate.valueOf() - 86400000/2;
            } else{
                return false;
            }
        },
        dateRangeControl: function(){
            this.$set(this.sOptions, 'disabledDate', this.disabledDateHandler_1);
            this.$set(this.eOptions, 'disabledDate', this.disabledDateHandler_2);
        },
        dateRangeStyleHandler_1: function(date){
            let valueArr = this.value,
                rawDate1 = valueArr[0],
                rawDate2 = valueArr[1];

            if(!(rawDate1 && rawDate1 instanceof Date && rawDate1.toString() !== 'Invalid Date') || !(rawDate2 && rawDate2 instanceof Date && rawDate2.toString() !== 'Invalid Date')) return;
            let sDatepickerCellsDOM = this.$refs.sDatepicker.$el.querySelector('.ivu-select-dropdown .ivu-date-picker-cells'),
                currentMonthDatesDOM = sDatepickerCellsDOM.querySelectorAll('.ivu-date-picker-cells-cell:not(.ivu-date-picker-cells-cell-prev-month)');
            sDatepickerCellsDOM = null;

            //在当月第一天把range样式清除
            this.clearDateRangeStyle('sDatepicker', date, currentMonthDatesDOM);
            //判断特定日期范围内执行操作
            this.addDateRangeStyle('sDatepicker', date, currentMonthDatesDOM);
            date = null;
            currentMonthDatesDOM = null;
            return;
        },
        dateRangeStyleHandler_2: function(date){
            let valueArr = this.value,
                rawDate1 = valueArr[0],
                rawDate2 = valueArr[1];

            if(!(rawDate1 && rawDate1 instanceof Date && rawDate1.toString() !== 'Invalid Date') || !(rawDate2 && rawDate2 instanceof Date && rawDate2.toString() !== 'Invalid Date')) return;
            let eDatepickerCellsDOM = this.$refs.eDatepicker.$el.querySelector('.ivu-select-dropdown .ivu-date-picker-cells'),
                currentMonthDatesDOM = eDatepickerCellsDOM.querySelectorAll('.ivu-date-picker-cells-cell:not(.ivu-date-picker-cells-cell-prev-month)');
            eDatepickerCellsDOM = null;

            //在当月第一天把range样式清除
            this.clearDateRangeStyle('eDatepicker', date, currentMonthDatesDOM);
            //判断特定日期范围内执行操作
            this.addDateRangeStyle('eDatepicker', date, currentMonthDatesDOM);
            date = null;
            currentMonthDatesDOM = null;
            return;
        },
        clearDateRangeStyle: function(Datepicker, date, currentMonthDatesDOM){    //供dateRangeStyleHandler方法调用，在当月第一天把所有range样式清除
            let enableClearRangeClass = Datepicker === 'sDatepicker' ? 'enableClearRangeClass_1' : 'enableClearRangeClass_2',
                picker = this.$refs[Datepicker].picker;
            if(this[enableClearRangeClass] && date.getMonth() === picker.month && date.getFullYear() === picker.year && date.getDate() === 1){
                picker = null;
                this[enableClearRangeClass] = false;
                this.clearDateRangeStyleCore(currentMonthDatesDOM);
            }
            picker = null;
            date = null;
            currentMonthDatesDOM = null;
        },
        clearDateRangeStyleCore: function(currentMonthDatesDOM){    //把所有range样式清除
            currentMonthDatesDOM.forEach((item)=>{
                let itemClassNameList = item.className.split(' '),
                    rangeClassIndex = itemClassNameList.indexOf('ivu-date-picker-cells-cell-range');
                if(rangeClassIndex !== -1){
                    itemClassNameList.splice(rangeClassIndex, 1);
                    item.className = itemClassNameList.join(' ');
                }
            });
            currentMonthDatesDOM = null;
        },
        addDateRangeStyle: function(Datepicker, date, currentMonthDatesDOM){   //供dateRangeStyleHandler方法调用，为一个日期添加一个range样式
            let rawDate1 = this.value[0],
                rawDate2 = this.value[1],
                picker = this.$refs[Datepicker].picker;
            rawDate1 = new Date(rawDate1.getFullYear(), rawDate1.getMonth(), rawDate1.getDate());
            rawDate2 = new Date(rawDate2.getFullYear(), rawDate2.getMonth(), rawDate2.getDate());

            if(date.valueOf() >= rawDate1.valueOf() && date.valueOf() <= rawDate2.valueOf() && date.getMonth() === picker.month &&  date.getFullYear() === picker.year){   
                picker = null;
                let dateIndex = date.getDate() - 1;
                dateIndex = dateIndex < 0 ? 0 : dateIndex;
                let currentDateDOM = currentMonthDatesDOM[dateIndex],
                    currentDateDOMclassNameList = [].concat(currentDateDOM.className.split(' ')),
                    rangeClassIndex = currentDateDOMclassNameList.indexOf('ivu-date-picker-cells-cell-range');
                currentMonthDatesDOM = null;
                if(rangeClassIndex === -1){
                    currentDateDOMclassNameList.push('ivu-date-picker-cells-cell-range');
                    currentDateDOM.className = currentDateDOMclassNameList.join(' ');
                } 
                currentDateDOM = null;
            }
            picker = null;
            date = null;
        },
    },
    props: {
        dateValue: {          //时间值
            type: Array,
            default: function(){
                return [];        //传两个Date类型 或者 空数组    eg: [new Date(), new Date()]
            }
        },
        dateType: {
            type: String,
            default: 'date'      // date: 显示日期   datetime: 显示时期及时分秒
        },
        size: {
            type: String,
            default: 'default'
        },
        width: {
            type: Number,
            default: function(){
                let res = null;
                let size = this.size;
                switch(this.dateType){
                    case 'date':
                    default:
                        if(size === 'large'){
                            res = 260;
                        } else{
                            res = 240;
                        }
                        break;
                    case 'datetime':
                        if(size === 'large'){
                            res = 380;
                        } else{
                            res = 300;
                        }
                        break;
                }
                return res;
            }
        },
    },
    watch: {
        value(val){
            if(val.length <= 0){
                this.dateText = '';
                this.dateValue[0] = new Date('Invalid Date');
                this.dateValue[1] = new Date('Invalid Date');
                return;
            } else if((val[0] && val[0].toString() === 'Invalid Date') || (val[1] && val[1].toString() === 'Invalid Date') || val[0] == undefined || val[1] == undefined || val[0] === '' || val[1] === ''){
                this.dateText = ''; 
                this.dateValue[0] = new Date('Invalid Date');
                this.dateValue[1] = new Date('Invalid Date');            
                return;
            } else{
                //判断两个时间大小，第一个时间必须小于等于第二个
                if(val[0].valueOf() > val[1].valueOf()){
                    val[1] = new Date(val[0].valueOf());    //后期考虑选择时间时  数字的复位
                }
                //手动改原组件DatePicker中的高亮日期 
                if(this.$refs.sDatepicker.picker){     
                    val[0] ? this.$set(this.$refs.sDatepicker.picker, 'value', val[0]) : '';
                    val[1] ? this.$set(this.$refs.eDatepicker.picker, 'value', val[1]) : '';
                }
                //设置显示时间
                let res = [];
                switch(this.dateType_inner){
                    case 'date':
                    default:
                        val.forEach((item, index)=>{
                            res[index] = this.formatDate(item, 'yyyy-MM-dd');
                        });
                        break;
                    case 'datetime':
                        val.forEach((item, index)=>{
                            res[index] = this.formatDate(item, 'yyyy-MM-dd hh:mm:ss');
                        });
                        break;
                }
                this.dateText = res.join(' - ');
                //同步dateValue的值  这样修改，vue不会监听到dateValue的变化
                this.dateValue[0] = val[0];
                this.dateValue[1] = val[1];
            }             
        },
        dateValue(val){
            setTimeout(()=>{
                this.importUserInput();
            }, 0);
        },
        enableClearRangeClass_1(val){    //在外部监听该值，并做延迟变化
            clearTimeout(this.enableClearRangeClassSwitchTimer_1);
            this.enableClearRangeClassSwitchTimer_1 = setTimeout(()=>{this.enableClearRangeClass_1 = true}, 0);
        },
        enableClearRangeClass_2(val){    //在外部监听该值，并做延迟变化
            clearTimeout(this.enableClearRangeClassSwitchTimer_2);
            this.enableClearRangeClassSwitchTimer_2 = setTimeout(()=>{this.enableClearRangeClass_2 = true}, 0);
        },
    },

}
</script>


<style>
.datepicker-style{display: block;}
.datepicker-style .ivu-select-dropdown{margin: 0 !important; box-shadow: none; border: 1px solid #E9EAEC;}
.date-btn-datetime .ivu-btn-group{float: right;}
.date-btn-date .ivu-btn-group{float: right;}
.large-datepicker-range .ivu-input-icon-normal{height: 100%;}
.large-datepicker-range .ivu-input-icon-normal+.ivu-input { height: 38px !important; line-height: 38px !important; font-size: 16px; border-color: #bbb;}
.large-datepicker-range .ivu-input-icon-normal+.ivu-input:hover {border-color: #2D8CF0;}
.large-datepicker-range .ivu-input-icon-normal+.ivu-input:focus {border-color: #2D8CF0;}
</style>

<style scoped>
.datepicker-range{height: 32px; position: relative; display: inline-block; line-height: normal; vertical-align: middle;}
.large-datepicker-range{height: 38px !important;}
.large-datepicker-range .date-display .ivu-input-wrapper{height: 38px !important;}
.datepicker-range .date-display{width: 100%; height: 100%; position: absolute;}
/*date-select*/
.datepicker-range .date-select{height: 335px; background-color: #fff; position: absolute; top: 35px; z-index: 1000; padding: 12px; border-radius: 4px; box-shadow: 0 0 3px 1px rgba(191,191,191,0.5); 
    overflow: hidden;}
.large-datepicker-range .date-select { top: 40px;}
.datepicker-range .date-select .date-control{width: 540px;}
.datepicker-range .date-select .date-btn-datetime{position: absolute; right: 10px; bottom: 15px; width: 440px;}
.datepicker-range .date-select .date-btn-date{position: absolute; right: 10px; bottom: 15px;}
.datepicker-range .date-select .date-btn-datetime .timepicker-btn{float: left; height: 24px; user-select: none;}
.datepicker-range .date-select .date-btn-datetime .timepicker-btn>span{line-height: 24px; color: #2d8cf0; font-size: 12px; cursor: pointer; transition: all .2s ease-in-out;}
.datepicker-range .date-select .date-btn-datetime .timepicker-btn>span:hover{color: #57a3f3}

.datepicker-range .date-select .dateselect-part{float: left; margin-right: 10px; position: relative;}
.datepicker-range .date-select .dateselect-part:nth-child(3){margin-right: 0;}
.datepicker-range .date-select .title{width: 100%; text-align: center; font-size: 13px; margin-bottom: 5px; color: #757575;}
/*date shortcut*/
.datepicker-range .date-select .date-shortcut{width: 88px; text-align: center; overflow: hidden;}
.datepicker-range .date-shortcut .content p{float: left; margin: 0 4px 8px 0; width: 40px; height: 24px; font-size: 12px; text-align: center; color: #559FF0; line-height: 24px; 
    background-color: #F5F5F5; border-radius: 3px; cursor: pointer; transition: .2s;}
.datepicker-range .date-shortcut .content p:hover{color: #fff; background-color: #2D8CF0;}
.datepicker-range .date-shortcut .content p.shortcut-selected{color: #fff; background-color: #2D8CF0;}
.datepicker-range .date-shortcut .content p:nth-child(8)~p{width: 84px;}
/*datepicker*/
.datepicker-range .date-select .date-datepicker{width: 216px;}
.datepicker-range .date-select .date-datepicker .content{height: 250px; position: relative;}

/*切换动画*/
.vertical-toggle-enter-to, .vertical-toggle-leave{opacity: 0; transform-origin: center top 0px;}
.vertical-toggle-enter-active{ animation: slide-toggle .3s ease-out; }
.vertical-toggle-leave-active{ animation: slide-toggle-reverse .3s ease-out; }
@keyframes slide-toggle{
    0%{
        transform-origin: 0% 0%;
        transform: scaleY(0.75);
        opacity: 0;
    }
    100%{
        transform-origin: 0% 0%;
        transform: scaleY(1);
        opacity: 1;
    }
}
@keyframes slide-toggle-reverse{
    0%{
        transform-origin: 0% 0%;
        transform: scaleY(1);
        opacity: 1;
    }
    100%{
        transform-origin: 0% 0%;
        transform: scaleY(0.8);
        opacity: 0;
    }
}
</style>