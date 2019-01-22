<template>
    <div class="calculator">
        <header class="header">
            <p>《阴阳师》御魂组合计算器</p>
        </header>
        <section class="main">
            <div class="control">
                <Button type="primary">选择输入文件</Button>
            </div>
            <Form class="filter-options-form">
                <FormItem label="">
                    <Select v-model="firstYunhun" placeholder="请选择主御魂套装">
                        <Option v-for="yuhun in yunhunOptions" :key="yuhun" :label="yuhun" :value="yuhun"></Option>
                    </Select>
                </FormItem>
                <FormItem label="">
                    <Select v-model="secondYunhun" placeholder="请选择副御魂套装">
                        <Option v-for="yuhun in yunhunOptions" :key="yuhun" :label="yuhun" :value="yuhun"></Option>
                    </Select>
                </FormItem>
                <FormItem>
                    <Checkbox v-model="usePackage" label="限定使用套装"></Checkbox>
                    <Checkbox v-model="useAttack" label="限定使用输出御魂"></Checkbox>
                </FormItem>
                <FormItem label="二号位属性" class="multi-checkbox">
                    <CheckboxGroup v-model="twoAttribute">
                        <Checkbox label="攻击+55%"></Checkbox>
                        <Checkbox label="生命+55%"></Checkbox>
                        <Checkbox label="防御+55%"></Checkbox>
                        <Checkbox label="速度+57"></Checkbox>
                    </CheckboxGroup>
                </FormItem>
                <FormItem label="四号位属性" class="multi-checkbox">
                    <CheckboxGroup v-model="fourAttribute">
                        <Checkbox label="攻击+55%"></Checkbox>
                        <Checkbox label="生命+55%"></Checkbox>
                        <Checkbox label="防御+55%"></Checkbox>
                        <Checkbox label="命中+55%"></Checkbox>
                        <Checkbox label="抵抗+55%"></Checkbox>
                    </CheckboxGroup>
                </FormItem>
                <FormItem label="六号位属性" class="multi-checkbox">
                    <CheckboxGroup v-model="sixAttribute">
                        <Checkbox label="攻击+55%"></Checkbox>
                        <Checkbox label="生命+55%"></Checkbox>
                        <Checkbox label="防御+55%"></Checkbox>
                        <Checkbox label="暴击+55%"></Checkbox>
                        <Checkbox label="爆伤+89%"></Checkbox>
                    </CheckboxGroup>
                </FormItem>
                <FormItem label="">
                    <Input placeholder="忽略指定关键字御魂(以,分隔)" />
                </FormItem>
            </Form>
            <Form class="expect-options-form">
                <FormItem class="input-item" label="伤害期望">
                    <Input placeholder="格式:基础攻击,基础爆伤,期望值" />
                </FormItem>
                <FormItem class="input-item" label="治疗期望">
                    <Input placeholder="格式:基础生命,基础爆伤,期望值" />
                </FormItem>
                <FormItem label="目标属性下限">
                    <br/>
                    <div class="attribute-inputs">
                        <label>属性:</label>
                        <Select v-model="lowerAttribute">
                            <Option v-for="attr in attributes" :key="attr" :label="attr" :value="attr"></Option>
                        </Select>
                        <label>数值:</label>
                        <InputNumber :min="0" :max="999" size="small" v-model="lowerValue"></InputNumber>
                    </div>
                </FormItem>
                <FormItem class="attribute-results">
                    <span v-for="lower in this.lowerList" :key="lower" @click="removeLower(lower)">{{lower}}</span>
                </FormItem>
                <FormItem label="目标属性上限">
                    <br/>
                    <div class="attribute-inputs">
                        <label>属性:</label>
                        <Select v-model="upperAttribute">
                            <Option v-for="attr in attributes" :key="attr" :label="attr" :value="attr"></Option>
                        </Select>
                        <label>数值:</label>
                        <InputNumber :min="0" :max="999" size="small" v-model="upperValue"></InputNumber>
                    </div>
                </FormItem>
                <FormItem class="attribute-results">
                    <span v-for="upper in this.upperList" :key="upper" @click="removeUpper(upper)">{{upper}}</span>
                </FormItem>
            </Form>
        </section>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from 'vue-property-decorator';
import { Form, FormItem, Button, Select, Option, CheckboxGroup, Checkbox, Input, InputNumber, Message } from 'element-ui';

@Component({
    components: { Form, FormItem, Button, Select, Option, CheckboxGroup, Checkbox, Input, InputNumber },
})
export default class Calculator extends Vue {
    /**
     * 主御魂御魂
     */
    private firstYunhun = ''
    /**
     * 副御魂
     */
    private secondYunhun = ''

    /**
     * 是否使用套装
     */
    private usePackage = true
    /**
     * 是否使用攻击类御魂
     */
    private useAttack = false

    /**
     * 二号位属性
     */
    private twoAttribute: string[] = []
    /**
     * 四号位属性
     */
    private fourAttribute: string[] = []
    /**
     * 六号位属性
     */
    private sixAttribute: string[] = []

    /**
     * 下限属性
     */
    private lowerAttribute = ''
    /**
     * 下限数值
     */
    private lowerValue = 0
    /**
     * 下限配置列表
     */
    private lowerList: string[] = []
    /**
     * 上限属性
     */
    private upperAttribute = ''
    /**
     * 上限数值
     */
    private upperValue = 0
    /**
     * 上限配置列表
     */
    private upperList: string[] = []

    /**
     * 御魂套装选项
     */
    get yunhunOptions(): string[] {
        return ['不限', '针女', '破势', '网切', '三味', '镇墓兽', '伤魂鸟', '蝠翼', '鸣屋', '心眼', '狰', '轮入道', '狂骨', '阴摩罗', '魍魉之匣', '骰子鬼', '返魂香', '幽谷响', '蚌精', '火灵', '树妖', '地藏像', '薙魂', '镜姬', '钟灵', '被服', '涅槃之火', '招财猫', '魅妖', '反枕', '木魅', '日女己时', '雪幽魂', '珍珠', '荒骷髅', '土蜘蛛', '地震鲶', '蜃气楼', '胧车'];
    }
    /**
     * 属性选项
     */
    get attributes(): string[] {
        return ['暴击', '暴击伤害', '效果命中', '效果抵抗', '速度', '攻击加成', '生命加成', '防御加成'];
    }

    /**
     * 添加下限属性
     */
    addLower(): void {
        if (!this.lowerAttribute) {
            Message.error('请选择属性');
            return;
        }

        const newItem = `${this.lowerAttribute}+${this.lowerValue}${this.lowerAttribute === '速度' ? '' : '%'}`;
        const index = this.lowerList.findIndex(lower => lower.startsWith(this.lowerAttribute));
        if (index === -1) {
            this.lowerList.push(newItem);
        } else {
            this.lowerList.splice(index, 1, newItem);
        }
    }
    /**
     * 删除下限属性
     */
    removeLower(item: string): void {
        const index = this.lowerList.findIndex(lower => lower === item);
        if (index !== -1) {
            this.lowerList.splice(index, 1);
        }
    }
    @Watch('lowerAttribute')
    onLowerAttributeChange(val: string) {
        this.lowerValue = 0;
        this.addLower();
    }
    @Watch('lowerValue')
    onLowerValueChange(val: string) {
        this.addLower();
    }

    /**
     * 添加上限属性
     */
    addUpper(): void {
        if (!this.upperAttribute) {
            Message.error('请选择属性');
            return;
        }

        const newItem = `${this.upperAttribute}+${this.upperValue}${this.upperAttribute === '速度' ? '' : '%'}`;
        const index = this.upperList.findIndex(upper => upper.startsWith(this.upperAttribute));
        if (index === -1) {
            this.upperList.push(newItem);
        } else {
            this.upperList.splice(index, 1, newItem);
        }
    }
    /**
     * 删除下限属性
     */
    removeUpper(item: string): void {
        const index = this.upperList.findIndex(upper => upper === item);
        if (index !== -1) {
            this.upperList.splice(index, 1);
        }
    }
    @Watch('upperAttribute')
    onUpperttributeChange(val: string) {
        this.upperValue = 0;
        this.addUpper();
    }
    @Watch('upperValue')
    onUpperValueChange(val: string) {
        this.addUpper();
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
            width: 200px;
        }
        .filter-options-form {
            width: 290px;
            margin: 0 8px;
        }
        .expect-options-form {
            flex: 1;
            margin: 0 8px;

            .attribute-inputs {
                display: flex;
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
                .el-button {
                    height: 34px;
                    margin-left: 12px;
                    transform: translateY(2px);
                    span {
                        position: relative;
                        top: -3px;
                    }
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
            display: flex;
            align-items: center;

            .el-form-item__label {
                width: 50px;
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
    input, .el-checkbox__inner, .el-input-number__decrease, .el-input-number__increase {
        background-color: #11171f;
        border: 1px solid #333b47;
    }
}
</style>
