<style lang="less">
    @import '../../styles/common.less';
    @import '../../styles/table.less';
</style>

<template>
    <div>
        <Card>
            <p slot="title">
                <Icon type="code"></Icon>
                Scratch作品发布管理
            </p>
            <Row :gutter="16">
                <Col span="18"> 
                    <Input v-model="searchParam.name" placeholder="输入名称搜索..." style="width: 200px" />
                    <Input v-model="searchParam.student" placeholder="输入学生搜索..." style="width: 200px" />
                    <span @click="handleSearch" style="margin: 0 10px;"><Button type="primary" icon="search">搜索</Button></span>
                    <Button @click="handleCancel" type="ghost" >取消</Button>
                </Col>
                <Col span="6" class="align-right">
                    <Button @click="showAdd" type="primary" >新建</Button>
                </Col>
            </Row>
        </Card>    
        <Row class="margin-top-10 searchable-table-con1">
            <Table :columns="columns" :data="programList" :loading="loading"></Table>
        </Row>


        <Modal v-model="programModal" :closable='false' :mask-closable=false :width="500">
            <h3 slot="header" style="color:#2D8CF0">{{modalTitle}}</h3>
            <Form ref="programForm" :model="programForm" :label-width="100" label-position="right" :rules="programRules">
                <FormItem label="名称" prop="name">
                    <Input v-model="programForm.name" placeholder="请输入作品名称"></Input>
                </FormItem>
                <FormItem label="学生" prop="student">
                    <Input v-model="programForm.student" placeholder="请输入学生姓名"></Input>
                </FormItem>
                <FormItem label="描述" prop="desc">
                    <Input v-model="programForm.desc" type="textarea" :autosize="{minRows: 2,maxRows: 5}" placeholder="请输入作品描述"></Input>
                </FormItem>
                <FormItem label="封面图片" prop="imageUrl">
                    <Input v-model="programForm.imageUrl" placeholder="请输入封面图片地址"></Input>
                </FormItem>
                <FormItem label="作品文件" prop="assetUrl">
                    <Input v-model="programForm.assetUrl" placeholder="请输入作品文件地址"></Input>
                </FormItem>
            </Form>
            <div slot="footer">
                <Button type="text" @click="cancelModal">取消</Button>
                <Button type="primary" :loading="saveProgramLoading" @click="saveProgram">保存</Button>
            </div>
        </Modal>
    </div>
    
</template>

<script>
import {mockProgamList} from './data/mock-data';
export default {
    name: 'program-list',
    data () {
        return {
            loading: true,
            programList: [],
            initProgramList: [],
            searchParam: {
                name: '',
                student: ''
            },
            columns: [
                {key: 'name', title: '作品名称'},
                {key: 'desc', title: '作品介绍', width: 200},
                {key: 'student', title: '学生'},
                {key: 'image', title: '封面图'},
                {key: 'asset', title: '文件'},
                {
                    key: 'publishTime',
                    title: '发布时间',
                    render: (h, params) => {
                        return h('span', this.convertDate(params.row.publishTime));
                    }
                },
                {key: 'publishUser', title: '发布老师'},
                {
                    title: 'Action',
                    key: 'action',
                    width: 150,
                    align: 'center',
                    render: (h, params) => {
                        return h('div', [
                            h('Button', {
                                props: {
                                    type: 'primary',
                                    size: 'small'
                                },
                                style: {
                                    marginRight: '5px'
                                },
                                on: {
                                    click: () => {
                                        this.showEdit(params.row, params.index);
                                    }
                                }
                            }, '修改'),
                            h('Button', {
                                props: {
                                    type: 'error',
                                    size: 'small'
                                },
                                on: {
                                    click: () => {
                                        this.remove(params.index);
                                    }
                                }
                            }, '删除')
                        ]);
                    }
                }
            ],
            programModal: false,
            saveProgramLoading: false,
            programForm: {
                action: '',
                index: 0,
                programId: '',
                name: '',
                student: '',
                desc: '',
                imageUrl: '',
                assetUrl: ''
            },
            programRules: {
                name: [
                    { required: true, message: '请输入作品名称', trigger: 'blur' }
                ],
                student: [
                    { required: true, message: '请输入学生姓名', trigger: 'blur' }
                ]
            }
        };
    },
    computed: {
        modalTitle () {
            return this.programForm.action === 'create' ? '添加作品'
                : this.programForm.action === 'update' ? '修改作品' : '';
        }
    },
    methods: {
        init () {
            this.programList = this.initProgramList = mockProgamList;
            setTimeout(() => {
                this.loading = false;
            }, 1000);
        },
        search (data, argumentObj) {
            let res = data;
            let dataClone = data;
            for (let argu in argumentObj) {
                if (argumentObj[argu].length > 0) {
                    res = dataClone.filter(d => {
                        return d[argu].indexOf(argumentObj[argu]) > -1;
                    });
                    dataClone = res;
                }
            }
            return res;
        },
        handleSearch () {
            this.programList = this.initProgramList;
            this.programList = this.search(this.programList, this.searchParam);
        },
        handleCancel () {
            this.searchParam = {
                name: '',
                student: ''
            };
            this.programList = this.initProgramList;
        },
        convertDate (ts) {
            if (!ts) {
                return '';
            }
            let date = new Date(ts);
            if (date) {
                return date.getFullYear() + '-' + this.getMonth(date) + '-' + this.getDay(date);
            } else {
                return '';
            }
        },
        getMonth (date) {
            var month = '';
            month = date.getMonth() + 1;
            if (month < 10) {
                month = '0' + month;
            }
            return month;
        },
        getDay (date) {
            var day = '';
            day = date.getDate();
            if (day < 10) {
                day = '0' + day;
            }
            return day;
        },
        showAdd () {
            this.programForm.action = 'create';
            this.programForm.index = 0;
            this.programModal = true;
            if (this.$refs['programForm']) {
                this.$refs['programForm'].resetFields();
            }
        },
        showEdit (row, index) {
            this.programForm.action = 'update';
            this.programForm.index = index;
            this.programForm.programId = row.programId || '';
            this.programForm.name = row.name;
            this.programForm.student = row.student;
            this.programForm.desc = row.desc;
            this.programForm.imageUrl = row.imageUrl;
            this.programForm.assetUrl = row.assetUrl;
            this.programModal = true;
        },
        cancelModal () {
            this.programModal = false;
        },
        saveProgram () {
            this.$refs['programForm'].validate((valid) => {
                if (valid) {
                    this.saveProgramLoading = true;
                    setTimeout(() => {
                        if (this.programForm.action === 'create') {
                            this.initProgramList.splice(0, 0, this.programForm);
                            this.programList = this.initProgramList;
                        }
                        // if (this.programForm.action === 'update') {
                        //     this.$set(this.initProgramList, this.programForm.index, this.programForm);
                        //     this.programList = this.initProgramList;
                        // }
                        this.$Message.success('保存成功');
                        this.saveProgramLoading = false;
                        this.programModal = false;
                    }, 1000);
                }
            });
        },
        remove (index) {
            this.$Modal.confirm({
                title: '删除作品',
                content: '<p>是否确认删除此作品</p>',
                onOk: () => {
                    this.programList.splice(index, 1);
                },
                onCancel: () => {
                    console.log('cancel opt');
                }
            });
        }
    },
    mounted () {
        this.init();
    }
};
</script>

