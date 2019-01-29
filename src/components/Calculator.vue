<template>
    <div class="calculator">
        <header class="header">
            <p>《阴阳师》御魂计算器</p>
        </header>
        <section class="main">
            <Form class="filter-options-form">
                <FormItem label="御魂套装组合" class="yuhun-package-input">
                    <Tooltip content="点击已选御魂可将其删除" placement="right">
                        <i class="el-icon-info"></i>
                    </Tooltip>
                    <br/>
                    <Select class="yuhun" v-model="yuhunPackage" placeholder="选择御魂套装">
                        <Option v-for="yuhun in yunhunOptions" :key="yuhun" :label="yuhun" :value="yuhun"></Option>
                    </Select>
                    <Button type="primary" @click="addYuhunPackageLimit" :disabled="disableAddYuhunPackageButton">添加</Button>
                    <div class="select-yuhun-list">
                        <span v-for="yuhunPackage in this.yuhunPackageList" :key="yuhunPackage" @click="removeYuhunPackageLimit(yuhunPackage)">{{yuhunPackage}}</span>
                    </div>
                </FormItem>
                <FormItem>
                    <Checkbox v-model="usePackage" label="限定使用套装"></Checkbox>
                    <Checkbox v-model="useAttack" label="限定使用输出御魂"></Checkbox>
                </FormItem>
                <FormItem label="二号位属性" class="multi-checkbox">
                    <CheckboxGroup v-model="secondAttributeList">
                        <Checkbox label="攻击加成,55"></Checkbox>
                        <Checkbox label="生命加成,55"></Checkbox>
                        <Checkbox label="防御加成,55"></Checkbox>
                        <Checkbox label="速度,57"></Checkbox>
                    </CheckboxGroup>
                </FormItem>
                <FormItem label="四号位属性" class="multi-checkbox">
                    <CheckboxGroup v-model="fourthAttributeList">
                        <Checkbox label="攻击加成,55"></Checkbox>
                        <Checkbox label="生命加成,55"></Checkbox>
                        <Checkbox label="防御加成,55"></Checkbox>
                        <Checkbox label="效果命中,55"></Checkbox>
                        <Checkbox label="效果抵抗,55"></Checkbox>
                    </CheckboxGroup>
                </FormItem>
                <FormItem label="六号位属性" class="multi-checkbox">
                    <CheckboxGroup v-model="sixthAttributeList">
                        <Checkbox label="攻击加成,55"></Checkbox>
                        <Checkbox label="生命加成,55"></Checkbox>
                        <Checkbox label="防御加成,55"></Checkbox>
                        <Checkbox label="暴击,55"></Checkbox>
                        <Checkbox label="暴击伤害,89"></Checkbox>
                    </CheckboxGroup>
                </FormItem>
                <FormItem label="">
                    <Input placeholder="忽略指定关键字御魂(以,分隔)" v-model="ignoreSerial" />
                </FormItem>
            </Form>
            <Form class="expect-options-form">
                <FormItem class="input-item" label="伤害期望">
                    <Input placeholder="格式:基础攻击,基础爆伤,期望值" v-model="damageExpect" />
                </FormItem>
                <FormItem class="input-item" label="治疗期望">
                    <Input placeholder="格式:基础生命,基础爆伤,期望值" v-model="healthExpect" />
                </FormItem>
                <FormItem label="目标属性下限">
                    <Tooltip content="点击已选属性可将其删除" placement="right">
                        <i class="el-icon-info"></i>
                    </Tooltip>
                    <br/>
                    <div class="attribute-inputs">
                        <label>属性:</label>
                        <Select v-model="lowerAttribute">
                            <Option v-for="attr in attributes" :key="attr" :label="attr" :value="attr"></Option>
                        </Select>
                        <label v-show="lowerAttribute !== ''">数值:</label>
                        <InputNumber :min="0" :max="999" size="small" v-model="lowerValue" v-show="lowerAttribute !== ''"></InputNumber>
                    </div>
                </FormItem>
                <FormItem class="attribute-results">
                    <span v-for="lower in this.lowerList" :key="lower" @click="removeLower(lower)">{{lower}}</span>
                </FormItem>
                <FormItem label="目标属性上限">
                    <Tooltip content="点击已选属性可将其删除" placement="right">
                        <i class="el-icon-info"></i>
                    </Tooltip>
                    <br/>
                    <div class="attribute-inputs">
                        <label>属性:</label>
                        <Select v-model="upperAttribute">
                            <Option v-for="attr in attributes" :key="attr" :label="attr" :value="attr"></Option>
                        </Select>
                        <label v-show="upperAttribute !== ''">数值:</label>
                        <InputNumber :min="0" :max="999" size="small" v-model="upperValue" v-show="upperAttribute !== ''"></InputNumber>
                    </div>
                </FormItem>
                <FormItem class="attribute-results">
                    <span v-for="upper in this.upperList" :key="upper" @click="removeUpper(upper)">{{upper}}</span>
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
                    <Button type="primary" @click="run">开始计算</Button>
                </FormItem>
            </Form>
        </section>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from 'vue-property-decorator';
import { Form, FormItem, Button, Select, Option, CheckboxGroup, Checkbox, Input, InputNumber, Message, Tooltip } from 'element-ui';
import axios from 'axios';

@Component({
    components: { Form, FormItem, Button, Select, Option, CheckboxGroup, Checkbox, Input, InputNumber, Tooltip },
})
export default class Calculator extends Vue {
    /**
     * 御魂套装
     */
    private yuhunPackage = ''
    /**
     * 御魂套装限制列表
     */
    private yuhunPackageList: string[] = []

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
    private secondAttributeList: string[] = []
    /**
     * 四号位属性
     */
    private fourthAttributeList: string[] = []
    /**
     * 六号位属性
     */
    private sixthAttributeList: string[] = []

    /**
     * 忽略指定关键字御魂
     */
    private ignoreSerial = ''

    /**
     * 伤害期望
     */
    private damageExpect = ''
    /**
     * 生命期望
     */
    private healthExpect = ''

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
     * 选择的文件名称
     */
    private filename = ''

    /**
     * 御魂套装选项
     */
    get yunhunOptions(): string[] {
        return ['攻击加成,2', '暴击,2', '生命加成,2', '效果命中,2', '效果抵抗,2','针女,4', '破势,4', '网切,4', '三味,4', '镇墓兽,4', '伤魂鸟,4', '蝠翼,4', '鸣屋,4', '心眼,4', '狰,4', '轮入道,4', '狂骨,4', '阴摩罗,4', '魍魉之匣,4', '骰子鬼,4', '返魂香,4', '幽谷响,4', '蚌精,4', '火灵,4', '树妖,4', '地藏像,4', '薙魂,4', '镜姬,4', '钟灵,4', '被服,4', '涅槃之火,4', '招财猫,4', '魅妖,4', '反枕,4', '木魅,4', '日女己时,4', '雪幽魂,4', '珍珠,4', '荒骷髅,2', '土蜘蛛,2', '地震鲶,2', '蜃气楼,2', '胧车,2'];
    }
    /**
     * 属性选项
     */
    get attributes(): string[] {
        return ['暴击', '暴击伤害', '效果命中', '效果抵抗', '速度', '攻击加成', '生命加成', '防御加成'];
    }

    get disableAddYuhunPackageButton(): boolean {
        let currentSelectCount = 0;
        if (this.yuhunPackage) {
            const [, count] = this.yuhunPackage.split(',');
            currentSelectCount = +count;
        }

        return currentSelectCount + this.selectedYuhunPackageCount > 6;
    }

    get selectedYuhunPackageCount(): number {
        return this.yuhunPackageList
            .map(yuhunPackage => {
                const [, count] = yuhunPackage.split(',');
                return +count;
            })
            .reduce((sum, value) => sum += value, 0);
    }

    /**
     * 添加御魂套装限制
     */
    addYuhunPackageLimit(): void {
        this.yuhunPackageList.push(this.yuhunPackage);
    }
    removeYuhunPackageLimit(yuhunPackage: string): void {
        const index = this.yuhunPackageList.findIndex(x => x === yuhunPackage);
        this.yuhunPackageList.splice(index, 1);
    }

    /**
     * 添加下限属性
     */
    addLower(): void {
        const newItem = `${this.lowerAttribute},${this.lowerValue || 0}`;
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
        const newItem = `${this.upperAttribute},${this.upperValue || 0}`;
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

    /**
     * 选择文件
     */
    selectFile() {
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

    /**
     * 开始计算
     */
    run() {
        if (!this.filename) {
            Message.warning('请先选择御魂数据文件');
            return;
        }

        const api = process.env.NODE_ENV === 'development' ? 'http://localhost:2019/calculate' : '/calculate';
        axios.post(api, {
            src_filename: this.filename,
            mitama_suit: this.yuhunPackageList.join('.'),
            prop_limit: this.lowerList.join('.'),
            upper_prop_limit: this.upperList.join('.'),
            sec_prop_value: this.secondAttributeList.join('.'),
            fth_prop_value: this.fourthAttributeList.join('.'),
            sth_prop_value: this.sixthAttributeList.join('.'),
            ignore_serial: this.ignoreSerial,
            all_suit: this.usePackage ? 'True' : 'False',
            damage_limit: this.damageExpect || '0,0,0',
            health_limit: this.healthExpect || '0,0,0',
            attack_only: this.useAttack ? 'True' : 'False',
            effective_secondary_prop: '',
            effective_secondary_prop_num: '',
        }, {
            validateStatus: function (status) {
                return status >= 200 && status <= 500;
            },
        }).then((result) => {
            if (result.data.reason) {
                Message.error('计算失败');
                console.error(result.data.reason);
            } else {
                Message.success(`计算完毕, 组合数量:${result.data.result_num}`);
            }
        }).catch((err) => {
            Message.error('计算失败');
        });
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
        }
        .filter-options-form {
            width: 290px;
            margin: 0 8px;

            .yuhun-package-input {
                height: 94px;

                .el-tooltip {
                    position: relative;
                    top: -8px;
                }
                .el-form-item__label {
                    line-height: 24px;
                }
                .el-select {
                    width: 180px;
                    top: -15px;
                }
                .el-button {
                    margin-left: 10px;
                    width: 90px;
                    height: 34px;
                    position: relative;
                    top: -13px;

                    span {
                        position: relative;
                        top: -3px;
                    }
                }
                .select-yuhun-list {
                    width: 100%;
                    height: 30px;
                    position: absolute;
                    top: 70px;
                    line-height: 30px;

                    & > span {
                        color: #409EFF;
                        margin-right: 12px;
                        cursor: pointer;
                    }
                }
            }
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
        color: #ccc;
        &::placeholder {
            color: #606266;
        }
    }
}
</style>
