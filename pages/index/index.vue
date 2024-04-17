<template>
	<view>
		<page-head title="农场主"></page-head>
		<view class="uni-padding-wrap uni-common-mt">

			<view class="uni-flex uni-row">
				<view class="text" style="-webkit-flex: 1;flex: 1;" @click="nextRound">当前回合数:{{round}}</view>
				<view class="text" style="-webkit-flex: 1;flex: 1;">{{roundDesc[round-1].title}}</view>
			</view>
09


			<view class="uni-flex uni-row flex-item flex-item-V " 
				v-for="n in Math.ceil(resourceTypeList.length / 4)">
				<view class="text" style="-webkit-flex: 1;flex: 1;" v-for="index in (n<=4?4:2)">
					{{resourceTypeList[(n-1) * 4 + index-1 ].name}}
						<uni-number-box class="resource" :data-resource="resourceTypeList[(n-1) * 4 + index-1 ].name"
							v-model="player[0].resource[resourceTypeList[(n-1) * 4 + index-1 ].name]"
							:background="resourceTypeList[(n-1) * 4 + index-1 ].color" color="#fff" />
					
				</view> 

			</view>

			<view class="uni-flex uni-row">
				<button style="-webkit-flex: 1;flex: 1;" type="primary" @click="nextRound">下一回合</button>
				<button style="-webkit-flex: 1;flex: 1;" type="primary" @click="aiAction">ai操作</button>

			</view>





			<uni-section title="当前可选动作" type="line">
				<uni-list>

					<uni-list-item v-for="(item, index) in allAvailbleActionCardList" :title="(index+1)+'.'+item.name"
						:note="item.desc"
						thumb="https://vkceyugu.cdn.bspapp.com/VKCEYUGU-dc-site/460d46d0-4fcc-11eb-8ff1-d5dcf8779628.png"
						thumb-size="lg" :rightText="item.resourceType" showBadge="true" :badgeText="item.allResource">
						<template v-slot:header>
							<uni-badge v-for="item2 in item.actionSelectList" class="uni-badge-left-margin"
								:text="item2.size" type="primary" :customStyle="item2.style"
								@click="cardSelectCallback(item2.playerId,item)" />
						</template>
					</uni-list-item>

				</uni-list>
			</uni-section>

			<uni-section title="次要发展卡" type="line" padding>
				<textarea auto-height value="用于备注" />
				<uni-grid :column="4" :highlight="true" @change="change">
					<uni-grid-item v-for="(item, index) in allRandomMIList" :index="index" :key="index">
						 <view :data-image-index="index" data-type="MI" @tap="refreshCard">{{index}}</view>
						<image class="image" :data-image-index="index" data-type="MI" style="width: 200px;height: 300px;" mode="aspectFit" :data-src="item"
							:src="item" @tap="previewImage" />
					</uni-grid-item>
				</uni-grid>
			</uni-section>

			<uni-section title="职业卡" type="line" padding>
				<textarea auto-height value="用于备注" />
				<uni-grid :column="4" :highlight="true" @change="change">
					<uni-grid-item v-for="(item, index) in allRandomOList" :index="index" :key="index">
						 <view :data-image-index="index" data-type="O" @tap="refreshCard">{{index}}</view>
						<image class="image" :data-image-index="index" data-type="O" style="width: 200px;height: 300px;" mode="aspectFit" :data-src="item"
							:src="item" @tap="previewImage"  /> 
					</uni-grid-item>
				</uni-grid> 
			</uni-section>



			<uni-steps :options="roundDesc" active-color="#007AFF" :active="round-1" direction="column" />

		<view>
			<!-- 普通弹窗 -->
			<uni-popup ref="popup" background-color="#fff" @change="change">
				<view class="popup-content" ><text
						class="text">ai当前执行{{aiActionIndex}}号动作</text></view>
			</uni-popup>
		</view>

		</view>
	</view>
</template>
<script>
	export default {

		data() {
			return {
				index: 0,

				array: [{
					name: '木屋'
				}, {
					name: '砖屋'
				}, {
					name: '石屋'
				}],
				player: [{
					id: 1,
					color: "#0000ff",
					name: "玩家",
					resource: {}
				}, {
					id: 2,
					color: "#d40000",
					name: "ai"
				}],
				allActionCardList: [{
						id: 1,
						stage: 0,
						name: "钓鱼",
						desc: "获得1食物",
						resourceType: "食物",
						allResource: 0,
						eachAddResource: 1
					},
					{
						id: 2,
						stage: 0,
						name: "扩建房舍",
						desc: "扩建房屋/+建造畜棚 房间=5(木/砖/石)+2芦苇 畜棚=2木"
					},
					{
						id: 3,
						stage: 0,
						name: "职业卡",
						desc: "打出1张职业卡(每个玩家的第一张免费,后续需要支付1食物)"
					},
					{
						id: 4,
						stage: 0,
						name: "广场",
						desc: "获得起始玩家/+打出1张次要发展卡"
					},
					{
						id: 5,
						stage: 0,
						name: "小麦",
						desc: "获得1小麦",
						resourceType: "小麦",
						allResource: 0,
						stableResourse: 1
					},
					{
						id: 6,
						stage: 0,
						name: "犁地",
						desc: "开垦1个田地",
						resourceType: "耕地",
						allResource: 0,
						stableResourse: 1
					},
					{
						id: 7,
						stage: 0,
						name: "临时工",
						desc: "获得2食物",
						resourceType: "食物",
						allResource: 0,
						stableResourse: 2
					},
					{
						id: 8,
						stage: 0,
						name: "森林",
						desc: "获得3木头",
						resourceType: "木头",
						allResource: 0,
						eachAddResource: 3
					},
					{
						id: 9,
						stage: 0,
						name: "粘土坑",
						desc: "获得1粘土",
						resourceType: "粘土",
						allResource: 0,
						eachAddResource: 1
					},
					{
						id: 10,
						stage: 0,
						name: "芦苇滩",
						desc: "获得1芦苇",
						resourceType: "芦苇",
						allResource: 0,
						eachAddResource: 1
					},
					{
						id: 11,
						stage: 1,
						name: "羊市场",
						desc: "获得1羊",
						resourceType: "羊",
						allResource: 0,
						eachAddResource: 1
					},
					{
						id: 12,
						stage: 1,
						name: "使用谷物",
						desc: "耕种小麦或者蔬菜/+烘培面包"
					},
					{
						id: 13,
						stage: 1,
						name: "使用发展卡",
						desc: "使用主要发展卡/使用次要发展卡"
					},
					{
						id: 14,
						stage: 1,
						name: "建造栅栏",
						desc: "1栅栏=1木"
					},
					{
						id: 15,
						stage: 2,
						name: "西部采石场",
						desc: "获得1石头",
						resourceType: "石头",
						allResource: 0,
						eachAddResource: 1
					},
					{
						id: 16,
						stage: 2,
						name: "房屋重建",
						desc: "翻新房屋(房间数量相等的砖(C)/石(S)+1芦苇)/+使用发展卡"
					},
					{
						id: 17,
						stage: 2,
						name: "增加家庭成员",
						desc: "增加1家庭成员/+使用主发展卡"
					},
					{
						id: 18,
						stage: 3,
						name: "野猪市场",
						desc: "获得1野猪",
						resourceType: "野猪",
						allResource: 0,
						eachAddResource: 1
					},
					{
						id: 19,
						stage: 3,
						name: "蔬菜种子",
						desc: "获得1蔬菜",
						resourceType: "蔬菜",
						allResource: 0,
						eachAddResource: 1
					},
					{
						id: 20,
						stage: 4,
						name: "牛市场",
						desc: "获得1牛",
						resourceType: "牛",
						allResource: 0,
						eachAddResource: 1
					},
					{
						id: 21,
						stage: 4,
						name: "东部采石场",
						desc: "获得1石",
						resourceType: "石",
						allResource: 0,
						eachAddResource: 1
					},
					{
						id: 22,
						stage: 5,
						name: "耕作",
						desc: "开垦1个田地/+耕种小麦或者蔬菜"
					},
					{
						id: 23,
						stage: 5,
						name: "紧急增加家庭成员",
						desc: "无视房屋需求增加1家庭成员"
					},
					{
						id: 24,
						stage: 6,
						name: "农场重建",
						desc: "翻新房屋/+建造栅栏(1木1栅栏)"
					}
				],
				allAvailbleActionCardList: [],
				round: 0,
				stage: 1,
				roundDesc: [{
						title: '阶段1-早春',
						desc: ''
					}, {
						title: '阶段1-晚春',
						desc: ''
					}, {
						title: '阶段1-夏',
						desc: ''
					}, {
						title: '阶段1-秋',
						desc: '结束后,进行 喂食-作物丰收-动物繁殖'
					},
					{
						title: '阶段2-春',
						desc: ''
					}, {
						title: '阶段2-夏',
						desc: ''
					}, {
						title: '阶段2-秋',
						desc: '结束后,进行 喂食-作物丰收-动物繁殖'
					},
					{
						title: '阶段3-夏',
						desc: ''
					}, {
						title: '阶段3-秋',
						desc: '结束后,进行 喂食-作物丰收-动物繁殖'
					},
					{
						title: '阶段4-夏',
						desc: ''
					}, {
						title: '阶段4-秋',
						desc: '结束后,进行 喂食-作物丰收-动物繁殖'
					},
					{
						title: '阶段5-夏',
						desc: ''
					}, {
						title: '阶段5-秋',
						desc: '结束后,进行 喂食-作物丰收-动物繁殖'
					},
					{
						title: '阶段6-秋',
						desc: '结束后,进行 喂食-作物丰收-动物繁殖'
					}
				],
				allRandomOList: [],
				allRandomMIList: [],

				resourceTypeList: [{
						name: "木头",
						initNum: 0,
						color: "#9d4c00"
					},
					{
						name: "粘土",
						initNum: 0,
						color: "#3a1c00"
					},
					{
						name: "石头",
						initNum: 0,
						color: "#6c6c6c"
					},
					{
						name: "芦苇",
						initNum: 0,
						color: "#dadada"
					},
					{
						name: "小麦",
						initNum: 0,
						color: "#bcbc00"
					},
					{
						name: "蔬菜",
						initNum: 0,
						color: "#009000"
					},
					{
						name: "耕地",
						initNum: 0,
						color: "#613000"
					},
					{
						name: "食物",
						initNum: 2,
						color: "#49bd28"
					},
					{
						name: "羊",
						initNum: 0,
						color: "#d8d8d8"
					},
					{
						name: "牛",
						initNum: 0,
						color: "#8c4600"
					},
					{
						name: "野猪",
						initNum: 0,
						color: "#3b1c00"
					},
					{
						name: "栅栏",
						initNum: 0,
						color: "#aa5500"
					},
					{
						name: "房屋材质",
						initNum: 0,
						color: "#000000"
					},
					{
						name: "房间",
						initNum: 2,
						color: "#1e0e00"
					},
					{
						name: "农民",
						initNum: 2,
						color: "#ff0000"
					},
					{
						name: "主要发展卡",
						initNum: 0,
						color: "#870000"
					},
					{
						name: "次要发展卡",
						initNum: 0,
						color: "#008a00"
					},
					{
						name: "职业卡",
						initNum: 0,
						color: "#919100"
					},
				],
				stageIndex: [
					[1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
					[11, 12, 13, 14],
					[15, 16, 17],
					[18, 19],
					[20, 21],
					[22, 23],
					[24]
				],
				stageActionList:[
					[[1,1],[1,0],[0,0],[0,1]],
					[[1,1,1],[1,1,2],[1,2,1],[2,1,1]],
					[[1,1,1,2],[1,1,2,3],[1,2,3,1],[2,3,1,1],[3,1,1,1]],
					[[1,1,1,2,3],[1,1,2,3,4],[1,2,3,4,1],[2,3,4,1,1],[3,4,1,1,1],[4,1,1,1,2]],
					[[1,1,1,2,3],[1,1,2,3,4],[1,2,3,4,5],[2,3,4,5,1],[3,4,5,1,1],[4,5,1,1,1],[5,1,1,1,4]],
					[[1,1,1,2,3],[1,1,2,3,4],[1,2,3,4,5],[2,3,4,5,1],[3,4,5,1,1],[4,5,1,1,1],[5,1,1,1,2]]
				],
				aiRound:0,
				aiThisRoundActionIndex:0,
				aiThisRoundRandomAction:[],
				aiActionIndex:0


			}
		},
		onLoad() {
console.log("ai在阶段0动作里选择")
			this.allActionCardList = this.sortAndRandomizeByStage(this.allActionCardList);
			this.allAvailbleActionCardList = this.allActionCardList.slice(0, 10);
			this.initPlayerData();
			this.nextRound();
			this.allRandomMIList = this.getRandomCardUrl("MI", 7);
			this.allRandomOList = this.getRandomCardUrl("O", 7);
		},
		methods: {
			initCardSelectData: function() {
				for (var index in this.allAvailbleActionCardList) {
					this.allAvailbleActionCardList[index].actionSelectList = [];

					for (var playerIndex in this.player) {
						this.allAvailbleActionCardList[index].actionSelectList[playerIndex] = {
							playerId: this.player[playerIndex].id,
							size: "0",
							style: {
								background: this.player[playerIndex].color
							}
						}
					}
				}

			},
			bindPickerChange: function(e) {
				console.log('picker发送选择改变，携带值为：' + e.detail.value)
				this.index = e.detail.value
			},
			cardSelectCallback: function(playerId, actionCard) {
				var selectList = actionCard.actionSelectList.find(obj => obj.playerId === playerId);
				selectList.size = String(parseInt(selectList.size) + 1);

				if (playerId == 1) {
					//玩家操作,需要增加相应资源
					this.player[0].resource[actionCard.resourceType] = String(parseInt(this.player[0].resource[
						actionCard.resourceType]) + parseInt(actionCard.allResource))

				}
				if (actionCard.allResource != undefined) {
					actionCard.allResource = "0";
				}

			},
			nextRound: function() {
				this.round = this.round + 1;
				this.allAvailbleActionCardList.push(this.allActionCardList[this.round + 9])
				this.handleActionCardEachRound();
				this.initCardSelectData();
				this.aiThisRoundActionIndex=0;
			},
			sortAndRandomizeByStage: function(array) {
				// 首先按stage进行排序，注意这里使用对象的降序排序，以便stage为0的保持在前面
				array.sort(function(a, b) {
					if (a.stage == b.stage && a.stage == 0 && b.stage == 0) {
						return 0;
					} else if (a.stage == b.stage) {
						return Math.random() - 0.5;
					} else {
						return a.stage - b.stage;
					}
				});

				return array;

			},
			handleActionCardEachRound: function() {
				for (var index in this.allActionCardList) {

					if (this.allActionCardList[index].stableResourse != undefined) {
						//固定资源节点
						this.allActionCardList[index].allResource = String(this.allActionCardList[index]
							.stableResourse);
					} else if (this.allActionCardList[index].eachAddResource != undefined) {
						//增量节点
						this.allActionCardList[index].allResource = String(parseInt(this.allActionCardList[index]
							.eachAddResource) + parseInt(this.allActionCardList[index].allResource));

					}


				}
			},
			initPlayerData: function() {
				this.player[0].resource = {};
				for (var index in this.resourceTypeList) {
					this.player[0].resource[this.resourceTypeList[index].name] = this.resourceTypeList[index].initNum;
				}
				console.log('1')
			},
			getOneRandomCardUrl: function(type) {
				var baseUrl =
					"https://images.weserv.nl/?url=https://gitee.com/qianzise/img/raw/master/%E5%86%9C%E5%9C%BA%E4%B8%BB/";
				var randomNumber = 0;
				if (type === "MI") {

					var min = 1;
					var max = 2225;
					randomNumber = Math.floor(Math.random() * (max - min + 1)) + min;
					baseUrl = baseUrl + "%E6%AC%A1%E8%A6%81%E5%8F%91%E5%B1%95%E5%8D%A1/ciyao" + randomNumber + ".jpg";

				} else if (type === "O") {

					var min = 1;
					var max = 2613;
					randomNumber = Math.floor(Math.random() * (max - min + 1)) + min;
					baseUrl = baseUrl + "%E8%81%8C%E4%B8%9A%E5%8D%A1/zhiye" + randomNumber + ".jpg";
				}

				return baseUrl;

			},
			getRandomCardUrl: function(type, size) {
				var list = [];
				for (var i = 0; i < size; i++) {
					list.push(this.getOneRandomCardUrl(type));
				}

				return list;

			},
			aiAction: function() {
				
				if(this.aiRound!=this.round){
					//本回合未生成ai操作,生成先
					var thisRoundActionList=this.stageActionList[this.stage-1];
					//这个回合ai要执行的阶段动作列表
				    this.aiThisRoundRandomAction=thisRoundActionList[Math.floor(Math.random() * thisRoundActionList.length)];
					//ai当前已经执行到了这个阶段动作列表的弟几个
					this.aiThisRoundActionIndex=0;
					this.aiRound=this.round;
					console.log("本回合ai路线" + this.aiThisRoundRandomAction)
				}
				
				//ai目前要执行的阶段的所有动作列表
				var aiActionList=this.stageIndex[this.aiThisRoundRandomAction[this.aiThisRoundActionIndex]];
				aiActionList.sort(() => Math.random() - 0.5);
				console.log("ai在阶段" + this.aiThisRoundRandomAction[this.aiThisRoundActionIndex]+"动作里选择")
				for(var i in aiActionList){
					var actionIndex=aiActionList[i];
					if(this.actionCanSelect(actionIndex)){
						//当前随机到的动作可以执行
						this.aiActionIndex=actionIndex;
					    this.$refs.popup.open('top');
						this.aiThisRoundActionIndex+=1;
						console.log("ai选择了动作" + this.aiActionIndex)
						return;
					}
				}
				
				if(this.aiThisRoundRandomAction[this.aiThisRoundActionIndex]===0){
					//到这里还没返回,意味着基础动作都被占用了,报错
					
				}
				
				//当前stage动作被占用,则随机基础动作
				aiActionList=this.stageIndex[0];
				aiActionList.sort(() => Math.random() - 0.5);
				console.log("ai在阶段0动作里选择")
				for(var i in aiActionList){
					var actionIndex=aiActionList[i];
					if(this.actionCanSelect(actionIndex)){
						//当前随机到的动作可以执行
						this.aiActionIndex=actionIndex;
					    this.$refs.popup.open('top');
						this.aiThisRoundActionIndex+=1;
						console.log('ai选择了动作' + this.aiActionIndex)
						return;
					}
				}
				
				//报错
				

			},
			actionCanSelect:function(actionCardIndex){
				if(actionCardIndex<1 || actionCardIndex>(this.round+10)){
					return false;
				}
				
				if(this.allAvailbleActionCardList[actionCardIndex-1].actionSelectList==undefined){
					return true;
				}
				
				if(this.allAvailbleActionCardList[actionCardIndex-1].actionSelectList[0]!=undefined && 
				this.allAvailbleActionCardList[actionCardIndex-1].actionSelectList[0].size!="0"){
					return false;
				}
				if(this.allAvailbleActionCardList[actionCardIndex-1].actionSelectList[1]!=undefined &&
				this.allAvailbleActionCardList[actionCardIndex-1].actionSelectList[1].size!="0"){
					return false;
				}
				return true;
				
			},
			previewImage: function(e) {
				var type=e.currentTarget.dataset["type"];
				var index=e.currentTarget.dataset["imageIndex"];
			    var urls=null;
				if(type==="O"){
					urls=this.allRandomOList;
				}else if(type==="MI"){
					urls=this.allRandomMIList;
				}
			 
				uni.previewImage({
					current:index,
					urls: urls
				})
			},
			refreshCard:function(e){
				var type=e.currentTarget.dataset["type"];
				var index=e.currentTarget.dataset["imageIndex"];
				
				if(type==="O"){
					this.allRandomOList[index]=this.getOneRandomCardUrl("O");
				}else if(type==="MI"){
					this.allRandomMIList[index]=this.getOneRandomCardUrl("MI");
				}
				
				
			}
		}
	}
</script>

<style>
	.flex-item {
		width: 33.3%;
		height: 200rpx;
		text-align: center;
		line-height: 200rpx;
	}

	.flex-item-V {
		width: 100%;
		height: 150rpx;
		text-align: center;
		line-height: 150rpx;
	}

	.text {
		margin: 15rpx 10rpx;
		padding: 0 20rpx;
		background-color: #ebebeb;
		height: 70rpx;
		line-height: 70rpx;
		text-align: center;
		color: #777;
		font-size: 26rpx;
	}

	.desc {
		/* text-indent: 40rpx; */
	}

	.flex-pc {
		display: flex;
		justify-content: center;
	}
</style>