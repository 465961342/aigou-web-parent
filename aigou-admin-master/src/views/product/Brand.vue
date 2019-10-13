<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="姓名"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getbrands">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="brands" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="60">
			</el-table-column>
			<el-table-column prop="name" label="商品名称" width="120" sortable align="center">
			</el-table-column>
			<el-table-column prop="englishName" label="英文名称" width="130"  sortable align="center">
			</el-table-column>
			<el-table-column prop="firstLetter" label="首字母" width="100" sortable align="center">
			</el-table-column>
			<el-table-column prop="logo" label="商品logo" width="120" sortable align="center">
                <template scope="scope">
                    <img src="scope.row.logo" style="margin-top: 10px;height: 30px">
                </template>
			</el-table-column>
            <el-table-column prop="productType.name" label="商品类型" width="150" sortable align="center">
            </el-table-column>
			<el-table-column prop="description" label="描述" min-width="90" sortable align="center">
			</el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="rows" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
        <el-dialog title="编辑" v-model="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                <el-form-item label="商品名称" prop="name">
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="英文名称" prop="englishName">
                    <el-input v-model="editForm.englishName"></el-input>
                </el-form-item>

                <el-form-item label="品牌logo" prop="logo">
                    <el-input v-model="editForm.logo"></el-input>
                </el-form-item>

                <div class="block" style="margin-left: 15px">
                    <span class="demonstration">商品类型</span>
                    <el-cascader
                            @change="handleChange"
                            :change-on-select="true"
                            :props="defaultParams"
                            placeholder='请选择商品类型'
                            :options="options"
                            v-model="productOptions"
                            :clearable="true">
                    </el-cascader>
                </div>

                <el-form-item></el-form-item>

                <el-form-item label="描述">
                    <el-input type="textarea" v-model="editForm.description"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>


        <!--新增界面-->
        <el-dialog title="新增" v-model="addFormVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
                <el-form-item label="商品名称" prop="name">
                    <el-input v-model="addForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="英文名称" prop="englishName">
                    <el-input v-model="addForm.englishName"></el-input>
                </el-form-item>

                <el-form-item label="品牌logo" prop="logo">
                    <el-input v-model="addForm.logo"></el-input>
                </el-form-item>

                <div class="block" style="margin-left: 15px">
                    <span class="demonstration">商品类型</span>
                    <el-cascader
                            @change="handleChange"
                            :change-on-select="true"
                            :props="defaultParams"
                            placeholder='请选择商品类型'
                            :options="options"
                            v-model="addForm.productTypeId"
                            :clearable="true">
                    </el-cascader>
                </div>

                <el-form-item></el-form-item>

                <el-form-item label="描述">
                    <el-input type="textarea" v-model="addForm.description"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>
	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';

	export default {
		data() {
			return {
				filters: {
					keyword: ''
				},
                options: [],
                productOptions:[],
                // selectedOptions: [],
                defaultParams: {
                    label: 'name',
                    value: 'id',
                    children: 'children'
                },


                brands: [],
				total: 0,
				page: 1,
                rows:10,
				listLoading: false,
				sels: [],//列表选中列

				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '请输入商品名', trigger: 'blur' }
					]
				},
				//编辑界面数据
				editForm: {
                    name: '',
                    englishName:'',
                    firstLetter:'',
                    description: '',
                    productTypeId: [],
                    logo:''
				},

				addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//新增界面数据
				addForm: {
                    name: '',
                    englishName:'',
                    firstLetter:'',
                    description: '',
                    productTypeId: [],
                    logo:''
				}

			}
		},
		methods: {
			//性别显示转换
			formatSex: function (row, column) {
				return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
			},
			handleCurrentChange(val) {
				this.page = val;
				this.getbrands();
			},
            //加载类型树
            loadTypeTree(){
                this.$http.get("/product/productType/loadTypeTree")
                    .then(res=>{
                        console.debug(res);
                        /*this.options = this.productTypes = res.data;*/
                        this.options = res.data;
                    })
            },
            handleChange(){

            },

            /*编辑获取回填菜单*/
            loadGetPath(index, row){
                this.$http.get("/product/productType/"+row.productTypeId).then(res=>{
                    var productOptions = res.data.path.split('.');
                    let paths = [];
                    let indexs ='';
                    for (let index = 0 ; index < productOptions.length ; index++){
                        if (productOptions[index] != ''){
                            indexs = parseInt(productOptions[index]);
                            paths.push(indexs);
                        }
                    }
                    console.debug(paths);
                    this.productOptions = paths;
                }).catch({});
            },
            // 递归方法:去掉tree里面的空数组
            /*getTreeData(data) {
                // 循环遍历json数据
                for (var i = 0; i < data.length; i++) {

                    if (data[i].children.length < 1) {
                        // children若为空数组，则将children设为undefined
                        data[i].children = undefined;
                    } else {
                        // children若不为空数组，则继续 递归调用 本方法
                        this.getTreeData(data[i].children);
                    }
                }
                return data;
            },*/
            //获取品牌列表
			getbrands() {
				let para = {
					page: this.page,
					rows: this.rows,
                    keyword:this.filters.keyword
				};
				this.listLoading = true;
				this.$http.post("/product/brand/json",para)
                    .then(res=>{
                        this.listLoading = false;
                        let{total,rows} = res.data;

                        console.debug("total",total);
                        console.debug("rows",rows);

                        this.brands = rows;
                        this.total = total;
                    })

			},
			//删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    this.$http.delete("/product/brand/delete/"+ row.id)
                        .then(res=>{
                            this.listLoading = false;
                            let {success,message,restObj} = res.data;
                            if (success){
                                this.$message({
                                    message: '删除成功',
                                    type: 'success'
                                });
                                this.getbrands();
                            }else {
                                this.$message({
                                    message: message,
                                    type: 'error'
                                });
                            }

                        })
                }).catch(() => {

                });
            },

            //显示编辑界面
			handleEdit: function (index, row) {
				this.editFormVisible = true;
				this.editForm = Object.assign({}, row);
                this.loadGetPath(index, row)

            },
			//显示新增界面
			handleAdd: function () {
				this.addFormVisible = true;
				this.addForm = {
					name: '',
					sex: -1,
					age: 0,
					birth: '',
					addr: ''
				};
			},
			//编辑
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认修改吗？', '提示', {}).then(() => {
                            this.addLoading = true;
                            //NProgress.start();
                            var b = '';
                            let para = Object.assign({}, this.editForm);
                            for(var i = 0;i<this.productOptions.length ; i++){
                                b = this.productOptions[i];
                            }
                            console.debug(b)
                            para.productTypeId = b;
                            console.debug("----------------------")
                            console.debug(para.productTypeId);
                            this.$http.post("/product/brand/add",para).then(
                                res=>{
                                    let {success,message,restObj} = res.data;
                                    if(success){
                                        this.addLoading = false;
                                        //NProgress.done();
                                        this.$message({
                                            message: '修改成功',
                                            type: 'success'
                                        });
                                        this.$refs['editForm'].resetFields();
                                        this.editFormVisible = false;
                                        this.getbrands();
                                    }else{
                                        this.$message({
                                            message: '添加失败',
                                            type: 'error'
                                        });
                                    }
                                }
                            ).catch({

                            });
                        });
                    }
                });
            },
			//新增
            addSubmit: function () {
                this.$refs.addForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.addLoading = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.addForm);
                            for(var i = 0;i<this.addForm.productTypeId.length ; i++){
                                var b = this.addForm.productTypeId[i];
                            }
                            para.productTypeId = b;
                            console.debug(para);
                            this.$http.post("/product/brand/add",para).then(
                                res=>{
                                    let {success,message,restObj} = res.data;
                                    if(success){
                                        this.addLoading = false;
                                        //NProgress.done();
                                        this.$message({
                                            message: '添加成功',
                                            type: 'success'
                                        });
                                        this.$refs['addForm'].resetFields();
                                        this.addFormVisible = false;
                                        this.getbrands();
                                    }else{
                                        this.$message({
                                            message: '添加失败',
                                            type: 'error'
                                        });
                                    }
                                }
                            ).catch({

                            });
                        });
                    }
                });
            },
            selsChange: function (sels) {
				this.sels = sels;
			},
			//批量删除
			batchRemove: function () {
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
                    this.$http.delete("/product/brand/deleteBatch?ids="+ids)
                        .then(res=>{
                            this.listLoading = false;
                            let {success,message,restObj} = res.data;
                            if (success){
                                this.$message({
                                    message: '删除成功',
                                    type: 'success'
                                });
                                this.getbrands();
                            }else {
                                this.$message({
                                    message: message,
                                    type: 'error'
                                });
                            }

                        })
				}).catch(() => {

				});
			}
		},
		mounted() {
			this.getbrands();
            this.loadTypeTree();
		}
	}

</script>

<style scoped>
    .el-cascader{
        margin-left:5px;
    }

</style>