<template>
    <div class='register'>
        <div class='title'>账号注册</div>
        <div class='user'>
            <img src="../../../assets/register/yhm.png" alt="">
            <input type="text" placeholder="请设置用户名" v-model='userNum'>
        </div>
        <div class='password'>
            <img src="../../../assets/register/password.png" alt="">
            <input type="password" ref='password' placeholder="请设置密码" v-model='passwordNum'>
            <img v-if='eyeShow==true' class='passKeyNo' src="../../../assets/register/kj.png" alt="" @touchstart='eyePassword'>
            <img v-else class='passKey' src="../../../assets/register/bkj.png" alt=""  @touchstart='eyePasswordNo'>
        </div>
        <div class='iphone'>
            <img src="../../../assets/register/iphone.png" alt="">
            <input type="text" v-model='iphoneNum' placeholder="请绑定手机号">
        </div>
        <div class='verCode'>
            <img src="../../../assets/register/yzm.png" alt="">
            <input type="text" placeholder="请输入验证码" v-model='codeNum'>
            <button @touchend="codeOne && addBtnFlag && getlogin()" class="verCodeRight" type="primary">{{ btnText }}</button>
        </div>
        <button :class="btnState?'btnRegister':'btnRegisterNo'" @touchend='btnState?registerBtn():""'>立即注册</button>
        <!-- <router-link to='../loginPassword'><div class='goLogin'>已有账号？快去登录吧></div></router-link> -->
        <div class='agreement'>
            <span>注册表示您同意</span>
            <router-link to='../registerAgr'><span class='agreementSpan'>《用户注册协议》</span></router-link>
        </div>
        <toast v-model="showPositionValue" type="text" :time="1000" is-show-mask :text="toastText" :position="position" width='auto'></toast>
        <div class='returnGo' @click='$router.go(-1)'></div>
    </div>
</template>
<script>
import { getUserRepeatAPI,getCodeAPI,RegisterAPI,getPhoneRepeatAPI,getLoginValidatePhoneCodeAPI } from '@/api/login/login.js'
import md5 from 'blueimp-md5';
import Cookies from 'js-cookie';
export default {
    data(){
        return{
            showPositionValue: false, //是否显示提示
            position: 'center', //提示信息的位置
            toastText: '', // 提示文本
            iphoneNum:'',
            userNum:'',
            passwordNum:'',
            codeNum:'',
            eyeShow:false,
            interval:undefined,
            totalCount:0,
            addBtnFlag:true,
            passwordTwo:true,
            phoneUuid:'',
            codeOne:true,
        }
    },
    mounted(){
        document.body.style.backgroundColor = '#fff'
    },
    computed:{
        btnText(){
            return this.totalCount !==0? `重新获取(${this.totalCount}s)`: "获取验证码"
        },
        btnState(){
            return this.userNum!==''&& this.passwordNum!==''&& this.iphoneNum!=='' && this.codeNum!=='' 
            // && this.passwordTwo == true && this.nameTwo == true
        }
    },
    created(){

    },
    methods:{
        prev(){
            this.$router.go(-1)
        },
        getUserRepeat(name){
            getUserRepeatAPI(name).then(res=>{
                if( !res.data.status ){
                    this.showPositionValue = true
                    this.toastText = res.data.msg
                }
            })
        },
        getlogin(){
            this.codeOne = false
            if( this.iphoneNum.length != '11' ){
                this.showPositionValue = true;
                this.toastText = '请输入正确的手机号'
                return
            }else{
                let data = {
                    phone:this.iphoneNum,
                    type:0
                }
                getCodeAPI(data).then(res=>{
                    this.phoneUuid = res.data.uuid
                    if( res.data.code == 0 ){
                        this.totalCount=60
                        this.showPositionValue = true;
                        this.toastText = '已发送验证码'
                        
                        if( this.addBtnFlag ){
                            this.interval=setInterval(()=>{
                                this.totalCount--
                                this.addBtnFlag = false
                                if(this.totalCount === 0){
                                    clearInterval(this.interval)
                                    this.addBtnFlag = true
                                    this.codeOne = true
                                }
                            },1000);
                        }
                        if( res.data.data.count == '6' ){
                            this.showPositionValue = true;
                            this.toastText = '该手机号还可以验证4次，请尽快验证'
                        }
                    }else{
                        this.showPositionValue = true;
                        this.toastText = res.data.msg
                    }
                })
            }
        },
        registerBtn(){
            if( this.userNum.length > 0 ){
                if( this.userNum.length < 4  || this.userNum.length > 20 ){
                    this.showPositionValue = true;
                    this.toastText = '用户名长度必须是4-20个字符'
                    return
                }else if( this.userNum !== '' ){
                    this.getUserRepeat(this.userNum)
                }
            }
            if( this.passwordNum.length > 0 ){
                if( this.passwordNum.length < 6 || this.passwordNum.length > 20 ){
                    this.showPositionValue = true;
                    this.toastText = '密码长度必须在6-20个字符'
                    this.passwordTwo = false
                    return
                }else if( this.passwordNum !== '' ){
                    let reg = /^[0-9a-zA-Z@#-_]+$/
                    let regS = /^[0-9]+$/
                    let regA = /^[a-zA-Z]+$/
                    let regB = /^[@]+$/
                    let regC = /^[#]+$/
                    let regD = /^[-]+$/
                    let regE = /^[_]+$/
                    if( !reg.test(this.passwordNum) ){
                        this.showPositionValue = true;
                        this.toastText = '非法字符'
                        this.passwordTwo = false
                        return
                    }else if( regS.test(this.passwordNum) || regA.test(this.passwordNum ) || regB.test(this.passwordNum ) || regC.test(this.passwordNum )|| regD.test(this.passwordNum )|| regE.test(this.passwordNum ) ){
                        this.showPositionValue = true;
                        this.toastText = '密码不能为同一字符'
                        this.passwordTwo = false
                        return
                    }else{
                        this.passwordTwo = true
                    }
                }
            }
            if( this.iphoneNum.length > 0 ){
                let reg = /^1[3456789]\d{9}$/
                if( !reg.test(this.iphoneNum) ){
                    this.showPositionValue = true;
                    this.toastText = '请输入正确的手机号'
                    return
                }
            }
            let reg = /^[0-9]+$/
            if( !reg.test(this.codeNum) && this.codeNum.length !== 0){
                this.showPositionValue = true;
                this.toastText = '请输入正确字符'
                return
            }
            getPhoneRepeatAPI(this.iphoneNum).then(res=>{
                if( res.data.code == 0 ){
                    if( res.data.data ){
                        this.showPositionValue = true;
                        this.toastText = ('该手机号已注册过')
                    }else{
                        this.getLoginValidatePhoneCode()
                    }
                }
            })
            
        },
        getLoginValidatePhoneCode(){
            let data = {
                phone:this.iphoneNum,
                code:this.codeNum
            }
            getLoginValidatePhoneCodeAPI(data).then(res=>{
                if( res.data.code == 0 ){
                    this.register()
                }else{
                    this.showPositionValue = true;
                    this.toastText = '验证码输入错误！'
                }
            })
        },
        register(){
            let data ={
                account:this.userNum,
                phoneCode:this.codeNum,
                password:md5(this.passwordNum).toUpperCase(),
                phone:this.iphoneNum,
                phoneUuid:this.phoneUuid
            }
            RegisterAPI(data).then(res=>{
                if( res.data.code == 0 ){
                    this.showPositionValue = true;
                    this.toastText = '注册成功';
                    setTimeout(()=>{
                        this.$store.commit('setToken',res.data.data);
                        this.$router.push({path:'/loginPassword'});
                    },1000)
                    Cookies.set('publishName',this.userNum)
                    Cookies.set('publishPhone',this.iphoneNum)
                }else{
                    this.showPositionValue = true;
                    this.toastText = res.data.msg
                }
            })
        },
        eyePassword(){
            this.eyeShow = false
            this.$refs.password.setAttribute('type','password')
        },
        eyePasswordNo(){
            this.eyeShow = true
            this.$refs.password.setAttribute('type','text')
        },
    }
}
</script>
<style scoped lang='less'>
    .register{
        background:#fff;
        overflow:hidden;
        position:relative;
        top:0;
        .returnGo{
            width:0.57rem;
            height:0.305rem;
            background:url('../../../assets/img/list/fanhuiRight.png') no-repeat right;
            background-size: 0.57rem 0.305rem;
            position:fixed;
            right:0;
            bottom:0.9rem;
        }
        .title{
            color:#333;
            font-size:0.22rem;
            font-weight:bold;
            text-align: left;
            text-align: left;
            margin:0.4rem 0 0.15rem 0.24rem;
        }
        .user,.password,.iphone,.verCode{
            width:3.45rem;
            height:0.63rem;
            margin:auto;
            border-bottom:0.01rem solid #E3E3E3;
            img{
                float:left;
                margin:0.21rem 0.15rem 0 0.12rem;
            }
            input{
                width:2.4rem;
                border:none;
                font-size:0.15rem;
                float:left;
                margin-top:0.23rem;
            }
            .passKeyNo{
                float:right;
                width:0.17rem;
                height:0.1rem;
                margin:0.27rem 0.1rem;
            }
            .passKey{
                float:right;
                width:0.17rem;
                height:0.1rem;
                margin:0.27rem 0.1rem;
            }
            .verCodeRight{
                width:auto;
                height:0.18rem;
                background:url('../../../../static/img/zhucecode.png') no-repeat left #fff;
                float:right;
                font-size:0.13rem;
                color:#BC0606;
                margin:0.22rem 0.1rem;
                border:none;
                // border-left:0.01rem dashed #000;
                padding-left:0.1rem;
            }
        }
        .user{
            img{
                width:0.19rem;
                height:0.2rem;
            }
        }
        .password{
            img{
                width:0.16rem;
                height:0.19rem;
            }
        }
        .iphone{
            img{
                width:0.15rem;
                height:0.2rem;
            }
        }
        .verCode{
            .verCodeCup{
                width:1.2rem;
                height:100%;
                background:pink;
                float:right;
            }
            img{
                width:0.17rem;
                height:0.2rem;
            }
            input{
                width:1.5rem;
                padding-right:0.05rem;
            }
        }
        .btnRegister{
            display:block;
            width:3.3rem;
            height:0.42rem;
            border:none;
            border-radius: 0.21rem;
            color:#fff;
            background:linear-gradient(90deg,rgba(228,34,34,1) 0%,rgba(187,6,6,1) 100%);
            margin:0.4rem auto 0.27rem auto;
            font-size:0.16rem;
            letter-spacing: 0.02rem;
            font-weight:bold;
            text-align: center;
            line-height:0.42rem;
        }
        .btnRegisterNo{
            display:block;
            width:3.3rem;
            height:0.42rem;
            border:none;
            background-size:100%;
            border-radius: 0.21rem;
            font-weight:bold;
            text-align: center;
            line-height:0.42rem;
            color:#5F6775;
            background:#E6E6E6;
            margin:0.4rem auto 0.27rem auto;
            font-size:0.16rem;
            letter-spacing: 0.02rem;
        }
        .goLogin{
            color:#BA9F74;
            font-size:0.13rem;
            margin-bottom:0.95rem;
            text-align: center;
        }
        .agreement{
            font-size:0.13rem;
            text-align: center;
            .agreementSpan{
                color:rgba(187, 6, 6, 1);
            }
        }
    }
</style>

