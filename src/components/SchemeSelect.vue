<template>
    <Select v-model="selectedPresetSchemeName">
        <Option
            v-for="scheme in schemeList"
            :key="scheme.name"
            :label="scheme.name"
            :value="scheme.name"
        ></Option>
    </Select>
</template>

<script lang="ts">
import { Vue, Component, Prop, Watch } from 'vue-property-decorator';
import { Select, Option } from 'element-ui';

import Scheme from '@/definition/Scheme';

@Component({
    components: { Select, Option },
})
export default class SchemeSelect extends Vue {

    @Prop({
        default: [],
    })
    public schemeList!: Scheme[];
    private selectedPresetSchemeName = '';

    @Watch('selectedPresetSchemeName')
    public onSelectSchemeChange() {
        this.$emit('SelectSchemeChange', this.selectedPresetSchemeName);
    }
}
</script>

