<template>
    <div class="calculator">
        <header class="header">
            <p>《阴阳师》御魂组合计算器</p>
        </header>
        <section class="form">
            <Form>
                <FormItem label="主御魂套装">
                    <Select placeholder="请选择主御魂">
                        <Option v-for="yuhun in primaryYuhunOptions" :key="yuhun" :label="yuhun" :value="yuhun"></Option>
                    </Select>
                </FormItem>
                <FormItem label="副御魂套装">
                    <Select placeholder="请选择副御魂">
                        <Option v-for="yuhun in secondYuhunOptions" :key="yuhun" :label="yuhun" :value="yuhun"></Option>
                    </Select>
                </FormItem>
                <FormItem label="输入文件">
                    <Button type="primary">选择文件</Button>
                </FormItem>
            </Form>
        </section>
    </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import { Form, FormItem, Button, Select, Option } from 'element-ui';

@Component({
    components: {
        Form,
        FormItem,
        Button,
    },
})
export default class Calculator extends Vue {
    data() {
        return {
            primaryYuhunOptions: [
                '针女',
                '破势',
            ],
            secondYuhunOptions: [
                '荒骷髅',
                '蜃气楼',
            ]
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
    .form {
        flex: 1;
        padding: 8px 16px;
    }
}
</style>
