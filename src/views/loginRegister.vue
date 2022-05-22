<template>
	<div class="guanliConfigure" v-if="zhengzaiguanliConfigure">
		<button class="fanhuiButton" @click="buguanliConfigure">返回</button>
		<div>-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</div>
		<ul>
			<li class="peizhiList" v-for="(configure,index) in configures">
				<div v-if="configure.isPortal">
				<div>
					域名：<input type="text" v-model="configure.domain">；</div>
				<div>	对外宣称的port：<input type="text" v-model="configure.port">；</div>
					
				<div>	对外宣称的IP地址：<input type="text" v-model="configure.address">；</div>
				<div>	对bridge宣称的port：<input type="text" v-model="configure.porttobridge">；</div>
				<div>	位于私网的bridge的port：<input type="text" v-model="configure.bridgeport">；</div>
				<div>	密钥：<input type="text" v-model="configure.miyao"></div>
				</div>
				<div v-else>
					<div>
					域名：<input type="text" v-model="configure.domain">；</div>
					<div>转发服务器IP:port：<input type="text" v-model="configure.redirect">；</div>
					<div>位于公网的portal的IP：<input type="text" v-model="configure.address">；</div>
					<div>位于公网的portal的port：<input type="text" v-model="configure.port">；</div>
					<div>密钥：<input type="text" v-model="configure.miyao"></div>
				</div>
				<button class="operateConfigureButton" @click="shengchengConfigure(configure)" v-if="!isDeleting">生成这个的配置文件</button>
				<button class="operateConfigureButton" @click="shanchuConfigure(index)" v-if="!isDeleting">删掉这个</button>
				<button class="operateConfigureButton" @click="gengaiConfigure(configure,index)" v-if="!isDeleting">保存更改</button>
				<div>-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</div>
			</li>
		</ul>
	</div>
	<div class="login-register" v-else>
		<div class="containv" v-if="isGuanliingUser">
			<div><button class="fanhuiButton" @click="buGuanliUser">返回</button></div>
			<div><input type="text" placeholder="用户名" v-model="form.domain">
			<input type="text" placeholder="密码" v-model="form.port">
			<button class="changbutton" @click="tianJIaUser">添加用户</button></div>
			<div><input type="text" placeholder="新的管理员用户名" v-model="form.address">
			<input type="text" placeholder="新的管理员密码" v-model="form.miyao">
			<button class="changbutton" @click="xiugaiAdmin">修改</button></div>
			<div class="containnamelist">
				<ul>用户列表：
					<div>---</div>
					<li v-for="item in items">
						账号：{{item.userName}}；
						密码：
						<input type="text" v-model="item.password">
						<button class="changbutton" @click="changePassword(item.userName, item.password)" v-if="!isDeleting">改密码</button>
						<button class="changbutton" variant="solid" color="danger"  @click="erase(item.userName)" v-if="!isDeleting">删除</button>
					</li>
				</ul>
			</div>
		</div>

		<div class="containv" v-else-if="isguanliMianban">
			<div class="btitle" style="text-align:center">面板配置</div>
			<div class="bform">
			<input type="text" placeholder="面板监听ip" v-model="form.domain">
			<input type="text" placeholder="面板监听端口" v-model="form.port">
			<input type="text" placeholder="面板证书公钥文件路径" v-model="form.address">
			<input type="text" placeholder="面板证书密钥文件路径" v-model="form.porttobridge">
			<input type="text" placeholder="面板URL根路径" v-model="form.miyao">
		    </div>
			<div style="text-align:center"><button class="bbutton"  @click="saveMianban">保存</button>
			<button class="bbutton" @click="buguanliMianban">返回</button></div>
			
		</div>
	

		<div class="contain" v-else-if="hasLogined">
			<div class="big-box" :class="{active:isPortal}">
				<!-- <button class="logOutButton" @click="logOut">登出</button> -->
				<div class="big-contain" key="bigContainLogin" v-if="isPortal">
					<div class="btitle">我是位于公网的portal</div>
					<div class="bform">
						<input type="text" placeholder="我的域名（例：test.xray.com）" v-model="form.domain">
						<input type="text" placeholder="对外宣称的port" v-model="form.port">
						<input type="text" placeholder="对外宣称的IP地址" v-model="form.address">
						<input type="text" placeholder="对bridge宣称的port" v-model="form.porttobridge">
						<input type="text" placeholder="位于私网的bridge的port" v-model="form.bridgeport">
						<input type="text" placeholder="与bridge通信的密钥，这个密钥必须和bridge手中的一模一样" v-model="form.miyao">
					</div>
					<button class="bbutton" @click="makePortal">配置portal</button>
					
				</div>
				<div class="big-contain" key="bigContainRegister" v-else>
					<div class="btitle">我是位于私网的bridge</div>
					<div class="bform">
						<input type="text" placeholder="我的域名（例：test置.xray.com）" v-model="form.domain">
						<input type="text" placeholder="转发服务器 IP:port（例：127.0.0.1:80）" v-model="form.redirect">
						<input type="text" placeholder="位于公网的portal的IP地址" v-model="form.address">
						<input type="text" placeholder="位于公网的portal的port" v-model="form.port">
						<input type="text" placeholder="与portal通信的密钥，这个密钥必须和portal手中的一模一样" v-model="form.miyao">
					</div>
					<button class="bbutton" @click="register">配置bridge</button>
					
				</div>
			</div>
			<div class="small-box" :class="{active:isPortal}">
				<div class="small-contain" key="smallContainRegister" v-if="isPortal">
					<div class="stitle">你正在配置portal</div>
					<p class="scontent">如果你的机器位于私网，你应该将机器配置为bridge</p>
					<button class="sbutton" @click="changeType">去配置bridge</button>
					<p class="blank">                   </p>
					<button class="guanliPeizhi"  @click="guanliConfigure">管理配置</button>
					<p class="blank">                   </p>
					<button class="guanliUserButton" v-if="isAdmin" @click="guanliUser">管理用户</button>
					<p class="blank"  v-if="isAdmin">                   </p>
					<button class="guanliUserButton"  @click="guanliMianban">管理面板</button>
					<p class="blank">                   </p>
					<button class="logOutButton" @click="logOut">登出</button>
				</div>
				<div class="small-contain" key="smallContainLogin" v-else>
					<div class="stitle">你正在配置bridge</div>
					<p class="scontent">如果你的机器位于公网，你应该将该机器配置为portal</p>
					<button class="sbutton" @click="changeType">去配置portal</button>
					<p class="blank">                   </p>
					<button class="guanliPeizhi"  @click="guanliConfigure">管理配置</button>
					<p class="blank">                   </p>
					<button class="guanliUserButton" v-if="isAdmin" @click="guanliUser">管理用户</button>
					<p class="blank" v-if="isAdmin">                   </p>
					<button class="guanliUserButton"  @click="guanliMianban">管理面板</button>
					<p class="blank">                   </p>
					<button class="logOutButton" @click="logOut">登出</button>
				</div>
			</div>
		</div>
		<!--  -->
		<div class="contain" v-else>
			<div class="big-box" :class="{active:isPortal}">
				<div class="big-contain" key="bigContainLogin2" v-if="isPortal">
					<div class="btitle">你正在登录为管理员</div>
					<div class="bform">
						<input type="text" placeholder="账号" v-model="form.domain">
						<input type="text" placeholder="密码" v-model="form.port">
					</div>
					<button class="bbutton" @click="loginAsAdmin">登录</button>
				</div>
				<div class="big-contain" key="bigContainRegister2" v-else>
					<div class="btitle">你正在登录为用户</div>
					<div class="bform">
						<input type="text" placeholder="账号" v-model="form.domain">
						<input type="text" placeholder="密码" v-model="form.port">
					</div>
					<button class="bbutton" @click="loginAsUser">登录</button>
				</div>
			</div>
			<div class="small-box" :class="{active:isPortal}">
				<div class="small-contain" key="smallContainRegister" v-if="isPortal">
					<!-- <div class="stitle">你正在配置portal</div>
					<p class="scontent">如果你的机器位于私网，你应该将机器配置为bridge</p> -->
					<button class="sbutton" @click="changeType">我是用户</button>
				</div>
				<div class="small-contain" key="smallContainLogin" v-else>
					<!-- <div class="stitle">你正在配置bridge</div>
					<p class="scontent">如果你的机器位于公网，你应该将该机器配置为portal</p> -->
					<button class="sbutton" @click="changeType">我是管理员</button>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	export default{
		name:'login-register',
		data(){
			return {
				isPortal:false,
				emailError: false,
				passwordError: false,
				existed: false,
				hasLogined: false,
				isAdmin: false,
				isGuanliingUser: false,
				isDeleting: false,
				zhengzaiguanliConfigure: false,
				isguanliMianban: false,
				form:{
					domain:'',
					port:'',
					bridgeport:'',
					address:'',
					redirect:'',
					porttobridge:'',
					miyao:''
				},
				items:[],
				configures:[]
			}
		},
		methods:{
			changeType() {
				this.isPortal = !this.isPortal
				this.form.domain = ''
				this.form.port = ''
				this.form.bridgeport = ''
				this.form.address = ''
				this.form.redirect = ''
				this.form.miyao = ''
			},
			gengaiConfigure(configure,index){
				this.Deleting = true;
				this.$axios({
					method:'post',
					url:'http://127.0.0.1:10520/api/user/gengaiConfigure',
					data:{
						i: index,
						conf: configure
					}
				})
				.then(res=>{
					this.$axios({
						method:'post',
						url:'http://127.0.0.1:10520/api/user/getConfigures',
						data:{
						}
					})
					.then(res=>{
						this.configures = res.data;
						console.log(this.configures);
						this.isDeleting=false;
					});
				});
			},
			shanchuConfigure(index){
				 this.isDeleting = true;
				this.$axios({
					method:'post',
					url:'http://127.0.0.1:10520/api/user/shanchuConfigure',
					data:{
						i: index
					}
				})
				.then(res=>{
					this.$axios({
						method:'post',
						url:'http://127.0.0.1:10520/api/user/getConfigures',
						data:{
						}
					})
					.then(res=>{
						this.configures = res.data;
						console.log(this.configures);
						this.isDeleting=false;
					});
				});
			},
			shengchengConfigure(configure){
				if(configure.isPortal){
					this.$axios({
						method:'post',
						url: 'http://127.0.0.1:10520/api/user/makePortal',
						data: {
							domain: configure.domain,
							port: configure.port,
							address: configure.address,
							bridgeport: configure.bridgeport,
							porttobridge: configure.porttobridge,
							miyao: configure.miyao,
							used: true
						}
					})
					.then( res => {
						switch(res.data){
							case 0: 
								alert("portal配置成功！配置文件portal.json已存放在你的D盘根目录下。");
								break;
							case -1:
								this.emailError = true;
								break;
							case 1:
								this.passwordError = true;
								break;
						}
					})
					.catch( err => {
						console.log(err);
					})
				}else{
					this.$axios({
						method:'post',
						url: 'http://127.0.0.1:10520/api/user/makeBridge',
						data: {
							domain: configure.domain,
							port: configure.port,
							address: configure.address,
							redirect: configure.redirect,
							miyao: configure.miyao,
							used: true
						}
					})
					.then( res => {
						switch(res.data){
							case 0:
								alert("bridge配置成功！配置文件bridge.json已存放在你的D盘根目录下。");
								break;
							case -1:
								this.existed = true;
								break;
						}
					})
					.catch( err => {
						console.log(err);
					})
				}
			},
			guanliConfigure(){
				this.zhengzaiguanliConfigure = true;
				this.isDeleting = true;
				this.$axios({
					method:'post',
					url:'http://127.0.0.1:10520/api/user/getConfigures',
					data:{
					}
				})
				.then(res=>{
					this.configures = res.data;
					console.log(this.configures);
					this.isDeleting=false;
				});
			},
			buguanliConfigure(){
				this.zhengzaiguanliConfigure = false;
			},
			changePassword(un, pw){
				this.isDeleting=true;
				this.$axios({
					method:'post',
					url:'http://127.0.0.1:10520/api/user/changePassword',
					data:{
						userName: un,
						password: pw
					}
				})
				.then(res=>{
					this.isDeleting=false;
					this.$axios({
						method:'post',
						url:'http://127.0.0.1:10520/api/user/getUsers',
						data:{}
					})
					.then(res=>{
						this.items = res.data;
					})
				});
			},
			erase(un){
				console.log(typeof un);
				this.isDeleting=true;
				this.$axios({
					method:'post',
					url:'http://127.0.0.1:10520/api/user/eraseUser',
					data:{
						userName: un
					}
				})
				.then(res=>{
					this.isDeleting=false;
					this.$axios({
						method:'post',
						url:'http://127.0.0.1:10520/api/user/getUsers',
						data:{}
					})
					.then(res=>{
						this.items = res.data;
					})
				});
				
				
			},
			clean(){
				this.form.domain = ''
				this.form.port = ''
				this.form.bridgeport = ''
				this.form.address = ''
				this.form.redirect = ''
				this.form.miyao = ''
			},
			xiugaiAdmin(){
				this.$axios({
					method:'post',
					url:'http://127.0.0.1:10520/api/user/changeAdmin',
					data:{
						userName: this.form.address,
						password: this.form.miyao
					}
				})
				.then(res=>{
					switch(res.data){
						case 0:
							alert("修改成功");
							break;
						case -1:
							alert("修改失败？");
							break;
					}
				})
			},
			tianJIaUser(){
				this.$axios({
					method:'post',
					url:'http://127.0.0.1:10520/api/user/addUser',
					data:{
						userName: this.form.domain,
						password: this.form.port
					}
				})
				.then(res=>{
					switch(res.data){
						case 0:
							alert("添加成功");
							break;
						case -1:
							alert("添加失败：已经有此用户");
							break;
					}
					this.$axios({
						method:'post',
						url:'http://127.0.0.1:10520/api/user/getUsers',
						data:{}
					})
					.then(res=>{
						this.items = res.data;
					})
				});
			},
			buGuanliUser(){this.form.domain = ''
				this.form.port = ''
				this.form.bridgeport = ''
				this.form.address = ''
				this.form.redirect = ''
				this.isGuanliingUser = false;this.form.miyao = ''
			},
			guanliUser(){this.form.domain = ''
				this.form.port = ''
				this.form.bridgeport = ''
				this.form.address = ''
				this.form.redirect = ''
				this.isGuanliingUser = true;this.form.miyao = ''
				this.$axios({
					method:'post',
					url:'http://127.0.0.1:10520/api/user/getUsers',
					data:{
					}
				})
				.then(res=>{
					this.items = res.data;
					console.log(this.items);
					console.log(res.data);
				})
			},
			buguanliMianban(){this.form.domain = ''
				this.form.port = ''
				this.form.bridgeport = ''
				this.form.address = ''
				this.form.redirect = ''
				this.isguanliMianban = false
				this.isGuanliingUser = '';this.form.miyao = ''
			},
			guanliMianban(){this.form.domain = ''
				this.form.port = ''
				this.form.bridgeport = ''
				this.form.address = ''
				this.form.redirect = ''
				this.isguanliMianban = true
				this.isGuanliingUser = '';this.form.miyao = ''
			},
			
			saveMianban(){
				this.$axios({
					method:'post',
					url:'http://127.0.0.1:10520/api/user/saveMianban',
					data:{
						ip:this.form.domain,
						port:this.form.port,
						publickey:this.form.address,
						secertkey:this.form.porttobridge,
						uel:this.form.miyao
						
					}
				})
			},
			logOut(){
				this.form.domain = ''
				this.form.port = ''
				this.form.bridgeport = ''
				this.form.address = ''
				this.form.redirect = ''
				this.form.miyao = ''
				this.hasLogined = false;
				this.isAdmin = false;
			},
			loginAsUser(){
				this.$axios({
					method:'post',
					url: 'http://127.0.0.1:10520/api/user/login',
					data:{
						userName: this.form.domain,
						password: this.form.port,
						isAdmin: false
					}
				})
				.then(res=>{
					switch(res.data){
						case 0:
							alert("登录成功");
							this.form.domain = ''
							this.form.port = ''
							this.form.bridgeport = ''
							this.form.address = ''
							this.form.redirect = ''
							this.hasLogined = true;
							break;
						case -1:
							alert("用户名或密码错误");
							break;
					}
				})
				.catch(err=>{
					console.log(err)
				})
			},
			loginAsAdmin(){
				this.$axios({
					method:'post',
					url: 'http://127.0.0.1:10520/api/user/login',
					data:{
						userName: this.form.domain,
						password: this.form.port,
						isAdmin: true
					}
				})
				.then(res=>{
					switch(res.data){
						case 0:
							alert("登录成功");
							this.form.domain = ''
							this.form.port = ''
							this.form.bridgeport = ''
							this.form.address = ''
							this.form.redirect = ''
							this.hasLogined = true;
							this.isAdmin = true;
							break;
						case -1:
							alert("用户名或密码错误");
							break;
					}
				})
				.catch(err=>{
					console.log(err)
				})
			},
			makePortal() {
				// 配置portal
				const self = this;
				if (true) {
					self.$axios({
						method:'post',
						url: 'http://127.0.0.1:10520/api/user/makePortal',
						data: {
							domain: self.form.domain,
							port: self.form.port,
							address: self.form.address,
							bridgeport: self.form.bridgeport,
							porttobridge: self.form.porttobridge,
							miyao: self.form.miyao,
							used: false
						}
					})
					.then( res => {
						switch(res.data){
							case 0: 
								alert("portal配置成功！配置文件portal.json已存放在你的D盘根目录下。");
								break;
							case -1:
								this.emailError = true;
								break;
							case 1:
								this.passwordError = true;
								break;
						}
					})
					.catch( err => {
						console.log(err);
					})
				} else{
					alert("填写不能为空！");
				}
			},
			register(){
				// 配置bridge
				const self = this;
				
				if(true){
					console.log(this);
					self.$axios({
						method:'post',
						url: 'http://127.0.0.1:10520/api/user/makeBridge',
						data: {
							domain: self.form.domain,
							port: self.form.port,
							address: self.form.address,
							redirect: self.form.redirect,
							miyao: self.form.miyao,
							used: false
						}
					})
					.then( res => {
						switch(res.data){
							case 0:
								alert("bridge配置成功！配置文件bridge.json已存放在你的D盘根目录下。");
								break;
							case -1:
								this.existed = true;
								break;
						}
					})
					.catch( err => {
						console.log(err);
					})
				} else {
					alert("填写不能为空！");
				}
			}
		}
	}
</script>

<style scoped="scoped">
	.login-register{
		width: 100vw;
		height: 100vh;
		box-sizing: border-box;
	}
	.containnamelist{
		color: rgb(255, 255, 255);
		width: 80%;
		height: 65%;
		position: relative;
		top: 40%;
		left: 50%;
		display:inline-flex;
		flex-direction: column;
		padding:1%;
		transform: translate(-50%,-50%);
		background: linear-gradient(315deg,rgb(1, 115, 255),rgb(40, 2, 106));
		border-radius: 20px;
		box-shadow: 0 0 3px #ff00888f,
					0 0 6px #ffffff;
	}
	.containv{
		width: 80%;
		height: 90%;
		position: relative;
		top: 50%;
		left: 50%;
		padding: 1%;
		flex-direction: column;
		transform: translate(-50%,-50%);
		background: linear-gradient(135deg,rgb(135, 16, 125),rgb(82, 7, 210));
		border-radius: 20px;
		box-shadow: 0 0 3px #ff00888f,
					0 0 6px #ffffff;
	}
	.contain{
		width: 80%;
		height: 90%;
		position: relative;
		top: 50%;
		left: 50%;
		
		transform: translate(-50%,-50%);
		background: linear-gradient(135deg,rgb(135, 16, 125),rgb(82, 7, 210));
		border-radius: 20px;
		box-shadow: 0 0 3px #ff00888f,
					0 0 6px #ffffff;
	}
	.peizhiList{
		color:rgb(255, 204, 0);
		font-size: larger;
		font-style: unset;
		
	}
	.big-box{
		width: 70%;
		height: 150%;
		position: relative;
		top: -25%;
		left: 30%;
		transform: translateX(0%); 
		transition: all 1s;
	}
	.big-contain{
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
	.btitle{
		font-size: 1.5em;
		font-weight: bold;
		color: rgb(230, 156, 249);
	}
	.bform{
		width: 100%;
		height: 40%;
		padding: 2em 0;
		display: flex;
		flex-direction: column;
		justify-content: space-around;
		align-items: center;
	}
	.bform .errTips{
		display: block;
		width: 50%;
		text-align: left;
		color: red;
		font-size: 0.7em;
		margin-left: 1em;
	}
	.bform input{
		width: 50%;
		height: 30px;
		border: none;
		outline: none;
		border-radius: 10px;
		padding-left: 2em;
		background-color: #f0f0f0;
	}
	.bbutton{
		width: 20%;
		height: 40px;
		border-radius: 100px;
		border: none;
		outline: none;
		background: linear-gradient(135deg,rgb(190, 7, 210),rgb(135, 16, 85));
		color: rgb(255, 255, 255);
		font-size: 1.1em;
		cursor: pointer;
	}
	.operateConfigureButton{
		width: 10%;
		height: 20px;
		border-radius: 100px;
		border: none;
		outline: rgb(255, 255, 255);;
		background-color: rgb(218, 233, 247);;
		color: rgb(14, 13, 13);
		font-size: 0.5em;
		cursor: pointer;
	}
	.changbutton{
		width: 10%;
		height: 40px;
		border-radius: 0px;
		border: none;
		outline: rgb(255, 255, 255);;
		background-color: transparent;;

		color: rgb(255, 255, 255);
		font-size: 1.1em;
		cursor: pointer;
	}
	
	.fanhuiButton{
		width: 20%;
		height: 40px;
		border-radius: 24px;
		border: 1px solid #fff;
		outline: none;
		background-color: transparent;
		color: #fff;
		font-size: 0.9em;
		cursor: pointer;
	}
	.guanliPeizhi{
		width: 60%;
		height: 40px;
		border-radius: 24px;
		border: 1px solid #fff;
		outline: none;
		background-color: transparent;
		color: #fff;
		font-size: 0.9em;
		cursor: pointer;
	}
	.guanliUserButton{
		width: 60%;
		height: 40px;
		border-radius: 24px;
		border: 1px solid #fff;
		outline: none;
		background-color: transparent;
		color: #fff;
		font-size: 0.9em;
		cursor: pointer;
	}
	.logOutButton{
		width: 60%;
		height: 40px;
		border-radius: 24px;
		border: 1px solid #fff;
		outline: none;
		background-color: transparent;
		color: #fff;
		font-size: 0.9em;
		cursor: pointer;
	}
	.small-box{
		width: 30%;
		height: 100%;
		background: linear-gradient(45deg,rgb(40, 13, 85),rgb(61, 1, 56));
		position: absolute;
		top: 0;
		left: 0;
		transform: translateX(0%);
		transition: all 1s;
		border-top-left-radius: inherit;
		border-bottom-left-radius: inherit;
	}
	.small-contain{
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
	.stitle{
		font-size: 1.5em;
		font-weight: bold;
		color: #fff;
	}
	.scontent{
		font-size: 0.8em;
		color: #fff;
		text-align: center;
		padding: 2em 4em;
		line-height: 1.7em;
	}
	.blank{
		font-size: 0.3em;
		color: #fff;
		text-align: center;
		padding: 2em 4em;
		line-height: 0.3em;
	}
	.sbutton{
		width: 60%;
		height: 40px;
		border-radius: 24px;
		border: 1px solid #fff;
		outline: none;
		background-color: transparent;
		color: #fff;
		font-size: 0.9em;
		cursor: pointer;
	}
	
	.big-box.active{
		left: 0;
		transition: all 0.5s;
	}
	.small-box.active{
		left: 100%;
		border-top-left-radius: 0;
		border-bottom-left-radius: 0;
		border-top-right-radius: inherit;
		border-bottom-right-radius: inherit;
		/* background: linear-gradient(45deg,rgb(255, 255, 255),rgb(98, 0, 255)); */
		transform: translateX(-100%);
		 transition: all 1s; 
	}
</style>
