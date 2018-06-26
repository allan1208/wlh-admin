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
                <FormItem label="名称" prop="title">
                    <Input v-model="programForm.title" placeholder="请输入作品名称"></Input>
                </FormItem>
                <FormItem label="学生" prop="student_name">
                    <Input v-model="programForm.student_name" placeholder="请输入学生姓名"></Input>
                </FormItem>
                <FormItem label="描述" prop="descr">
                    <Input v-model="programForm.descr" type="textarea" :autosize="{minRows: 2,maxRows: 5}" placeholder="请输入作品描述"></Input>
                </FormItem>
                <!-- <FormItem label="封面图片" prop="imageUrl">
                    <Input v-model="programForm.imageUrl" placeholder="请输入封面图片地址"></Input>
                </FormItem> -->
                <FormItem label="作品文件" prop="program_file">
                    <input type="file" placeholder="请输入作品文件地址" ref="programFile"></input>
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
// import {mockProgamList} from './data/mock-data';
import axios from 'axios';

export default {
    name: 'program-list',
    data () {
        return {
            loading: false,
            programList: [],
            initProgramList: [],
            searchParam: {
                name: '',
                student: ''
            },
            columns: [
                {key: 'title', title: '作品名称'},
                {key: 'descr', title: '作品介绍', width: 200},
                {key: 'student_name', title: '学生'},
                {key: 'image', title: '封面图'},
                {key: 'program_file', title: '文件'},
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
                title: '',
                student_name: '',
                descr: '',
                imageUrl: '',
                assetUrl: '',
                program_file: ''
            },
            programRules: {
                title: [
                    { required: true, message: '请输入作品名称', trigger: 'blur' }
                ],
                student_name: [
                    { required: true, message: '请输入学生姓名', trigger: 'blur' }
                ],
                descr: [
                    { required: true, message: '请输入描述', trigger: 'blur' }
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
            this.queryPrograms();
            // this.programList = this.initProgramList = mockProgamList;
        },
        queryPrograms () {
            this.loading = true;
            axios.get('http://testapi.wlhcode.com/programs.json').then(res => {
                let programData = res.data.data;
                this.programList = this.initProgramList = JSON.parse(programData);
                this.loading = false;
            });
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
            this.programForm.programId = row.id || '';
            this.programForm.name = row.title;
            this.programForm.student = row.student_name;
            this.programForm.desc = row.descr;
            this.programForm.imageUrl = row.imageUrl;
            this.programForm.assetUrl = row.program_file;
            this.programModal = true;
        },
        getFile (event) {
            this.programForm.program_file = event.target.files[0];
            console.log(this.programForm.program_file);
        },
        cancelModal () {
            this.programModal = false;
        },
        saveProgram () {
            this.$refs['programForm'].validate((valid) => {
                if (valid) {
                    if (this.programForm.action === 'create') {
                        this.createProgram();
                    }
                    if (this.programForm.action === 'update') {
                        this.$Modal.info({
                            title: '提示',
                            content: '修改功能开发中'
                        });
                    }
                }
            });
        },
        createProgram () {
            if (this.$refs.programFile.files.length === 0) {
                this.$Modal.warning({
                    title: '提示',
                    content: '请选择上传文件'
                });
                return;
            }
            this.saveProgramLoading = true;

            // let payload = {
            //     'program[title]': this.programForm.title,
            //     'program[student_name]': this.programForm.student_name,
            //     'program[descr]': this.programForm.descr,
            //     'program[program_file]': this.programForm.program_file
            // };
            // var qs = require('qs');
            let formData = new FormData();
            formData.append('program[title]', this.programForm.title);
            formData.append('program[student_name]', this.programForm.student_name);
            formData.append('program[descr]', this.programForm.descr);
            formData.append('program[program_file]', this.$refs.programFile.files[0]);
            let config = {
                headers: {
                    'Content-Type': 'multipart/form-data'
                }
            };
            const vm = this;
            axios.post('http://testapi.wlhcode.com/programs.json', formData, config)
                .then(function (res) {
                    if (res.status === 200) {
                        console.log(res);
                        if (res.data.isok) {
                            vm.$Message.success('保存成功');
                            vm.saveProgramLoading = false;
                            vm.programModal = false;
                            vm.queryPrograms();
                        } else {
                            vm.$Modal.error({
                                title: '提示',
                                content: '保存失败,[' + res.data.err + ']' + res.data.msg
                            });
                            vm.saveProgramLoading = false;
                        }
                    }
                })
                .catch(function () {
                    vm.$Modal.error({
                        title: '提示',
                        content: '服务器异常'
                    });
                    vm.saveProgramLoading = false;
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

