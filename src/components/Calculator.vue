<template>
    <div class="calculator">
        <header class="header">
            <p>《阴阳师》御魂计算器</p>
        </header>
        <section class="main">
            <Form class="filter-options-form">
                <div class="yuhun-package-input">
                    <Popover placement="right" trigger="hover" ref='presetYuhunSet'>
                      <RadioGroup class="preset" size="mini" @change="onSelectPresetYuhun" v-model="presetYuhunPackageList">
                        <RadioButton v-for="yuhunPackage in this.presetYuhunPackages"
                        :label="yuhunPackage.value" :key="yuhunPackage.name"
                        >
                          {{ yuhunPackage.name }}
                        </RadioButton>
                      </RadioGroup>
                    </Popover>
                    <p class="title">
                      御魂套装组合
                      <Tag v-popover:presetYuhunSet title='常用'>+</Tag>
                    </p>
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
                    <Popover placement="right" trigger="hover" ref='presetAttribute'>
                      <RadioGroup class='preset' size="mini" @change="onSelectPresetAttribute" v-model="presetAttribute">
                        <RadioButton v-for="attributePreset in this.presetAttributes"
                        :label="attributePreset.value" :key="attributePreset.value"
                        >
                          {{ attributePreset.value }}
                        </RadioButton>
                      </RadioGroup>
                    </Popover>
                </FormItem>
                <FormItem class="multi-checkbox">
                    <p class="title">二号位属性<Tag v-popover:presetAttribute title='常用'>+</Tag></p>
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
                    <p class="title">忽略指定关键字御魂</p>
                    <Input placeholder="御魂列表(以,分隔)" v-model="ignoreSerial" />
                </FormItem>
            </Form>
            <Form class="expect-options-form">
                <FormItem class="input-item" label="伤害期望">
                    <Input placeholder="格式: 式神基础攻击,式神基础爆伤,期望伤害值" v-model="damageExpect" />
                    <label>攻击加成: </label>
                    <Select v-model="attackBuff">
                        <Option v-for="attBuff in attackBuffs" :key="attBuff.name" :label="attBuff.name" :value="attBuff.value"></Option>
                    </Select>
                </FormItem>
                <FormItem class="input-item" label="治疗期望">
                    <Input placeholder="格式: 式神基础生命,式神基础爆伤,期望治疗值" v-model="healthExpect" />
                </FormItem>
                <FormItem class="input-item" label="有效副属性">
                    <Autocomplete placeholder="属性列表(以,分隔)  例如: 暴击,暴击伤害" v-model="effectiveAttributes"
                      popper-class="esp-autocomplete" :fetch-suggestions="queryEffectiveAttributes">
                      <template slot-scope="{ item }">
                        <span>{{ item.name }}</span>
                        <span>&lt;{{ item.value }}&gt;</span>
                      </template>
                    </Autocomplete>
                    <Autocomplete placeholder="各位置加成次数(以,分隔)  例如: 3,3,3,3,3,0" v-model="effectiveAttributesBonusCount"
                      :fetch-suggestions="queryEffectiveAttributesBonusCount">
                    </Autocomplete>
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
import {
    Form,
    FormItem,
    Button,
    ButtonGroup,
    Select,
    Option,
    OptionGroup,
    CheckboxGroup,
    Checkbox,
    CheckboxButton,
    Radio,
    RadioButton,
    RadioGroup,
    Input,
    InputNumber,
    Message,
    Tag,
    Notification,
    Dialog,
    Autocomplete,
    Popover,
} from 'element-ui';
import axios from 'axios';

import Scheme from '../definition/Scheme';
import CustomScheme from './CustomScheme.vue';

const apiRoot = '//localhost:2019';
const axiosOption = {
    validateStatus(status: number) {
        return status >= 200 && status <= 500;
    },
};

interface Options {
  name: string;
  value: any;
}

@Component({
    components: {
        Form,
        FormItem,
        Button,
        ButtonGroup,
        Select,
        Option,
        OptionGroup,
        CheckboxGroup,
        Checkbox,
        CheckboxButton,
        Radio,
        RadioButton,
        RadioGroup,
        Input,
        InputNumber,
        Tag,
        Dialog,
        CustomScheme,
        Autocomplete,
        Popover,
    },
})
export default class Calculator extends Vue {

    /**
     * 御魂套装选项
     */
    get yuhunOptions(): object[] {
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
            list: ['珍珠,4', '魅妖,4', '雪幽魂,4', '招财猫,4', '反枕,4', '日女巳时,4', '木魅,4'],
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
     * 攻击加成
     */
    get attackBuffs(): Options[] {
        return [
          {name: '无buff', value: 0},
          {name: '1级兔子舞', value: 10},
          {name: '满级兔子舞', value: 20},
          {name: '满级兄弟之绊', value: 25},
          {name: '黑晴明+1级兔子舞', value: 30},
          {name: '黑晴明+满级兔子舞', value: 40},
          {name: '黑晴明+满级兄弟之绊', value: 45},
        ];
    }
    /**
     * 预设常用246主属性
     */
    get presetAttributes(): Options[] {
        return [
          {name: '攻攻暴', value: '攻攻暴'},
          {name: '速攻暴', value: '速攻暴'},
          {name: '生生暴', value: '生生暴'},
          {name: '速生暴', value: '速生暴'},
          {name: '速命生', value: '速命生'},
          {name: '速抗生', value: '速抗生'},
        ];
    }
    /**
     * 预设常用246主属性
     */
    get presetYuhunPackages(): Options[] {
        return [
          {name: '破荒', value: ['破势,4', '荒骷髅,2']},
          {name: '心荒', value: ['心眼,4', '荒骷髅,2']},
          {name: '狂荒', value: ['狂骨,4', '荒骷髅,2']},
          {name: '针荒', value: ['针女,4', '荒骷髅,2']},
          {name: '散件生命', value: ['生命加成,2', '生命加成,2', '生命加成,2']},
        ];
    }
    /**
     * 有效属性
     */
    get effectiveAttributesChoices(): Options[] {
      return [
        {name: '输出', value: '暴击,暴击伤害,攻击加成,速度'},
        {name: '奶盾', value: '暴击,暴击伤害,生命加成,速度'},
        {name: '命中', value: '效果命中,速度'},
        {name: '抵抗', value: '效果抵抗,速度'},
        {name: '双堆', value: '速度,效果命中,效果抵抗'},
      ];
    }
    /**
     * 有效属性
     */
    get effectiveAttributesBonusCountChoices(): Options[] {
      return [
        {name: '1', value: '1,1,1,1,1,0'},
        {name: '2', value: '3,3,3,3,3,1'},
        {name: '3', value: '3,2,3,2,3,0'},
        {name: '4', value: '5,3,5,3,5,1'},
      ];
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
    get currentScheme(): object {
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
        };
    }
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
     * 预设御魂套装限制列表
     */
    private presetYuhunPackageList: string[] = [];

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
     * 选中的预设常用246主属性
     */
    private presetAttribute: string = '';
    /**
     * 忽略指定关键字御魂
     */
    private ignoreSerial = '';

    /**
     * 伤害期望
     */
    private damageExpect = '';
    /**
     * 攻击加成
     */
    private attackBuff = '';
    /**
     * 生命期望
     */
    private healthExpect = '';
    /**
     * 有限副属性
     */
    private effectiveAttributes = '';
    /**
     * 有效副属性各位置加成次数
     */
    private effectiveAttributesBonusCount = '';

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

    /**
     * 校验输入框格式
     */
    public verifyInputValue(): boolean {
        if (this.damageExpect && !/^[0-9]+,[0-9]+,[0-9]+(?:,[0-9]+)?$/.test(this.damageExpect)) {
            Message.warning('"伤害期望" 格式错误');
            return false;
        }
        if (this.healthExpect && !/^[0-9]+,[0-9]+,[0-9]+(?:,[0-9]+)?$/.test(this.healthExpect)) {
            Message.warning('"治疗期望" 格式错误');
            return false;
        }
        if (
            this.effectiveAttributes
            && !/^(攻击加成|生命加成|防御加成|速度|效果命中|效果抵抗|暴击|暴击伤害)(,(攻击加成|生命加成|防御加成|速度|效果命中|效果抵抗|暴击|暴击伤害))*$/
                .test(this.effectiveAttributes)
        ) {
            Message.warning('"有效副属性 => 属性列表" 格式错误');
            return false;
        }
        if (
            this.effectiveAttributesBonusCount
            && !/^[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+$/.test(this.effectiveAttributesBonusCount)
        ) {
            Message.warning('"有效副属性 => 各位置加成次数" 格式错误');
            return false;
        }

        return true;
    }

    /**
     * 开始计算
     */
    public run() {
        if (!this.filename) {
            Message.warning('请先选择御魂数据文件');
            return;
        }

        if (!this.verifyInputValue()) {
            return;
        }

        axios.post(apiRoot + '/calculate', {
            src_filename: this.filename,
            mitama_suit: this.yuhunPackageList.join('.'),
            prop_limit: this.targetAttributeList.map((attr) => {
                const [attrName, value] = attr.split(/ |-/);
                return attrName + ',' + value;
            }).join('.'),
            upper_prop_limit: this.targetAttributeList.map((attr) => {
                const [attrName, , , , value] = attr.split(/ |-/);
                return attrName + ',' + value;
            }).join('.'),
            sec_prop_value: this.getPropValue(this.secondAttributeList),
            fth_prop_value: this.getPropValue(this.fourthAttributeList),
            sth_prop_value: this.getPropValue(this.sixthAttributeList),
            ignore_serial: this.ignoreSerial,
            all_suit: this.usePackage,
            damage_limit: this.damageExpect || '0,0,0',
            health_limit: this.healthExpect || '0,0,0',
            attack_only: this.useAttack,
            effective_secondary_prop: this.effectiveAttributes || '',
            effective_secondary_prop_num: this.effectiveAttributesBonusCount || '',
            attack_buff: this.attackBuff || 0,
        }, axiosOption).then((result) => {
            switch (result.status) {
                case 500: {
                    Message({
                        type: 'error',
                        message: `计算失败, 服务端错误: ${result.data.reason}`,
                        duration: 5000,
                    });
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

        setTimeout(this.getCalculateStatus.bind(this), 500);
    }

    private mounted() {
        axios.get(apiRoot + '/status').catch(() => {
            Notification.error({
                title: '提示',
                message: '未检测到服务端, 请先启动服务端后再试',
                duration: 0,
            });
            this.serverOnline = false;
        });
    }

    /**
     * 处理选择御魂方案事件
     */
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
        this.attackBuff = '';
    }

    /**
     * 获取计算进度
     */
    private getCalculateStatus() {
        axios
            .get(apiRoot + '/status')
            .then((result) => {
                const {progress} = result.data;
                if (progress === undefined) {
                    return;
                }
                this.calculateProgress = Math.floor(progress * 100);
                if (progress !== 1) {
                    setTimeout(this.getCalculateStatus.bind(this), 100);
                }
            })
            .catch(() => {
                console.warn('获取计算进度失败');
            });
    }

    /**
     * 获取后端所需的 prop_value 格式文本
     */
    private getPropValue(attributeList: string[]): string {
        return attributeList.map((attr) => {
            switch (attr) {
                case '速度': return '速度,57';
                case '暴击伤害': return '暴击伤害,89';
                default: return attr + ',55';
            }
        }).join('.');
    }

    /**
     * 根据用户输入过滤有效属性
     */
    private queryEffectiveAttributes(queryString: string, cb: (arg0: any[]) => void) {
      let results = this.effectiveAttributesChoices;
      if (queryString) {
        results = results.filter((choice: Options) =>
          (choice.name.toLowerCase().indexOf(queryString.toLowerCase()) === 0 ||
          choice.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0)
        , );
      }
      return cb(results);
    }
    /**
     * 根据用户输入过滤有效属性条数
     */
    private queryEffectiveAttributesBonusCount(queryString: string, cb: (arg0: any[]) => void) {
      let results = this.effectiveAttributesBonusCountChoices;
      if (queryString) {
        results = results.filter((choice: Options) => choice.value.indexOf(queryString.toLowerCase()) !== -1);
      }
      return cb(results);
    }
    /**
     * 预设属性选择快捷按钮
     */
    private onSelectPresetAttribute(label: string): void {
      const l2p: {[key: string]: string[]} = {生: ['生命加成'], 攻: ['攻击加成'], 暴: ['暴击', '暴击伤害'],
                 命: ['效果命中'], 抗: ['效果抵抗'], 速: ['速度']};
      if (label.length === 3) {
        const props = label.split('').map((x) => l2p[x]);
        this.secondAttributeList = props[0];
        this.fourthAttributeList = props[1];
        this.sixthAttributeList = props[2];
      }
      return;
    }
    /**
     * 预设御魂选择快捷按钮
     */
    private onSelectPresetYuhun(label: string): void {
      this.yuhunPackageList = this.presetYuhunPackageList.slice();
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
            .el-autocomplete {
              width: 100%;
            }
            .esp-autocomplete {
              span {
                text-overflow: ellipsis;
                overflow: hidden;
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

        .preset {
          .el-radio-button {
              &.is-active {
                  .el-radio-button__inner {
                      color: #fff;
                      background-color: #409EFF;
                      border-color: #409EFF;
                      box-shadow: -1px 0 0 0 #8cc5ff;
                  }
              }
          }
          .el-radio-button__inner {
              min-width: 50px;
              padding: 7px 5px;
              color: #fff;
              background-color: #409EFF;
              border: 1px solid #333b47;
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
          .el-radio-group {
              display: flex;
              align-items: center;
              justify-content: flex-start;
              flex-wrap: wrap;
              transform: translateY(-5px);
          }
          .el-radio {
              margin-left: 10px;
              height: 30px;
          }

        }

        &.multi-checkbox, {
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
        font-size: 14px;

        .el-tag {
          background-color: inherit;
          border: 0px;
          transform: scale(1.5);
          transition: transform .2s ease-in-out;
        }
        .el-tag:hover {
          transform: scale(2) rotate(45deg);
        }
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
