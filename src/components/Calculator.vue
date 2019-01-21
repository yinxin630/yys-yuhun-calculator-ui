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
                        <Option v-for="yuhun in firstYunhunOptions" :key="yuhun" :label="yuhun" :value="yuhun"></Option>
                    </Select>
                </FormItem>
                <FormItem label="">
                    <Select v-model="secondYunhun" placeholder="请选择副御魂套装">
                        <Option v-for="yuhun in secondYuhunOptions" :key="yuhun" :label="yuhun" :value="yuhun"></Option>
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
                        <Select>
                            <Option v-for="attr in attributes" :key="attr" :label="attr" :value="attr"></Option>
                        </Select>
                        <label>数值:</label>
                        <InputNumber :min="0" :max="89"></InputNumber>
                        <Button type="primary">添加</Button>
                    </div>
                </FormItem>
                <FormItem class="attribute-results">
                    <span>暴击+95%</span>
                </FormItem>
                <FormItem label="目标属性上限">
                    <br/>
                    <div class="attribute-inputs">
                        <label>属性:</label>
                        <Select>
                            <Option v-for="attr in attributes" :key="attr" :label="attr" :value="attr"></Option>
                        </Select>
                        <label>数值:</label>
                        <InputNumber :min="0" :max="89"></InputNumber>
                        <Button type="primary">添加</Button>
                    </div>
                </FormItem>
                <FormItem class="attribute-results">
                    <span>暴击+195%</span>
                </FormItem>
            </Form>
        </section>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import { Form, FormItem, Button, Select, Option, CheckboxGroup, Checkbox, Input, InputNumber } from 'element-ui';

@Component({
    components: { Form, FormItem, Button, Select, Option, CheckboxGroup, Checkbox, Input, InputNumber },
    computed: {
        firstYunhunOptions(): string[] {
            return ['针女', '破势'];
        },
        secondYuhunOptions(): string[] {
            return ['荒骷髅', '蜃气楼'];
        },
        attributes(): string[] {
            return ['暴击', '爆伤'];
        }
    }
})
export default class Calculator extends Vue {
    data() {
        return {
            /**
             * 主御魂御魂
             */
            firstYunhun: '',
            /**
             * 副御魂
             */
            secondYunhun: '',
            /**
             * 是否使用套装
             */
            usePackage: true,
            /**
             * 是否使用攻击类御魂
             */
            useAttack: false,
            /**
             * 二号位属性
             */
            twoAttribute: [],
            /**
             * 四号位属性
             */
            fourAttribute: [],
            /**
             * 六号位属性
             */
            sixAttribute: [],
        }
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
                    width: 110px;
                }
                .el-input-number {
                    width: 110px;
                }
                .el-input-number__decrease, .el-input-number__increase {
                    width: 30px;
                    height: 34px;
                    transform: translateY(1px);
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
    input {
        background-color: #f9f9f9;
    }
}
</style>
