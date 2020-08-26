<template>
    <div class="test">
        <div style="margin: 10px;padding-left: 10px">
            <el-button type="primary" @click.native="gen_timestamp()" size="small">生成当前时间戳</el-button>
            <el-button type="primary" @click.native="xmind2testcaseshow()" size="small">测试用例转换</el-button>

        </div>

        <div style="margin: 20px 0;"></div>

        <el-dialog title="用例转化" :visible.sync="xMindToTCData.xMindToTCStatus" width="30%">
            <el-form :inline="true" class="demo-form-inline">
                <el-form-item label="文件地址">
                    <el-input v-model="xMindToTCData.xMindToTCAddress" size="medium" :disabled="true">
                    </el-input>
                </el-form-item>
                <el-form-item>
                    <el-upload
                            class="upload-demo"
                            :action="this.$api.fileUploadingApi"
                            :before-upload="beforeFile"
                            :show-file-list='false'
                            :on-success="getFileAddress">
                        <el-button size="small" type="primary">点击上传</el-button>
                    </el-upload>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button size="small" @click="xMindToTCData.xMindToTCStatus = false">取 消</el-button>
                <el-button type="primary" size="small" @click.native="initCaseChange()">确 定</el-button>
            </div>
        </el-dialog>

        <el-dialog title="测试用例转换" :visible.sync="xMindToTCData.xMindToTCStatus" width="30%">
            <el-form>
            </el-form>
            <el-form :inline="true" class="demo-form-inline">
                <el-form-item label="文件地址">
                    <el-input v-model="xMindToTCData.xMindToTCAddress" size="medium" :disabled="true">
                    </el-input>
                </el-form-item>
                <el-form-item>
                    <el-upload
                            class="upload-demo"
                            :action="this.$api.fileUploadingApi"
                            :show-file-list='false'
                            :on-success="getFileAddress">
                        <el-button size="small" type="primary">点击上传</el-button>
                    </el-upload>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button size="small" @click.native="previewcsv()">预 览</el-button>
                <el-button type="primary" size="small" @click.native="xmind2testcase()">下 载</el-button>
            </div>
        </el-dialog>

    </div>
</template>

<script>

    export default {
        name: 'test',
        data() {
            return {
                data: [{
                    id: 1,
                    label: '一级 1',
                    children: [{
                        id: 4,
                        label: '二级 1-1',
                        children: [{
                            id: 9,
                            label: '三级 1-1-1'
                        }, {
                            id: 10,
                            label: '三级 1-1-2'
                        }]
                    }]
                }, {
                    id: 2,
                    label: '一级 2',
                    children: [{
                        id: 5,
                        label: '二级 2-1'
                    }, {
                        id: 6,
                        label: '二级 2-2'
                    }]
                },],
                defaultProps: {
                    children: 'children',
                    label: 'label'
                },
                status: 1,

                value6: '',
                token: '',
                showData: '',
                testCase: {
                    viewStatus: false,
                    address: '',
                },
                xMindToTCData: {
                    xMindToTCStatus: false,
                    xMindToTCAddress: null,
                    xMindToTCFilename:''
                },
            };
        },
        mounted() {
        },
        methods: {
        initCase(status) {
            this.status = status;
            this.testCase.viewStatus = true;
            this.testCase.address = ''
        },
        beforeFile(file){
            let name = file.name;
            window.console.log("filename:",name);
            let type = name.substring(name.lastIndexOf('.')+1);
            if (type !== 'xmind') {
              this.$message('请上传xmind文件格式');
              return false
            }
        },
        getFileAddress(response, file) {
            if (response['status'] === 0) {
                this.$confirm('服务器已存在相同名字文件，是否覆盖?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    let form = new FormData();
                    form.append("file", file.raw);
                    form.append("skip", '1');
                    this.$axios.post('/api/upload', form).then((resp) => {
                            this.$message({
                                showClose: true,
                                message: resp.data['msg'],
                                type: 'success',
                            });
                            this.testCase.address = resp['data']['data']['file_address'];
                            this.xMindToTCData.xMindToTCAddress = resp['data']['data']['file_address'];
                            this.xMindToTCData.xMindToTCFilename = resp['data']['data']['filename'];
                        }
                    );
                }).catch(() => {
                });
            } else {
                if (response['msg']) {
                    this.$message({
                        showClose: true,
                        message: response['msg'],
                        type: 'success',
                    });
                }
                this.testCase.address = response['data']['file_address'];
                this.xMindToTCData.xMindToTCAddress = response['data']['file_address'];
                this.xMindToTCData.xMindToTCFilename = response['data']['filename'];
            }

        },
        buildIdentity() {
            // 调用 callback 返回建议列表的数据
            this.$axios.get('/api/buildIdentity', {}).then((response) => {
                    if (response.data['status'] === 0) {
                        this.$message({
                            showClose: true,
                            message: response.data['msg'],
                            type: 'warning',
                        });
                    } else {
                        this.showData = response.data['data'];

                    }
                }
            )
        },
        gen_timestamp() {
                let timestamp = (new Date()).getTime();
                this.$message({
                    message: timestamp,
                    type: 'success'
                })
        },
        xmind2testcaseshow() {
                this.xMindToTCData.xMindToTCStatus = true;
                this.xMindToTCData.xMindToTCAddress = ''
        },
        previewcsv(){
                let filename = this.xMindToTCData.xMindToTCFilename
                this.$axios.get(this.$api.previewCsvFile,{
                    params:{
                        'filename': filename
                    }
                }).then((resp) => {
                    let url = resp.request['responseURL'];
                    let baseName = url.split('=').pop();
                    let index0 = baseName.lastIndexOf(".");
                    let csvFileName = baseName.substr(0,index0) + '.csv';
                    let previewWindow = window.open(url, '_blank');
                    this.$route.query;
                    //监听预览页面是否被关闭，关闭页面后把弹窗消掉，并删除xmind文件和csv文件
                    let that = this;
                    previewWindow.onload = function(){
                        previewWindow.onunload = function(){
                            that.xMindToTCData.xMindToTCStatus = false;
                            that.$axios.post(that.$api.deleteFile,{
                                'filename': filename
                            });
                            that.$axios.post(that.$api.deleteFile,{
                                'filename': csvFileName
                            });
                        }

                    };
                })
        },
        xmind2testcase() {
            this.$axios.get(this.$api.xmind2testcese, {
                params: {
                    'filename': this.xMindToTCData.xMindToTCFilename
                }
            }).then((response) => {
                if (response.data['status'] === 1) {
                    let filename = response.data['data'];
                    let host = "http://" + response.request['responseURL'].split('/', 3)[2];
                    let baseurl = this.$api.downloadFile;
                    let fileparams = 'filename=' + filename;
                    let url = host + baseurl + '?' + fileparams; //创建下载链接
                    let link = document.createElement('a');//创建a标签
                    link.style.display = 'none';//将a标签隐藏
                    link.href = url; //给a标签添加下载链接
                    link.setAttribute('download', filename);// 此处注意，要给a标签添加一个download属性，属性值就是文件名称 否则下载出来的文件是没有属性的，空白白
                    document.body.appendChild(link);
                    link.click();  //执行a标签
                    this.xMindToTCData.xMindToTCStatus = false;

                    //删除xmind文件
                    this.$axios.post(this.$api.deleteFile,{
                        'filename': this.xMindToTCData.xMindToTCFilename
                    });
                    //删除csv文件
                    this.$axios.post(this.$api.deleteFile,{
                        'filename': filename
                    });

                }else{
                    this.$message({
                    showClose: true,
                    message: response.data['msg'],
                    type: 'warning'
                });
            }
                }
            )
        },
        initCaseChange() {
            // 调用 callback 返回建议列表的数据
            this.$axios.post('/api/caseChange', {
                'address': this.testCase.address,
                'choice': this.status
            }).then((response) => {
                    if (response.data['status'] === 0) {
                        this.$message({
                            showClose: true,
                            message: response.data['msg'],
                            type: 'warning',
                        });
                    } else {
                        // let blob = new Blob([response.data['data']],{type:"application/application/vnd.ms-excel"})
                        // let objectUrl = URL.createObjectURL(blob)
                        // window.location.href=objectUrl
                        let link = document.createElement('a');
                        link.style.display = 'none';
                        link.href = response.data['data'];
                        // link.setAttribute('download', 'excel.xlsx')
                        document.body.appendChild(link);
                        link.click();
                        this.testCase.viewStatus = false;
                    }
                }
            )
        },


    }
    ,

    }
</script>

<style>
    .list {
        max-height: 200px;
    }
</style>
