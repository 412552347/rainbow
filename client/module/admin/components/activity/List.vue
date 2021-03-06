<template>
	<div class="add-form-box">
		<full-screen :title="activityId===undefined?'新增活动':'编辑活动'">
			<div slot="content">
				<el-form :model="activityForm" :rules="activityRules" label-position="left" label-width="120px" ref="activityForm">
					<el-form-item label="活动名称" prop="name">
						<el-input v-model="activityForm.name" placeholder="不能超过20个字符"></el-input>
					</el-form-item>
					<el-form-item label="活动开始时间" prop="startDate">
						<el-date-picker
					      v-model="activityForm.startDate"
					      type="date"
					      placeholder="开始时间">
					    </el-date-picker>
					</el-form-item>
					<el-form-item label="活动结束时间" prop="endDate">
						<el-date-picker
					      v-model="activityForm.endDate"
					      type="date"
					      placeholder="结束时间">
					    </el-date-picker>
					</el-form-item>
					<el-form-item label="活动承办团队" prop="teamId">
					    <el-select v-model="activityForm.teamId" placeholder="请选择团队">
					      <el-option v-for="item in teamList"
					      	:label="item.name"
					      	:key="item.id"
					      	:value="item.id"></el-option>
					    </el-select>
					  </el-form-item>
					<el-form-item label="招募人数" prop="maxNum">
						<el-input type="number" v-model="activityForm.maxNum"></el-input>
					</el-form-item>
					<el-form-item label="活动状态">
						<el-switch :disabled="activityId === undefined" v-model="activityForm.status" text-on="" text-off=""></el-switch>
					</el-form-item>
					<el-form-item label="活动描述">
						<div id="editor"></div>
					</el-form-item>
				</el-form>
			</div>

			<div slot="footer">
				<el-button @click="handlerValidateList">确认</el-button>
				<el-button class="cancel" @click="$router.push('/activity')">取消</el-button>
			</div>
		</full-screen>
	</div>
</template>
<script>
import wangEditor from 'wangeditor'
import FullScreen from '../layout/FullScreen.vue'

export default{
	created(){
		const t = this;
		t.activityId = t.$route.query.activityId;

		//获取团队列表
		t.ajaxGetTeam(); 

		// 编辑获取活动信息
		if(t.activityId !== undefined) {
			t.ajaxGetForm();
		}
	},
	mounted(){
		const t = this;
		t.createEditor();
	},
	data(){
		return {
			activityId:'',
			activityForm:{  // 添加/删除发送数据
				activityId:"", 
				name:"",
				startDate:"", 
				endDate:"", 
				note:"",
				teamId:"",
				maxNum:"",
				status:true    // 1删除 0进行 2结束
			},
			editor:'',
			teamList:[
				{id:'0',name:'wang'},
				{id:'1',name:'li'}
			],
		}
	},
	methods:{
		// -------- ajax请求 ----------
		// 活动信息
		ajaxGetForm(){
			const t = this;
			t.$http({
				method:'post',
				url:'/activity/ajax-get-activity',
				data:{
					id:t.activityId
				}
			}).then(res => {
				const result = res.data;
				if(!result.status){
					return t.$message({
						message:result.message,
						type:'error'
					})
				}

				t.activityForm = result.data;
				t.activityForm['startDate'] = new Date(t.activityForm['startDate']);
				t.activityForm['endDate'] = new Date(t.activityForm['endDate']);

				t.activityForm['status'] = !t.activityForm['status'];

				//编辑器内容填充
				t.editor.txt.html(t.activityForm.note);
			})
		},
		// 获取团队列表
		ajaxGetTeam(){
			const t = this;

			t.$http({
				method:'post',
				url:'/team/ajax-get-team-id-name-list'
			}).then(res => {
				const result = res.data;
				if(!result.status) {
					return  t.$message({
			          message: result.message,
			          type: 'Error'
			        });
				}

				t.teamList = result.data;
			})
		},
		// 添加/编辑
		ajaxUpdateForm(postData){
			const t = this;
			let url = t.activityId === undefined ? '/activity/ajax-add-activity' : '/activity/ajax-edit-activity';

			t.$http({
				method:'post',
				url:url,
				data:postData
			}).then(res => {
				const result = res.data;
				if(!result.status){
					return t.$message({
						message:result.message,
						type:'error'
					})
				}
				// 更新成功，跳回列表页
				t.$router.push('/activity');
				return t.$message({
					message:result.message,
					type:'success'
				})
			})
		},
		// -------- 其他操作 ----------
		//初始化富文本编辑器
		createEditor(){
			const t = this;
			t.editor = new wangEditor('#editor');
			t.editor.customConfig.uploadImgShowBase64 = true; //图片base64编码

			// 自定义菜单配置
		    t.editor.customConfig.menus = [
		        'head',
		        'bold',
		        'italic',
		        'underline',
		        'fontSize',
		        'fontName', 
		    ]
			t.editor.create();
		},
		// 数据处理
		handlerpostData(){
			const t = this;
			let postData = JSON.parse(JSON.stringify(t.activityForm));

			//处理数据
			postData.endDate = new Date(postData.endDate).getTime();
			postData.startDate = new Date(postData.startDate).getTime();
			postData.status = postData.status ? 0 : 2;
			postData.note = t.editor.txt.html();
			return postData;
		},
		// 添加/编辑
		handlerValidateList(){
			const t = this;
			t.$refs['activityForm'].validate(valid => {
				if(valid){
					let postData = t.handlerpostData();
					// 数据发送
					t.ajaxUpdateForm(postData);
				}else{
					 return false;
				}
			})
		},
		// 日期校验
		validateDate(rule, value, callback){
			const t = this;
			if(t.activityForm.startDate !== '' && t.activityForm.endDate !== ''){
				if(new Date(t.activityForm.startDate).getTime() > new Date(t.activityForm.endDate).getTime()){
					return callback(new Error('开始时间不得小于结束时间！'));
				}
				callback();
			}
			callback();
		}
	},
	computed:{
		activityRules(){
			const t = this;
			return {
				name:[
					{ required: true, message: '不能为空', trigger: 'blur' },
					{ min: 1, max: 20, message: '长度在 1 到 20 个字符', trigger: 'blur' }
				],
				startDate:[
					{ type: 'date', required: true, message: '请选择时间', trigger: 'change' },
					{validator:t.validateDate,trigger:'change'}
				],
				endDate:[
					{ type: 'date', required: true, message: '请选择时间', trigger: 'change' },
					{validator:t.validateDate,trigger:'change'}
				],
				teamId: [
		            { required: true, message: '请选择志愿者团队', trigger: 'change' },
		        ],
		        maxNum: [
		        	{ required: true, message: '不能为空', trigger: 'blur' },
		        ]
			}
		}
	},
	components:{
		'full-screen':FullScreen
	}
}
</script>
<style lang="scss">
.add-form-box{
	width:100%;
	height: 100%;
}
</style>