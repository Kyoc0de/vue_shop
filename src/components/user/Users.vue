<template>
    <div>
<!--       面包屑 导航-->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
<!--        卡片视图-->
        <el-card class="box-card">
<!--            搜索与添加区域 -->
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
                        <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>
<!--            用户列表区域-->
            <el-table :data="userlist" border stripe>
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column label="姓名" prop="username" width="200px"></el-table-column>
                <el-table-column label="邮箱" prop="email" width="500px"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                <el-table-column label="角色" prop="role_name"></el-table-column>
                <el-table-column label="采纳状态" prop="mg_state">
                    <template slot-scope="scope">
                        <el-switch @change="userStateChanged(scope.row)"
                                v-model="scope.row.mg_state">
                        </el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="180px" >
                    <template slot-scope="scope">
<!--                        修改-->
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
<!--                        删除-->
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
<!--                        分配-->
                        <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>
<!--            分页区域-->
            <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="queryInfo.pagenum"
                    :page-sizes="[1, 2, 3, 4]"
                    :page-size="queryInfo.pagesize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="total">
            </el-pagination>
        </el-card>
<!--添加用户对话框-->
        <el-dialog
                title="添加用户"
                :visible.sync="addDialogVisible"
                width="60%"
                @close="addDialogClosed"
        >
<!--            内容主体区域-->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
<!--            底部区域    -->
            <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addUser">确 定</el-button>
  </span>
        </el-dialog>

<!--        修改用户对话框-->
        <el-dialog
                title="修改用户"
                :visible.sync="editDialogVisible"
                width="60%"
                @close="editDialogClosed"
        >
            <!--            内容主体区域-->
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
                <el-form-item label="用户名" >
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <!--            底部区域    -->
            <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editUserInfo">确 定</el-button>
  </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        data(){
            return {
                //获取用户列表的参数对象
                queryInfo:{
                    query: '',
                    pagenum: 1,
                    pagesize: 2
                },
                userlist: [],
                total: 0,
                //控制对话框显示
                addDialogVisible: false,
                //控制修改用户对话框
                editDialogVisible: false,
                //添加用户表单数据
                addForm:{
                    username: '',
                    password: '',
                    email: '',
                    mobile: ''
                },
                //编辑用户数据表单
                editForm:{

                },
                //添加表单的验证规则
                addFormRules:{
                    username:[
                        { required: true, message: '请输入用户名字', trigger: 'blur'},
                        { min:3, max:10, message: '用户名长度3-10', trigger: 'blur'}
                    ],
                    password:[
                        { required: true, message: '请输入密码', trigger: 'blur'},
                        { min:3, max:10, message: '用户名长度3-10', trigger: 'blur'}
                    ],
                    email:[
                        { required: true, message: '请输入邮箱', trigger: 'blur'}
                    ],
                    mobile:[
                        { required: true, message: '请输入手机号', trigger: 'blur'}
                    ]
                },
                editFormRules:{
                    username:[
                        { required: true, message: '请输入用户名字', trigger: 'blur'},
                        { min:3, max:10, message: '用户名长度3-10', trigger: 'blur'}
                    ],
                    email:[
                        { required: true, message: '请输入邮箱', trigger: 'blur'}
                    ],
                    mobile:[
                        { required: true, message: '请输入手机号', trigger: 'blur'}
                    ]
                }
            }
        },
        created() {
            this.getUserList()
        },
        methods:{
            async getUserList(){
                const { data: res } = await this.$http.get('users',{params: this.queryInfo})
                if(res.meta.status !== 200) return this.$message.error('获取用户列表失败!')
                this.userlist = res.data.users
                this.total = res.data.total
            },
        //    监听pagesize改变事件
            handleSizeChange(newSize){
                // console.log(newSize)
                this.queryInfo.pagesize = newSize
                this.getUserList()
            },
        //    监听页码改变事件
            handleCurrentChange(newPage){
                this.queryInfo.pagenum = newPage
                this.getUserList()
            },
        //
            async userStateChanged(userinfo){
                // console.log(userinfo)
                const { data:res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
                if(res.meta.status !== 200){
                    userinfo.mg_state = !userinfo.mg_state
                    return this.$message.error('更新用户状态失败! ')
                }
                this.$message.success('更新用户状态成功!')
            },
            //监听对话框开启与关闭
            addDialogClosed(){
                this.$refs.addFormRef.resetFields()
            },
            editDialogClosed(){
                this.$refs.editFormRef.resetFields()
            },
            addUser(){
                this.$refs.addFormRef.validate(async valid => {
                    // console.log(valid)
                    if (!valid) return
                    const {data: res} = await this.$http.post('users/', this.addForm)
                    if(res.meta.status !== 201){
                        this.$message.error('添加用户失败!')
                    }

                    this.$message.success('添加用户成功!')
                    this.addDialogVisible = false
                    this.getUserList()
                })

            },
            //展示编辑用户对话框
            async showEditDialog(id){
                const { data:res } = await this.$http.get('users/' + id)
                if(res.meta.status !== 200){
                    return this.$message.error("查询用户失败!")
                }
                this.editForm = res.data
                this.editDialogVisible = true

            },
            //修改用户信息并提交
            editUserInfo(){
                this.$refs.editFormRef.validate(async valid => {
                    if(!valid) return
                    //发起修改用户的请求
                    const {data: res} = await this.$http.put('users/', this.editForm.id,
                        {
                        email: this.editForm.email,
                        mobile: this.editForm.mobile
                        }
                    )
                    if(res.meta.status !== 200){
                        this.$message.error('修改用户失败!')
                    }

                    this.$message.success('修改用户成功!')
                    this.editDialogVisible = false
                    this.getUserList()
                })
            },
            //根据id删除
            async removeUserById(id){
                //弹框提升
                const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch((err) => {
                  return err
                });

                if(confirmResult !== 'confirm'){
                    return this.$message.info("已取消删除")
                }
                const { data:res } = await this.$http.delete('users/' + id)
                if(res.meta.status !== 200){
                    return this.$message.error('删除失败')
                }
                this.$message.success('删除成功')
                this.getUserList()
            }
        }
    }
</script>

<style lang="less" scoped>

</style>