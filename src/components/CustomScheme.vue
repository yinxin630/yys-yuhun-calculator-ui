<template>
    <div class="custom-scheme">
        <FormItem>
            <Button type="primary" @click="saveScheme">保存当前方案</Button>
        </FormItem>
        <FormItem>
            <Button type="primary">选择自定义方案</Button>
        </FormItem>
        <FormItem>
            <Button type="primary" disabled>选择预设方案</Button>
        </FormItem>
    </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator';
import { FormItem, Button, MessageBox } from 'element-ui';

const StorageKey = 'customSchemeList';
const storageSchemeList = window.localStorage.getItem(StorageKey) || '[]';

@Component({
    components: { FormItem, Button }
})
export default class CustomScheme extends Vue {
    /**
     * 当前编辑中的方案
     */
    @Prop({
        default: {}
    })
    currentScheme: Object

    private schemeList: Object[] = JSON.parse(storageSchemeList)

    /**
     * 保存自定义方案
     */
    private saveScheme() {
        MessageBox.prompt('', '请输入自定义方案名称').then(({value}) => {
            const scheme = Object.assign({ name: value }, this.currentScheme);
            this.schemeList.push(scheme);
            window.localStorage.setItem(StorageKey, JSON.stringify(this.schemeList));
        });
    }
}
</script>


<style lang="less">
.custom-scheme {
    margin-top: 50px;
    .el-button {
        margin-bottom: 5px;
    }
}
</style>
