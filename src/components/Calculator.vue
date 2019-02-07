<template>
    <div class="calculator">
        <header class="header">
            <p>《阴阳师》御魂计算器</p>
        </header>
        <section class="main">
            <Form class="filter-options-form">
                <div class="yuhun-package-input">
                    <p class="title">御魂套装组合</p>
                    <Select class="yuhun" v-model="yuhunPackage" placeholder="选择御魂套装">
                        <OptionGroup v-for="group in yuhunOptions" :key="group.name" :label="group.name">
                            <Option v-for="yuhun in group.list" :key="yuhun" :label="yuhun" :value="yuhun"></Option>
                        </OptionGroup>
                    </Select>
                    <Button type="primary" @click="addYuhunPackageLimit" :disabled="disableAddYuhunPackageButton">添加</Button>
                    <div class="select-yuhun-list">
                        <Tag v-for="(yuhunPackage, index) in this.yuhunPackageList" closable :key="index" @close="removeYuhunPackageLimit(yuhunPackage)">{{yuhunPackage}}</Tag>
                    </div>
                </div>
                <FormItem>
                    <Checkbox v-model="usePackage" label="限定使用套装"></Checkbox>
                    <Checkbox v-model="useAttack" label="限定使用输出御魂"></Checkbox>
                </FormItem>
                <FormItem class="multi-checkbox">
                    <p class="title">二号位属性</p>
                    <CheckboxGroup v-model="secondAttributeList" size="mini">
                        <CheckboxButton label="攻击加成"></CheckboxButton>
                        <CheckboxButton label="生命加成"></CheckboxButton>
                        <CheckboxButton label="防御加成"></CheckboxButton>
                        <CheckboxButton label="速度"></CheckboxButton>
                    </CheckboxGroup>
                </FormItem>
                <FormItem class="multi-checkbox">
                    <p class="title">四号位属性</p>
                    <CheckboxGroup v-model="fourthAttributeList" size="mini">
                        <CheckboxButton label="攻击加成"></CheckboxButton>
                        <CheckboxButton label="生命加成"></CheckboxButton>
                        <CheckboxButton label="防御加成"></CheckboxButton>
                        <CheckboxButton label="效果命中"></CheckboxButton>
                        <CheckboxButton label="效果抵抗"></CheckboxButton>
                    </CheckboxGroup>
                </FormItem>
                <FormItem class="multi-checkbox">
                    <p class="title">六号位属性</p>
                    <CheckboxGroup v-model="sixthAttributeList" size="mini">
                        <CheckboxButton label="攻击加成"></CheckboxButton>
                        <CheckboxButton label="生命加成"></CheckboxButton>
                        <CheckboxButton label="防御加成"></CheckboxButton>
                        <CheckboxButton label="暴击"></CheckboxButton>
                        <CheckboxButton label="暴击伤害"></CheckboxButton>
                    </CheckboxGroup>
                </FormItem>
                <FormItem label="">
                    <Input placeholder="忽略指定关键字御魂(以,分隔)" v-model="ignoreSerial" />
                </FormItem>
            </Form>
            <Form class="expect-options-form">
                <FormItem class="input-item" label="伤害期望">
                    <Input placeholder="格式: 式神基础攻击,式神基础爆伤,期望伤害值" v-model="damageExpect" />
                </FormItem>
                <FormItem class="input-item" label="治疗期望">
                    <Input placeholder="格式: 式神基础生命,式神基础爆伤,期望治疗值" v-model="healthExpect" />
                </FormItem>
                <FormItem>
                    <p class="title">目标属性限制</p>
                    <label>属性:</label>
                    <Select v-model="targetAttribute">
                        <Option v-for="attr in attributes" :key="attr" :label="attr" :value="attr"></Option>
                    </Select>
                    <br/>
                    <div v-show="targetAttribute !== ''">
                        <label>下限:</label>
                        <InputNumber :min="0" :max="999" size="small" v-model="lowerValue" controls-position="right"></InputNumber>
                        <label>上限:</label>
                        <InputNumber :min="0" :max="999" size="small" v-model="upperValue" controls-position="right"></InputNumber>
                    </div>
                </FormItem>
                <FormItem class="attribute-results">
                    <Tag v-for="attr in this.targetAttributeList" closable :key="attr.split(' ')[0]" @close="removeTargetAttribute(attr)">{{attr}}</Tag>
                </FormItem>
            </Form>
            <Form class="control">
                <FormItem label="输入文件">
                    <br/>
                    <p v-if="filename">{{filename}}</p>
                    <p v-else>尚未选择御魂数据文件</p>
                    <Button type="primary" @click="selectFile">选择文件</Button>
                </FormItem>
                <FormItem>
                    <Button v-if="calculateProgress === 100" type="primary" @click="run" :disabled="!serverOnline">开始计算</Button>
                    <Button v-else type="primary" loading>计算中... {{calculateProgress}}%</Button>
                </FormItem>
                <CustomScheme :currentScheme="currentScheme" @selectScheme="handleSelectScheme"></CustomScheme>
            </Form>
        </section>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from 'vue-property-decorator';
import { Form, FormItem, Button, Select, Option, OptionGroup, CheckboxGroup, Checkbox, CheckboxButton, Input, InputNumber, Message, Tag, Notification, Dialog } from 'element-ui';
import axios from 'axios';

import Scheme from '../definition/Scheme';
import CustomScheme from './CustomScheme.vue';

const apiRoot = '//localhost:2019';
const axiosOption = {
    validateStatus(status: number) {
        return status >= 200 && status <= 500;
    },
};

@Component({
    components: { Form, FormItem, Button, Select, Option, OptionGroup, CheckboxGroup, Checkbox, CheckboxButton, Input, InputNumber, Tag, Dialog, CustomScheme },
})
export default class Calculator extends Vue {
    private serverOnline = true;
    /**
     * 御魂套装
     */
    private yuhunPackage = '';
    /**
     * 御魂套装限制列表
     */
    private yuhunPackageList: string[] = [];

    /**
     * 是否使用套装
     */
    private usePackage = true;
    /**
     * 是否使用攻击类御魂
     */
    private useAttack = false;

    /**
     * 二号位属性
     */
    private secondAttributeList: string[] = [];
    /**
     * 四号位属性
     */
    private fourthAttributeList: string[] = [];
    /**
     * 六号位属性
     */
    private sixthAttributeList: string[] = [];

    /**
     * 忽略指定关键字御魂
     */
    private ignoreSerial = '';

    /**
     * 伤害期望
     */
    private damageExpect = '';
    /**
     * 生命期望
     */
    private healthExpect = '';

    /**
     * 目标属性
     */
    private targetAttribute = '';
    /**
     * 下限数值
     */
    private lowerValue = 0;
    /**
     * 上限数值
     */
    private upperValue = 999;
    /**
     * 属性限制列表
     */
    private targetAttributeList: string[] = [];

    /**
     * 选择的文件名称
     */
    private filename = '';

    /**
     * 计算进度
     */
    private calculateProgress = 100;

    /**
     * 御魂套装选项
     */
    get yuhunOptions(): Object[] {
        return [{
            name: '散件',
            list: ['攻击加成,2', '暴击,2', '生命加成,2', '效果命中,2', '效果抵抗,2'],
        }, {
            name: '首领御魂',
            list: ['荒骷髅,2', '土蜘蛛,2', '地震鲶,2', '蜃气楼,2', '胧车,2'],
        }, {
            name: '暴击',
            list: ['针女,4', '破势,4', '网切,4', '三味,4', '伤魂鸟,4', '镇魂兽,4'],
        }, {
            name: '攻击加成',
            list: ['蝠翼,4', '鸣屋,4', '心眼,4', '狰,4', '轮入道,4', '狂骨,4', '阴摩罗,4'],
        }, {
            name: '生命加成',
            list: ['树妖,4', '地藏像,4', '薙魂,4', '镜姬,4', '钟灵,4', '涅槃之火,4', '被服,4'],
        }, {
            name: '防御加成',
            list: ['珍珠,4', '魅妖,4', '雪幽魂,4', '招财猫,4', '反枕,4', '日女己时,4', '木魅,4'],
        }, {
            name: '效果命中',
            list: ['蚌精,4', '火灵,4'],
        }, {
            name: '效果抵抗',
            list: ['骰子鬼,4', '返魂香,4', '幽谷响,4', '魍魉之匣,4' ],
        }];
    }
    /**
     * 属性选项
     */
    get attributes(): string[] {
        return ['暴击', '暴击伤害', '效果命中', '效果抵抗', '速度', '攻击加成', '生命加成', '防御加成'];
    }

    /**
     * 是否禁用添加御魂套装按钮
     */
    get disableAddYuhunPackageButton(): boolean {
        if (!this.yuhunPackage) {
            return true;
        }

        let currentSelectCount = 0;
        if (this.yuhunPackage) {
            const [, count] = this.yuhunPackage.split(',');
            currentSelectCount = +count;
        }

        return currentSelectCount + this.selectedYuhunPackageCount > 6;
    }

    /**
     * 已选的御魂套装组合所用的御魂个数
     */
    get selectedYuhunPackageCount(): number {
        return this.yuhunPackageList
            .map((yuhunPackage) => {
                const [, count] = yuhunPackage.split(',');
                return +count;
            })
            .reduce((sum, value) => sum += value, 0);
    }

    /**
     * 当前编辑中的方案
     */
    get currentScheme(): Object {
        return {
            yuhunPackageList: this.yuhunPackageList,
            usePackage: this.usePackage,
            useAttack: this.useAttack,
            secondAttributeList: this.secondAttributeList,
            fourthAttributeList: this.fourthAttributeList,
            sixthAttributeList: this.sixthAttributeList,
            ignoreSerial: this.ignoreSerial,
            damageExpect: this.damageExpect,
            healthExpect: this.healthExpect,
            targetAttributeList: this.targetAttributeList,
        }
    }

    private mounted() {
        axios.get(apiRoot + '/status').catch(() => {
            Notification.error('未检测到服务端, 请先启动服务端后再试');
            this.serverOnline = false;
        })
    }

    /**
     * 添加御魂套装限制
     */
    @Watch('yuhunPackage')
    public addYuhunPackageLimit(): void {
        if (!this.disableAddYuhunPackageButton) {
            this.yuhunPackageList.push(this.yuhunPackage);
        }
    }
    public removeYuhunPackageLimit(yuhunPackage: string): void {
        const index = this.yuhunPackageList.findIndex((x) => x === yuhunPackage);
        this.yuhunPackageList.splice(index, 1);
    }

    /**
     * 添加目标属性规则
     */
    public addTargetAttribute(): void {
        const newItem = `${this.targetAttribute} ${this.lowerValue || 0} - ${this.upperValue || 0}`;
        const index = this.targetAttributeList.findIndex((attr) => attr.split(' ')[0] === this.targetAttribute);
        if (index === -1) {
            this.targetAttributeList.push(newItem);
        } else {
            this.targetAttributeList.splice(index, 1, newItem);
        }
    }
    /**
     * 删除目标属性规则
     */
    public removeTargetAttribute(item: string): void {
        const index = this.targetAttributeList.findIndex((attr) => attr === item);
        if (index !== -1) {
            this.targetAttributeList.splice(index, 1);
        }
    }
    @Watch('targetAttribute')
    public ontargetAttributeChange(val: string) {
        if (val) {
            this.lowerValue = 0;
            this.upperValue = 0;
            this.addTargetAttribute();
        }
    }
    @Watch('lowerValue')
    public onLowerValueChange(val: number, oldVal: number) {
        if (oldVal === 0) {
            this.upperValue = val + 10;
        }
        if (this.upperValue < val) {
            this.upperValue = val;
        }
        this.addTargetAttribute();
    }
    @Watch('upperValue')
    public onUpperValueChange(val: number) {
        if (this.lowerValue > val) {
            this.lowerValue = val;
        }
        this.addTargetAttribute();
    }


    /**
     * 选择文件
     */
    public selectFile() {
        const $input = document.createElement('input');
        $input.style.display = 'none';
        $input.setAttribute('type', 'file');
        $input.setAttribute('accept', '.json,.xls');
        // 判断用户是否点击取消, 原生没有提供专门事件, 用hack的方法实现
        $input.onclick = () => {
            $input.value = '';
            document.body.onfocus = () => {
                // onfocus事件会比$input.onchange事件先触发, 因此需要延迟一段时间
                setTimeout(() => {
                    if ($input.value.length === 0) {
                        // 未选择文件
                    }
                    document.body.onfocus = null;
                }, 500);
            };
        };
        $input.onchange = (e: any) => {
            const file = e.target.files[0];
            if (!file) {
                return;
            }

            this.filename = file.name;
        };
        $input.click();
    }

    private handleSelectScheme(scheme: Scheme) {
        this.yuhunPackageList = scheme.yuhunPackageList;
        this.usePackage = scheme.usePackage;
        this.useAttack = scheme.useAttack;
        this.secondAttributeList = scheme.secondAttributeList;
        this.fourthAttributeList = scheme.fourthAttributeList;
        this.sixthAttributeList = scheme.sixthAttributeList;
        this.ignoreSerial = scheme.ignoreSerial;
        this.damageExpect = scheme.damageExpect;
        this.healthExpect = scheme.healthExpect;
        this.targetAttributeList = scheme.targetAttributeList;
        this.targetAttribute = '';
    }

    private getClaculateStatus() {
        axios
            .get(apiRoot + '/status')
            .then(result => {
                const {progress} = result.data;
                if (progress === undefined) {
                    return;
                }
                this.calculateProgress = Math.floor(progress * 100);
                if (progress !== 1) {
                    setTimeout(this.getClaculateStatus.bind(this), 100);
                }
            })
            .catch(() => {
                console.warn('获取计算进度失败');
            })
    }

    /**
     * 获取后端所需的 prop_value 格式文本
     */
    private getPropValue(attributeList: string[]): string {
        return attributeList.map(attr => {
            switch(attr) {
                case '速度': return '速度,57';
                case '暴击伤害': return '暴击伤害,89';
                default: return attr + ',55'
            }
        }).join('.');
    }

    /**
     * 开始计算
     */
    public run() {
        if (!this.filename) {
            Message.warning('请先选择御魂数据文件');
            return;
        }

        axios.post(apiRoot + '/calculate', {
            src_filename: this.filename,
            mitama_suit: this.yuhunPackageList.join('.'),
            prop_limit: this.targetAttributeList.map(attr => {
                const [attrName, value] = attr.split(/ |-/);
                return attrName + ',' + value;
            }).join('.'),
            upper_prop_limit: this.targetAttributeList.map(attr => {
                const [attrName, , , ,value] = attr.split(/ |-/);
                return attrName + ',' + value;
            }).join('.'),
            sec_prop_value: this.getPropValue(this.secondAttributeList),
            fth_prop_value: this.getPropValue(this.fourthAttributeList),
            sth_prop_value: this.getPropValue(this.sixthAttributeList),
            ignore_serial: this.ignoreSerial,
            all_suit: this.usePackage ? 'True' : 'False',
            damage_limit: this.damageExpect || '0,0,0',
            health_limit: this.healthExpect || '0,0,0',
            attack_only: this.useAttack ? 'True' : 'False',
            effective_secondary_prop: '',
            effective_secondary_prop_num: '',
        }, axiosOption).then((result) => {
            switch (result.status) {
                case 500: {
                    Message.error('计算失败, 服务端错误');
                    console.error(result.data.reason);
                    break;
                }
                case 200: {
                    Message.success(`计算完毕, 组合数量:${result.data.result_num}`);
                    break;
                }
                default: {
                    Message.error(`计算失败, 服务端返回状态码:${result.status}`);
                    break;
                }
            }
        }).catch((err) => {
            Message.error('计算失败');
        });

        setTimeout(this.getClaculateStatus.bind(this), 200);
    }
}
</script>

<style lang="less">
.calculator {
    width: 940px;
    height: 540px;
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    background-color: #1D2633;
    display: flex;
    flex-direction: column;
    border-radius: 6px;

    .header {
        background-image: url('../assets/header.png');
        width: 100%;
        height: 60px;
        position: relative;

        p {
            color: #f1f1f1;
            position: absolute;
            top: 22px;
            left: 60px;
        }
    }
    .main {
        flex: 1;
        padding: 8px 16px;
        display: flex;

        .control {
            width: 210px;
            padding: 4px 8px;

            p {
                color: #ccc;
                line-height: 28px;
                position: relative;
                top: -6px;
            }
            .el-button {
                width: 100%;
                margin-bottom: 12px;
            }
            .custom-setting {
                margin-top: 50px;
                .el-button {
                    margin-bottom: 5px;
                }
            }
        }
        .filter-options-form {
            width: 300px;
            margin: 0 8px;

            .yuhun-package-input {
                .el-form-item__label {
                    line-height: 24px;
                }
                .el-select {
                    width: 200px;
                    margin-bottom: 8px;

                    .el-select__caret {
                        line-height: 34px;
                    }
                }
                .el-button {
                    margin-left: 10px;
                    width: 90px;
                    height: 34px;
                    transform: translateY(2px);

                    span {
                        position: relative;
                        top: -3px;
                    }
                }
                .select-yuhun-list {
                    width: 100%;
                    min-height: 36px;

                    .el-tag {
                        margin-right: 8px;
                        margin-bottom: 4px;
                    }
                }
            }
        }
        .expect-options-form {
            flex: 1;
            margin: 0 8px;

            label {
                color: #f1f1f1;
                margin-right: 4px;
                &:nth-child(3) {
                    margin-left: 12px;
                }
            }
            .el-select {
                width: 130px;
            }
            .el-input-number {
                width: 100px;
            }
            .el-button {
                height: 34px;
                margin-left: 12px;
                transform: translateY(2px);
                span {
                    position: relative;
                    top: -3px;
                }
            }
            .attribute-results {
                height: 66px;
                color: #409EFF;
                span {
                    margin-right: 12px;
                    cursor: pointer;
                    user-select: none;
                }
            }
        }
        .el-icon-info {
            color: #ddd;
        }
    }
    .el-form-item__label, .el-checkbox__label {
        color: #eee;
    }
    .el-input__inner {
        height: 34px;
    }
    .el-form-item {
        margin-bottom: 5px;

        &.multi-checkbox {
            .el-checkbox-button {
                &.is-checked {
                    .el-checkbox-button__inner {
                        color: #fff;
                        background-color: #409EFF;
                        border-color: #409EFF;
                        box-shadow: -1px 0 0 0 #8cc5ff;
                    }
                }
            }
            .el-checkbox-button__inner {
                min-width: 60px;
                padding: 7px 5px;
                background-color: #11171f;
                border: 1px solid #333b47;
                color: #ccc;
            }
            .el-form-item__label {
                // width: 50px;
                padding: 0;
                text-align: center;
                line-height: 22px;
            }
            .el-form-item__content {
                flex: 1;
            }
            .el-checkbox-group {
                display: flex;
                align-items: center;
                justify-content: flex-start;
                flex-wrap: wrap;
                transform: translateY(-5px);
            }
            .el-checkbox {
                margin-left: 10px;
                height: 30px;
            }
        }
        &.input-item {
            .el-form-item__label {
                line-height: 24px;
            }
        }
    }
    .el-select {
        width: 100%;
    }
    input, .el-checkbox__inner {
        background-color: #11171f;
        border: 1px solid #333b47;
        color: #ccc;
        &::placeholder {
            color: #606266;
        }
    }
    p.title {
        color: #eee;
        line-height: 24px;
    }
    .el-input-number__decrease, .el-input-number__increase {
        background-color: #11171f;
        border-left-color: #333b47 !important;
        color: #ccc;
    }
    .el-input-number__increase {
        border-bottom-color: #333b47 !important;
    }
}
</style>
