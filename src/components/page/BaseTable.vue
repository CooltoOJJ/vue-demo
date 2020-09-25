<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item> <i class="el-icon-lx-cascades"></i> 基础表格 </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <el-button type="primary" icon="el-icon-delete" class="handle-del mr10" @click="delAllSelection">批量删除</el-button>
                <el-input v-model="query.studentName" placeholder="用户名" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="el-icon-search" @click="selectLike">搜索</el-button>
                <el-button type="primary" @click="handleCreate">新增</el-button>
            </div>
            <el-table
                :data="tableData"
                border
                class="table"
                ref="multipleTable"
                header-cell-class-name="table-header"
                @selection-change="handleSelectionChange"
            >
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column>
                <el-table-column prop="studentName" label="学生名"></el-table-column>

                <el-table-column prop="email" label="邮箱"></el-table-column>
                <el-table-column prop="age" label="年龄"></el-table-column>
                <el-table-column prop="createTime" label="注册时间"></el-table-column>
                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button type="text" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                        <el-button type="text" icon="el-icon-delete" class="red" @click="handleDelete(scope.$index, scope.row)"
                            >删除</el-button
                        >
                    </template>
                </el-table-column>
            </el-table>
            <!-- @size-change:选中当前页触发方法   
                 @current-change：点击下一页触发方法
                 :current-page.sync 当前页码
            -->
            <div class="pagination">
                <el-pagination
                    background
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page.sync="query.pageIndex"
                    layout="prev, pager, next"
                    :total="pageTotal"
                ></el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="70px">
                <el-form-item label="学生名">
                    <el-input v-model="form.studentName"></el-input>
                </el-form-item>
                <el-form-item label="邮箱">
                    <el-input v-model="form.email"></el-input>
                </el-form-item>
                <el-form-item label="年龄">
                    <el-input v-model="form.age"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 新增弹出框 -->
        <el-dialog title="新增" :visible.sync="createVisible" width="30%">
            <el-form ref="student" :model="student" label-width="70px">
                <el-form-item label="学生名">
                    <el-input v-model="student.studentName"></el-input>
                </el-form-item>
                <el-form-item label="年龄">
                    <el-input v-model="student.age"></el-input>
                </el-form-item>
                <el-form-item label="邮箱">
                    <el-input v-model="student.email"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="createVisible = false">取 消</el-button>
                <el-button type="primary" @click="createStudent">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
import { fetchData } from '../../api/index';
const axios = require('axios');
export default {
    name: 'basetable',
    data() {
        return {
            query: {
                studentName: '',
                pageIndex: 1,
                pageSize: 10
            },
            students: null,
            tableData: [],
            multipleSelection: [],
            delList: [],
            editVisible: false,
            createVisible: false,
            pageTotal: 0,
            form: {},
            student: {
                studentName: '',
                age: 0,
                email: ''
            }
            //pageNo: 1,
            //count: 5,
        };
    },
    created() {
        this.data();
    },
    methods: {
        handleSizeChange(val) {
            //console.log('handleSizeChange:'+this.pageNo);
            this.data();
        },
        handleCurrentChange(val) {
            //console.log('handleCurrentChange:'+this.pageNo);
            this.data();
        },
        data() {
            axios
                .post('http://192.168.0.11:8080/student/student/', this.query)
                .then((res) => {
                    console.log(res.data);
                    this.tableData = res.data.data.records;
                    this.pageTotal = res.data.data.pages;
                    console.log(this.tableData);
                })
                .catch((err) => {
                    console.log(err);
                });
        },
        // 获取 easy-mock 的模拟数据
        getData() {
            fetchData(this.query).then((res) => {
                console.log(res);
                this.tableData = res.list;
                this.pageTotal = res.pageTotal || 50;
            });
        },
        // 触发搜索按钮
        handleSearch() {
            this.$set(this.query, 'pageIndex', 1);
            this.getData();
        },
        // 删除操作
        handleDelete(index, row) {
            this.form = row;
            console.log(this.form);
            // 二次确认删除
            this.$confirm('确定要删除吗？', '提示', {
                type: 'warning'
            })
                .then(() => {
                    axios
                        .delete('http://192.168.0.11:8080/student/student/' + this.form.id)
                        .then((res) => {
                            this.$message.success(`删除成功`);
                            this.data();
                        })
                        .catch((err) => {
                            this.$message.error('删除失败');
                            console.log(err);
                        });
                })
                .catch(() => {});
        },
        // 多选操作
        handleSelectionChange(val) {
            this.delList = [];
            this.multipleSelection = val;
            this.multipleSelection.forEach((item) => {
                this.delList.push(item.id);
            });
            //console.log(this.multipleSelection);
            console.log(this.delList);
        },
        delAllSelection() {
            const length = this.multipleSelection.length;
            let str = '';
            for (let i = 0; i < length; i++) {
                str += this.multipleSelection[i].studentName + ' ';
            }
            axios
                .delete('http://192.168.0.11:8080/student/student/',{data:this.delList})
                .then((res) => {
                    this.$message.error(`删除了${str}`);
                    this.data();
                })
                .catch((err) => {
                    this.$message.error('删除失败');
                    console.log(err);
                });
        },
        handleCreate() {
            this.createVisible = true;
        },
        // 编辑操作
        handleEdit(index, row) {
            this.idx = index;
            this.form = row;
            this.editVisible = true;
        },
        // 保存编辑
        saveEdit() {
            this.editVisible = false;

            // this.$set(this.tableData, this.idx, this.form);
            axios
                .put('http://192.168.0.11:8080/student/update/', this.form)
                .then((res) => {
                    this.$message.success(`修改第 ${this.idx + 1} 行成功`);
                    this.data();
                })
                .catch((err) => {
                    this.$message.error('修改失败');
                    console.log(err);
                });
        },
        //根据字段模糊查询数据
        selectLike() {
            this.query.pageIndex = 1;
            console.log(this.query.name);
            this.data();
        },
        //新增
        createStudent() {
            this.createVisible = false;
            axios
                .put('http://192.168.0.11:8080/student/student/', this.student)
                .then((res) => {
                    this.$message.success('新增成功');
                    this.data();
                })
                .catch((err) => {
                    this.$message.error('新增失败');
                    console.log(err);
                });
        }
    }
};
</script>

<style scoped>
.handle-box {
    margin-bottom: 20px;
}

.handle-select {
    width: 120px;
}

.handle-input {
    width: 300px;
    display: inline-block;
}
.table {
    width: 100%;
    font-size: 14px;
}
.red {
    color: #ff0000;
}
.mr10 {
    margin-right: 10px;
}
.table-td-thumb {
    display: block;
    margin: auto;
    width: 40px;
    height: 40px;
}
</style>
