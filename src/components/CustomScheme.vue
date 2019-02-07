<template>
    <div class="custom-scheme">
        <FormItem>
            <Button type="primary" @click="saveScheme">保存当前方案</Button>
        </FormItem>
        <FormItem>
            <Button type="primary" @click="selectCustomScheme">选择自定义方案</Button>
        </FormItem>
        <FormItem>
            <Button type="primary" @click="selectPresetScheme">选择预设方案</Button>
        </FormItem>
    </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator';
import { FormItem, Button, MessageBox, Select, Option, Dialog, Message } from 'element-ui';

import Scheme from '../definition/Scheme';
import presetSchemeList from '../data/presetScheme.json';

const StorageKey = 'customSchemeList';
const storageSchemeList = window.localStorage.getItem(StorageKey) || '[]';

@Component({
    components: { FormItem, Button, Select, Option, Dialog }
})
export default class CustomScheme extends Vue {
    /**
     * 当前编辑中的方案
     */
    @Prop({
        default: {}
    })
    currentScheme!: Scheme

    /**
     * 方案列表
     */
    private schemeList: Scheme[] = JSON.parse(storageSchemeList)

    /**
     * 选中的自定义方案名称
     */
    private selectedCustomSchemeName = ''

    /**
     * 选中的预设方案名称
     */
    private selectedPresetSchemeName = ''


    /**
     * 保存到本地存储
     */
    private saveStorage() {
        window.localStorage.setItem(StorageKey, JSON.stringify(this.schemeList));
    }

    /**
     * 保存自定义方案
     */
    private saveScheme() {
        MessageBox.prompt('', '请输入自定义方案名称').then(({value}) => {
            const scheme = Object.assign({ name: value }, this.currentScheme);
            this.schemeList.push(scheme);
            this.saveStorage();
            Message.success(`保存自定义方案「${scheme.name}」成功`);
        });
    }

    /**
     * 选择自定义方案
     */
    private selectCustomScheme() {
        const h = this.$createElement;
        MessageBox({
            title: '自定义方案列表',
            message: h(
                'Select',
                {
                    props:{
                        value: this.selectedCustomSchemeName,
                    },
                    on: {
                        change: (schemeName: string) => {
                            this.selectedCustomSchemeName = schemeName;
                            const $el = document.querySelector('.customSchemeSelect input');
                            if ($el) {
                                setTimeout(() => {
                                    (<any>$el).value = schemeName;
                                })
                            }
                        }
                    },
                    staticClass: 'customSchemeSelect'
                },
                [
                    this.schemeList.map(scheme => h(
                        'Option',
                        {
                            props: {
                                key: scheme.name,
                                label: scheme.name,
                                value: scheme.name,
                            }
                        }
                    ))
                ]
            ),
            showCancelButton: true,
            confirmButtonText: '选择',
            cancelButtonText: '删除',
            cancelButtonClass: 'delete',
            distinguishCancelAndClose: true,
        }).then(() => {
            const selectScheme = this.schemeList.find(scheme => scheme.name === this.selectedCustomSchemeName);
            if (selectScheme) {
                this.$emit('selectScheme', selectScheme);
                Message.success(`应用自定义方案「${selectScheme.name}」成功`);
            }
        }).catch((action) => {
            if (action === 'close'){
                return;
            }

            const schemeIndex = this.schemeList.findIndex(scheme => scheme.name === this.selectedCustomSchemeName);
            if (schemeIndex !== -1) {
                Message.success(`删除自定义方案「${this.schemeList[schemeIndex].name}」成功`);
                this.schemeList.splice(schemeIndex, 1);
                this.saveStorage();
                this.selectedCustomSchemeName = '';
            }
        })
    }

    /**
     * 选择预设方案
     */
    private selectPresetScheme() {
        const h = this.$createElement;
        MessageBox({
            title: '预设方案列表',
            message: h(
                'Select',
                {
                    props:{
                        value: this.selectedPresetSchemeName,
                    },
                    on: {
                        change: (schemeName: string) => {
                            this.selectedPresetSchemeName = schemeName;
                            const $el = document.querySelector('.preset-scheme-select input');
                            if ($el) {
                                setTimeout(() => {
                                    (<any>$el).value = schemeName;
                                })
                            }
                        }
                    },
                    staticClass: 'preset-scheme-select'
                },
                [
                    (presetSchemeList as Scheme[]).map(scheme => h(
                        'Option',
                        {
                            props: {
                                key: scheme.name,
                                label: scheme.name,
                                value: scheme.name,
                            }
                        }
                    ))
                ]
            ),
            confirmButtonText: '选择',
            distinguishCancelAndClose: true,
        }).then(() => {
            const selectScheme = (presetSchemeList as Scheme[]).find(scheme => scheme.name === this.selectedPresetSchemeName);
            if (selectScheme) {
                this.$emit('selectScheme', selectScheme);
                Message.success(`应用预设方案「${selectScheme.name}」成功`);
            }
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
.el-select {
    width: 250px;
}
.el-message-box__title {
    font-size: 16px;
}
.el-button {
    &.delete {
        color: #fff;
        background-color: #f56c6c;
        border-color: #f56c6c;
        &:hover {
            background: #f78989;
            border-color: #f78989;
            color: #fff;
        }
    }
}
</style>
