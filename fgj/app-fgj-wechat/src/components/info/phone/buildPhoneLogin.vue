<template>
    <div class="main">
        <div class="form">
            <div class="cell">
                <span class="phone"></span>
                <input type="number" class="cell_input" placeholder="请输入手机号码" maxlength="11" v-model="mobile">
            </div>

            <div id="yzm">
                <div class="cell">
                    <span class="yzm"></span>
                    <input type="number" class="cell_input" placeholder="请输入验证码" v-model="code">
                </div>

                <div class="yzm_btn" @click="sendIdentifyingCode" v-if="yzmShow">
                    <span>获取验证码</span>
                </div>
                <div class="yzm_btn disabled" v-else>
                    <span>{{count}}秒</span>
                </div>
            </div>
        </div>

        <div class="btn" :class="{disabled:mobile==''||code==''}" @click="bindingMobile">
            立即绑定
        </div>
    </div>
</template>

<script>

    import {sendIdentifyingCode,bindingMobile} from '../../../api/getData';
    import { Toast } from 'mint-ui';
    import { MessageBox } from 'mint-ui';
    import { Indicator } from 'mint-ui';
    export default{

        data(){
            return{
                yzmShow:true,
                count:60,
                timer: null,
                mobile:'',
                code:''
            }
        },
        created () {

        },
        mounted(){

        },
        watch : {

        },
		components: {

		},
        methods:{
            sendIdentifyingCode(){
                if (!this.timer && /(^1[3|4|5|6|7|8|9]\d{9}$)|(^09\d{8}$)/.test(this.mobile)) {
                    this.count = 60;
                    this.yzmShow = false;
                    this.sendIdentifyingCodeAjax();
                }else{
                    Toast('手机号码格式错误');
                }
            },
            async sendIdentifyingCodeAjax(){
                try {
                    const res = await sendIdentifyingCode({mobile:this.mobile,type:2});

                    if (res.data.code == 0) {
                        Toast(res.data.data);
                        this.timer = setInterval(() => {
                        if (this.count > 0 && this.count <= 60) {
                                this.count--;
                            } else {
                                this.yzmShow = true;
                            clearInterval(this.timer);
                                this.timer = null;
                            }
                        }, 1000)
                    } else {
                        this.yzmShow = true;
                        Toast(res.data.error);
                    }

                } catch (err) {
                    this.yzmShow = true;
                    Toast(err);
                }
            },
            async bindingMobile(){
                try {
                    if(this.mobile==''||this.code==''){
                        return;
                    }
                    const res = await bindingMobile({mobile:this.mobile,type:1,identityCode:this.code,weiXinOpenId:this.$store.state.Cookie.mutations.fetch('openId')});
                    if (res.data.code == 0) {
                        Toast('绑定成功');
                        this.$router.push('/home');
                    } else {
                        Toast(res.data.error);
                    }

                } catch (err) {
                    Toast(err);
                }
            }
        }
    }
</script>

<style lang="scss" scoped>
@import "../../../style/info/buildPhone.scss";
</style>
