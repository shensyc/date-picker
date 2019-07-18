<template>
    <div v-click-outside>
        <input type="text" :value=" formatDate"> 
        <div class="pannel" v-if="isVisible">
           <div class="pannel-nav">
               <span>&lt;</span>
               <span @click="prevMonth">&lt;&lt;</span>
               <span>{{time.year}}年</span>
               <span>{{time.month+1}}月</span>
                <span @click="nextMonth">&gt;&gt;</span>
               <span>&gt;</span>
           </div>
           <div class="pannel-content">
               <div class="days">

                    <span v-for="j in 7" :key="`_`+j" class="cell" >
                        {{weekDays[j-1]}}
                    </span>

                   <!-- 列出一个6*7的列表 -->
                   <div v-for="i in 6" :key="i">
                       <span
                        class="cell cell-days"
                        @click="chooseDate(visibleDays[(i-1)*7+(j-1)])"
                        :class = "[
                            {notCurrentMonth:!isCurrentMonth(visibleDays[(i-1)*7+(j-1)])},
                            {today:isToday(visibleDays[(i-1)*7+(j-1)])},
                            {select:isSelect(visibleDays[(i-1)*7+(j-1)])}
                        ]"
                        v-for="j in 7" :key="j">
                           {{visibleDays[(i-1)*7+(j-1)].getDate()}}
                       </span>
                   </div>
                   
               </div>
           </div>
           <div class="pannel-footer">
               今天
           </div>
        </div>
    </div>
</template>

<script>
    import * as utils from './util.js'
    export default {
        directives:{
            clickOutside:{
                bind(el,bindings,vnode){//context
                    //把事件绑定给document上看一下点击的是否是当前这个元素的内部
                    let handler = (e)=>{
                        //console.log(e.target);
                        if(el.contains(e.target)){
                            if(!vnode.context.isVisible){
                                vnode.context.focus();
                                console.log('focus');
                            }
                        }else{
                            if(vnode.context.isVisible){
                                vnode.context.blur();
                                console.log('blur');
                            }
                        }
                    }
                    el.handler = handler;
                    document.addEventListener('click',handler);
                },
                unbind(el){
                    document.addEventListener('click',el.handler);
                }
            }
        },
        data(){
            let {year,month} = utils.getYearMonthDay(this.value);
            return {
                weekDays:['日','一','二','三','四','五','六'],
                time:{year,month},
                isVisible:false//这个变量是用来控制面板是否可见的
            }
        },
        methods: {
            focus() {
                this.isVisible = true;
            },
            blur(){
                this.isVisible = false;
            },
            isCurrentMonth(date){//判断是不是当前月
                let {year,month} = utils.getYearMonthDay(utils.getDate(this.time.year,this.time.month,1));
                let {year:y,month:m} = utils.getYearMonthDay(date);
                return year === y && month === m;
            },
            isToday(date){//是不是今天
                let {year,month,day} = utils.getYearMonthDay(new Date());
                let {year:y,month:m,day:d} = utils.getYearMonthDay(date);
                return year === y && month === m && day === d;
            },
            chooseDate(date){
                this.time = utils.getYearMonthDay(date);
                this.$emit('input',date);
                this.blur();//关闭弹层
            },
            isSelect(date){
                let {year,month,day} = utils.getYearMonthDay(this.value);
                let {year:y,month:m,day:d} = utils.getYearMonthDay(date);
                return year === y && month === m && day === d;
            },
            prevMonth(){
                let d = utils.getDate(this.time.year,this.time.month,1);
                d.setMonth(getMonth()-1);
                this.time = utils.getYearMonthDay(d);
            },
            nextMonth(){
                let d = utils.getDate(this.time.year,this.time.month,1);
                d.setMonth(getMonth()+1);   
                this.time = utils.getYearMonthDay(d);
            }
        },
       props:{
           value:{
               type:Date,
               default:()=>new Date()  //返回的默认值必须是一个函数类型
           }
       },
      
       computed:{
           visibleDays(){
               //先获取当前是周几
               let {year,month} = utils.getYearMonthDay(utils.getDate(this.time.year,this.time.month,1));
               //获取当前月份的第一天
               let currentFirstDay = utils.getDate(year,month,1);
               //先获取当前是周几，把天数往前移几天
               let week = currentFirstDay.getDay();
               //当前开始的天数
               let startDay = currentFirstDay - week*60*60*1000*24;
               //循环42天
               let arr = [];
               for(let i=0;i<42;i++){
                   arr.push(new Date(startDay+i * 60 * 60 * 24 * 1000));
               }
               return arr;
           },
           formatDate(){
            let {year,month,day} = utils.getYearMonthDay(this.value);//getFullYear getMonth getDate
            return `${year}-${month+1}-${day}`;
           }
       }
    }
</script>

<style scoped>
    .pannel{
        position:absolute;
        width: 32px*7;
        box-shadow: 2px 2px 2px pink,-2px -2px 2px pink;
    }
    .pannel .pannel-nav{
        display: flex;
        justify-content: space-around;
        height: 30px;
    }
    .pannel .pannel-nav span{
        cursor: pointer;
        user-select: none;
    }
    .pannel .pannel-content .cell{
        display: inline-flex;
        justify-content: center;
        align-items: center;
        width: 32px;
        height: 32px;
        font-weight: bold;
        box-sizing: border-box;
    }
    .pannel .pannel-content .cell-days:hover,.select{
        border:1px solid pink;
        box-sizing: 
    }
    .pannel .pannel-footer{
        height: 30px;
        text-align: center;
    }
    .notCurrentMonth{
        color: gray;
    }
    .today{
        background: red;
        color: #fff;
        border-radius: 4px;
    }
</style>