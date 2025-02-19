<template>
  <Layout>
    <Tabs class-prefix="type" :data-source="recordTypeList"/>
    <ol class="group-wrapper" v-if="groupedList.length>0">
      <li class="group" v-for="group in groupedList" :key="group.title">
        <h3 class="title">
          <span>{{ beautify(group.title) }}</span>
          <span>￥{{ group.total }}</span>
        </h3>
        <ol class="record-wrapper">
          <li v-for="item in group.items" :key="item.createdAt"
              class="record" :class="{lastRecord: item === group.items[group.items.length-1]}">
            <Icon :name="`${item.tags.icon}`"/>
            <span class="name">{{ item.tags.name }}</span>
            <span class="notes">{{ item.notes }}</span>
            <span>￥{{ item.amount }} </span>
          </li>
        </ol>
      </li>
    </ol>
    <div v-else class="noResult">
      <Icon name="records"/>
      <span class="remind">目前没有相关记录，快去记一笔吧~</span>
    </div>
  </Layout>
</template>

<script lang="ts">
  import {Vue, Component} from 'vue-property-decorator';
  import Tabs from '@/components/Tabs.vue';
  import recordTypeList from '@/constants/recordTypeList';
  import dayjs from 'dayjs';
  import clone from '@/lib/clone';

  @Component({
    components: {Tabs}
  })
  export default class Statistics extends Vue {
    recordTypeList = recordTypeList;
    beforeCreate() {
      this.$store.commit('fetchRecords');
    }
    get tabsType() {
      return this.$store.state.tabsType;
    }
    get recordList() {
      return (this.$store.state as RootState).recordList;
    }
    get groupedList() {
      const {recordList} = this;
      const newList = clone(recordList)
        .filter(r => r.tags.type === this.tabsType)
        .sort((a, b) => dayjs(b.createdAt).valueOf() - dayjs(a.createdAt).valueOf());
      if (newList.length === 0) {
        return [];
      }
      type Result = { title: string; total?: number; items: RecordItem[] }[]
      const result: Result = [{title: dayjs(newList[0].createdAt).format('YYYY-MM-DD'), items: [newList[0]]}];
      for (let i = 1; i < newList.length; i++) {
        const current = newList[i];
        const last = result[result.length - 1];
        if (dayjs(last.title).isSame(dayjs(current.createdAt), 'day')) {
          last.items.push(current);
        } else {
          result.push({title: dayjs(current.createdAt).format('YYYY-MM-DD'), items: [current]});
        }
      }
      result.forEach(group => {
        group.total = group.items.reduce((sum, item) => {
          return sum + item.amount;
        }, 0);
      });
      return result;
    }
    beautify(string: string) {
      const day = dayjs(string);
      const now = dayjs();
      if (day.isSame(now, 'day')) {
        return '今天';
      } else if (day.isSame(now.subtract(1, 'day'), 'day')) {
        return '昨天';
      } else if (day.isSame(now.subtract(2, 'day'), 'day')) {
        return '前天';
      } else if (day.isSame(now, 'year')) {
        return day.format('M月D日');
      } else {
        return day.format('YYYY年M月D日');
      }
    }
  }
</script>

<style lang="scss" scoped>
  ::v-deep .content {
    display: flex;
    flex-direction: column;
  }
  .group-wrapper {
    flex-grow: 1;
    overflow: auto;
    %item {
      line-height: 24px;
      display: flex;
    }
    .title {
      padding: 8px 16px;
      justify-content: space-between;
      @extend %item;
    }
    .record-wrapper {
      padding: 0 16px;
      background: white;
      .record {
        padding: 8px 0;
        border-bottom: 1px solid #e6e6e6;
        align-items: center;
        @extend %item;
        .name {
          margin-left: 6px;
        }
        .notes {
          margin-right: auto;
          margin-left: 16px;
          color: #999;
        }
      }
      .lastRecord {
        border-bottom: 1px solid white;
      }
    }
  }
  .noResult {
    padding: 64px;
    display: flex;
    text-align: center;
    align-items: center;
    flex-direction: column;
    .icon {
      width: 64px;
      height: 64px;
    }
    .remind {
      padding: 32px;
    }
  }
</style>