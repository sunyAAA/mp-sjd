<template>
	<div class="taskProgress-wrap">
		<!-- 导航 -->
		<div class="nav">
			<ul>
				<li v-for="(item, index) in navArr" :key="index" :class="[ status == index ? 'current' : '' ]" @click="toggleTab(index)" v-text="item.title"></li>
			</ul>
		</div>
		<!-- 内容 -->
		<scroll-view scroll-y :style='scrollHeight' @scrolltolower='loadMore'>
			<task-card v-for="(item,index) in curData" :key="index" :item="item" :current="status"></task-card>
			<div v-show='isNoMore' class="no-more">—— 我们是有底线的 ——</div>
		</scroll-view>
	</div>
</template>

<script>
import taskCard from "../../components/taskCard";
import { getMyTask, getApprove } from "../../api";
import { formMyTask } from "../../model";
import { _loading } from "../../utils";
const pageSize = 8;
export default {
    components: {
        taskCard
    },
    data() {
        return {
            taskStatus: 2,
            pageNum: 1,
            curData: [],
            navArr: [
                {
                    title: "进行中"
                },
                {
                    title: "我的审核"
                },
                {
                    title: "已完成"
                }
            ],
            status: 0,
            scrollHeight: "height:560px",
            total: 0,
            isNoMore: false
        };
    },
    onShow() {
        this.getMyData();
        this.setScrollViewHeight();
    },
    methods: {
        toggleTab(index) {
            this.status = index;
            this.taskStatus = index == 0 ? 2 : index == 1 ? 3 : 6;
            this.pageNum = 1;
            this.total = 0;
            this.isNoMore = false;
        },
        getMyData() {
            _loading("加载中...");
            getMyTask(this.taskStatus, pageSize, this.pageNum).then(res => {
                if (res.code == 1) {
                    this.total = res.totalCount;
                    this.curData =
                        this.pageNum > 1
                            ? this.curData.concat(formMyTask(res.data))
                            : formMyTask(res.data);
                }
                _loading();
            });
        },
        loadMore() {
            if (curData.length > this.total) {
                return (this.isNoMore = true);
            }
            this.pageNum++;
            this.getMyTask();
        },
        setScrollViewHeight() {
            wx.getSystemInfo({
                success: res => {
                    this.scrollHeight = `height:${res.windowHeight - 55}px;`;
                }
            });
        }
    },
    watch: {
        taskStatus(val) {
            this.getMyData();
        }
    }
};
</script>

<style lang="stylus" scoped>
.taskProgress-wrap {
	background-color: #fff;
	min-height: 100vh;

	// 导航nav
	.nav {
		display: flex;
		width: 100%;
		height: 100rpx;
		line-height: 100rpx;

		ul {
			display: flex;
			width: 100%;
			justify-content: space-around;

			li {
				font-size: 30rpx;
				color: #000;
			}

			li.current {
				color: #ff4b2b;
			}
		}
	}
}

.no-more {
	text-align: center;
	font-size: 12px;
	color: #aaa;
	padding: 10px 0;
}
</style>
