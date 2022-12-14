<template>
  <div :class="{fullscreen:fullscreen}" class="tinymce-container editor-container">
    <textarea :id="tinymceId" class="tinymce-textarea"/>
  </div>
</template>

<script>
import plugins from "./plugins";
import toolbar from "./toolbar";
import { qiuNiuUpToken, createFile } from "../../api/file/upload";

export default {
    name: "Tinymce",
    props: {
        id: {
            type: String,
            default: function() {
                return (
                    "vue-tinymce-" +
                    +new Date() +
                    ((Math.random() * 1000).toFixed(0) + "")
                );
            }
        },
        value: {
            type: String,
            default: ""
        },
        toolbar: {
            type: Array,
            required: false,
            default() {
                return [];
            }
        },
        menubar: {
            type: String,
            default: "file edit insert view format table"
        },
        height: {
            type: Number,
            required: false,
            default: 360
        }
    },
    data() {
        return {
            hasChange: false,
            hasInit: false,
            tinymceId: this.id,
            fullscreen: false
        };
    },
    computed: {},
    watch: {
        value(val) {
            if (!this.hasChange && this.hasInit) {
                this.$nextTick(() =>
                    window.tinymce.get(this.tinymceId).setContent(val || "")
                );
            }
        }
    },
    mounted() {
        this.initTinymce();
    },
    activated() {
        this.initTinymce();
    },
    deactivated() {
        this.destroyTinymce();
    },
    destroyed() {
        this.destroyTinymce();
    },
    methods: {
        initTinymce() {
            const _this = this;
            window.tinymce.init({
                language: "zh_CN",
                selector: `#${this.tinymceId}`,
                height: this.height,
                body_class: "panel-body ",
                object_resizing: false,
                toolbar: this.toolbar.length > 0 ? this.toolbar : toolbar,
                menubar: this.menubar,
                plugins: plugins,
                end_container_on_empty_block: true,
                powerpaste_word_import: "clean",
                code_dialog_height: 450,
                code_dialog_width: 1000,
                advlist_bullet_styles: "square",
                advlist_number_styles: "default",
                imagetools_cors_hosts: ["www.tinymce.com", "codepen.io"],
                default_link_target: "_blank",
                link_title: false,
                nonbreaking_force_tab: true, // inserting nonbreaking space &nbsp; need Nonbreaking Space Plugin
                init_instance_callback: editor => {
                    if (_this.value) {
                        editor.setContent(_this.value);
                    }
                    _this.hasInit = true;
                    editor.on("NodeChange Change KeyUp SetContent", () => {
                        this.hasChange = true;
                        this.$emit("input", editor.getContent());
                    });
                },
                setup(editor) {
                    editor.on("FullscreenStateChanged", e => {
                        _this.fullscreen = e.state;
                    });
                },
                // ??????????????????
                // images_dataimg_filter(img) {
                //   setTimeout(() => {
                //     const $image = $(img);
                //     $image.removeAttr('width');
                //     $image.removeAttr('height');
                //     if ($image[0].height && $image[0].width) {
                //       $image.attr('data-wscntype', 'image');
                //       $image.attr('data-wscnh', $image[0].height);
                //       $image.attr('data-wscnw', $image[0].width);
                //       $image.addClass('wscnph');
                //     }
                //   }, 0);
                //   return img
                // },
                images_upload_handler(blobInfo, success, failure, progress) {
                    progress(0);
                    qiuNiuUpToken()
                        .then(response => {
                            if (response.code > 0) {
                                failure("?????????????????????????????????");
                                return;
                            }
                            const url = response.upload_url;
                            const formData = new FormData();
                            formData.append("token", response.up_token);
                            formData.append("file", blobInfo.blob());
                            const domain = response.domain;
                            createFile(url, formData)
                                .then(response => {
                                    if (response.key) {
                                        success(domain + "/" + response.key);
                                        progress(100);
                                        return;
                                    }
                                    failure("????????????");
                                })
                                .catch(() => {
                                    failure("????????????");
                                });
                        })
                        .catch(err => {
                            failure("?????????????????????????????????");
                            console.log(err);
                        });
                }
            });
        },
        destroyTinymce() {
            if (window.tinymce.get(this.tinymceId)) {
                window.tinymce.get(this.tinymceId).destroy();
            }
        },
        setContent(value) {
            window.tinymce.get(this.tinymceId).setContent(value);
        },
        getContent() {
            window.tinymce.get(this.tinymceId).getContent();
        }
    }
};
</script>

<style scoped>
.tinymce-container {
    position: relative;
}
.tinymce-container >>> .mce-fullscreen {
    z-index: 10000;
}
.tinymce-textarea {
    visibility: hidden;
    z-index: -1;
}
.editor-custom-btn-container {
    position: absolute;
    right: 4px;
    top: 4px;
    /*z-index: 2005;*/
}
.fullscreen .editor-custom-btn-container {
    z-index: 10000;
    position: fixed;
}
.editor-upload-btn {
    display: inline-block;
}
</style>
