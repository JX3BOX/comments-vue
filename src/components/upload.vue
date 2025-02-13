<template>
    <div>
        <el-upload
            action="https://cms.jx3box.com/api/cms/upload"
            ref="upload"
            list-type="picture-card"
            :on-preview="handlePictureCardPreview"
            :auto-upload="false"
            :file-list="fileList"
            :limit="maxCount"
            :accept="acceptedExtensions.reduce((acc, cur) => acc + `.${cur},`, '')"
            multiple
            with-credentials
            :on-exceed="onExceed"
            :on-change="onChange"
            :on-success="onSuccess"
            :on-error="onError"
        >
            <i class="el-icon-plus"></i>
            <div slot="tip" class="el-upload__tip">
                最多上传 {{ maxCount }} 张 {{ acceptedExtensions.join(" / ").toUpperCase() }} 格式图片，单张图片不能超过
                {{ maxSize / 1024 / 1024 }} MB
            </div>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
            <img width="60%" :src="dialogImageUrl" alt />
        </el-dialog>
    </div>
</template>

<script>
export default {
    props: {
        fileMaxSize: {
            type: Number,
            default: 10,
        },
    },
    data() {
        return {
            dialogImageUrl: "",
            dialogVisible: false,
            fileList: [],
            successList: [],

            acceptedExtensions: ["jpg", "jpeg", "png", "gif"],
            maxCount: 5,
            maxSize: this.fileMaxSize * 1024 * 1024,
        };
    },
    methods: {
        handlePictureCardPreview(file) {
            this.dialogImageUrl = file.url;
            this.dialogVisible = true;
        },
        onExceed() {
            this.$notify({
                title: "",
                message: `最多上传 ${this.maxCount} 张图片！`,
                type: "error",
                duration: 3000,
                position: "bottom-right",
            });
        },
        onChange(file, fileList) {
            if (file.status == "ready") {
                if (file.size > this.maxSize) {
                    this.$notify({
                        title: "",
                        message: `单张图片大小不能超过 ${this.maxSize / 1024 / 1024} MB！`,
                        type: "error",
                        duration: 3000,
                        position: "bottom-right",
                    });
                    fileList.pop();
                } else {
                    this.fileList = fileList;
                }
            }
        },
        upload() {
            if (this.fileList.length > 0) {
                this.$refs.upload.submit();
            } else {
                this.$emit("onFinish", []);
            }
        },
        onSuccess(response) {
            this.successList = this.successList.concat(response.data);
            if (response.code === 0) {
                if (this.successList.length == this.fileList.length) {
                    this.$emit("onFinish", this.successList || []);
                    this.fileList = [];
                    this.successList = [];
                }
            } else {
                this.onError(response.msg);
            }
        },
        onError() {
            this.$notify({
                title: "",
                message: "图片上传失败!",
                type: "error",
                duration: 3000,
                position: "bottom-right",
            });
            this.$emit("onError");
            this.fileList = [];
        },
        addFile(file) {
            this.$refs.upload.handleStart(file);
            return false;
        },
    },
};
</script>
