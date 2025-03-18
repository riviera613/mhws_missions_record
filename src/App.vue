<template>
  <Card style="margin: 32px 48px 0 48px;">
    <template #title>
      <b>大小金冠及任务记录</b>
    </template>
    <template #extra>
      <Button size="small" type="warning" @click="clearLocalStorage" style="font-size: 12px;">
        <Icon type="md-trash" />清空数据
      </Button>
    </template>

    <!-- 主表格 -->
    <Table border stripe :columns="columns" :data="data">
      <template #name="{ row, index }">
        <div class="name-pic-cell">
          <img :src="row.pic" style="width: 40px; margin-right: 16px;">
          <span>{{ row.name }}</span>&nbsp;
        </div>
      </template>
      <template #bigFlag="{ row, index }">
        <Checkbox v-model="data[index].bigFlag" :disabled="row.disabled" @on-change="saveToLocalStorage"></Checkbox>
      </template>
      <template #smallFlag="{ row, index }">
        <Checkbox v-model="data[index].smallFlag" :disabled="row.disabled" @on-change="saveToLocalStorage"></Checkbox>
      </template>
      <template #bigMissions="{ row, index }">
        <!-- 大金冠任务列表 -->
        <div v-for="(mission, i) in row.bigMissions">
          <div v-if="mission.count > 0" style="padding: 4px; margin: 4px; border: 1px solid #dcdee2">
            <!-- 单个任务信息 -->
            <Icon type="md-star" /><span>{{ mission.star }}</span>&nbsp;
            <Tag :color="TYPE_COLOR_MAP[mission.type]">{{ TYPE_MAP[mission.type] }}</Tag>
            <Tag v-if="mission.place" :color="PLACE_COLOR_MAP[mission.place]">{{ PLACE_MAP[mission.place] }}</Tag>
            <Tag v-else>未记录地区</Tag>
            <Button size="small" style="font-size: 12px; float: right;" @click="clickCountButton(2, index, i)">
              {{ mission.count }}次
            </Button>
          </div>
        </div>
        <!-- 新增任务按钮 -->
        <Button type="default" shape="circle" size="small" :disabled="row.disabled"
          @click="openAddModal(2, index, row.name)">
          <Icon type="md-add" />
        </Button>
      </template>
      <!-- 小金冠任务列表 -->
      <template #smallMissions="{ row, index }">
        <div v-for="(mission, i) in row.smallMissions">
          <div v-if="mission.count > 0" style="padding: 4px; margin: 4px; border: 1px solid #dcdee2">
            <Icon type="md-star" /><span>{{ mission.star }}</span>&nbsp;
            <Tag :color="TYPE_COLOR_MAP[mission.type]">{{ TYPE_MAP[mission.type] }}</Tag>
            <Tag v-if="mission.place" :color="PLACE_COLOR_MAP[mission.place]">{{ PLACE_MAP[mission.place] }}</Tag>
            <Tag v-else>未记录地区</Tag>
            <Button size="small" style="font-size: 12px; float: right;" @click="clickCountButton(1, index, i)">
              {{ mission.count }}次
            </Button>
          </div>
        </div>
        <Button type="default" shape="circle" size="small" :disabled="row.disabled"
          @click="openAddModal(1, index, row.name)">
          <Icon type="md-add" />
        </Button>
      </template>
    </Table>
  </Card>

  <!-- 新增任务表单 -->
  <Modal v-model="addModal.display" title="新增任务">
    <Form :label-width="80" label-position="left" :model="addModal.form" ref="addModalForm"
      :rules="addModalFormValidate">
      <FormItem label="名称" prop="name">
        {{ addModal.form.name }}
      </FormItem>
      <FormItem label="大小" prop="bigOrSmall">
        <p v-if="addModal.form.bigOrSmall === 2">大金冠</p>
        <p v-else-if="addModal.form.bigOrSmall === 1">小金冠</p>
      </FormItem>
      <FormItem label="星级">
        <Rate v-model="addModal.form.star" :count="5" icon="md-star"></Rate>
      </FormItem>
      <FormItem label="类型" prop="type">
        <RadioGroup v-model="addModal.form.type" type="button">
          <Radio v-for="(v, k) in TYPE_MAP" :key="k" :label="k">{{ v }}</Radio>
        </RadioGroup>
      </FormItem>
      <FormItem label="地区" prop="place">
        <Select v-model="addModal.form.place" prop="place">
          <Option v-for="(v, k) in PLACE_MAP" :value="k" :label="v">{{ v }}</Option>
        </Select>
      </FormItem>
    </Form>
    <template #footer>
      <Button long type="primary" @click="addModalSubmit">提交</Button>
    </template>
  </Modal>
</template>

<script>
import { TYPE_MAP, TYPE_COLOR_MAP, PLACE_MAP, PLACE_COLOR_MAP, MONSTER_LIST } from './consts.js'
const LOCAL_STORAGE_KEY = 'mhws_crowns_mission';
export default {
  data() {
    return {
      columns: [
        {
          title: '怪物',
          key: 'name',
          slot: 'name',
        },
        {
          title: '大金冠',
          key: 'bigFlag',
          slot: 'bigFlag',
        },
        {
          title: '小金冠',
          key: 'smallFlag',
          slot: 'smallFlag',
        },
        {
          title: '大金冠任务',
          key: 'bigMissions',
          slot: 'bigMissions',
        },
        {
          title: '小金冠任务',
          key: 'smallMissions',
          slot: 'smallMissions',
        }
      ],
      data: [

      ],
      addModal: {
        display: false,
        form: {
          name: '',
          bigOrSmall: 0,
          star: 3,
          type: '',
          place: '',
        },
        index: 0,
      },
      addModalFormValidate: {
        type: [
          { required: true, message: 'Can not be empty', trigger: 'blur' }
        ],
      }
    };
  },
  computed: {
    TYPE_MAP() {
      return TYPE_MAP;
    },
    TYPE_COLOR_MAP() {
      return TYPE_COLOR_MAP;
    },
    PLACE_MAP() {
      return PLACE_MAP;
    },
    PLACE_COLOR_MAP() {
      return PLACE_COLOR_MAP;
    },
    MONSTER_LIST() {
      return MONSTER_LIST;
    }
  },
  methods: {
    // 任务计数按钮响应
    clickCountButton(bigOrSmall, index, i) {
      if (bigOrSmall === 2) {
        // 每点击一次代表任务被使用一次，次数-1
        this.data[index].bigMissions[i].count -= 1;
        if (this.data[index].bigMissions[i].count === 0) {
          // 次数为零时删除任务
          this.data[index].bigMissions.splice(i, 1);
          this.$Message.info('任务次数已清零');
        }
      } else if (bigOrSmall === 1) {
        this.data[index].smallMissions[i].count -= 1;
        if (this.data[index].smallMissions[i].count === 0) {
          this.data[index].smallMissions.splice(i, 1);
          this.$Message.info('任务次数已清零');
        }
      }
      this.saveToLocalStorage();
    },
    // 打开新增任务表单对话框
    openAddModal(bigOrSmall, index, name) {
      this.addModal.display = true;
      this.addModal.form = {
        name: name,
        bigOrSmall: bigOrSmall,
        star: 3,
        type: '',
        place: '',
      };
      this.addModal.index = index;
    },
    // 新增任务提交
    addModalSubmit() {
      this.$refs['addModalForm'].validate((valid) => {
        if (!valid) {
          this.$Message.warning('参数不完整');
          return;
        }
        let newMission = {
          star: this.addModal.form.star,
          count: 3,
          type: this.addModal.form.type,
          place: this.addModal.form.place,
        };
        if (this.addModal.form.bigOrSmall === 2) {
          this.data[this.addModal.index].bigMissions.push(newMission);
        } else if (this.addModal.form.bigOrSmall === 1) {
          this.data[this.addModal.index].smallMissions.push(newMission);
        }
        this.addModal.display = false;
        this.saveToLocalStorage();
      })
    },
    // 每次操作后，自动存储到浏览器缓存
    saveToLocalStorage() {
      console.log(this.$data.data);
      localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(this.$data.data));
    },
    // 清空本地数据记录
    clearLocalStorage() {
      this.$Modal.confirm({
        title: '清空数据后无法恢复',
        onOk: () => {
          localStorage.clear(LOCAL_STORAGE_KEY);
          this.$data.data = MONSTER_LIST;
          this.$Message.info('本地数据已清空')
        },
        onCancel: () => {
          return;
        }
      });
    }
  },
  mounted() {
    // 启动时，从浏览器缓存中读取数据
    let cache = localStorage.getItem(LOCAL_STORAGE_KEY);
    if (!cache) {
      // 缓存中无数据时，读取怪物列表，其中没有任务信息
      this.$data.data = MONSTER_LIST;
    } else {
      if (cache.length < MONSTER_LIST.length) {
        // 怪物列表的长度比缓存中数据长时，说明有追加新怪物，补充到末尾
        for (let i = cache.length; i < MONSTER_LIST.length; i++) {
          cache.push(MONSTER_LIST[i]);
        }
      }
      this.$data.data = JSON.parse(cache);
    }
  }
}
</script>

<style scoped>
.name-pic-cell {
  display: flex;
  /* justify-content: center; */
  align-items: center;
}
</style>
