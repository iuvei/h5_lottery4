
<template>
    <div class="so-main-down">
        <ul>
            <li>
                <p>
                    <!-- id 8 -->
                 <!--   第 <span class="now-date">{{(lotteryID==8)?now_pcode.toString().substr(4, 8):now_pcode}}</span> 期  -->
                    第 <span class="now-date">{{now_pcode}}</span> 期
                </p>
            </li>
            <li>
                <i></i>
                <span>封盘</span>
                <a class="close-time">
                    {{(lt_time_leave_over <= 0) ? '已封盘' : overTimeSpanStr}}
                     
                    <!-- 00:00 --></a>
                    
            </li>
            <li class="kjdjs-timer">
                <i></i>
                <span >开奖</span>
                <a class="open-time">{{timeSpanStr}}</a>
            </li>
        </ul>
    </div>
</template>


<script>
import Mixin from '@/Mixin'

export default {
    name: 'CountdownTimer',
    mixins:[Mixin],
    // start服务器开始时间，end当前期开奖结束时间，overend 封盘结束时间
    props:['start', 'end', 'overend', 'now_pcode', 'lotteryID'],
    data () {
        return {
            timer:null,
            lt_time_leave:0, 
            lt_time_leave_over:0, 
            timeSpanStr:'',
            overTimeSpanStr:'',
            wrongFlag: false,

        }
    },
    created:function(){

    },
    mounted:function() {
        var that = this;
        $(document).on('visibilitychange', (function() {
            // 页面变为可见时触发
            if (document.visibilityState == 'visible') {
                that.$emit('visibility');
            }
        }).bind(this));
    },
    beforeDestroy:function(){
        clearInterval(this.timer);
    },
    methods:{
        timerInit:function(start, end, overend){
            const format = this.format;
            const theStart = start ? start : this.start;
            const theEnd = end ? end : this.end;
            const theOverend = overend ? overend : this.overend;
            let timeSpan = this.lt_time_leave = (this.format(theEnd).getTime() - this.format(theStart).getTime()) / 1000;//总秒数
            this.lt_time_leave_over = (this.format(theOverend).getTime() - this.format(theStart).getTime()) / 1000;//总秒数
          
           // console.log(this.lt_time_leave_over  ,'timebreak')

            if (this.lt_time_leave_over <0){ // 封盘倒计时结束
                this.$emit('entertainCountdownBreak');   
                console.log(1)    
            }
            //计数器，计算间隔时间触发
            const counter = () => {
                const spanSrrived = () => {
                    this.$emit('spanArrived');
                }
                const lt_time_leave = this.lt_time_leave;
                if (timeSpan-lt_time_leave<60*1 && lt_time_leave % 10 == 0){
                    spanSrrived();
                } else if (timeSpan-lt_time_leave<60*2 && lt_time_leave % 20 == 0){
                    spanSrrived();
                } else if (timeSpan-lt_time_leave<60*3 && lt_time_leave % 30 == 0){
                    spanSrrived();
                }else if (lt_time_leave==30){
                    spanSrrived();
                }
                 if(this.lotteryID=='2'&&lt_time_leave % 60 == 0&&timeSpan-lt_time_leave<60*6){
                    spanSrrived();     
                }
            }

            clearInterval(this.timer);
            this.timer = window.setInterval((function() {
               // console.log('蒂ff冈你')
                counter();
                if (this.lt_time_leave <= -1) {
                    this.wrongFlag = true;
                } else {
                    this.wrongFlag = false;
                }
               // console.log(this.lt_time_leave,'timeouver')

                if (this.lt_time_leave <= 0) {   // 开奖倒计时结束
                    clearInterval(this.timer);
                    this.$emit('countdownOver');
                    console.log(2)
                }
               //  console.log(this.lt_time_leave_over)

                if(this.lt_time_leave_over == 0  ){  // 封盘倒计时结束
                    this.$emit('entertainCountdownOver');
                    console.log(3)
                }

                this.lt_time_leave = this.lt_time_leave - 1;
                var oDate = this.diff(this.lt_time_leave);  // 开奖倒计时
                this.lt_time_leave_over = this.lt_time_leave_over - 1;
                var over_oDate = this.diff(this.lt_time_leave_over);  // 封盘倒计时
               

                if(oDate.hour =='00'){  // 平常时间没有小时
                    this.timeSpanStr = this.fftime(oDate.minute) + ':' + this.fftime(oDate.second); // 开奖倒计时
                    this.overTimeSpanStr = this.fftime(over_oDate.minute) + ':' + this.fftime(over_oDate.second); // 封盘倒计时
                }else{   // 跨天时间有小时
                    this.timeSpanStr = this.fftime(oDate.hour) + ':' + this.fftime(oDate.minute) + ':' + this.fftime(oDate.second); // 开奖倒计时
                    this.overTimeSpanStr = this.fftime(over_oDate.hour) + ':' + this.fftime(over_oDate.minute) + ':' + this.fftime(over_oDate.second); // 封盘倒计时
                }
                // console.log( oDate.day ,'day')
                if (oDate.day != 0) {
                    console.log(22)
                    this.timeSpanStr = this.fftime(oDate.day) + '天' + this.fftime(oDate.hour) + ':' + this.fftime(oDate.minute) + ':' + this.fftime(oDate.second); // 开奖倒计时
                    this.overTimeSpanStr = this.fftime(oDate.day) + '天' + this.fftime(over_oDate.hour) + ':' + this.fftime(over_oDate.minute) + ':' + this.fftime(over_oDate.second); // 封盘倒计时
                }


            }).bind(this), 1000);
        }


    }
}
</script>
