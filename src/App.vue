<template>
  <PageHeader style="padding: 0 32px 0 32px">
    <template #title>
      <b>MHWs大小金冠及任务记录</b>
    </template>
    <template #logo>
      <a href="https://github.com/riviera613/mhws_missions_record" target="_blank">
        <Icon type="logo-github" style="font-size: 24px;" />
      </a>
    </template>
    <template #action>
      <Poptip placement="bottom-end">
        <Button type="text" shape="circle" style="font-size: 20px; margin-right: 8px;">
          <Icon type="ios-help-circle-outline" />
        </Button>
        <template #content>数据存储在浏览器缓存中</template>
      </Poptip>
      <Button type="primary" @click="openAddModal" style="margin-right: 8px;">
        <Icon type="md-add" />新增任务
      </Button>
      <Button type="warning" @click="clearLocalStorage">
        <Icon type="md-trash" />清空数据
      </Button>
    </template>
  </PageHeader>
  <!-- 主表格 -->
  <Table border stripe :columns="columns" :data="data" :width="tableWidth" style="margin: 16px 32px 0 32px;">
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
      <MissionList :data="row.bigMissions" :index="index" @onUpdate="updateBigMissions"></MissionList>
    </template>
    <!-- 小金冠任务列表 -->
    <template #smallMissions="{ row, index }">
      <MissionList :data="row.smallMissions" :index="index" @onUpdate="updateSmallMissions"></MissionList>
    </template>
  </Table>

  <!-- 新增任务表单 -->
  <Modal v-model="addModal.display" title="新增任务">
    <Form :label-width="80" label-position="left" :model="addModal.form" ref="addModalForm"
      :rules="addModalFormValidate">
      <FormItem label="名称" prop="name">
        <Select v-model="addModal.form.name" filterable>
          <Option v-for="monster in MONSTER_LIST" :disabled="monster.disabled" :value="monster.name"
            :label="monster.name">
            {{
              monster.name }}</Option>
        </Select>
      </FormItem>
      <FormItem label="大小" prop="bigOrSmall">
        <RadioGroup v-model="addModal.form.bigOrSmall" type="button">
          <Radio :label="BIG" :key="BIG">大金冠</Radio>
          <Radio :label="SMALL" :key="SMALL">小金冠</Radio>
        </RadioGroup>
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
        <Select v-model="addModal.form.place" prop="place" filterable>
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
import { BIG, SMALL, TYPE_MAP, PLACE_MAP, MONSTER_LIST } from './consts.js'
import MissionList from './components/MissionList.vue';
const LOCAL_STORAGE_KEY = 'mhws_crowns_mission';
export default {
  data() {
    return {
      columns: [
        {
          title: '怪物',
          key: 'name',
          slot: 'name',
          maxWidth: 200,
        },
        {
          title: '大金冠',
          key: 'bigFlag',
          slot: 'bigFlag',
          maxWidth: 100,
        },
        {
          title: '小金冠',
          key: 'smallFlag',
          slot: 'smallFlag',
          maxWidth: 100,
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
          bigOrSmall: '',
          star: 3,
          type: '',
          place: '',
        },
        index: 0,
      },
      addModalFormValidate: {
        name: [
          { required: true, message: 'Can not be empty', trigger: 'blur' }
        ],
        bigOrSmall: [
          { type: 'number', required: true, message: 'Can not be empty', trigger: 'blur' }
        ],
        type: [
          { required: true, message: 'Can not be empty', trigger: 'blur' }
        ],
      }
    };
  },
  components: {
    MissionList
  },
  computed: {
    TYPE_MAP() {
      return TYPE_MAP;
    },
    PLACE_MAP() {
      return PLACE_MAP;
    },
    MONSTER_LIST() {
      return MONSTER_LIST;
    },
    BIG() {
      return BIG;
    },
    SMALL() {
      return SMALL;
    },
    tableWidth() {
      if (window.screen.width < window.screen.height) {
        let screenWidth = window.screen.width;
        return screenWidth > 1000 ? screenWidth : 1000;
      } else {
        return '';
      }
    }
  },
  methods: {
    updateBigMissions(data, index) {
      this.data[index].bigMissions = data;
      this.saveToLocalStorage();
    },
    updateSmallMissions(data, index) {
      this.data[index].smallMissions = data;
      this.saveToLocalStorage();
    },
    // 打开新增任务表单对话框
    openAddModal() {
      this.addModal.display = true;
      this.addModal.form = {
        name: '',
        bigOrSmall: '',
        star: 3,
        type: '',
        place: '',
      };
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
        let index = -1;
        for (let i in this.data) {
          if (this.data[i].name === this.addModal.form.name) {
            index = i;
            break;
          }
        }
        if (index < 0) {
          this.$Message.error('怪物名错误');
          return;
        }
        if (this.addModal.form.bigOrSmall === this.BIG) {
          this.data[index].bigMissions.push(newMission);
        } else if (this.addModal.form.bigOrSmall === this.SMALL) {
          this.data[index].smallMissions.push(newMission);
        }
        this.addModal.display = false;
        this.saveToLocalStorage();
      })
    },
    // 每次操作后，自动存储到浏览器缓存
    saveToLocalStorage() {
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
    },
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

a:not(:hover) {
  color: black;
}
</style>
