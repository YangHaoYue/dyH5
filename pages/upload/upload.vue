<template>
	<view>
		<view class="card">
			<view style="text-align: center;">
				<image src="../../static/localPrint.png" class="img"></image>
			</view>
			<view class="flex">
				<button id="fileBtn" class="flex-sub cu-btn shadow line-yellow button" @click="choose_file(1)">合并</button>
				<button id="fileBtn" class="flex-sub cu-btn shadow line-yellow button" @click="choose_file(2)">上传</button>
			</view>
			<view v-html='ipt'></view>
		</view>
		
		<view style="text-align: center;font-weight: 600;color: #F43F3B;">
			提示：点击文档名称右侧的三个点设置需求
		</view>
		
		<view v-show="progress" style="margin: 0upx 5%;">
			<view class="flex margin-top">
				<view class="cu-progress round shadow">
					<view class="bg-yellow" :style="[{ width:chunks!='0%'?this.chunks:''}]"></view>
				</view>
				<text class="margin-left">{{chunks}}</text>
			</view>
			<view style="text-align: center;">{{filestatus}}</view>
		</view>
		
		<view v-show="files!=''||countCombineFiles!=''">
			<view class="file flex" style="font-weight: 600;">
				<view class="flex-sub"></view>
				<view class="fileName flex-treble">文件名称</view>
				<view class="paper flex-twice">页数</view>
				<view class="price flex-twice">价格/元</view>
				<view v-if="false" class="flex-sub size cuIcon-add text-yellow" @tap="addCombineFile()"></view>
			</view>
			<vuedraggable class="wrapper" v-model="files" group="description" handle=".fileName" animation="0">
			    <transition-group class="cu-list menu-avatar">
			      	<view v-for="(item,index) in files" :key="index" class="cu-item file flex" style="height: 80upx;">
						<view class="flex-sub" v-show="item.name.slice(-3)=='doc'||item.name.slice(-4)=='docx'">
							<image src="../../static/icon/word.png" class="iconsize"></image>
						</view>
						<view class="flex-sub" v-show="item.name.slice(-3)=='ppt'||item.name.slice(-4)=='pptx'">
							<image src="../../static/icon/PPT.png" class="iconsize"></image>
						</view>
						<view class="flex-sub" v-show="item.name.slice(-3)=='jpg'||item.name.slice(-4)=='jpeg'||item.name.slice(-3)=='png'||item.name.slice(-3)=='bmp'||item.name.slice(-3)=='gif'">
							<image src="../../static/icon/PIC.png" class="iconsize"></image>
						</view>
						<view class="flex-sub" v-show="item.name.slice(-3)=='pdf'">
							<image src="../../static/icon/PDF.png" class="iconsize"></image>
						</view>
			      		<view class="fileName flex-treble">{{item.name}}</view>
			      		<view class="paper flex-twice">{{item.paper}}</view>
			      		<view class="price flex-twice">{{showPrice(item.price)}}</view>
			      		<view class="cuIcon-moreandroid size flex-sub text-black" @tap="tapFiletext($event,1,index)"></view>
			      	</view>
			    </transition-group>
			</vuedraggable>
			
			<view v-for="(item,id) in countCombineFiles" :key="id" class="combineCard">
				<view class="file flex" style="font-weight: 600;">
					<view class="flex-sub"></view>
					<view class="fileName flex-treble">合并列表</view>
					<view class="price flex-twice" style="color: #007AFF;" v-html="'￥'+(parseFloat(item[0].price/100,2)).toFixed(2)"></view>
					<view class="cuIcon-add size flex-sub text-red" @tap="choose_combinfile(id)"></view>
				</view>
				<vuedraggable class="wrapper" v-model="countCombineFiles[id]" group="description" handle=".fileName" animation="0">
					<transition-group class="cu-list menu-avatar">
						<view v-for="(row,index) in item" :key="index" class="cu-item file flex" style="height: 80upx;">
							<view class="flex-sub" v-show="row.name.slice(-3)=='doc'||row.name.slice(-4)=='docx'">
								<image src="../../static/icon/word.png" class="iconsize"></image>
							</view>
							<view class="flex-sub" v-show="row.name.slice(-3)=='ppt'||row.name.slice(-4)=='pptx'">
								<image src="../../static/icon/PPT.png" class="iconsize"></image>
							</view>
							<view class="flex-sub" v-show="row.name.slice(-3)=='jpg'||row.name.slice(-4)=='jpeg'||row.name.slice(-3)=='png'||row.name.slice(-3)=='bmp'||row.name.slice(-3)=='gif'">
								<image src="../../static/icon/PIC.png" class="iconsize"></image>
							</view>
							<view class="flex-sub" v-show="row.name.slice(-3)=='pdf'">
								<image src="../../static/icon/PDF.png" class="iconsize"></image>
							</view>
							<view class="flex-treble"  :class="row.paper==0?'title':'fileName'">{{row.name}}</view>
							<view class="paper flex-twice" v-show="row.paper!=0">{{row.paper}}页</view>
							
							<view class="cuIcon-moreandroid size flex-sub text-black" v-show="row.paper!=0" @tap="tapCombineFiletext($event,2,id,index)"></view>
						</view>
					</transition-group>
				</vuedraggable>
			</view>
			<textarea maxlength="500" v-model="textarea"  @input="textareaAInput" placeholder="备注" @focus="textareaClassChange()" @blur="textareaClassChange()" class="card" :class="textareaClass?'textarea':''" style="height: 200upx;padding: 10upx;"></textarea>
			<view class="card flex" style="height: 100upx;">
				<view class="justify-between" style="font-size: 32upx;font-weight: 600;margin: auto 10upx;">共{{countFile}}个文件，共计{{countPrice}}元</view>
				<button class="cu-btn shadow bg-yellow justify-end" data-target="DialogModal1" style="margin: auto;" @tap="submit">提交</button>
			</view>
		</view>
		<!-- 默认文件菜单 -->
		<chunLei-popups v-model="value1" :popData="data1" @tapPopup="tapPopup" :x="x" :y="y" direction="row" theme="dark" placement="bottom-end" dynamic>
		</chunLei-popups>
		<chunLei-popups v-model="value2" :popData="data2" @tapPopup="tapPopup" :x="x" :y="y" direction="row" theme="dark" placement="bottom-end" dynamic>
		</chunLei-popups>
		
		<view class="cu-modal" :class="modalName=='DialogModal1'?'show':''||modalName=='DialogModal2'?'show':''||modalName=='DialogModal3'?'show':''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">选择打印格式</view>
				</view>
				<view class="padding-xl">
					<view class="Radio-h5">
						<view class="flex" v-for="(item,index) in option" :key="index" style="margin: 40upx auto 20upx;">
							<view class="title">{{item.group_name}}</view>
							<view  class="select">
								<select v-model="optionChoise[index]"  name="" id="">
									<option v-for="(row,id) in item.options"  :key="id"  v-if="is_disable(row.id,index)" :value="row.id"  v-text="row.option_name"></option>
								</select>
							</view>
						</view>
					</view>
					
					<view class="Radio-h5">
						<view  class="flex" v-for="(item,i) in attach" :key="i">
							<view class="title">{{item.group_name}}</view>
							<view  class="select">
								<select v-model="attachChoise[i]"  name="" id="">
									<option v-for="(it,j) in item.options" :key="j" :value="it.id"  v-text="it.option_name"></option>
								</select>
							</view>
						</view>
					</view>

					<view class="flex Radio-h5">
						<view class="title">版数:</view>
						<input v-model="Editionnumber" type="number" class="input" maxlength="10"/>
					</view>
					<view class="flex Radio-h5">
						<view class="title">打印份数:</view>
						<input v-model="copies" type="number" class="input"/>
					</view>
				</view>
				<view class="cu-bar bg-white justify-end">
					<view class="action">
						<button class="cu-btn line-yellow text-yellow" @tap="hideModal">取消</button>
						<button v-show="modalName=='DialogModal2'" class="cu-btn bg-yellow margin-left" @tap="combineSetSynchronization">一键同步</button>
						<!-- <button v-show="modalName=='DialogModal1'" class="cu-btn bg-yellow margin-left" @tap="start_upload">确定上传</button> -->
						<button v-show="modalName=='DialogModal2'" class="cu-btn bg-yellow margin-left" @tap="set">确定</button>
						<button v-show="modalName=='DialogModal3'" class="cu-btn bg-yellow margin-left" @tap="combineSet">确定</button>
					</view>
				</view>
			</view>
		</view>
		<view class="cu-modal" :class="modalName=='Image'?'show':''">
			<view class="cu-dialog lines-yellow" style="border: #fbbd08 solid 5upx;border-radius: 10uxp;">
				<view class="bg-img">
					<view class="cu-bar justify-center text-yellow">
						{{msg}}
					</view>
				</view>
			</view>
		</view>
	</view>
</template>
<script>
	import upUrl from'../../common/url.js'
	import chunLeiPopups from "@/components/chunLei-popups/chunLei-popups.vue";
	import vuedraggable from 'vuedraggable';
	const wx=require('weixin-js-sdk')
	/* var VConsole = require('../../node_modules/vconsole/dist/vconsole.min.js');
	var vConsole = new VConsole(); */
	export default {
		components:{
			vuedraggable,
			chunLeiPopups
		},
		updated() {
			for(let i=0;i<this.countCombineFiles.length;i++){
				for(let j=0;j<this.countCombineFiles[i].length;j++){
					if(this.countCombineFiles[i][j].paper==0&&this.countCombineFiles[i].length>1){
						this.countCombineFiles[i].splice(j,1);
						this.computeCountprice();
						this.computeCountfile();
					}
				}
				if(this.countCombineFiles[i].length==0){
					this.countCombineFiles.splice(i,1);
					this.computeCountprice();
					this.computeCountfile();
				}
			}
		},
		data() {
			return {
				msg:'',
				
				width:'',
				user_id:'',
				order_id:'',
				token:'',
				isagain:'',
				groupid:'',
				
				modalName:"",
				textareaAValue:null,
				textareaClass:false,
				textarea:'',
				listTouchStart: 0,
				listTouchDirection: null,
				
				//下拉菜单
				value1:false,
				value2:false,
				x:0,
				y:0,
				data1:[{title:'设置需求',id:1},{title:'删除',id:2}],
				data2:[{title:'设置需求',id:3},{title:'删除',id:4}],
				
				fileTitle:'',
				optionChoise:[],
				attachChoise:[],
				lines:{},
				
				option:[],
				attach:[],
				price_list:{},
				copies:1,
				Editionnumber:1,
				inputTitle:false,
				
				price:'',
				countPrice:0,
				countFile:0,
				AttachPrice:{},
				whichBtn:0,
				//储存修改的哪一个文件
				whichfile:0,
				whichAttach:[],
				whichcombinefile:[0,0],
				whichinputcombinefile:0,
				
				files:[],
				combineFile:[{
					name:'请将上面的文件拖入！',
					copies:0,
					paper:0,
					price:0
				}],
				Temporaryfile:[],
				countCombineFiles:[],
				
				
				filename:"待选择",
				filesize:0,
				filestatus:"待上传",
				file:{},
				file_lists:[],
				ipt:"<input type='file' style='display:none' id='file'  multiple='multiple' onchange='file_c()'>",
				
				chunks:'0%',
				progress:false,
				
				
			// 对应需要显示的标题
			      props: {
			        label: 'name'
			      }
			}
		},
		onReady(e) {
			this.user_id= this.GetQueryString('userid');
			this.order_id =this.GetQueryString('orderid');
			this.token=this.GetQueryString('token');
			this.isagain=this.GetQueryString('isagain');
			this.groupid=this.GetQueryString('groupid');
			
			/* this.user_id= 9;
			this.order_id =202101011482231;
			this.token="Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiI3IiwianRpIjoiZjk1YWM3YjExNWFiMzYwZjRjNjI4NjJhZTM4NzI1NjRjNGE5NTU2YjE4M2ZiNThjYjYwYWM0N2M5N2QxYTRiY2I5MDkzZjM4MmQ0NjVjM2EiLCJpYXQiOjE2MDk0ODIwODAsIm5iZiI6MTYwOTQ4MjA4MCwiZXhwIjoxNjQxMDE4MDgwLCJzdWIiOiI5Iiwic2NvcGVzIjpbXX0.yQuC963n08LxMorvolB5z-tZdqZhEeEQmex9nWtYVs02MI4AqbjLo9P_YQTf04ZI5hYdYfYjSZSOMu5FoADV67CtS4vqMb991qUnzy0qNAURSMe8e_F5HZXrpAVyhCYD0CRW74Dw7UL4gZthIU_rsur9QuJKUQptcSebRuHzIRoPka9XPz7IiDVEoeEwRNawOv-Ziy3SluG9f6c2qqMfoRb6OGvFnGJnd7RDIClmg9QUYSMDwB0-PESLTujVoQwIu7q4i8Uq-ubeuaa06yImGrIivwS5FUEjV-CZFP08pakm_sZ6X2B-9bXoqARsKkOMcDRl9vS5LN-MW1DPiBg4bz4gu0rO4CE9M2BCC7OlBJ-m4qwHQsQXZxlsOJX4BjC7wK1XbCP1bDf4cVr7C4AayblZm_emgHjvsCeo-YT9wWNsXUjIwo9NwBULcD24-wzRY3BnomXsP6cuezDPUMhfSVa0dzo9pqMMfS_ZOGoz8iWRL2lZ08whPkVNVgP5FQDPjHJr1KcQQOomMAAxQ17EzlWk4h6olf1tgGu5Xn2x5JoSbGUIxaI08DBEtMlwpUQtwKIpnCDEdiRgmSNyiGSsjG7VPuZRNlGxcHDx9XvBMKskoNnThbUQDTe29CHbd5IAdgrXTP7P0Ag5wfCuogih2eeZLTZ2n_2rEp1oueXVl0o";
			this.isagain="no";
			this.groupid=57; */
			this.getinfo();
		},
		//监听滚动隐藏
		onPageScroll(){
			for (let i = 0; i < 6; i++) {
				this[`value${i}`] = false
			}
		},
		methods: {
			getwidth(){
				uni.getSystemInfo({
					success: (res) => {
						this.width = parseInt(res.screenWidth*0.9);
					}
				})
			},
			showPrice(p){
				return parseFloat((p/100).toFixed(2));
			},
			
			is_disable(id,index){
				return this.is_ext(index,id);
			},
			is_ext(ii,id){
				console.log('option:'+this.optionChoise[0])
				let current = this.optionChoise[0];
				let t = this.lines[current];

				let arr = [current];
				if(ii == 0)
				{
					if(!this.lines.hasOwnProperty(id)){
						return false;
					}
					return true;
				}
				else if(ii == 1)
					arr.push(id);
				else if(ii == 2){
					arr.push(this.optionChoise[1]);
					arr.push(id);
				}else if(ii == 3){
					arr.push(this.optionChoise[1]);
					arr.push(this.optionChoise[2]);
					arr.push(id);
				}
				for (let i = 0; i < t.length; i++) {
					let stre= arr.join(',');
					console.log(stre);
					if( t[i].options_ids.indexOf(stre)>-1)
						return true;

				}
				return false;
				
			}
			,
			
			AttachChange(item,$event){
				this.$set(this.attachChoise,item,$event.target.value);
				console.log(this.attachChoise);
			
			},
			//获取当前url地址上的值
			GetQueryString(name){
			     var reg = new RegExp("(^|&)"+ name +"=([^&]*)(&|$)");
			     var r = window.location.search.substr(1).match(reg);
			     if(r!=null){
					 return  unescape(r[2]); 
				 }
				 return null;
			},
			getinfo(){
				//获取选项卡组price
				uni.request({
					url: upUrl.websiteUrl+'options/getLines',
					method: 'GET',
					success: res => {
						let first = -1;
						let _this = this;
						
						if(res.data.code==200){
							res.data.data.forEach(function(item){
								if(first == -1&&item.type_id==_this.groupid)
									{
									first = item.type_id;
									_this.lines[item.type_id] = [];
									
									}
									if(!_this.lines.hasOwnProperty(item.type_id)){
										_this.lines[item.type_id] = [];
								
									}
									_this.lines[item.type_id].push(item);
									_this.price_list[item.options_ids] = parseInt(item.price);
								
							});
							console.log(_this.lines);
							
							uni.request({
								url: upUrl.websiteUrl+'options/get_options',
								method: 'GET',
								data:{
									group_id:this.groupid
								},
								success: res => {
									if(res.data.code==200){
										if(_this.lines[first].length>0)
											{
												_this.optionChoise = _this.lines[first][0].options_ids.split(',');
												console.log(_this.optionChoise);
											}
										for(let j=0;j<res.data.data.option.length;j++){
											if(res.data.data.option[j].id == 10){
												res.data.data.option[j].options.forEach(function(item){
													if(item.attach)
													_this.AttachPrice[item.id] = JSON.parse(item.attach);
										
												});
												
											}
											_this.option.push(_this._groupOptionFormat(res.data.data.option[j],j))
										}
										console.log(_this.AttachPrice);
										for(let i=0;i<res.data.data.attach.length;i++){
											_this.attach.push(_this._groupAttachFormat(res.data.data.attach[i],i))
										}
									}
								}
							});
						}
					}
				});
				//获取选项卡组

				
				//判断是否再次下单
				if(this.isagain=='again'){
					let token=uni.getStorageSync('token')
					uni.request({
						url:upUrl.websiteUrl + 'orders/order_again',
						method: 'GET',
						header: {
							'Authorization': this.token
						},
						data: {
							order_id:this.order_id,
							user_id:this.user_id
						},
						success: res => {
							if(res.data.code==200){
								this.order_id=res.data.data.order_id
								let files=res.data.data;
								for(let i=0;i<files.normal.length;i++){
									this.files.push(this._formatfiles(files.normal[i]))
								}
								for(let i=0;i<files.combine.length;i++){
									let combineFile=[];
									for(let j=0;j<files.combine[i].length;j++){
										combineFile.push(this._formatfiles(files.combine[i][j]))
									}
									this.countCombineFiles.push(combineFile)
								}
								if(files.memo!=null){
									this.textarea=files.memo
								}
								this.computeCountprice();
								this.computeCountfile();
								this.combineSet();
							}
							if(res.data.code==500){
								this.msg=res.data.msg;
								this.modalName="Image";
								setTimeout(()=>{
									wx.miniProgram.navigateBack({										delta:1									})
								},3000)
							}
						}
					});
				}
			},
			_groupOptionFormat(e,j){
				let temOptions=[];
				
				for(let i=0;i<e.options.length;i++){
					
					temOptions.push(this._optionsFormat(e.options[i]))
				}
				return{
					id:e.id,
					group_name:e.group_name,
					options:temOptions
				}
			},
			_optionsFormat(e){
				return{
					id:e.id,
					option_name:e.option_name,
					price:e.price
				}
			},
			_groupAttachFormat(e,j){
				let temOptions=[];
				for(let i=0;i<e.options.length;i++){
					if(i==0){
						this.attachChoise.push(e.options[0].id);
					}
					temOptions.push(this._optionsFormat(e.options[i]))
				}
				return{
					id:e.id,
					group_name:e.group_name,
					options:temOptions
				}
			},
			
			hideModal(e) {
				this.modalName = null
				this.startSet();
			},
			//打印选项
			RadioOptionChange(index,id) {
				this.$set(this.optionChoise,index,id)
			},
			//下拉菜单
			tapPopup(e){
				if(e.id==1){
					this.setting(this.whichfile);
				}else if(e.id==2){
					this.delet(this.whichfile)
				}else if(e.id==3){
					let index=this.whichcombinefile[1];
					let id=this.whichcombinefile[0];
					this.combineSetting(id,index)
				}else if(e.id==4){
					let index=this.whichcombinefile[1];
					let id=this.whichcombinefile[0];
					this.combineDelte(id,index)
				}
			},
			tapFiletext(e,res,index){
				this.whichfile=index;
				this.x = e.touches[0].clientX
				this.y = e.touches[0].clientY
				
				this[`value${res}`] = !this[`value${res}`]
			},
			tapCombineFiletext(e,res,id,index){
				this.whichcombinefile[0]=id;
				this.whichcombinefile[1]=index;
				
				this.x = e.touches[0].clientX
				this.y = e.touches[0].clientY
				
				this[`value${res}`] = !this[`value${res}`]
			},
			tapOut(e,index){
				let dom = uni.createSelectorQuery().in(this)
				dom.select("#text").boundingClientRect()
				dom.exec((data) => {
					this.x = (data[0].left+data[0].right)/2
					this.y = data[0].top
					this[`value${index}`] = !this[`value${index}`]
				})
			},
			
			//计算价格 bs 板数  copies分数
			compute(option,attach,pages,bs,copies){
				//首先判断是否为单页打印
				let Price = 0;
				const SinglePage_Id = 28, DoublePage_Id = 25;
				let ids = option.join(",");
				pages = parseInt(pages);
				copies = parseInt(copies);
				bs = parseInt(bs);
				if(option.indexOf(SinglePage_Id) > -1){
					//单面打印,获得这个价格
					Price =parseInt(this.price_list[ids]) * pages;
				}else{
					let S_ids = "";
					let Dp = 0,Sp = 0;
					if(pages % 2 == 0){
					
						Price = parseInt(this.price_list[ids]) * pages;
						
					}else{
						let tempOption = option.join(",");
						S_ids = tempOption.replace(new RegExp(DoublePage_Id.toString(),'g'),SinglePage_Id.toString());
						if(!this.price_list.hasOwnProperty(S_ids)){
							uni.showToast({
								icon:'none',
								title:'请在后台设置该属性的单面'
							})
							return 0;
						}
						Sp = 1;
						Dp = pages - 1;
						let Sp_price= parseInt(this.price_list[S_ids]) * Sp;
						Price = parseInt(this.price_list[ids]) * Dp + Sp_price;
					}

					
				}
				
				Price = Price / bs;
				let _this = this;
				console.log(this.AttachPrice)
				console.log(option)
				console.log(this.AttachPrice[option[0]])
				console.log(attach)
				attach.forEach(function(item){
					Price += parseInt(_this.AttachPrice[option[0]][item].price);
				});	
				Price *= copies
				console.log("File Price"+ Price);
				return Price;
			},
			//计算总价
			computeCountprice(){
				let filePrice=0;
				for(let i=0;i<this.files.length;i++){
					
					
					filePrice+=parseInt(this.files[i].price);
				}
				let countCombineFilesPrice=0,temp_Price = 0;
				
				for(let i=0;i<this.countCombineFiles.length;i++){
					let paper=0,option_temp = [],attach_temp = [],bs = 0,copies_temp = 0;
					
					for(let j=0;j<this.countCombineFiles[i].length;j++){
						if(j == 0)
						{
							option_temp = this.countCombineFiles[i][0].optionChoise;
							attach_temp =  this.countCombineFiles[i][0].attachChoise;
							bs =  this.countCombineFiles[i][0].Editionnumber;
							copies_temp =  this.countCombineFiles[i][0].copies;
						}
						paper+=this.countCombineFiles[i][j].paper;
					}
					temp_Price = 0;
					temp_Price = this.compute(option_temp,attach_temp,paper,bs,copies_temp);
					for(let j=0;j<this.countCombineFiles[i].length;j++){
						this.countCombineFiles[i][j].price = temp_Price;
					}
					
					this.countCombineFiles[i][0].price  = temp_Price;
					countCombineFilesPrice += temp_Price;
				
				}
				this.countPrice=parseFloat(((filePrice+countCombineFilesPrice)/100).toFixed(2));
			},
			//计算文件总个数
			computeCountfile(){
				let CountCombineFiles=0;
				for(let i=0;i<this.countCombineFiles.length;i++){
					for(let j=0;j<this.countCombineFiles[i].length;j++){
						CountCombineFiles++;
					}
				}
				let Files=this.files.length;
				this.countFile=CountCombineFiles+Files;
			},
			
			//合并文件分组
			addCombineFile(){
				this.countCombineFiles.push(this.combineFile);
			},
			
			//备注的文本框样式改变
			textareaAInput(e) {
				this.textareaAValue = e.detail.value
			},
			textareaClassChange(){
				this.textareaClass=!this.textareaClass;
			},
			
			// ListTouch触摸开始
			ListTouchStart(e) {
				this.listTouchStart = e.touches[0].pageX
			},
			
			// ListTouch计算方向
			ListTouchMove(e) {
				this.listTouchDirection = e.touches[0].pageX - this.listTouchStart > 0 ? 'right' : 'left'
			},
			
			// ListTouch计算滚动
			ListTouchEnd(e) {
				if (this.listTouchDirection == 'left') {
					this.modalName = e.currentTarget.dataset.target
				} else {
					this.modalName = null
				}
				this.listTouchDirection = null
			},
			//重置
			startSet(){
				this.fileTitle='';
				this.attachChoise=[];
				this.optionChoise=[];
				this.copies=1;
				this.Editionnumber=1;
				this.inputTitle=false;
				let _this = this,first = true;
				console.log("line"+JSON.stringify(this.lines))
				for (let line in this.lines) {
					
						if(first){
							_this.optionChoise = this.lines[this.groupid][0].options_ids.split(',');
							first =!false;
						}
				}

				for(let i=0;i<this.attach.length;i++){
					this.attachChoise.push(this.attach[i].options[0].id);
				}
			},
			//重新设置默认文件打印方式
			setting(index){
				this.modalName = 'DialogModal2';
				
				for(let i=0;i<this.files[index].optionChoise.length;i++){
					this.optionChoise[i]=this.files[index].optionChoise[i];
				}
				
				for(let i=0;i<this.files[index].attachChoise.length;i++){
					this.attachChoise[i]=this.files[index].attachChoise[i];
					
					
				}
				this.copies=this.files[index].copies;
				this.Editionnumber=this.files[index].Editionnumber;
			},
			set(){
				if(this.copies==0){
					/* uni.showToast({
						icon:'none',
						title: '打印份数不能为0！'
					}); */
					this.copies=1;
					return;
				}
				if(this.Editionnumber==0){
					/* uni.showToast({
						icon:'none',
						title: '打印份数不能为0！'
					}); */
					this.Editionnumber=1;
					return;
				}
				let  _this=this;
				let index=this.whichfile;
				this.files[index].optionChoise = [];
				this.optionChoise.forEach(function(item){
					_this.files[index].optionChoise.push(item);
			
				})
					
				this.files[index].attachChoise = [];
				this.attachChoise.forEach(function(item){
					_this.files[index].attachChoise.push(item);
		
				})
		
				this.files[index].copies= this.copies;
				this.files[index].fileTitle= this.fileTitle;
				this.files[index].Editionnumber= this.Editionnumber;
				this.files[index].price=this.compute(this.optionChoise,this.attachChoise,this.files[index].paper,this.Editionnumber,this.copies);
				this.computeCountprice();
				this.hideModal();
			},
			//重新设置合并文件打印方式
			combineSetting(id,index){
				this.modalName = 'DialogModal3';
				this.optionChoise = this.countCombineFiles[id][index].optionChoise;
				this.attachChoise = this.countCombineFiles[id][index].attachChoise;
				
				this.copies=this.countCombineFiles[id][index].copies;
				this.Editionnumber=this.countCombineFiles[id][index].Editionnumber;
			},
			combineSetSynchronization(){
				if(this.copies==0){
					/* uni.showToast({
						icon:'none',
						title: '打印份数不能为0！'
					}); */
					this.copies=1;
					return;
				}
				if(this.Editionnumber==0){
					/* uni.showToast({
						icon:'none',
						title: '打印份数不能为0！'
					}); */
					this.Editionnumber=1;
					return;
				}
				for(let j=0;j<this.files.length;j++){
					this.files[j].optionChoise=this.optionChoise;
					this.files[j].attachChoise=this.attachChoise;
					this.files[j].copies= this.copies;
					this.files[j].Editionnumber= this.Editionnumber;
					this.files[j].price=this.compute(this.optionChoise,this.attachChoise,this.files[j].paper,this.Editionnumber,this.copies);
					this.files[j].fileTitle=this.fileTitle;
				}
				this.computeCountprice();
				this.hideModal();
			},
			combineSet(){
				if(this.copies==0){
					/* uni.showToast({
						icon:'none',
						title: '打印份数不能为0！'
					}); */
					this.copies=1;
					return;
				}
				if(this.Editionnumber==0){
					/* uni.showToast({
						icon:'none',
						title: '打印份数不能为0！'
					}); */
					this.Editionnumber=1;
					return;
				}
				let id=this.whichcombinefile[0];
				for(let j=0;j<this.countCombineFiles[id].length;j++){
					
					this.countCombineFiles[id][j].optionChoise=this.optionChoise;
			
					this.countCombineFiles[id][j].attachChoise=this.attachChoise;
					
					this.countCombineFiles[id][j].copies= this.copies;
					this.countCombineFiles[id][j].Editionnumber= this.Editionnumber;
					this.countCombineFiles[id][j].price=0;
					this.countCombineFiles[id][j].fileTitle=this.fileTitle;
				}
				this.computeCountprice();
				this.hideModal();
			},
			combineDelte(id,index){
				uni.showModal({
					content: '是否删除文件？',
					showCancel: true,
					cancelText: '取消',
					confirmText: '确认',
					success: res => {
						if(res.confirm){
							uni.request({
								url:upUrl.websiteUrl+'file/delete_files',
								method: 'GET',
								data: {
									id:this.countCombineFiles[id][index].id
								},
								header: {
									'Authorization': this.token
								},
								success: res => {
									if(res.data.code==200){
										this.countCombineFiles[id].splice(index,1);
										this.computeCountprice();
										this.computeCountfile();
									}
								},
								fail: () => {
									uni.showToast({
										icon:'none',
										title: '删除失败！请重试！'
									});
								},
								complete: () => {}
							});
						}
					}
				});
			},
			//删除合并分组
			deletCombine(id){
				uni.showModal({
					content: '是否删除合并分组？',
					showCancel: true,
					cancelText: '取消',
					confirmText: '确认',
					success: res => {
						if(res.confirm){
							for(let i=0;i<this.countCombineFiles[id].length;i++){
								uni.request({
									url:upUrl.websiteUrl+'file/delete_files',
									method: 'GET',
									data: {
										id:this.countCombineFiles[id][i].id
									},
									header: {
										'Authorization': this.token
									},
									success: res => {
										
									},
									fail: () => {
										uni.showToast({
											icon:'none',
											title: '删除失败！请重试！'
										});
									}
								});
							}
							this.countCombineFiles.splice(id,1);
							this.computeCountprice();
							this.computeCountfile();
						}
					}
				});
			},
			//删除
			delet(e){
				uni.showModal({
					content: '是否删除文件？',
					showCancel: true,
					cancelText: '取消',
					confirmText: '确认',
					success: res => {
						if(res.confirm){
							uni.request({
								url:upUrl.websiteUrl+'file/delete_files',
								method: 'GET',
								data: {
									id:this.files[e].id
								},
								header: {
									'Authorization': this.token
								},
								success: res => {
									if(res.data.code==200){
										this.files.splice(e,1);
										this.computeCountprice();
										this.computeCountfile();
									}
								},
								fail: () => {
									uni.showToast({
										icon:'none',
										title: '删除失败！请重试！'
									});
								},
								complete: () => {}
							});
						}
					}
				});
			},
			
			_formatfiles(e){
				console.log(e.optionChoise)
				console.log(e.attachChoise)
				let option=[];
				e.optionChoise.forEach(function(item){
					option.push(item);
				})
				let attach=[];
				e.attachChoise.forEach(function(item){
					attach.push(item);
				})
				return{
					name:e.name,
					copies:e.copies,
					Editionnumber:e.Editionnumber,
					paper:e.paper,
					price:e.price,
					fileTitle:e.fileTitle,
					optionChoise:option,
					attachChoise:attach,
					id:e.id,
					path:e.path
				}
			},
			_fileFormat(e,path,paper,id){
				let option=[];
				let attach=[];
				if(this.whichBtn==3){
					option =this.countCombineFiles[this.whichinputcombinefile][0].optionChoise;
					attach = this.countCombineFiles[this.whichinputcombinefile][0].attachChoise;
				}else{
					this.optionChoise.forEach(function(item){
						option.push(item);
					})
					
					this.attachChoise.forEach(function(item){
						attach.push(item);
					})
				}
				
				return{
					name:e.name,
					copies:this.copies,
					Editionnumber:this.Editionnumber,
					paper:paper,
					id:id,
					price:this.compute(option,attach,paper,this.Editionnumber,this.copies),
					path:path,
					optionChoise:option,
					attachChoise:attach,
					fileTitle:e.name
				}
			},
			//上传
			choose_file(e) {
			// 模拟点击file input触发选择文件，注意：不能在任何方式的回调里面执行此语句
				/* this.$refs.upload.click(); */
				this.whichBtn=e;
				var input=document.getElementById('file');
				input.value = ''; // 虽然file的value不能设为有字符的值，但是可以设置为空值
				input.click();
			},
			choose_combinfile(id){
				this.whichinputcombinefile=id;
				this.choose_file(3);
			},
			upload_next(index){
				/* this.progress=false;
				this.chunks="0%"; */
				if(this.whichBtn==1&&this.Temporaryfile!=''&&index >= this.file_lists.length){
					this.countCombineFiles.push(this.Temporaryfile);
					this.computeCountfile();
					this.computeCountprice();
					this.startSet();
					this.Temporaryfile=[];
					console.log(this.countCombineFiles)
					return;
				}else if(this.whichBtn==2&&index>=this.file_lists.length&&this.Temporaryfile!=''){
					this.Temporaryfile.forEach(item=>{
						this.files.push(item);
					})
					/* for(let i=0;i<this.Temporaryfile.length;i++){
						this.files.push(this.Temporaryfile[i]);
					} */
					this.computeCountfile();
					this.computeCountprice();
					this.startSet();
					this.Temporaryfile=[];
					console.log(this.files)
					return;
				}else if(this.whichBtn==3&&index>=this.file_lists.length&&this.Temporaryfile!=''){
					this.Temporaryfile.forEach(item=>{
						this.countCombineFiles[this.whichinputcombinefile].push(item);
					})
					/* for(let i=0;i<this.Temporaryfile.length;i++){
						this.countCombineFiles[this.whichinputcombinefile].push(this.Temporaryfile[i]);
					} */
					this.computeCountfile();
					this.computeCountprice();
					this.startSet();
					this.Temporaryfile=[];
					console.log(this.countCombineFiles)
					return;
				}else if(index == this.file_lists.length){
					return;
				}
				var _this=this;
				console.log(this.file_lists);
					this.filestatus = "开始上传"+index+"/"+this.file_lists.length;
					console.log(this.file_lists[index]);
					this.uploadFile(index,this.file_lists[index],
					this.user_id,(res)=>{
						uni.hideLoading();
						this.filestatus = "上传成功,文件名："+res.data.path;
						uni.showLoading({
							title:'正在获取第'+(index+1)+"/"+_this.file_lists.length+'文件页数...',
							mask:true
						})
						
						let getPaperTimes=0;
						let intervalTimes=0
						/* for(let j=0;j<6;j++){
						} */
						let stopGetPaper = setInterval(function(){
							if(getPaperTimes==intervalTimes){
								intervalTimes++
								uni.request({
									url: upUrl.websiteUrl+'file/getPages',
									method: 'GET',
									data: {
										id:res.data.id,
										user_id:_this.user_id
									},
									header: {
										'Authorization':_this.token
									},
									success: e => {
										getPaperTimes++;
										if(e.data.code==500||getPaperTimes==6){
											//隐藏进度条，重置进度
											_this.progress=false;
											_this.chunks="0%";
											uni.hideLoading();
											clearInterval(stopGetPaper);
											uni.showToast({
												icon:'none',
												title: '获取页数失败，请重新上传！'
											});
											_this.upload_next(++index);
										}
										if(e.data.code==200&&e.data.data.pages>0){
											uni.showToast({
												icon:'success',
												title:"上传成功！"
											});
											//隐藏进度条，重置进度
											_this.progress=false;
											_this.chunks="0%";
											clearInterval(stopGetPaper);
											let f =_this._fileFormat(_this.file_lists[index],res.data.path,e.data.data.pages,res.data.id);
											_this.Temporaryfile.push(f);
											_this.upload_next(++index);
										}
									}
								});
							}
						},15000)
					},
					(current,total)=>{
						this.filestatus = "总块数："+total+"已上传块："+current;
						uni.showLoading({
							title:'正在上传中:'+(((parseFloat((current+1)/total)).toFixed(2))*100).toString()+"%",
							mask:true
						})
						});
				
			}
			,
			start_upload(){
				this.filestatus = "开始上传";
				this.upload_next(0);
				
			},
			file_change(e){
				//对选择的文件进行处理
				this.file_lists = event.target.files;
				/* this.modalName = 'DialogModal1'; */
				this.start_upload();
			},
						
			uploadFile(index,file,user_id,success_callbacks,upload_callback){
				var _this=this;
				//撤销model
				this.modalName = null
				// 上传文件块的大小，可自定义
			   var chunkSize = 2097152,
			    // 计算改文件的可分为多少块,总块数
			    Total_chunks = Math.ceil(file.size / chunkSize),
				//
				Current_chunk = 0,
				
				File_name = file.name;

				
				function load(){
					upload_callback && upload_callback(Current_chunk,Total_chunks);
					if(Current_chunk < Total_chunks){
						
						var start = Current_chunk * chunkSize,
						end = start + (chunkSize > file.size ? file.size : chunkSize);
						if(Current_chunk == Total_chunks - 1){
							end = file.size;
							console.log(start,end);
						}
						let res = file.slice(start, end);
						up(res);
					}
				};
				
				function up(res){
					_this.progress=true;
					// 用FormData传输文件对象
					let fd = new FormData()
					// 设置文件上传接口的需要的参数
					fd.append('user_id', user_id)
					fd.append('file_name', File_name)
					fd.append('origin', file.name)
					fd.append('current_part', ++Current_chunk)
					fd.append('total_part', Total_chunks)
					
					// 设置上传的当前的文件块
					let b = new Blob([res]);
					fd.append('file', b)
					 
					let xhr = new XMLHttpRequest()
					xhr.open('post', upUrl.websiteUrl+'file/upload', true)
					xhr.onreadystatechange = function() {
						uni.hideLoading();
					    // 上传成功
					    if (xhr.readyState == 4 && xhr.status == 200) {
							console.log(Current_chunk,Total_chunks);
							
							
							let result = JSON.parse(xhr.responseText)
							if(result.code == 500){
								_this.progress=false;
								_this.chunks="0%";
								uni.showToast({
									icon:'none',
									title: result.msg
								});
								_this.upload_next(++index);
								return;
							}
							File_name = result.data.file_name;
					        if(Current_chunk == Total_chunks){
								upload_callback && upload_callback(Current_chunk,Total_chunks);
								success_callbacks && success_callbacks(result);
							}else{
								
								load();
							}
					        xhr = null
					        return
					    }
					    if (xhr.readyState == 4 && xhr.status == 500) {
					        // 文件上传出错
							uni.showToast({
								icon:'none',
								title:'上传失败！请重试！'
							})
							//隐藏进度条，重置进度
							_this.progress=false;
							_this.chunks="0%";
							_this.upload_next(++index);
							return;
					    }
					}
					// 文件上传进度条
					xhr.upload.onprogress = function(e) {
					    // 计算上传的进度
						var count=parseInt((Current_chunk/Total_chunks)*100);
						_this.chunks=count.toString()+'%'
						/* console.log('chunks'+_this.chunks)
						console.log(Current_chunk/Total_chunks)
						console.log(e)
						console.log(Current_chunk,Total_chunks) */
					}
					xhr.onerror = xhr.upload.onerror = function() {
					    // 文件上传出错
						uni.showToast({
							icon:'none',
							title:'上传失败！请重试！'
						})
						//隐藏进度条，重置进度
						_this.progress=false;
						_this.chunks="0%";
						_this.upload_next(++index);
						return;
					}
					//开始发送
					xhr.send(fd)
					fd = null
				}
				load();
			},
			submit(){
				uni.request({
					url: upUrl.websiteUrl+'orders/submit_orders',
					method: 'POST',
					header: {
						'Authorization': this.token
					},
					data: {
						user_id:this.user_id,
						order_id:this.order_id,
						normal:this.files,
						combine:this.countCombineFiles,
						memo:this.textarea
					},
					success: res => {
						console.log(res)
						if(res.data.code==200){
							wx.miniProgram.navigateTo({
								url:'../home/filePay?orderid='+this.order_id
							})
						}else if(res.data.code==500){
							uni.showToast({
								icon:'none',
								title: '提交失败！请重新提交！'
							});
						}
					}
				});
			}
		},
		mounted() {
			window.file_c = this.file_change;
		},
		watch:{
			
		}
	} 
</script>  

<style lang="scss" scoped>
	.iconsize{
		width: 36upx;
		height: 36upx;
	}
	.test{
		background-color: black;
	}
	#app {
	  font-family: 'Avenir', Helvetica, Arial, sans-serif;
	  -webkit-font-smoothing: antialiased;
	  -moz-osx-font-smoothing: grayscale;
	  text-align: center;
	  color: #2c3e50;
	  margin-top: 60px;
	}
	.img{
		width: 500upx;
		height: 439upx;
		justify-content: center;
	}
	.combineCard{
		border: 5upx solid #FBBD08;
		border-radius: 15upx;
		width: 90%;
		margin: 20upx 5%;
		.file{
			border-radius: 15upx;
			background-color: #FFFFFF;
			height: 80upx;
			margin: 20upx auto;
			padding: 0upx 20upx;
		}
	}
	.card{
		border-radius: 15upx;
		background-color: #FFFFFF;
		height: 550upx;
		width: 90%;
		margin: 20upx 5%;
		.button{
			margin: auto 40upx;
		}
	}
	.Radio-h5{
		margin: 40upx auto 20upx;
		.title{
			font-weight: 600;
			font-size: 32upx;
		}
		.group{
			margin-left: 10upx;
			.item{
				transform: scale(0.8);
			}
			.name{
				line-height:200%;
			}
		}
		.input-title{
			border: 5upx solid #9A9A9A;
			border-radius: 15upx;
			font-size: 34upx;
			text-align: left;
			padding-left: 10upx;
			width: 75%;
		}
		.input{
			border: 5upx solid #FFDC16;
			border-radius: 15upx;
			width: 30%;
			font-size: 32upx;
		}
	}
	.file{
		border-radius: 15upx;
		background-color: #FFFFFF;
		height: 80upx;
		width: 90%;
		margin: 20upx 5%;
		padding: 0upx 20upx;
		.fileName{
			font-size: 32upx;
			margin: auto;
			max-width: 40%;
			overflow: hidden;
			text-overflow:ellipsis;
			white-space: nowrap
		}
		.title{
			font-size: 32upx;
			margin: auto;
			max-width: 100%;
			overflow: visible;
			white-space: nowrap;
			text-overflow:inherit;
		}
		.paper{
			font-size: 32upx;
			margin: auto;
			text-align: center;
		}
		.price{
			font-size: 32upx;
			margin: auto;
			text-align: center;
		}
		.size{
			font-size: 48upx;
			margin: auto;
			text-align: center;
		}
	}
	.textarea{
		border: #FFDC16 solid 5upx;
		box-shadow: 3px 3px 4px rgba(224, 170, 7, 0.2);
	}
	
	.select {
			display: inline-block;
			width: 300px;
			position: relative;
			vertical-align: middle;
			padding: 0;
			overflow: hidden;
			background-color: #fff;
			color: #555;
			border: 1px solid #aaa;
			text-shadow: none;
			border-radius: 4px;	
			transition: box-shadow 0.25s ease;
			z-index: 2;
		}
	 
		.select:hover {
			box-shadow: 0 1px 4px rgba(0, 0, 0, 0.15);
		}
	 
		.select:before {
			content: "";
			position: absolute;
			width: 0;
			height: 0;
			border: 10px solid transparent;
			border-top-color: #ccc;
			top: 14px;
			right: 10px;
			cursor: pointer;
			z-index: -2;
		}
		.select select {
			cursor: pointer;
			padding: 10px;
			width: 100%;
			border: none;
			background: transparent;
			background-image: none;
			-webkit-appearance: none;
			-moz-appearance: none;
		}
	 
		.select select:focus {
			outline: none;
		}
</style>