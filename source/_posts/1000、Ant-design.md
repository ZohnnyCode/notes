---
title: 1000、Ant-design
date: 2021-04-16 16:18:50
tags:
---

#### 自定义 Modal 底部

```js
<Modal
    title="yourtitle"
    visible={MailOutModal}     //控制对话框的弹出与消失
    footer={[
        <Button
        type="primary"
        key="ok"
        onClick={() => {
            // ...//自己写点击事件
        }}
        >
        确认
        </Button>,
        <Button
        type="primary"
        key="cancel"
        onClick={() => {
            setMailOutModal(!MailOutModal);
        }}
        >
        取消
        </Button>,
    ]}
>
```

#### Textarea 不可编辑

```js
<TextArea rows={4} disabled />
```

#### 深刻理解表单默认值

```js
initialValues	表单默认值，只有初始化以及重置时生效

要动态更改表单项，必须：
form.setFieldsValue({
    sights: defaultShow,
});

动态增减表单项，可以通过默认值来实现
自定义添加：
onClick={() => {
    const sights = form.getFieldValue('sights');
    const nextList = sights.concat([{ user: 'all' }]); // 括号内为默认值
    form.setFieldsValue({
        sights: nextList,
    });
}}
```

#### 清楚历史记录值

```js
//清除输入记录代码:
<Form.Item label="商品名称" name="name" rules={[{ required: true }]}>
  <Input autoComplete="off" />
</Form.Item>
```

#### form 的操作

```js
const [form] = Form.useForm();

form.getFieldValue("sights"); // 获取某一项的（Form.Item）的值

form.setFieldsValue({}); // 动态设置表单项的值

form.resetFields(); // 重置表单
```

#### 解决 antd select 组件、日历组件下拉框随页面滚动分离的问题

```js
<Select getPopupContainer={(triggerNode) => triggerNode.parentNode}>
  // some code
</Select>

<DatePicker getCalendarContainer={triggerNode => triggerNode.parentNode} />
```

#### Form.List 用法详解

```js
Form.List中可以有多个字段，name要定义为:
name={[field.name, "lastName"]}   //第二项是组内设置的name值
fieldKey={[field.fieldKey, "lastName"]}

如何给Form.List添加默认值：（使用initialValues）
<Form form={form} onFinish={onFinish} onFinishFailed={onFinishFailed}
    initialValues={{
        list: itemList
}}>

Form.List添加验证：（使用validator，自定义验证）
<Form.Item className='num'
    name={[field.name, 'maxRange']}
    fieldKey={[field.fieldKey, 'maxRange']}
    rules={[{validator: handleValitorMax }]}
>
    <Input addonAfter='人' type='number' />
</Form.Item>
const handleValitorMax = (rule, value, callback)=>{
    let arr = rule.field.split('.');
    let index = Number(arr[1]) || 0;
    let discountItem = form.getFieldValue('list');
    let prevNum = Number(discountItem[index].minRange) || 2;
    value = Number(value);
    // console.log('max:', arr, prevNum, discountItem);
    if (!value) {
        return  Promise.reject('人数必填');
    }else if(value <= prevNum){
        return Promise.reject('人数要大于最小人数');
    }
    return  Promise.resolve();
}

Form.List自定义添加一项（不使用默认的add方法）
const handleAddItem = ()=>{
    const list = form.getFieldValue('list');
    const nextList = list.concat(discountObj);
    form.setFieldsValue({
        list: nextList,
    });
};
```
