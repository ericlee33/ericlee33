---
title: Formily服务端控制自定义校验x-rules规则实践
date: 2021-03-29 23:15:12
tags:
- Formily
---
## 提供给服务端自定义校验函数
由于后端要求动态提供校验规则，并且每一个表单字段都需要很多种校验方式，这里进行探索。


```javascript
// /public/index.html
<script src="https://服务器地址/xRulesMethods.js"></script>
```


这里提供给后端模板，文件格式如下：


```javascript
(function (window) {
    window.rulesMethods = {
        customRule2: (value) => {
            return value === '123' ? '不能等于123' : value === '234' ? '不能等于234' : ''
        },
        customRule3: (value) => {
            return value === '14' ? '不能等于14' : ''
        },
        customRule4: (value) => {
            return value === '1' ? '不能等于1' : ''
        },
    };
})(window);
```


我们在index.js引入该自定义校验规则即可


```javascript
import {
    registerValidationRules,
} from '@formily/antd'; 
registerValidationRules(window.rulesMethods)
```


这样服务端就可以进行完整的自定义校验操纵了


JSON schema使用自定义函数方式如下：


```json
{
    "name": {
        "key": "name",
        "type": "string",
        "title": "姓名",
        "name": "name",
        "required": true,
        "x-props": {
            "itemClassName": "form-item"
        },
        "x-component-props": {
            "bordered": false
        },
        "x-rules": {
            "customRule2": true
        },
        "x-component": "input"
    },
}
```


## 如何进行联动校验？


[https://github.com/alibaba/formily/issues/478](https://github.com/alibaba/formily/issues/478)
看到官方之前提供的RFC中，其中有一个讨论举的例子很不错，但是看起来最终并没有实现。


```json
{
    "name": {
        "type": "string",
        "title": "姓名",
        "default": "xxx",
        "x-component": "Input",
        "x-component-props": {
            "value": "{{root.value.fieldA === 'xxx' ? 0 : 1}}",
            "disabled": "{{root.value.fieldA > root.value.fieldB}}", //支持嵌套字段值获取，支持JS原生方法、逻辑表达式
        },
        "x-rules": [{
            "required": true
        }]
    }
}
```


鉴于上面👆这种方式并没有最终被采纳，我们只能通过前端控制effect的方式进行，下面是官网提供的例子：


```javascript
const useLinkageValidateEffects = () => {
    const { setFieldState, getFieldState } = createFormActions();
    onFieldValueChange$('*(name,family_name)').subscribe(fieldState => {
        const selfName = fieldState.name;
        const selfValue = fieldState.value;
        const otherName = selfName == 'name' ? 'family_name' : 'name';
        const otherValue = getFieldState(otherName, state => state.value);
        setFieldState(otherName, state => {
            if (selfValue && otherValue && selfValue !== otherValue) {
                state.errors = '两次密码输入不一致';
            } else {
                state.errors = '';
            }
        });
        setFieldState(selfName, state => {
            if (selfValue && otherValue && selfValue !== otherValue) {
                state.errors = '两次密码输入不一致';
            } else {
                state.errors = '';
            }
        });
    });
};
```


通过这种方式我们就可以进行不同表单字段的联动校验了
