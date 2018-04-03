<style lang="less">
    @import '../main.less';
</style>

<template>
    <div ref="scrollCon" @DOMMouseScroll="handlescroll" @mousewheel="handlescroll" class="tags-outer-scroll-con">
        <div class="close-all-tag-con">
            <Dropdown transfer @on-click="handleTagsOption">
                <Button size="small" type="primary">
                    标签选项
                    <Icon type="arrow-down-b"></Icon>
                </Button>
                <DropdownMenu slot="list">
                    <DropdownItem name="clearAll">关闭所有</DropdownItem>
                    <DropdownItem name="clearOthers">关闭其他</DropdownItem>
                </DropdownMenu>
            </Dropdown>
        </div>
        <div ref="scrollBody" class="tags-inner-scroll-body" :style="{left: tagBodyLeft + 'px'}">
            <transition-group id="viewTagList" name="taglist-moving-animation">
                <views-tag
                        type="dot"
                        v-for="(item, index) in myPageTagsList"
                        ref="tagsPageOpened"
                        :key="item.name+(item.query&&(item.query.viewKey!== undefined)?item.query.viewKey:'')"
                        :name="item.name"
                        :query="item.query"
                        :argu="item.argu"
                        @on-close="closePage"
                        @click.native="linkTo(item)"
                        @refresh="refreshView"
                        :closable="item.name==='home_index'?false:true"
                        :color="item.children?(item.children[0].name+(item.children[0].query&&(item.children[0].query.viewKey!== undefined)?item.children[0].query.viewKey:'')===currentPageName?'blue':'default'):(item.name+(item.query&&(item.query.viewKey!== undefined)?item.query.viewKey:'')===currentPageName?'blue':'default')"
                >{{ itemTitle(item) }}
                </views-tag>
            </transition-group>
        </div>
    </div>
</template>

<script>
    import Sortable from 'sortablejs';
    import Vue from 'vue';
    import VueI18n from 'vue-i18n';
    import viewsTag from './views-tag/views-tag.vue';

    Vue.use(VueI18n);
    export default {
        name: 'tagsPageOpened',
        data () {
            return {
                myPageTagsList: this.pageTagsList,
                currentPageName: this.$route.name + (this.$route.query.viewKey !== undefined ? this.$route.query.viewKey : ''),
                tagBodyLeft: 0,
                refsTag: [],
                tagsCount: 1
            };
        },
        components: {
            'views-tag': viewsTag
        },
        props: {
            pageTagsList: Array,
            beforePush: {
                type: Function,
                default: (item) => {
                    return true;
                }
            }
        },
        computed: {
            title () {
                return this.$store.state.app.currentTitle;
            },
            tagsList () {
                return this.$store.state.app.pageOpenedList;
            }
        },
        methods: {
            itemTitle (item) {
                if (item.query && item.query.viewTitle) {
                    if (typeof item.title === 'object') {
                        return this.$t(item.query.viewTitle.i18n);
                    } else {
                        return item.query.viewTitle;
                    }
                }
                if (typeof item.title === 'object') {
                    return this.$t(item.title.i18n);
                } else {
                    return item.title;
                }
            },
            closePage (event, {name, query = {}}) {
                let pageOpenedList = this.$store.state.app.pageOpenedList;
                let lastPageObj = pageOpenedList[0];
                var closePageName = name + (query.viewKey ? query.viewKey : '');
                if (this.currentPageName === closePageName) {
                    let len = pageOpenedList.length;
                    for (let i = 1; i < len; i++) {
                        if (pageOpenedList[i].name === name) {
                            let itemQuery = pageOpenedList[i].query ? pageOpenedList[i].query : {};
                            if (itemQuery.viewKey === query.viewKey) {
                                if (i < (len - 1)) {
                                    lastPageObj = pageOpenedList[i + 1];
                                } else {
                                    lastPageObj = pageOpenedList[i - 1];
                                }
                                break;
                            }
                        }
                    }
                } else {
                    let tagWidth = event.target.parentNode.offsetWidth;
                    this.tagBodyLeft = Math.min(this.tagBodyLeft + tagWidth, 0);
                }
                this.$store.commit('removeTag', {name, query});
                this.$store.commit('closePage', name);
                pageOpenedList = this.$store.state.app.pageOpenedList;
                localStorage.pageOpenedList = JSON.stringify(pageOpenedList);
                if (this.currentPageName === closePageName) {
                    this.linkTo(lastPageObj);
                }
            },
            linkTo (item) {
                let routerObj = {};
                routerObj.name = item.name;
                if (item.argu) {
                    routerObj.params = item.argu;
                }
                if (item.query) {
                    routerObj.query = item.query;
                }
                if (this.beforePush(item)) {
                    this.$router.push(routerObj);
                }
            },
            refreshView (name) {
                this.$emit('refreshView', name);
            },
            handlescroll (e) {
                var type = e.type;
                let delta = 0;
                if (type === 'DOMMouseScroll' || type === 'mousewheel') {
                    delta = (e.wheelDelta) ? e.wheelDelta : -(e.detail || 0) * 40;
                }
                let left = 0;
                if (delta > 0) {
                    left = Math.min(0, this.tagBodyLeft + delta);
                } else {
                    if (this.$refs.scrollCon.offsetWidth - 100 < this.$refs.scrollBody.offsetWidth) {
                        if (this.tagBodyLeft < -(this.$refs.scrollBody.offsetWidth - this.$refs.scrollCon.offsetWidth + 100)) {
                            left = this.tagBodyLeft;
                        } else {
                            left = Math.max(this.tagBodyLeft + delta, this.$refs.scrollCon.offsetWidth - this.$refs.scrollBody.offsetWidth - 100);
                        }
                    } else {
                        this.tagBodyLeft = 0;
                    }
                }
                this.tagBodyLeft = left;
            },
            handleTagsOption (type) {
                if (type === 'clearAll') {
                    this.$store.commit('clearAllTags');
                    this.$router.push({
                        name: 'home_index'
                    });
                } else {
                    this.$store.commit('clearOtherTags', this);
                }
                this.tagBodyLeft = 0;
            },
            moveToView (tag) {
                if (tag.offsetLeft < -this.tagBodyLeft) {
                    // 标签在可视区域左侧
                    this.tagBodyLeft = -tag.offsetLeft + 10;
                } else if (tag.offsetLeft + 10 > -this.tagBodyLeft && tag.offsetLeft + tag.offsetWidth < -this.tagBodyLeft + this.$refs.scrollCon.offsetWidth - 100) {
                    // 标签在可视区域
                    this.tagBodyLeft = Math.min(0, this.$refs.scrollCon.offsetWidth - 100 - tag.offsetWidth - tag.offsetLeft - 20);
                } else {
                    // 标签在可视区域右侧
                    this.tagBodyLeft = -(tag.offsetLeft - (this.$refs.scrollCon.offsetWidth - 100 - tag.offsetWidth) + 20);
                }
            },
            // 页面标签拖拽
            initViewTagSortable () {
                document.body.ondrop = function (event) {
                    event.preventDefault();
                    event.stopPropagation();
                };
                let doList = document.getElementById('viewTagList');
                let vm = this;
                Sortable.create(doList, {
                    group: {
                        name: 'viewTagList',
                        pull: false
                    },
                    animation: 120,
                    ghostClass: 'view-tag-moved',
                    fallbackClass: 'view-tag-moving',
                    onEnd ({oldIndex, newIndex}) {
                        if (oldIndex === newIndex) {
                            return;
                        }
                        vm.$store.commit('sortOpenedList', {oldIndex, newIndex});
                    }
                });
            }
        },
        mounted () {
            this.refsTag = this.$refs.tagsPageOpened;
            setTimeout(() => {
                this.refsTag.forEach((item, index) => {
                    if (this.$route.name === item.name && this.$route.query.viewKey === item.query.viewKey) {
                        let tag = this.refsTag[index].$el;
                        this.moveToView(tag);
                    }
                });
            }, 1); // 这里不设定时器就会有偏移bug
            this.tagsCount = this.tagsList.length;
            this.$nextTick(() => {
                this.initViewTagSortable();
            });
        },
        watch: {
            '$route' (to) {
                this.currentPageName = to.name + (to.query.viewKey !== undefined ? to.query.viewKey : '');
                this.$nextTick(() => {
                    this.refsTag.forEach((item, index) => {
                        if (to.name === item.name && to.query.viewKey === item.query.viewKey) {
                            let tag = this.refsTag[index].$el;
                            this.moveToView(tag);
                        }
                    });
                });
                this.tagsCount = this.tagsList.length;
            },
            'pageTagsList' (value) {
                this.myPageTagsList = value;
            }
        }
    };
</script>
