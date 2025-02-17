<!--
 * @Author: Dhx
 * @Date: 2023-12-23 19:53:26
 * @Description: 
 * @FilePath: \RvcWeb\src\components\rvcModel\modelCommentsComponent.vue
-->
<script setup lang="ts">
import { CommentAddForm, GetCommentForm, ModelComment } from '@/api/rvcModel/modelType'
import { onMounted, ref } from 'vue'
import WaterFallComponent from '@/components/layout/waterFallComponent.vue'
import modelCommentComponent from '@/components/rvcModel/modelCommentComponent.vue'
import { getRootComments, commentAdd } from '@/api/rvcModel/commentApi.ts'
import { message } from '@/utils/message'
import { storage } from '@/utils/storage'
import { useUserStore } from '@/view/user/info/userStore.js'
import LoadingComponent from '../common/loadingComponent.vue'
const userStore = useUserStore()
const props = defineProps<{
    modelId: string
}>()
const WaterFallComponentRef = ref<any>()
let sendCommentDialogVisible = ref(false)
let comments = ref<ModelComment[]>([])
let page = ref(1)
let loading = ref(false)
let getCommentsForm = ref<GetCommentForm>({
    id: '',
    page: '',
    limit: '10',
    sortType: '3'
})
let sendCommentForm = ref<CommentAddForm>({
    replyId: '',
    modelId: '',
    content: ''
})
let disabled = ref(false)
let loaded = ref(false)
const load = function () {
    if (disabled.value) return
    disabled.value = true
    getCommentsForm.value.page = String(page.value)
    loading.value = true
    getCommentsForm.value.id = props.modelId
    getRootComments(getCommentsForm.value).then((res: any) => {
        if (res.code == 200) {
            let data = ref<ModelComment[]>(res.data.records)
            for (let i = 0; i < data.value.length; i++) {
                if (!comments.value.some((item: ModelComment) => {
                    return item.id == data.value[i].id
                })) {
                    comments.value.push(data.value[i])
                }
            }
            if (data.value.length != 0) {
                //
            }
            else {
                disabled.value = true
                loading.value = false
                message.warning('没有更多评论了')
                return
            }

            page.value++
        } else {
            //
        }
        loaded.value = true
        disabled.value = false
        loading.value = false
    })
}
let sendCommentDisabled = ref(false)
const commentAddFunc = function () {
    if (sendCommentForm.value.content == '') return
    if (sendCommentDisabled.value == true) {
        message.warning('请稍后再试')
        return
    }
    sendCommentDisabled.value = true
    setTimeout(() => {
        sendCommentDisabled.value = false
    }, 3000)
    sendCommentForm.value.modelId = props.modelId
    commentAdd(sendCommentForm.value).then((res: any) => {
        if (res.code == 200) {
            message.success('发表成功')
            let newComment = ref<ModelComment>({
                id: res.data,
                uid: storage.get<string>('uid')!,
                nickname: userStore.getProfile.nickname,
                picture: userStore.getProfile.avatar,
                content: sendCommentForm.value.content,
                likesNum: '0',
                commentTime: '刚刚',
                modelId: props.modelId,
                likes: '0'
            })
            sendCommentDisabled.value = false
            sendCommentDialogVisible.value = false
            sendCommentForm.value.content = ''
            comments.value = [newComment.value].concat(comments.value)
        } else {
            message.error(res.msg)
        }
    })
}
const getLength = function (str: string) {
    return str.length
}
onMounted(() => {
    load()
});
</script>
<template>
    <div class="model-comments">
        <el-dialog v-model="sendCommentDialogVisible" style="background-color: rgba(26,27,30);border-radius: 5px;"
            width="20%">
            <div class="dialog-title">添加评论</div>
            <div class="dialog-input">
                <input class="input" v-model="sendCommentForm.content" maxlength="200">
                <div style="text-align: right;">
                    <span>{{ getLength(sendCommentForm.content) }}/200</span>
                </div>
            </div>
            <template #footer>
                <span class="dialog-footer">
                    <div @click="sendCommentDialogVisible = false" class="dialog-footer__cancel">取消</div>
                    <div type="primary" class="dialog-footer__confirm" @click="commentAddFunc">
                        发表
                    </div>
                </span>
            </template>
        </el-dialog>

        <div class="model-comments__title">
            <span>评论</span>
            <div class="add-comment" @click="sendCommentDialogVisible = true">
                发表评论
            </div>
        </div>
        <div class="model-comments__content">
            <WaterFallComponent ref="WaterFallComponentRef" :minWidth="240" v-infinite-scroll="load"
                infinite-scroll-distance="100" :infinite-scroll-disabled="disabled" :infinite-scroll-immediate="false">
                <modelCommentComponent style="" v-for="(comment, index) in comments" :key="comment.id"
                    v-show="WaterFallComponentRef.visibility[index]" :comment="comment">
                </modelCommentComponent>
            </WaterFallComponent>
            <el-empty :image-size="200" v-if="loaded&&comments.length == 0" style="font-family: 'ZCool';" description="暂无评论"
                image="/icon/comment-empty.svg" />
            <LoadingComponent :diameter="60" :loading="loading"></LoadingComponent>
            <!-- <div class="model-comments__content__more" v-show="!disabled" @click="load">加载更多</div> -->
        </div>
    </div>
</template>
<style scoped>
.model-comments {
    width: 100%;
    background-color: rgba(26, 27, 30);
}

.dialog-title {
    position: absolute;
    left: 20px;
    top: 20px;
    font-size: 14px;
    color: rgba(193, 194, 197);
}

.dialog-input {
    width: 100%;
}

.input {
    outline: none;
    width: calc(100% - 30px);
    height: 40px;
    line-height: 40px;
    padding: 0 15px;
    background-color: rgba(26, 27, 30);
    color: white;
    font-size: 16px;
    border-radius: 5px;
    margin-top: 10px;
    border: rgba(55, 58, 64) 1px solid;
}

.dialog-footer {
    display: flex;
}

.dialog-footer__cancel {
    color: white;
    height: 35px;
    padding: 0 20px;
    line-height: 35px;
    border-radius: 5px;
    border: rgba(55, 58, 64) 1px solid;
    background-color: rgba(37, 38, 43);
    cursor: pointer;
}

.dialog-footer__cancel:hover {
    background-color: rgba(44, 46, 51);
}

.dialog-footer__confirm {
    position: absolute;
    right: 20px;
    color: white;
    height: 35px;
    padding: 0 20px;
    line-height: 35px;
    border-radius: 5px;
    background-color: rgba(25, 113, 194);
    cursor: pointer;
    user-select: none;
}

.dialog-footer__confirm:hover {
    background-color: rgba(24, 100, 171);
}

.add-comment {
    position: relative;
    top: 50%;
    transform: translate(0, -50%);
    height: 35px;
    padding: 0 15px;
    border: rgba(51, 154, 240) 1px solid;
    font-size: 16px;
    line-height: 35px;
    border-radius: 5px;
    margin-left: 10px;
    cursor: pointer;
    color: rgba(51, 154, 240);
}

.add-comment:hover {
    background-color: rgba(28, 34, 40);
}

.model-comments__title {
    position: relative;
    display: flex;
    justify-content: space-between;
    width: 75%;
    height: 70px;
    left: 50%;
    transform: translate(-50%);
    color: rgba(193, 194, 197);
    font-size: 30px;
    line-height: 70px;
    font-family: 'ZCool';
    border-bottom: solid 3px #cccccc;
}

.model-comments__content {
    position: relative;
    top: 30px;
    width: 75%;
    min-height: 300px;
    left: 50%;
    transform: translate(-50%);
    padding-bottom: 50px;
}

.model-comments__content__more {
    cursor: pointer;
    color: white;
}
</style>