<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="关键字"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getProducts">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleViewProperties">显示属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleSkuProperties">SKU属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleOnSale">上架</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleOffSale">下架</el-button>
                </el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="products" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="50">
			</el-table-column>
			<el-table-column prop="name" label="标题" width="150" sortable align="center">
			</el-table-column>
			<el-table-column prop="subName" label="副标题" width="130"  sortable align="center">
			</el-table-column>
			<el-table-column prop="productType.name" label="商品类型" width="120" sortable align="center">
			</el-table-column>
			<el-table-column prop="brand.name" label="品牌" width="120" sortable align="center">
			</el-table-column>
			<el-table-column prop="onSaleTime" label="上架时间" min-width="120" sortable :formatter="formatOnSaleTime" align="center">
			</el-table-column>
            <el-table-column prop="offSaleTime" label="下架时间" min-width="120" sortable :formatter="formatOffSaleTime" align="center">
            </el-table-column>
            <el-table-column prop="state" label="状态" min-width="100" sortable align="center">
                <template scope="scope">
                    <span style="color: green" v-if="scope.row.state==1">上架</span>
                    <span style="color: red" v-else>下架</span>
                </template>
            </el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="loadExtdate(scope.$index, scope.row),handleEdit(scope.$index, scope.row)">编辑</el-button>
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
                <el-form-item label="标题" prop="name">
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="副标题" prop="subName">
                    <el-input v-model="editForm.subName" auto-complete="off"></el-input>
                </el-form-item>

                <el-form-item label="媒体属性">
                    <template>
                        <el-upload
                                class="upload-demo"
                                action="http://127.0.0.1:8082/services/common/file"
                                list-type="picture"
                                :on-success="saveimg"
                                :before-upload="saveBefo"
                                :on-remove="changeremove"
                                :file-list="fileList">
                            <el-button size="small" type="primary">点击上传</el-button>
                        </el-upload>
                    </template>
                </el-form-item>

                <el-form-item label="品牌">
                    <!--<el-input v-model="addForm.brandId" auto-complete="off">
                    </el-input>-->
                    <el-select v-model="editForm.brandId" clearable
                               placeholder="请选择品牌">
                        <el-option
                                v-for="item in brands"
                                :label="item.name"
                                :value="item.id">
                        </el-option>
                    </el-select>
                </el-form-item>

                <div class="block" style="margin-left: 15px">
                    <span class="demonstration">商品类型</span>
                    <el-cascader
                            :show-all-levels="false"
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

                <el-form-item label="商品描述">
                    <el-input type="textarea" v-model="editForm.ext.description"
                              auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="商品详情">
                    <quill-editor
                            ref="QuillEditor"
                            v-model="editForm.ext.richContent"
                    ></quill-editor>
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
				<el-form-item label="标题" prop="name">
					<el-input v-model="addForm.name" auto-complete="off"></el-input>
				</el-form-item>
                <el-form-item label="副标题" prop="subName">
                    <el-input v-model="addForm.subName" auto-complete="off">
                    </el-input>
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

                <el-form-item label="品牌">
                    <el-select v-model="addForm.brandId" clearable
                               placeholder="请选择品牌">
                        <el-option
                                v-for="item in brands"
                                :label="item.name"
                                :value="item.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="媒体属性">
                    <template>
                        <el-upload
                                class="upload-demo"
                                action="http://127.0.0.1:8082/services/common/file"
                                list-type="picture"
                                :on-success="saveimg"
                                :before-upload="saveBefo"
                                :on-remove="changeremove"
                                :file-list="fileList">
                            <el-button size="small" type="primary">点击上传</el-button>
                        </el-upload>
                    </template>
                </el-form-item>
                <el-form-item label="商品描述">
                    <el-input type="textarea" v-model="addForm.ext.description"
                              auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="商品详情">
                    <quill-editor
                            ref="QuillEditor"
                            v-model="addForm.ext.richContent"
                    ></quill-editor>
                </el-form-item>

			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="addFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
			</div>
		</el-dialog>

        <!--显示属性维护-->
          <el-dialog size="tiny" title="显示属性" v-model="viewPropertiesDialogVisible" :close-on-click-modal="false">
            <el-form label-width="80px">
              <el-form-item v-for="viewProperty in viewProperties"
                            :label="viewProperty.specName">
                    <el-input v-model="viewProperty.value" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
              <el-button @click.native="viewPropertiesDialogVisible = false">取消</el-button>
              <el-button type="primary" @click.native="handleSaveViewProperties">提交</el-button>
            </div>
          </el-dialog>

        <!--SKU属性维护-->
        <el-dialog title="SKU属性" v-model="skuPropertiesDialogVisible" :close-on-click-modal="false">
                <!--外层循环展示sku属性-->
                <el-card class="box-card" v-for="(skuProperty,i) in skuProperties">
                  <div slot="header" class="clearfix">
                    <span style="line-height: 36px;">{{skuProperty.specName}}</span>
                  </div>
                  <!--内层循环展示sku属性选项-->
                  <div v-for="index in skuProperty.options.length+1" class="textitem">
                    <el-row>
                      <el-col :span="18">
                        <el-input v-model="skuProperty.options[index-1]" auto-complete="off"></el-input>
                      </el-col>
                      <el-col :span="6">
                        <el-button @click="removeProperty(i,index-1)">删除</el-button>
                      </el-col>
                    </el-row>
                  </div>
                </el-card>

            <!--sku动态table的展示-->
            <el-table :data="skus" highlight-current-row style="width: 100%;">
                  <el-table-column v-for="(value,key) in skus[0]" :label="key" :prop="key">
                  </el-table-column>
            </el-table>

            <div slot="footer" class="dialog-footer">
                <el-button @click.native="skuPropertiesDialogVisible = false">取消</el-button>
                <el-button type="primary" @click.native="handleSaveSkuProperties">提交</el-button>
            </div>
        </el-dialog>




	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getProductListPage, removeProduct, batchRemoveProduct, editProduct, addProduct } from '../../api/api';

	export default {
		data() {
			return {
                fileList:[],//用作回显
                brands:[],
                defaultParams: {
                    label: 'name',
                    value: 'id',
                    children: 'children'
                },
                //显示属性
                viewProperties:[],
                //显示属性维护的模态框
                viewPropertiesDialogVisible:false,
                //sku属性
                skuProperties:[],
                skus:[],
                //显示属性维护的模态框
                skuPropertiesDialogVisible:false,
                producttypes:[],
				filters: {
                    keyword: ''
				},
				products: [],
				total: 0,
				page: 1,
                rows:10,
                options: [],
                productOptions:[],
				listLoading: false,
				sels: [],//列表选中列

				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//编辑界面数据
				editForm: {
                    name: '',
                    subName: '',
                    productTypeId: null,
                    brandId: null,
                    medias:'',
                    productTypes:[],
                    ext:{
                        description:'',
                        richContent:''
                    }
				},

				addFormVisible: false,//新增界面是否显示
				addLoading: false,
                addFormRules: {
                    name: [
                        { required: true, message: '请输入标题', trigger: 'blur' }
                    ],
                    subName: [
                        { required: true, message: '请输入副标题', trigger: 'blur'
                        }
                    ]
                },
                //新增界面数据
                addForm: {
                    name: '',
                    subName: '',
                    productTypeId: [],
                    brandId: null,
                    medias:'',
                    productTypes:[],
                    ext:{
                        description:'',
                        richContent:''
                    }
                }
			}
		},
		methods: {
            //删除sku属性选项
            removeProperty(index1,index2){
                //index1 第几个sku属性
                //index2 属性的第几个选项
                this.skuProperties[index1].options.splice(index2,1);
                //删除options的索引为index2的元素
                //index 从第几个开始删除  count 删除几个 item... 删除完之后在num处添加几个
                //数组对象.splice(index,count,item...)
            },
            loadExtdate(index,row){
                this.$http.get("/product/productExt/findOne?productId="+row.id).then(res=>{
                    // this.editForm.ext.description = res.data.description;
                    // this.editForm.ext.richContent = res.data.richContent;
                    this.editForm.ext = res.data;
                }).catch({});
            },
		    //获取品牌
            getBrands(){
                this.$http.get("/product/brand/list")
                    .then(res=>{
                        this.brands = res.data;
                    })
            },
            //加载类型树
            loadTypeTree(){
                this.$http.get("/product/productType/loadTypeTree")
                    .then(res=>{
                        console.debug(res);
                        /*this.options = this.productTypes = res.data;*/
                        this.options = res.data;
                        this.getTreeData(this.options);
                    })
            },
            // 递归方法:去掉tree里面的空数组
            getTreeData(data) {
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
            },
            handleChange(){

            },
            //文件上传成功前的钩子函数
            saveBefo(file){
                /*//console.log("filecount",this.fileList.length);
                if(this.fileList.length>0){
                    this.$message({
                        message: '只能上传一张logo图片',
                        type: 'warning'
                    });
                    return false;//停止上传
                }*/
            },
            //文件上传成功后的钩子函数
            saveimg(response,file,fileList){
                let {success,message,restObj} = response;
                if(success){
                    this.$message({
                        message: '上传成功',
                        type: 'success'
                    });
                    this.addForm.logo = restObj;
                    this.editForm.logo = restObj;
                }else{
                    this.$message({
                        message: message,
                        type: 'error'
                    });
                }
                this.fileList = fileList;
            },
            changeremove(file,fileList){
                let fileId = '';
                if(file.size){
                    fileId =file.response.restObj;
                console.debug("fileId================"+fileId)
                }else{
                    fileId =file.url.slice(19);
                }
                this.$http.delete("/common/file?fileId="+fileId)
                    .then(res=>{
                        if(res.data.success){
                            this.$message({
                                message: '删除成功!',
                                type: 'success'
                            });
                            this.fileList=[];
                        }else{
                            this.$message({
                                message: '删除失败!',
                                type: 'error'
                            });
                            this.fileList=[];
                        }
                    })
            },
            handleRemove(file, fileList){
                console.log(file, fileList);
            },
            handlePreview(file){
                console.log(file);
            },
            getBrands(){
                this.$http.get("/product/brand/list")
                    .then(res=>{
                        this.brands = res.data;
                    })
            },


            //显示属性维护
            handleViewProperties(){
                //只能选中一行数据
                if(this.sels.length==0){
                    this.$message({
                        message: '请选中一行数据',
                        type: 'warning'
                    });
                    return;
                }
                if(this.sels.length>1){
                    this.$message({
                        message: '只能选中一行数据',
                        type: 'warning'
                    });
                    return;
                }
                let productId = this.sels[0].id;
                //查询要维护商品的显示属性
                this.$http.get("/product/product/viewProperties/"+productId)
                    .then(res=>{
                        this.viewProperties = res.data;
                    });
                //打开模态框
                this.viewPropertiesDialogVisible = true;
            },
            //显示属性保存
            handleSaveViewProperties(){
                let productId = this.sels[0].id;
                this.$confirm('确认保存吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$http.post("/product/product/updateViewProperties?productId="+productId,this.viewProperties)
                        .then(res=>{
                            let {success,message,restObj} = res.data;
                            if(success){
                                this.$message({
                                    message: '保存成功!',
                                    type: 'success'
                                });
                                this.viewPropertiesDialogVisible =
                                    false;
                            }else{
                                this.$message({
                                    message: message,
                                    type: 'error'
                                });
                            }
                        })
                }).catch(() => {

                });
            },
            //sku属性维护
            handleSkuProperties(){
                //只能选中一行数据
                if(this.sels.length==0){
                    this.$message({
                        message: '请选中一行数据',
                        type: 'warning'
                    });
                    return;
                }
                if(this.sels.length>1){
                    this.$message({
                        message: '只能选中一行数据',
                        type: 'warning'
                    });
                    return;

                }
                let productId = this.sels[0].id;
                //查询要维护商品的显示属性
                this.$http.get("/product/product/skuProperties/"+productId)
                    .then(res=>{
                        console.debug(res);
                        this.skuProperties = res.data;
                    });
                //打开模态框
                this.skuPropertiesDialogVisible = true;
            },
            //sku属性保存
            handleSaveSkuProperties(){

            },
            //上架
            handleOnSale(){},
            //下架
            handleOffSale(){},
            //格式化时间
            formatOnSaleTime(row, column){
                return this.formatTime(row.onSaleTime)
            },
            formatOffSaleTime(row, column){
                return this.formatTime(row.offSaleTime)
            },
            formatTime(time){
                if(!time){
                    return null;
                }
                let date = new Date(time);
                let year = date.getFullYear();
                let month = date.getMonth()+1;
                let day = date.getDate();//获取一个月中的第几天
                let hour = date.getHours();
                let minute = date.getMinutes();
                let second = date.getSeconds();
                let timeStr = year+"-"+this.formatTimeNum(month)+"-"+this.formatTimeNum(day)+" "+this.formatTimeNum(hour)+":"+this.formatTimeNum(minute)+":"+this.formatTimeNum(second);
                return timeStr;
            },
            formatTimeNum(num){
                if(num>=10){
                    return num;
                }
                return "0"+num;
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
			//性别显示转换
			formatSex: function (row, column) {
				return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
			},
			handleCurrentChange(val) {
				this.page = val;
				this.getProducts();
			},
			//获取商品列表
			getProducts() {
				let para = {
                    page: this.page,
                    rows: this.rows,
                    keyword: this.filters.keyword
				};
				this.listLoading = true;
                this.$http.post("/product/product/json",para)
                    .then(res=>{
                        this.listLoading = false;
                        let {total,rows} = res.data;
                        this.total = total;
                        this.products = rows;
                        console.debug(rows);
                    })
				//NProgress.start();
				/*getProductListPage(para).then((res) => {
					this.total = res.data.total;
					this.products = res.data.products;
					this.listLoading = false;
					//NProgress.done();
				});*/
			},
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    this.$http.delete("/product/product/delete/"+ row.id)
                        .then(res=>{
                            this.listLoading = false;
                            let {success,message,restObj} = res.data;
                            if (success){
                                this.$message({
                                    message: '删除成功',
                                    type: 'success'
                                });
                                this.getProducts();
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
                console.debug("row------------",row);
                this.fileList = [];
				this.editFormVisible = true;
				this.editForm = Object.assign({}, row);
                this.fileList.push({"url":"http://172.16.4.218" + row.medias});
                this.loadGetPath(index, row);
                console.debug(this.editForm);
			},
            //显示新增界面
            handleAdd: function () {
                this.fileList = [];
                this.addFormVisible = true;
                this.addForm = {
                    name: '',
                    subName: '',
                    productTypeId: [],
                    brandId: null,
                    medias:'',
                    productTypes:[],
                    ext:{
                        description:'',
                        richContent:''
                    }
                };
            },
			//编辑
			editSubmit: function (file) {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.addLoading = true;
                            let para = Object.assign({}, this.editForm);
                            for(var i = 0;i<this.productOptions.length ; i++){
                                var b = this.productOptions[i];
                            }
                            //从fileList中获取文件路径用，拼接给medias赋值
                            console.debug(this.fileList);
                            if(file.response != null){
                                para.medias = this.fileList.map(file=>file.url).join(",");
                            }else{
                                para.medias = this.fileList.map(file=>file.response.restObj).join(",");
                            }
                            console.debug("-----------------"+para.medias)
                            /*para.medias = this.fileList;*/
                            para.productTypeId = b;
                            this.$http.post("/product/product/add",para).then(
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
                                        this.getProducts();
                                    }else{
                                        this.$message({
                                            message: '修改失败',
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
                            let para = Object.assign({}, this.addForm);
                            for(var i = 0;i<this.addForm.productTypeId.length ; i++){
                                var b = this.addForm.productTypeId[i];
                            }
                            //从fileList中获取文件路径用，拼接给medias赋值
                            para.medias = this.fileList.map(file=>file.response.restObj).join(",");
                            para.productTypeId = b;
                            this.$http.post("/product/product/add",para).then(
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
                                        this.fileList = [];
                                        this.getProducts();

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
                    this.$http.delete("/product/product/deleteBatch?ids="+ids)
                        .then(res=>{
                            this.listLoading = false;
                            let {success,message,restObj} = res.data;
                            if (success){
                                this.$message({
                                    message: '删除成功',
                                    type: 'success'
                                });
                                this.getProducts();
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
			this.getProducts();
			this.loadTypeTree();
			this.getBrands();
		},
        //核心方法，监听skuProperties属性值的变化
        watch:{
            skuProperties:{
                handler(val,oldval){
                    //过滤掉options为空数组的sku属性
                    let skuPropertiesArr = this.skuProperties.filter(e=>e.options.length>0);
                    let result = skuPropertiesArr.reduce((pre,cur,currentIndex)=>{
                        //pre  [{}]
                        //cur {specName:"年龄",options:["白皙","小麦黄"]}
                        //结果: [{年龄:"白皙"},{"年龄":"小麦黄"}]
                        let temp = [];
                        pre.forEach(e1=>{ //e1 {} 如果是第二次reduce{年龄:"萝莉"}
                            cur.options.forEach((e2,index)=>{ //e2 "白皙"  "小麦黄"
                                let obj = Object.assign({},e1);
                                obj[cur.specName] = e2;
                                //判断是否是最后一次reduce
                                if(currentIndex==skuPropertiesArr.length-1){
                                    obj.price = 0;
                                    obj.store = 0;
                                }
                                temp.push(obj);
                            })
                        });
                        return temp;
                    },[{}]);

                    this.skus = result;
                },
                deep:true
            }
        }
	}

</script>

<style scoped>
    .el-cascader{
        margin-left:5px;
    }
</style>