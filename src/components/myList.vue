<template>
  <div class="myList-wrap">
    <header class="top">
      <div class="top-filter-btn" @click="changeShowFilter()">
        <span class="top-filter-btn-name">筛选</span>
        <i :class="showFilter?'triangle-up':'triangle-down'"></i>
      </div>
      <div class="top-search">
        <div class="input-wrap" @click="showSearch">
          <span>{{selectedResult}}</span>
          <!--<input type="text" :value="selectedResult">-->
        </div>
        <div class="top-search-btn" @click="doSearch">
          <i class="search-icon"></i>
          <span>搜索</span>
        </div>
      </div>
    </header>

    <div class="filter-panel" v-show="showFilter" ref="filterPanel">
      <div class="filter-wrap" @click.stop ref="filterWrap"  @click="changeShowFilter">
        <div class="filter-tab" @click.stop>
          <div class="material title">
            <div>
              <span class="item">物料</span>
            </div>
            <div>
              <span class="title-right">全部物料</span>
              <i :class="materialExpand?'up-icon':''" @click="toggleMaterialUpDown(materialOptions)"></i>
            </div>
          </div>
          <!--<transition name="updown">-->
          <div class="material-options options" ref="materialOptions">
            <span class="item" :class="{'active':isInArray(chosenMaterielList,item.materielId)}" @click="toggle(item)"
                  v-for="(item,index) in materielList" :key="index">{{item.materiel}}</span>
          </div>
          <!--</transition>-->

          <div class="organization title">
            <div>
              <span class="item">采购组织</span>
            </div>
            <div>
              <span class="title-right">全部组织</span>
              <i :class="organizationExpand?'up-icon':''" @click="toggleOrganizationUpDown(organizationOptions)"></i>
            </div>
          </div>
          <div class="organization-options options" ref="organizationOptions">
            <span class="item" :class="{'active':isInArray(chosenOrganizationList,item.organizationId)}"
                  @click="toggle(item)" v-for="(item,index) in organizationList"
                  :key="index">{{item.organization}}</span>
          </div>
        </div>
        <div class="btns" @click.stop>
          <a class="reset" @click="resetOptions">重置</a>
          <a class="sure" @click="doSearch(),changeShowFilter()">确定</a>
        </div>
      </div>
    </div>

    <div class="search-wrap" v-show="!searchStatus" @click="changeShowSearch" ref="searchWrap">
      <div class="search-bar-wrap" @click.stop="">
        <div class="search-bar">
          <i class="search-icon"></i>
          <label for="search" class="showName" v-show="!labelStatus">请输入供应商名称</label>
          <input type="text" id="search" @input="search($event)" @focus="focus" @blur="blur" :value="selectedResult">
        </div>
        <span class="cancelBtn" @click="hiddenSearch">取消</span>
      </div>
      <div class="result-wrap" @click.stop="" v-show="showResult">
        <ul class="result">
          <li v-for="item in searchResult" @click="resultToSearchBar($event)">{{item.name}}</li>
        </ul>
      </div>
    </div>

    <div class="date-wrap">
      <span class="show">{{this.date?this.date:initDate()}}</span>
      <div class="data-plugin-wrap">
        <input type="date" @input="getDate($event)">
        <i class="date-icon"></i>
      </div>
    </div>

    <div class="list-wrap" ref="wrapper">
      <div class="wrap">
        <ul class="content">
          <li v-for="(item,index) in list" :key="index">
            <div class="head">
              <div class="left">
                <span class="materiel-name">{{controlLength(item.materiel,6)}}</span>
                <span class="materiel-id">({{controlLength(item.id,20)}})</span>
              </div>
              <div class="right">
                <span>{{item.status === 1 ? '正常':'关闭'}}</span>
                <i class="normal-icon"></i>
              </div>
            </div>
            <div class="supplier">{{item.supplier}}</div>
            <div class="bottom">
              <div class="label-wrap">
                <span class="label">累计收货检斤量{{item.weight}}T</span>
                <span class="label">{{controlLength(item.organization,8)}}</span>
              </div>
              <div class="more">
                <router-link :to="{name: 'detail', params: {id: item.id}}">更多</router-link>
              </div>
            </div>

          </li>
        </ul>
        <div class="loading-wrapper">
          <p class="line"></p>
          <img src="../assets/loading-bottom.gif" alt="">
          <p class="line"></p>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
  import Vue from 'vue'
  import BScroll from 'better-scroll'
  import axios from 'axios'

  let json = {}
  export default {
    name: "myList",
    data() {
      return {
        showLoadingImg: false,
        page: 1,
        list: [],
        scroll: null,
        filterScroll: null,
        showFilter: false,
        materialExpand:false,
        organizationExpand:false,
        labelStatus: false,
        searchStatus: true,
        searchResult: [],
        selectedResult: '',
        showResult: true,
        viewHeight: 0,
        timer: null,
        materielList: [],
        chosenMaterielList: [],
        organizationList: [],
        chosenOrganizationList: [],
        date: '',
        //materielWrapHeight: '0px',
        materielWrapHeightFixed: 0,
        organizationlWrapHeightFixed: 0,
        materielElementHeight: 0,
        organizationElementHeight: 0,
        materialOptions: {},
        organizationOptions: {},
        materielWrapHeight: 0,
        organizationWrapHeight: 0,
        filterWrap: 0,
      }
    },
    // computed:{
    //   value(){
    //     return this.selectedResult
    //   }
    // },
    methods: {
      loadData(page, pageAmount, materiel, organization, date, search) {
        console.log(organization)
        // localStorage.setItem('materiel',JSON.stringify(materiel))
        // localStorage.setItem('organization',JSON.stringify(organization))
        // localStorage.setItem('date',date)
        // localStorage.setItem('search',search)
        this.showLoadingImg = true
        axios.get('http://rap2api.taobao.org/app/mock/121282/getList', {
          params: {
            page: page,
            pageAmount: pageAmount,
            materiel: materiel,
            organization: organization,
            date: date,
            search: search,
          }
        })
          .then((response) => {
            this.showLoadingImg = false
            this.page++
            this.list = response.data.data.concat(this.list);
            this.$nextTick(() => {
              if (!this.scroll) {
                this.scroll = new BScroll(this.$refs.wrapper, {
                  click: true,
                  pullUpLoad: {
                    threshold: -50 // 当上拉距离超过30px时触发 pullingUp 事件
                  }
                })
                this.scroll.on('pullingUp', () => {
                  this.loadData(this.page, 10, this.chosenMaterielList, this.chosenOrganizationList, this.date, this.selectedResult)
                  this.scroll.finishPullUp()
                  // 事情做完，需要调用此方法告诉 better-scroll 数据已加载，否则上拉事件只会执行一次
                })
              } else {
                this.scroll.refresh()
              }
            })
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      getMateriel() {
        axios.get('http://rap2api.taobao.org/app/mock/121282/getMateriel')
          .then((response) => {
            this.materielList = response.data.data;
            // this.$nextTick(() => {
            //   if (!this.filterScroll) {
            //     this.filterScroll = new BScroll(this.$refs.filterPanel, {
            //       click: true
            //     })
            //   } else {
            //     this.filterScroll.refresh()
            //   }
            // })
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      getOrganization() {
        axios.get('http://rap2api.taobao.org/app/mock/121282/getOrganization')
          .then((response) => {
            this.organizationList = response.data.data;
            // this.$nextTick(() => {
            //   if (!this.filterScroll) {
            //     this.filterScroll = new BScroll(this.$refs.filterPanel, {
            //       click: true
            //     })
            //   } else {
            //     this.filterScroll.refresh()
            //   }
            // })
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      toggle(item) {
        if (item.active) {
          Vue.set(item, 'active', false);//为item添加不存在的属性，需要使用vue提供的Vue.set( object, key, value )方法。
          if (item.materiel) {
            this.$remove(this.chosenMaterielList, item.materielId)
          } else if (item.organization) {
            this.$remove(this.chosenOrganizationList, item.organizationId)
          }
        } else {
          Vue.set(item, 'active', true);
          if (item.materiel) {
            this.chosenMaterielList.push(item.materielId)
          } else if (item.organization) {
            this.chosenOrganizationList.push(item.organizationId)
          }
        }

        // if (item.materiel) {
        //   this.chosenMaterielList.push(item.materiel)
        // }else{
        //   this.chosenOrganizationList.push(item.organization)
        // }

      },
      doSearch() {
        this.list.length = 0
        this.loadData(1, 10, this.chosenMaterielList, this.chosenOrganizationList, this.date, this.selectedResult)
        // this.chosenMaterielList
        // this.chosenOrganizationList
        // this.searchResult
      },
      controlLength(str, len) {
        return str.length > 6 ? str.slice(0, len) + '...' : str
      },
      isInArray(arr, value) {
        for (let i = 0; i < arr.length; i++) {
          // console.log('arr:'+arr[i])
          // console.log('value:'+value)
          if (value === arr[i]) {
            console.log('InArray-----------------')
            return true;
          }
        }
        return false;
      },
      changeShowFilter() {
        this.showFilter = !this.showFilter
      },
      changeShowSearch() {
        this.searchStatus = !this.searchStatus
      },
      labelHidden(e) {
        this.labelStatus = true
        if (e.target.value === '') {
          this.labelStatus = false
        }

      },
      search(e, keyword) {
        this.selectedResult = e.target.value
        this.labelHidden(e)
        //----------------
        //函数节流
        clearTimeout(this.timer)
        this.timer = setTimeout(() => {
          if (e.target.value !== '') {
            axios.get('http://rap2api.taobao.org/app/mock/121282/search', {
              params: {
                keyword: e.target.value
              }
            })
              .then((response) => {
                this.searchResult = response.data.data;
              })
              .catch(function (error) {
                console.log(error);
              });
          } else {
            this.showResult = false
            this.searchResult = []
          }
        }, 300)
        //this.selectedResult = e != null ? e.target.value : keyword


      },
      hiddenSearch() {
        this.searchStatus = true
      },
      showSearch() {
        this.searchStatus = false
        this.showResult = true
        //this.search(null,this.selectedResult)
      },
      resultToSearchBar(e) {
        console.log(e.target.textContent)
        this.selectedResult = e.target.textContent
        this.hiddenSearch()
        this.searchResult = []
      },
      getDate(e) {
        this.date = e.target.value
        this.loadData(1, 10, this.chosenMaterielList, this.chosenOrganizationList, this.date, this.selectedResult)
      },
      toggleMaterialUpDown(dom) {
        let wrap = dom

        let lineNum = wrap.style.height.slice(0, wrap.style.height.length - 2) / this.materielElementHeight

        if (lineNum > 1) {

          wrap.style.height = this.materielElementHeight + 'px';
          wrap.style.overflow = 'hidden';
          this.materialExpand = false
        } else {

          wrap.style.height = this.materielWrapHeightFixed + 'px';
          this.materialExpand = true
        }
        this.materielWrapHeight = wrap.style.height

        // this.filterScroll.refresh()

        // console.log(this.filterScroll)
      },
      toggleOrganizationUpDown(dom) {

        let wrap = dom

        let lineNum = wrap.style.height.slice(0, wrap.style.height.length - 2) / this.organizationElementHeight

        if (lineNum > 1) {

          wrap.style.height = this.organizationElementHeight + 'px';
          wrap.style.overflow = 'hidden';
          this.organizationExpand = false

        } else {

          wrap.style.height = this.organizationWrapHeightFixed + 'px';
          this.organizationExpand = true

        }

        this.organizationWrapHeight = wrap.style.height

        // this.filterScroll.refresh()
        // console.log(this.filterScroll)

      },
      getMaterialHeight(dom) {
        //获取容器dom
        let wrap = dom
        let wrapRect = wrap.getBoundingClientRect()
        //容器高度
        let wrapHeight = wrapRect.height

        //容器的padding-top
        let paddingTopHeight = window.getComputedStyle(wrap).paddingTop.slice(0, window.getComputedStyle(wrap).paddingTop.length - 2)

        //容器的content高度
        let contentHeight = wrapHeight - paddingTopHeight

        //这个高度是容器内元素全部加载时容器的最大高度，要记录下来用于高度的恢复
        //this.materielWrapHeightFixed = contentHeight
        this.materielWrapHeightFixed = contentHeight

        //获取子元素dom
        let element = dom.children[0]
        let elementRect = element.getBoundingClientRect()

        //子元素margin-bottom
        let marginBottomHeight = window.getComputedStyle(element).marginBottom.slice(0, window.getComputedStyle(element).marginBottom.length - 2)

        //子元素的真实高度
        //this.elementHeight = elementRect.height + parseInt(marginBottomHeight)
        this.materielElementHeight = elementRect.height + parseInt(marginBottomHeight)

        //获取一行标签的高度 赋值给父容器的作为实时高度，因为父容器要使用这个高度初始化，让用户最开始只能看到一行数据
        wrap.style.height = this.materielElementHeight + 'px';
        wrap.style.overflow = 'hidden';
      },
      getOrganizationHeight(dom) {
        //获取容器dom
        let wrap = dom
        let wrapRect = wrap.getBoundingClientRect()

        //容器高度
        let wrapHeight = wrapRect.height

        //容器的padding-top
        let paddingTopHeight = window.getComputedStyle(wrap).paddingTop.slice(0, window.getComputedStyle(wrap).paddingTop.length - 2)

        //容器的content高度
        let contentHeight = wrapHeight - paddingTopHeight

        //这个高度是容器内元素全部加载时容器的最大高度，要记录下来用于高度的恢复
        //this.materielWrapHeightFixed = contentHeight
        this.organizationWrapHeightFixed = contentHeight

        //获取子元素dom
        let element = dom.children[0]

        let elementRect = element.getBoundingClientRect()

        //子元素margin-bottom
        let marginBottomHeight = window.getComputedStyle(element).marginBottom.slice(0, window.getComputedStyle(element).marginBottom.length - 2)

        //子元素的真实高度
        //this.elementHeight = elementRect.height + parseInt(marginBottomHeight)
        this.organizationElementHeight = elementRect.height + parseInt(marginBottomHeight)

        //获取一行标签的高度 赋值给父容器的作为实时高度，因为父容器要使用这个高度初始化，让用户最开始只能看到一行数据
        wrap.style.height = this.organizationElementHeight + 'px';
        wrap.style.overflow = 'hidden';
      },
      initDate() {
        let date = new Date();
        let seperator1 = "-";
        let year = date.getFullYear();
        let month = date.getMonth() + 1;
        let strDate = date.getDate();
        if (month >= 1 && month <= 9) {
          month = "0" + month;
        }
        if (strDate >= 0 && strDate <= 9) {
          strDate = "0" + strDate;
        }
        this.date = year + seperator1 + month + seperator1 + strDate;
      },
      focus() {
        this.$refs.searchWrap.style.height = this.viewHeight
      },
      blur() {
        this.$refs.searchWrap.style.height = '100%'
      },
      resetOptions(){
        this.chosenMaterielList = []
        this.chosenOrganizationList = []
      }
    },
    mounted() {
      this.getMateriel()
      this.getOrganization()
      this.viewHeight = window.innerHeight
    },
    beforeRouteEnter(to, from, next) {

      next((vm) => {
        //console.log(vm.chosenMaterielList, vm.chosenOrganizationList, vm.date, vm.selectedResult)
        vm.loadData(1, 10, vm.chosenMaterielList, vm.chosenOrganizationList, vm.date, vm.selectedResult)

      })
    },
    watch: {
      //数据只会在mounted中改变一次，所以这里监听的是数据初始化
      materielList: function () {
        //我们要获取的页面上元素的高度，但这个元素所在的父容器是v-show:false状态，所以这里要使用一个非常规方法
        //将这个父容器在页面上渲染出来，紧接着让它不可见，这个时候就可以获取到数据并切用户还看不到，后面获取完数据后会把它改回成初始状态
        console.log('change mater')
        if (!this.showFilter) {
          this.showFilter = true
          this.$refs.filterPanel.style.visibility = 'hidden'

          //必须在this.$nextTick中才能获取到数据，
          this.$nextTick(() => {
            //html中要用到这个dom
            this.materialOptions = this.$refs.materialOptions

            //获取元素的真实高度
            this.getMaterialHeight(this.$refs.materialOptions)

            // 将该组件改回初始状态
            this.showFilter = false
            this.$refs.filterPanel.style.visibility = 'visible'
          })
        }

      },
      organizationList() {
        if (!this.showFilter) {
          this.showFilter = true
          this.$refs.filterPanel.style.visibility = 'hidden'

          //必须在this.$nextTick中才能获取到数据，
          this.$nextTick(() => {
            //html中要用到这个dom
            this.organizationOptions = this.$refs.organizationOptions
            //获取元素的真实高度
            this.getOrganizationHeight(this.$refs.organizationOptions)
            //this.getHeight(this.$refs.organizationOptions,this.organizationlWrapHeightFixed,this.organizationlElementHeight)

            // 将该组件改回初始状态
            this.showFilter = false
            this.$refs.filterPanel.style.visibility = 'visible'
          })
        }
      },
      materielWrapHeight() {
        // this.filterScroll.refresh()
        // this.filterWrap = this.materielWrapHeight + this.organizationWrapHeight
        // console.log('refresh')
      },
      organizationWrapHeight() {
        // this.filterScroll.refresh()
        // this.filterWrap = this.materielWrapHeight + this.organizationWrapHeight
      },
      filterWrap() {
        // this.filterScroll.refresh()
      }
    }
  }
</script>

<style scoped>
  .myList-wrap {
    height: 100%;
    position: relative;
  }

  .top {
    display: flex;
    justify-content: space-between;
    align-items: center;
    /*border-bottom: 1px solid #e9e9e9;*/
  }

  .top::after {
    content: "  ";
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 1px;
    background-color: #e9e9e9;
    /* 如果不用 background-color, 使用 border-top:1px solid #e0e0e0; */
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .top::after {
      transform: scaleY(.5);
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .top::after {
      transform: scaleY(.33333);
    }
  }

  .top-filter-btn {
    margin: 0.14rem 0 0.14rem 0.12rem;
    padding-right: 0.2rem;
    border-right: 1px solid #b5b5b5;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .top-filter-btn {
      border-right: 0.5px solid #b5b5b5;
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .top-filter-btn {
      border-right: 0.333333px solid #b5b5b5;
    }
  }

  .top-filter-btn > .top-filter-btn-name {
    padding-right: 0.08rem;
    display: inline-block;
    font-size: 0.12rem;
    color: #8e8e93;
  }

  .top-filter-btn > .top-filter-btn-name.btn-color {
    color: #007aff;
  }

  .triangle-down {
    display: inline-block;
    width: 0.12rem;
    height: 0.07rem;
    background: url("../assets/triangle-down.png") no-repeat;
    background-size: contain;
  }

  .triangle-up {
    display: inline-block;
    width: 0.12rem;
    height: 0.07rem;
    background: url("../assets/triangle-up.png");
    background-size: contain;
  }

  .top-search {
    margin-right: 0.13rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex: 1;
  }

  .top-search .input-wrap {
    flex: 1;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  }

  .top-search .input-wrap span {
    margin-left: 0.1rem;
    display: inline-block;
    width: 100%;
    height: 0.17rem;
    font-size: 0.12rem;
    border: 0;
  }

  .top-search-btn {
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  }

  .top-search-btn > span {
    padding-left: 0.04rem;
    font-size: 0.12rem;
    color: #8e8e93;
  }

  .search-icon {
    display: inline-block;
    width: 0.11rem;
    height: 0.11rem;
    background: url("../assets/search.png") no-repeat;
    background-size: contain;
  }

  .date-wrap {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #F2F6F9;
    /*border-bottom: 1px solid #e9e9e9;*/
  }

  .date-wrap::after {
    content: "  ";
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 1px;
    background-color: #e9e9e9;
    /* 如果不用 background-color, 使用 border-top:1px solid #e0e0e0; */
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .date-wrap::after {
      transform: scaleY(.5);
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .date-wrap::after {
      transform: scaleY(.33333);
    }
  }

  .date-wrap .show {
    padding: 0.08rem 0 0.08rem 0.12rem;
    font-size: 0.12rem;
    color: #030303;
  }

  .date-wrap .data-plugin-wrap {
    margin-right: 0.12rem;
    position: relative;
    font-size: 0.12rem;
    color: #030303;
  }

  .date-wrap .data-plugin-wrap input {
    position: absolute;
    top: 0;
    right: 0;
    opacity: 0;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  }

  .date-wrap .data-plugin-wrap .date-icon {
    display: inline-block;
    width: 0.11rem;
    height: 0.11rem;
    background: url("../assets/date.png") no-repeat;
    background-size: contain;
  }

  .list-wrap {
    height: calc(100% - 0.8rem);
    overflow: hidden
  }

  .list-wrap li {
    margin-left: 0.12rem;
    padding-top: 0.18rem;
    padding-bottom: 0.19rem;
    padding-right: 0.12rem;
    /*border-bottom: 1px solid #efefef;*/
    font-size: 0.1rem;
  }

  .list-wrap li::after {
    content: "  ";
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 1px;
    background-color: #efefef;
    /* 如果不用 background-color, 使用 border-top:1px solid #e0e0e0; */
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .list-wrap li::after {
      transform: scaleY(.5);
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .list-wrap li::after {
      transform: scaleY(.33333);
    }
  }

  .list-wrap .content li .head {
    margin-bottom: 0.08rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .list-wrap .content li .head .left {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .list-wrap .content li .head .left .materiel-name {
    font-size: 0.14rem;
    color: #030303;
  }

  .list-wrap .content li .head .left .materiel-id {
    font-size: 0.12rem;
    color: #4b4b4b;
  }

  .list-wrap .content li .head .right {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .list-wrap .content li .head .right > span {
    padding-right: 0.07rem;
    font-size: 0.12rem;
    color: #19be6b;
  }

  .list-wrap .content li .head .right .normal-icon {
    display: inline-block;
    width: 0.14rem;
    height: 0.14rem;
    background: url("../assets/normal.png") no-repeat;
    background-size: contain;
  }

  .list-wrap .content li .supplier {
    margin-bottom: 0.09rem;
    font-size: 0.12rem;
    color: #4b4b4b;
  }

  .list-wrap .content li .bottom {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .list-wrap .content li .bottom .label-wrap span {
    margin-right: 0.14rem;
    padding: 0.03rem 0.14rem;
    display: inline-block;
    font-size: 0.11rem;
    color: #3897ff;
    background-color: #d4e9ff;
    border-radius: 0.06rem;
  }

  .list-wrap .content li .bottom .more a {
    font-size: 0.09rem;
    color: #bbbbbb;
  }

  .list-wrap .loading-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 0.8rem;
    background-color: #F2F6F9;
  }

  .list-wrap .loading-wrapper .line {
    height: 0px;
    width: 0.85rem;
    position: relative;
  }

  .list-wrap .loading-wrapper .line::after {
    content: "  ";
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 0px;
    background-color: #cacaca;
    border-bottom: 1px solid #cacaca;
    /* 如果不用 background-color, 使用 border-top:1px solid #e0e0e0; */
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .list-wrap .loading-wrapper .line::after {
      transform: scaleY(.5);
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .list-wrap .loading-wrapper .line::after {
      transform: scaleY(.33333);
    }
  }

  .list-wrap .loading-wrapper img {
    margin: 0 0.08rem;
    width: 0.25rem;
    height: 0.25rem;
  }

  .filter-panel {
    width: 100%;
    position: absolute;
    /*position: fixed;*/
    z-index: 2;
    top: 0.45rem;
    left: 0;
    height: calc(100% - 0.46rem);
    background-color: rgba(0, 0, 0, 0.3);
    /*overflow: scroll;*/
    -webkit-overflow-scrolling:touch;
    /*overflow-y: scroll*/
    /*overflow: hidden;*/
  }

  .filter-panel .filter-wrap {
    height: calc(100% + 1px);
  }

  .filter-panel .filter-wrap .filter-tab{
    background-color: #fff;
  }

  .filter-panel .filter-wrap .filter-tab .title {
    margin-left: 0.12rem;
    padding: 0.09rem 0.12rem 0.14rem 0;
    height: 0.2rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    /*border-bottom: 1px solid #979797;*/
    position: relative;
  }

  .filter-panel .filter-wrap .filter-tab .title::after {
    content: "  ";
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 1px;
    background-color: #979797;
    /* 如果不用 background-color, 使用 border-top:1px solid #e0e0e0; */
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .filter-panel .filter-wrap .filter-tab .title::after {
      transform: scaleY(.5);
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .filter-panel .filter-wrap .filter-tab .title::after {
      transform: scaleY(.33333);
    }
  }

  .filter-panel .filter-wrap .filter-tab .title .item {
    font-size: 0.12rem;
    color: #8e8e93;
  }

  .filter-panel .filter-wrap .filter-tab .title .title-right {
    padding-right: 0.03rem;
  }

  .filter-panel .filter-wrap .filter-tab .title i {
    margin: 0 0.02rem;
    display: inline-block;
    width: 0.12rem;
    height: 0.2rem;
    background: url("../assets/right.png") no-repeat;
    background-size: contain;
    transition: all 0.3s;
  }
  /*.filter-panel .filter-wrap .filter-tab .title i.right-icon {*/
    /*transform: rotate(90deg);*/
    /*transition: all 0.3s;*/
  /*}*/
  .filter-panel .filter-wrap .filter-tab .title i.up-icon {
    transform: rotate(-90deg);
    transition: all 0.3s;
  }

  .filter-panel .filter-wrap .filter-tab .options {
    margin-left: 0.12rem;
    padding: 0.09rem 0.12rem 0 0;
    transition: height 0.3s;
  }

  .filter-panel .filter-wrap .filter-tab .options .item {
    margin-right: 0.19rem;
    margin-bottom: 0.16rem;
    padding: 0.04rem 0.19rem;
    display: inline-block;
    font-size: 0.09rem;
    color: #8e8e93;
    background-color: #e7e7e7;
    border-radius: 0.04rem;
    border: 1px solid #fff;
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .filter-panel .filter-wrap .filter-tab .options .item {
      border: 0.5px solid #fff;
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .filter-panel .filter-wrap .filter-tab .options .item {
      border: 0.333333px solid #fff;
    }
  }

  /*.filter-panel .filter-wrap .filter-tab .options .item::after{*/
  /*content: "  ";*/
  /*position: absolute;*/
  /*left: 0;*/
  /*bottom: 0;*/
  /*width: 100%;*/
  /*height: 1px;*/
  /*background-color: #fff;*/
  /*!* 如果不用 background-color, 使用 border-top:1px solid #e0e0e0; *!*/
  /*}*/
  /*@media screen and (-webkit-min-device-pixel-ratio: 2) {*/
  /*.filter-panel .filter-wrap .filter-tab .options .item::after{ transform:scaleY(.5); }*/
  /*}*/
  /*@media screen and (-webkit-min-device-pixel-ratio: 3) {*/
  /*.filter-panel .filter-wrap .filter-tab .options .item::after { transform:scaleY(.333333); }*/
  /*}*/

  .filter-panel .filter-wrap .filter-tab .options .item:last-child {
    margin-right: 0;
  }

  .filter-panel .filter-wrap .filter-tab .options .item.active {
    border: 1px solid #007aff;
    color: #007aff;
    background-color: #fff;
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .filter-panel .filter-wrap .filter-tab .options .item.active {
      border: 0.5px solid #007aff;
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .filter-panel .filter-wrap .filter-tab .options .item.active {
      border: 0.333333px solid #007aff;
    }
  }

  .filter-panel .filter-wrap .filter-tab .organization-options .item {
    padding: 0.04rem 0.35rem;
  }

  .filter-panel .filter-wrap .btns {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .filter-panel .filter-wrap .btns a {
    padding: 0.15rem 0;
    font-size: 0.14rem;
    flex: 1;
    text-align: center;
  }

  .filter-panel .filter-wrap .btns a.reset {
    background: #d4e9ff;
    color: #007aff;
  }

  .filter-panel .filter-wrap .btns a.sure {
    background: #007aff;
    color: #fff;
  }

  .search-wrap {
    width: 100%;
    position: fixed;
    z-index: 2;
    top: 0;
    left: 0;
    height: 100%;
    background-color: #fff;
  }

  .search-wrap .search-bar-wrap {
    padding: 0.1rem 0 0.1rem 0.12rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #F2F6F9;
  }

  .search-wrap .search-bar-wrap .search-bar {
    margin-right: 0.06rem;
    padding: 0.08rem 0;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    background: #e7e7e7;
    border-radius: 0.1rem;
    flex: 1;
    position: relative;
  }

  .search-wrap .search-bar-wrap .search-bar label {
    position: absolute;
    top: 50%;
    left: 0.35rem;
    transform: translateY(-50%);
    font-size: 0.14rem;
    color: #8e8e93;
  }

  .search-wrap .search-bar-wrap .search-bar .search-icon {
    margin: 0 0.12rem;
  }

  .search-wrap .search-bar-wrap .search-bar input {
    margin-right: 0.5rem;
    height: 0.22rem;
    font-size: 0.14rem;
    background: #e7e7e7;
    border: 0;
    flex: 1;
  }

  .search-wrap .search-bar-wrap .cancelBtn {
    margin-right: 0.1rem;
    font-size: 0.14rem;
    color: #007aff;
  }

  .search-wrap .result-wrap {
    margin-left: 0.12rem;
  }

  .search-wrap .result-wrap .result li {
    padding: 0.14rem 0;
    font-size: 0.12rem;
    color: #929292;
    /*border-bottom: 1px solid #efefef;*/
  }

  .search-wrap .result-wrap .result li::after {
    content: "  ";
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 1px;
    background-color: #efefef;
    /* 如果不用 background-color, 使用 border-top:1px solid #e0e0e0; */
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .search-wrap .result-wrap .result li::after {
      transform: scaleY(.5);
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .search-wrap .result-wrap .result li::after {
      transform: scaleY(.333333);
    }
  }

</style>
