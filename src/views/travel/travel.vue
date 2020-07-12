<template>
    <div class="house-index">
        <wrapper-content>
            <div class="travel-search">
                <a-form
                        layout="inline"
                        :autoFormCreate="(form)=>{this.searchForm = form}"
                        @submit.prevent="handleSearchSubmit"
                >
                    <a-form-item
                            fieldDecoratorId="title1"
                    >
                        <a-input placeholder='请输入出差人部门'/>
                    </a-form-item>
                    <a-form-item
                            fieldDecoratorId="title2"
                    >
                        <a-input placeholder='请输入出差人姓名'/>
                    </a-form-item>
                    <a-form-item
                            fieldDecoratorId="date"
                    >
                        <a-date-picker style="width: 100%" placeholder='请输入出发时间' />
                    </a-form-item>
                    <a-form-item
                            fieldDecoratorId="title3"
                    >
                        <a-cascader :field-names="{ label: 'name', value: 'code', children: 'children' }"
                            :options="options" placeholder="请输入出发地" />
                    </a-form-item>
                    <a-form-item
                            fieldDecoratorId="title4"
                    >
                        <a-cascader :field-names="{ label: 'name', value: 'code', children: 'children' }"
                             :options="options" placeholder="请输入到达地" />
                    </a-form-item>
                    <a-form-item
                    >
                        <a-button class="m-r-sm" icon="search" type="primary" htmlType='submit'
                                :loading="searchLoading"
                        >搜索
                        </a-button>
                        <a-button class="m-r-sm" icon="reload" :loading="searchLoading"
                        >重置
                        </a-button>
                    </a-form-item>
                </a-form>
            </div>
            <div class="action">
                <a-button class="m-r-sm" type="primary" icon="plus" @click="doAction(null,'new')">新增</a-button>
                <a-button icon="check" class="m-r-sm" :disabled="!selectedRowKeys.length" @click="listAction({key:'setReadied'})">
                    <span>批量标记已读</span>
                </a-button>
                <a-button icon="delete" type="danger" :disabled="!selectedRowKeys.length" @click="listAction({key:'delete'})">
                    <span>批量删除</span>
                </a-button>
                <span style="padding-left: 12px;" v-show="selectedRowKeys.length">已选择 <span class="text-warning">{{selectedRowKeys.length}}</span> 项</span>
            </div>
            <a-table :columns="columns" :dataSource="dataSource" :loading="loading" :showTotal="pagination.showTotal" :pagination="pagination"
                     @change="pageChange"
                     :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
                     rowKey="id">
                <template slot="action" slot-scope="text,record,index">
                    <a @click="doAction(null,'select')">查看</a>
                    <Divider type="vertical"/>
                    <a @click="rowClick(record,'del', index)">删除</a>
                </template>
                
            </a-table>
        </wrapper-content>
        <a-modal
                class="travel-modal"
                :width="800"
                v-model="actionInfo.modalStatus"
                :title="actionInfo.modalTitle"
                :bodyStyle="{paddingBottom:'1px'}"
                :footer="null"
        >
        <div>
            <a-form :autoFormCreate="(form)=>{this.form = form}" @submit.prevent="handleSubmit" >
                <a-row :gutter="24">
                    <a-col  :span="12" :style="{ display: 'block' }">
                        <a-form-item label="出差人" :labelCol="{ span: 6 }" :wrapperCol="{ span: 18 }"
                            fieldDecoratorId="name3"
                            :fieldDecoratorOptions="{rules: [{ required: true, message: '请输入姓名' }]}"
                        >
                            <a-input placeholder='请输入出差人姓名'/>
                        </a-form-item>
                    </a-col>
                    <a-col :span="12">
                        <a-form-item label="出差时间" :labelCol="{ span: 6 }" :wrapperCol="{ span: 18 }"
                            fieldDecoratorId="name3"
                            :fieldDecoratorOptions="{rules: [{ required: true, message: '请输入出差时间' }]}"
                        >
                            <a-date-picker placeholder='请输入出差时间' style="width: 100%" />
                        </a-form-item>
                    </a-col>
                    <a-col :span="12">
                        <a-form-item label="出发地" :labelCol="{ span: 6 }" :wrapperCol="{ span: 18 }"
                            fieldDecoratorId="name3"
                            :fieldDecoratorOptions="{rules: [{ required: true, message: '请输入出发地' }]}"
                        >
                            <a-cascader :field-names="{ label: 'name', value: 'code', children: 'children' }"
                            :options="options" placeholder="请输入出发地" />
                        </a-form-item>
                    </a-col>
                    <a-col :span="12">
                        <a-form-item label="到达地" :labelCol="{ span: 6 }" :wrapperCol="{ span: 18 }"
                            fieldDecoratorId="name3"
                            :fieldDecoratorOptions="{rules: [{ required: true, message: '请输入到达地' }]}"
                        >
                            <a-cascader :field-names="{ label: 'name', value: 'code', children: 'children' }"
                            :options="options" placeholder="请输入到达地" />
                        </a-form-item>
                    </a-col>
                    <a-col :span="24">
                        <a-form-item label="出差事由" :labelCol="{ span: 3 }" :wrapperCol="{ span: 21 }"
                            fieldDecoratorId="name3"
                            :fieldDecoratorOptions="{rules: [{ required: true, message: '出差事由' }]}"
                        >
                            <a-input placeholder='请输入出差事由'
                                v-decorator="['name', {rules: [{ required: true, message: '请输入出差人姓名' }]}]"/>
                        </a-form-item>
                    </a-col>
                    <a-col :span="24">
                        <a-form-item label="备注" :labelCol="{ span: 3 }" :wrapperCol="{ span: 21 }">
                            <a-textarea :rows="4" />
                        </a-form-item>
                    </a-col>
                    <a-col :span="24">
                        <a-form-item label="调研函" :labelCol="{ span: 3 }" :wrapperCol="{ span: 21 }">
                            <a-upload
                                    name="cover"
                                    class="cover-uploader"
                                    :file-list="fileList"
                                    :headers="headers"
                                    :customRequest="updloadFile"
                                    :action="uploadAction"
                                    :beforeUpload="beforeUpload"
                                    :remove="removeFile"
                                    
                            >
                                <a-button size="large" class="upload">上传文件</a-button>
                            </a-upload>
                        </a-form-item>
                    </a-col>
                    <a-col :span="24">
                        <a-form-item label="缴费凭证" :labelCol="{ span: 3 }" :wrapperCol="{ span: 21 }">
                            <a-upload
                                    name="cover"
                                    class="cover-uploader"
                                    
                                    :headers="headers"
                                    :action="uploadAction"
                                    :beforeUpload="beforeUpload"
                                    @change="handleChange"
                            >
                                <a-button size="large" class="upload">上传文件</a-button>
                            </a-upload>
                        </a-form-item>
                    </a-col>
                    <a-col :span="24">
                        <a-form-item label="调研报告" :labelCol="{ span: 3 }" :wrapperCol="{ span: 21 }">
                            <a-upload
                                    name="cover"
                                    class="cover-uploader"
                                    
                                    :headers="headers"
                                    :action="uploadAction"
                                    :beforeUpload="beforeUpload"
                                    @change="handleChange"
                            >
                                <a-button size="large" class="upload">上传文件</a-button>
                            </a-upload>
                        </a-form-item>
                    </a-col>
                </a-row>
                <a-form-item :wrapper-col="{ span: 12, offset: 5 }">
                    <div class="action-btn" style="text-align: center">
                        <a-button type="primary" htmlType='submit'
                                size="large"
                                :loading="actionInfo.confirmLoading"
                                class="middle-btn">保存
                        </a-button>
                        <a-button class="middle-btn" size="large" @click="actionInfo.modalStatus = false">取消
                        </a-button>
                    </div>
                </a-form-item>
            </a-form>
        </div>
        </a-modal>
    </div>
</template>
<script>
    import {Divider} from 'ant-design-vue';
    import {list, del} from '@/api/notify';
    import {checkResponse, getApiUrl, getAuthorization, getBase64} from '@/assets/js/utils';
    import {batchDel, setReadied} from "../../api/notify";
    import pagination from "@/mixins/pagination";
    import areaData from "../../config/area.json";
    import $http from '@/assets/js/http'

    const columns = [{
        title: '出差人部门',
        dataIndex: 'department',
    },{
        title: '出差人姓名',
        dataIndex: 'name',
    },{
        title: '出发时间',
        dataIndex: 'time',
    },{
        title: '出发地',
        dataIndex: 'address',
    },{
        title: '到达地',
        dataIndex: 'add',
    },{
        title: '出差事由',
        dataIndex: 'details',
    },{
        title: '操作',
        scopedSlots: {
            customRender: 'action'
        }
    }];

    export default {
        components: {
            Divider,
        },
        mixins: [pagination],
        data() {
            return {
                columns,
                selectedRowKeys: [],
                loading: false,
                searchForm: {},
                searchLoading: false,
                options: areaData,
                formInfo: {},
                actionInfo: {
                    modalStatus: false,
                    confirmLoading: false,
                    modalTitle: '编辑任务',
                },
                fileList: [
                    {
                    uid: 'ds',
                    name: 'xxx.png',
                    status: 'done',
                    response: 'Server Error 500', // custom error message to show
                    url: 'http://www.baidu.com/xxx.png',
                    },
                    {
                    uid: 'dsa',
                    name: 'yyy.png',
                    status: 'done',
                    url: 'http://www.baidu.com/yyy.png',
                    },
                ],
                uploadAction: getApiUrl('project/travel/upload'),
                dataSource: [
                    {
                        department: '研发',
                        name: '小明',
                        time: '2020-10-11',
                        address: '北京',
                        add: '上海',
                        details: 'x项目'
                    },
                    {
                        department: '研发',
                        name: '小明',
                        time: '2020-10-11',
                        address: '北京',
                        add: '上海',
                        details: 'x项目'
                    },
                    {
                        department: '研发',
                        name: '小明',
                        time: '2020-10-11',
                        address: '北京',
                        add: '上海',
                        details: 'x项目'
                    },
                ],
            }
        },
        computed: {
            headers() {
                return getAuthorization();
            },
        },
        created() {
            this.init();
        },
        methods: {
            init() {
               
            },
            onSelectChange(selectedRowKeys) {
                this.selectedRowKeys = selectedRowKeys
            },
            listAction(type) {
                
            },
            rowClick(record, action) {
                

            },
            handleSearchSubmit() {
                let app = this;
                app.searchForm.validateFields(
                    (err, values) => {
                        if (!err) {
                            app.search();
                        }
                    })
            },
            search(){
                let obj = this.searchForm.getFieldsValue();
                if (obj.date && obj.date.length > 0) {
                    const beginDate = obj.date[0].format('YYYY-MM-DD');
                    const endDate = obj.date[1].format('YYYY-MM-DD');
                    obj.date = beginDate + '~' + endDate;
                }
                Object.assign(this.requestData, obj);
                this.init();
            },
            //查看和新增
            doAction(record, action) {
                let app = this;
                if (action == 'new') {
                    app.actionInfo.modalStatus = true;
                    app.actionInfo.modalTitle = '添加出差信息';
                    
                } else if (action == 'select') {
                    app.actionInfo.modalStatus = true;
                    app.actionInfo.modalTitle = '查看出差信息';
                }
            },
            //表单信息提交
            handleSubmit(){
                let app = this;
                app.form.validateFields(
                    (err, values) => {
                        if (!err) {
                            app.handleOk();
                        }
                    })
            },
            handleOk() {
                let app = this;
                app.actionInfo.confirmLoading = true;
                let obj = app.form.getFieldsValue();
                if (app.newData.code) {
                    //编辑
                    obj.code = app.newData.code;
                } else {
                    //新增
                    Object.assign(obj, app.newData);
                }
                doAccount(obj).then(res => {
                    app.actionInfo.confirmLoading = false;
                    if (!checkResponse(res)) {
                        return;
                    }
                    if (app.newData.code) {
                        app.newData.email = obj.email;
                        app.newData.name = obj.name;
                        app.newData.mobile = obj.mobile;
                        app.newData.description = obj.description;
                    } else {
                        app.dataSource.push(res.data);
                    }
                    app.form.resetFields();
                    app.newData = {id: 0};
                    app.actionInfo.modalStatus = false;

                });
            },
            //文件限制
            beforeUpload(file) {
                const isLt2M = file.size / 1024 / 1024 < 2;
                if (!isLt2M) {
                    this.$message.error('图片不能超过2MB!')
                }
                return isLt2M
            },
            //上传
            handleChange(info) {
                debugger
                if (info.file.status === 'uploading') {
                    this.uploadLoading = true;
                    return
                }
                if (info.file.status === 'done') {
                    this.project.cover = info.file.response.data.url;
                    this.uploadLoading = false;
                    notice({
                        title: '封面上传成功',
                    }, 'notice', 'success');
                }
            },
            updloadFile(data){
                debugger
                console.info(data);
                $http.post('project/task/index', data).then(res => {
                    debugger
                    let fileInfo = {
                    uid: 'dsww',
                    name: 'xxx.png',
                    status: 'done',
                    response: 'Server Error 500', // custom error message to show
                    url: 'http://www.baidu.com/xxx.png',
                    };
                    this.fileList.push(fileInfo);
                    
                    data.onSuccess()
                    console.info(res);
                });
            },
            //移除文件
            removeFile(data){
                debugger
                console.info(data);
            }
        }
    }
</script>
<style>
/* .ant-advanced-search-form .ant-form-item {
  display: flex;
  width: "600px";
} */
.travel-search .ant-form-item-control {
    width: 230px;
}

</style>
