<template>
    <div v-for="(mission, i) in missionList">
        <div v-if="mission.count > 0" style="padding: 4px; margin: 4px; border: 1px solid #dcdee2">
            <Icon type="md-star" /><span>{{ mission.star }}&nbsp;</span>
            <Tag :color="TYPE_COLOR_MAP[mission.type]">{{ TYPE_MAP[mission.type] }}</Tag>
            <Tag type="border">{{ PLACE_MAP[mission.place] }}</Tag>&nbsp;
            <Button size="small" style="font-size: 12px;" @click="clickCountButton(1, index, i)">{{ mission.count
                }}次</Button>
        </div>
    </div>
    <Button type="default" shape="circle" size="small" :disabled="row.disabled"
        @click="openAddModal(1, index, row.name)">
        <Icon type="md-add" />
    </Button>

</template>

<script>
export default {
    props: ['missionList'],
    setup(props) {
        console.log(props.missionList);
    },
    data() {
        return {
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
                place: [
                    { required: true, message: 'Can not be empty', trigger: 'blur' }
                ]
            }
        };
    },
    methods: {
        penAddModal(bigOrSmall, index, name) {
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
        clickCountButton(bigOrSmall, index, i) {
            if (bigOrSmall === 2) {
                this.data[index].bigMissions[i].count -= 1;
                if (this.data[index].bigMissions[i].count === 0) {
                    this.$Message.info('任务次数已清零');
                }
            } else if (bigOrSmall === 1) {
                this.data[index].smallMissions[i].count -= 1;
                if (this.data[index].smallMissions[i].count === 0) {
                    this.$Message.info('任务次数已清零');
                }
            }
            this.saveToLocalStorage();
        },
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
                    console.log(this.data[this.addModal.index]);
                    this.data[this.addModal.index].bigMissions.push(newMission);
                } else if (this.addModal.form.bigOrSmall === 1) {
                    this.data[this.addModal.index].smallMissions.push(newMission);
                }
                this.addModal.display = false;
                this.saveToLocalStorage();
            })
        },
    }
}
</script>