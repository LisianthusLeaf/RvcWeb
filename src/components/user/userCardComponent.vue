<script lang="ts" setup>
import { UserInfoVO } from '@/api/user/userTypes';
import router from '@/router';

const props = defineProps<{
    user: UserInfoVO
}>()
const routerGo = function () {
    if (props.user.uid) {
        router.push('/user?id=' + props.user.uid)
    }
}
</script>
<template>
    <div class="user-card horizontal-center">
        <div class="user-card__info vertical-center">
            <div @click="routerGo" class="user-card__info__avatar"
                :style="{ backgroundImage: 'url(\'' + props.user.avatar + '\')' }">

            </div>
            <div class="user-card__info__text">
                <div class="user-card__info__text__username" @click="routerGo">
                    {{ props.user.nickname }}
                </div>
                <div class="user-card__info__text__creatAt">
                    {{ props.user.description }}
                </div>
            </div>
        </div>
        <div class="user-card__button  vertical-center">
            <!-- <span class="user-card__button__item follow vertical-center" v-show="props.user.isfollow!=null">
                关注
            </span> -->
        </div>
    </div>
</template>
<style scoped>
.user-card {
    width: 100%;
    height: 100%;
    background-color: transparent;
    display: flex;
}

.user-card__info {
    height: 80%;
    width: 80%;
    display: flex;
}

.user-card__info__avatar {
    height: 100%;
    aspect-ratio: 1 / 1;
    border-radius: 50%;
    background-repeat: no-repeat;
    background-size: cover;
    background-position: center center;
    cursor: pointer;
}

.user-card__info__text {
    width: calc(100% - 20px);
    height: 100%;
}

.user-card__info__text__username {
    padding: 0 5px;
    width: calc(100% - 10px);
    height: 50%;
    font-size: 14px;
    line-height: 20px;
    text-align: left;
    font-weight: 500;
    cursor: pointer;
    color: rgba(144, 146, 150);
}

.user-card__info__text__creatAt {
    padding: 0 5px;
    width: calc(100% - 10px);
    height: 50%;
    font-size: 10px;
    text-align: left;
    word-wrap: normal;
    color: rgba(144, 146, 150);

}

.user-card__button {
    height: 100%;
    width: 20%;
    display: flex;
    justify-content: space-around;
}

.user-card__button__item {
    position: relative;
    display: inline-block;
    height: 25px;
    padding: 0 15px;
    line-height: 25px;
    font-size: 12px;
    color: white;
    border-radius: 12px;
    cursor: pointer;
}

.follow {
    background-color: rgba(25, 113, 194)
}

.follow:hover {
    background-color: rgba(24, 100, 171);
}
</style>