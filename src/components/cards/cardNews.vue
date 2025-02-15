<template>
    <transition name="card-fade">
        <div class="card_container" v-show="cardShow">
            <Card hoverable class="card">
                <template #cover>
                    <div class="img-container">
                        <div v-for="(imgSrc, index) in cardInfo.imgs.slice(0, 2)" :key="index" class="img-item">
                            <img :src="imgSrc" class="img-ele" />
                        </div>
                        <div class="calender" v-if="!isMobile">
                            <div class="year_month">{{ date.yearMonth }}</div>
                            <div class="day-container">
                                <div class="date">{{ date.date }}</div>
                                <div class="day">{{ date.day }}</div>
                            </div>
                            <div class="tech-nice">
                                <div class="page-icon" @click="turnPage('pre')">
                                    <LeftSquareFilled style="color: rgb(24, 144, 255);" />
                                </div>
                                <div class="suitable">{{ date.suitable }}</div>
                                <div class="page-icon" @click="turnPage('next')">
                                    <RightSquareFilled style="color: rgb(24, 144, 255);" />
                                </div>
                            </div>
                            <div class="introduction">我们是秋水灌河团队，借助人工智能，每天更新近期全球范围有趣的科技进展。今日份科学请君小观</div>
                        </div>
                    </div>

                </template>
                <div class="content-container">
                    <div class="title">
                        {{ cardInfo.title }}
                        <a class="from" :href="cardInfo.titleUrl" target="_blank" v-if="cardInfo.website">(转自{{
                            cardInfo.website }})</a>
                    </div>
                    <div class="meta">
                        {{ cardInfo.organization }}
                    </div>
                    <div class="content">
                        <div class="content-item" v-for="content, index in cardInfo.content" :key="index">
                            {{ content }}
                        </div>
                    </div>
                </div>

            </Card>
        </div>
    </transition>
</template>

<script setup lang="ts">
import { message, Card } from 'ant-design-vue';
import { useStore } from 'vuex';
import { ref, reactive, onMounted, onBeforeMount, watch, computed } from 'vue';
import { LeftSquareFilled, RightSquareFilled } from '@ant-design/icons-vue';
import { TimelineType } from '@/components/timeline/renderTimeline.ts';
import { getTimeline } from '@/api/timeline.ts';

const store = useStore();
const styleConfig = computed(() => {
    return store.state.style
})
const isMobile = styleConfig.value.isMobile
const cardShow = computed(() => {
    const cardShow = store.state.content.cardShow;
    if (cardShow) {
        const state = store.state.content;
        cardInfo.imgs = state.content.imgs;
        cardInfo.content = state.content.content.split('。').map((item, index, origin) => {
            if (index != origin.length - 1) {
                item += '。'
            }
            return item
        });
        cardInfo.title = state.content.title;
        cardInfo.subtitle = state.content.subtitle;
        cardInfo.date = state.content.date;
        cardInfo.locationName = state.content.locationName;
        cardInfo.website = state.content.website;
        cardInfo.titleUrl = state.content.titleUrl;
        cardInfo.organization = state.content.organization;
        date.suitable = suitable[Math.floor(Math.random() * suitable.length)]
    }
    return cardShow
})
interface CardInfo {
    title: string;
    subtitle: string;
    content: Array<string>;
    imgs: Array<string>;
    date: string;
    locationName: string;
    website: string;
    titleUrl: string;
    organization: string;
}
interface Calender {
    current: Date;
    yearMonth: string;
    date: string;
    day: string;
    suitable: string;
}

const cardInfo: CardInfo = reactive({
    title: "",
    subtitle: "",
    content: [],
    imgs: [],
    date: "",
    locationName: "",
    website: "",
    titleUrl: "",
    organization: "耶鲁"
})

let date: Calender = reactive({
    current: new Date(),
    yearMonth: "",
    date: "",
    day: "",
    suitable: ""
});

onMounted(async () => {
    await getTimelineData()
    date = getDate()
})
async function getTimelineData() {
    const body = {}
    let timelineType: TimelineType
    timelineType = TimelineType.TECHNEWS
    const data = await getTimeline(timelineType, body)
    store.commit('timeline/updateTimeline', {
        msgType: TimelineType.HISTORYGEOMAP,
        updateData: data
    })
}

const suitable = ['宜亲近科技', '宜关注paper', '宜学习知识', '宜探查世界', '宜保持好奇', '宜开脑洞', '宜求索未知'];
const getDate: () => Calender = () => {
    const today = new Date()
    // format yyyy年mm月
    const year = today.getFullYear();
    const month = (today.getMonth() + 1).toString();
    const yearMonth = year + '年' + month + '月';
    //获取今天是几号
    const date = today.getDate().toString();
    const day = today.getDay();
    const dayMap = ['日', '一', '二', '三', '四', '五', '六'];
    const res: Calender = {
        current: today,
        yearMonth,
        date,
        day: '星期' + dayMap[day],
        suitable: suitable[Math.floor(Math.random() * suitable.length)]
    }
    return res
}

const turnPage = (type: 'pre' | 'next') => {
    message.info('抱歉，此demo暂时尚未实时更新数据');
    // let payload = new Date();
    // if (type === 'pre') {
    //     payload.setDate(date.current.getDate() - 1)
    // } else {
    //     payload.setDate(date.current.getDate() + 1)
    // }
    // store.commit('timeline/updateTimeline', {
    //     msgType: 'tech_news',
    //     // 字符串时间 yyyy-mm-dd，current日期往前推一天
    //     updateData: {
    //         date: payload.toLocaleDateString().split(' ')[0].replace(/\//g, '-')
    //     }
    // })
}
</script>

<style scoped>
.card_container {
    position: absolute;
    z-index: 1;
    width: 100vw;
    height: calc(var(--screen-height--) - var(--timeline-container-height--));
    background: rgba(0, 0, 0, 0);
    padding: 20px 0;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;

    .card {
        cursor: default;
        padding: 10px;
        box-shadow: 6px 6px 6px #ccc;
        border-radius: 10px;
        background: #e5e5e5;
        width: var(--card-width--);

        @media screen and (min-width: 600px) {
            padding-right: var(--card-padding-right--);
        }

        height: 100%;
        overflow-y: auto;
        border-bottom: 16px solid transparent;

        .img-container {
            padding: 0px;
            height: calc(0.45*(100vh - var(--timeline-container-height--) - 80px));
            width: 100%;
            display: flex;
            align-items: center;
            justify-content: center;

            .img-item {
                width: 100%;
                height: 100%;
                box-sizing: border-box;
                margin-right: 4px;

                &:last-child {
                    margin-right: 0;
                }

                .img-ele {
                    height: 100%;
                    max-width: 100%;
                    object-fit: contain;
                }
            }

            .calender {
                position: absolute;
                width: 130px;
                height: 120px;
                display: flex;
                flex-direction: column;
                top: 10px;
                right: 10px;
                margin: 4px;
                padding: 4px;
                background: rgba(255, 255, 255, 1);
                border-radius: 5px;
                color: #333;
                font-size: 16px;
                border-radius: 10px;

                .year_month {
                    font-weight: bold;
                    margin-bottom: 4px;
                    color: #aaa;
                }

                .day-container {
                    display: flex;
                    justify-content: space-around;
                    align-items: center;
                    font-weight: bold;
                    line-height: 54px;

                    .date {
                        font-weight: bold;
                        font-size: 50px;
                        background: #fff;
                    }

                    .day {
                        font-size: 16px;
                        writing-mode: vertical-rl;
                        text-orientation: upright;
                        line-height: 1.5;
                        color: rgb(49, 205, 240);
                    }
                }

                .tech-nice {
                    display: flex;
                    justify-content: space-around;
                    color: #333;
                    font-size: 12px;
                    text-align: right;
                    margin-top: 4px;

                    .suitable {
                        color: #666;
                    }

                    .page-icon {
                        cursor: pointer;
                        font-size: 100px;
                        display: flex;
                        justify-content: center;
                        align-items: center;
                        font-size: 20px;
                        color: #333;
                        cursor: pointer;
                    }
                }

                .introduction {
                    position: absolute;
                    top: 140px;

                    border-radius: 10px;
                    text-align: justify;
                    height: 260px;
                    width: 110px;
                    margin-top: 10px;
                    font-size: 16px;
                    writing-mode: vertical-rl;
                    text-orientation: upright;
                    line-height: 2;
                    color: #888;
                }
            }

        }

        .content-container {
            height: calc(0.55*(100vh - var(--timeline-container-height--) - 80px));
            display: flex;
            flex-direction: column;

            .title {
                text-align: left;
                font-size: var(--card-title-size--);
                font-weight: bold;
                display: flex;
                align-items: center;

                .from {
                    font-size: 12px;
                    margin-left: 10px;
                }
            }

            .meta {
                width: 100%;
                font-size: 12px;
                line-height: 20px;
                color: #999;
                text-align: left;
            }

            .content {
                margin-top: 8px;
                text-align: left;
                font-size: 14px;
                color: #333;
                line-height: 1.4;
                display: flex;
                flex-direction: column;
                justify-content: flex-start;

                &>div {
                    margin-bottom: var(--card-paragraph-margin--)
                }
            }
        }
    }
}

.card-fade-enter-from,
.card-fade-leave-to {
    transform: translateX(20px);
    opacity: 0;
}

.card-fade-enter-active {
    transition: all 0.9s ease-out;
}

.card-fade-leave-active {
    transition: all 1.0s cubic-bezier(1, 0.5, 0.8, 1);
}
</style>