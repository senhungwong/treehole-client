<template>
    <at-card :noHover="true">
        <div slot="title" style="fontSize: 18px">
            {{ titleString }}
        </div>
        <div slot="extra">
            <at-button @click="isPickingEmoji=true" icon="icon-sun"></at-button>
            <at-button @click="isAddingImage=true" icon="icon-image" :disabled="!!imageId"></at-button>
            <at-button 
                icon="icon-corner-left-down" 
                :disabled="!isValidToBury"
                hollow 
                @click="burySecret(content)"
            >
                {{ buryButtonText }}
            </at-button>
        </div>
        <at-textarea
            id="secret-field"
            v-model="content" 
            :placeholder="storyPlaceholder" 
            autosize 
            resize="none"
            minRows=3
            :autofocus="true"
        >
        </at-textarea>
        <image-presenter v-if="!!imageId" :source="uploadedImagePreviewUrl"></image-presenter>
        <at-modal v-model="isPickingEmoji" :showFooter="false" width="385" :showClose="false" @on-cancel="onCloseEmojiPickerModal()">
            <picker 
                :native="true"
                @select="addEmoji" 
                title="Insert an emoji" 
                emoji="laughing"
                :infiniteScroll="false"
                :include="includedEmoji"
                :showCategories="false"
                :showPreview="false"
                />
        </at-modal>
        <at-modal v-model="isAddingImage" :showFooter="false" width="390" :showClose="false" @on-cancel="onCloseEmojiPickerModal()">
            <el-upload
                drag
                action=""
                :http-request="upload"
                :multiple="false"
                :before-upload="validateImage"
                >
                <i class="el-icon-upload"></i>
                <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
                <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
        </at-modal>
    </at-card>
</template>

<script>
import { create } from '../../../server/posts.js'
import ImageHttp from '../../../server/images.js'
import { Picker } from 'emoji-mart-vue';
import ImagePresenter from '../Share/ImagePresenter';

export default {
    data() {
        return {
            content: '',
            lastCaretIndex: 0,
            isPickingEmoji: false,
            includedEmoji: [
                'people',
            ],
            isAddingImage: false,
            imageId: null,
        }
    },

    props: {
        lang: String
    },

    components: {
        Picker,
        ImagePresenter,
    },

    watch: {
        content: function () {
            document.getElementById('secret-field').addEventListener('keyup', e => {
                this.lastCaretIndex = e.target.selectionStart;
            });
        }
    },

    computed: {
        /**
         * A basic validation for bury button
         * 
         * @return {Boolean} is valid to bury
         */
        isValidToBury() {
            if (this.content && this.content.length > 0) {
                return true;
            }

            return false;
        },

        /**
         * Placeholder tree hollow story in different languages
         * 
         * @return {String} placeholder string
         */
        storyPlaceholder() {
            /* chinese */
            if (this.lang == 'zh-CN') {
                return "以前的人，心中如果有什么不可告人的秘密，他们会跑到山上，找一棵树，在树上挖一个洞，然后把秘密全说进去，再用泥巴把洞封上，"
                    + "那秘密就会永远留在那棵树里，没有人会知道。";
            }

            /* default english */
            return "They say in ancient times, people go into the mountains and the forests to find a tree hole, and tell the "
                + "tree hole their secrets, then they seal the hole with some mud, and the secrets would stay there forever.";
        },

        /**
         * Title line string that indicates current section is for posting
         * 
         * @return {String} title line string
         */
        titleString() {
            /* chinese */
            if (this.lang == 'zh-CN') {
                return "将你的秘密告诉树洞听 ...";
            }

            /* default english */
            return "Hide your secrets into the tree hollow ...";
        },

        /**
         * The bury button text in different languages
         * 
         * @return {String} bury button text
         */
        buryButtonText() {
            /* chinese */
            if (this.lang == 'zh-CN') {
                return "埋";
            }

            /* default english */
            return "Bury";
        },

        uploadedImagePreviewUrl() {
            return ImageHttp.show(this.imageId);
        },
    },

    methods: {
        /**
         * Create a secret post
         * 
         * @param {String} content the post content
         */
        burySecret(content) {
            create(content, this.imageId).then(response => {
                location.reload(); /* reload the page */
            }, error => {
                this.errorMessageHandling();
            });
        },

        /**
         * Insert the emoji to content
         * 
         * @param {Object} emoji the emoji from emoji mart vue
         */
        addEmoji(emoji) {
            this.content = [
                this.content.slice(0, this.lastCaretIndex),
                emoji.native,
                this.content.slice(this.lastCaretIndex),
            ].join('');
            this.lastCaretIndex += emoji.native.length;
        },

        /**
         * Focus on text area when close emoji picker
         */
        onCloseEmojiPickerModal() {
            const textAreaElement = document.getElementsByTagName('textarea')[0];
            textAreaElement.focus();
            textAreaElement.setSelectionRange(this.lastCaretIndex, this.lastCaretIndex);
        },

        validateImage(file) {
            const isValidType = file.type === 'image/jpeg' || file.type === 'image/png';
            const isValidSize = file.size < 524288000;

            if (!isValidType) {
                this.$Message.error('上传图片只能是png或jpg格式');
            }

            if (!isValidSize) {
                this.$Message.error('上传图片大小不能超过500MB');
            }

            return isValidType && isValidSize;
        },

        /**
         * Upload an image
         */
        upload(item) {
            ImageHttp.store(item.file).then(res => {
                this.imageId = res.body.id;
                this.isAddingImage = false;
            }, err => {
                this.$Message.error('Some error occured when uploading the image');
            });
        },

        /* services methods */

        /**
         * Raise an error message indicating error occurring
         * 
         * @param {String} message the error message that is going to be raised
         */
        errorMessageHandling(message = "Some error occurred when sending data to the server :(") {
            this.$Message.error(message);
        }
    }
}
</script>

<style>

</style>
