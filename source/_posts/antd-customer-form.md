---
title: 自定义 Form 控件
date: 2021-05-03 12:50:28
tags: ["react", "antd"]
categories: 编程
---

其实很多时候官方文档给我们写好了许多东西，只是我们没有细心去观察，去使用。

<!-- more -->

自定义的控件需要遵循以下规定

- 提供受控属性 value 或其它与 valuePropName 的值同名的属性。
- 提供 onChange 事件或 trigger 的值同名的事件。

被设置了 name 属性的 Form.Item 包装的控件，表单控件会自动添加 value（或 valuePropName 指定的其他属性） onChange（或 trigger 指定的其他属性），数据同步将被 Form 接管，这会导致以下结果：

- 你不再需要也不应该用 onChange 来做数据收集同步（你可以使用 Form 的 onValuesChange），但还是可以继续监听 onChange 事件。
- 你不能用控件的 value 或 defaultValue 等属性来设置表单域的值，默认值可以用 Form 里的 initialValues 来设置。注意 initialValues 不能被 setState 动态更新，你需要用 setFieldsValue 来更新。
- 你不应该用 setState，可以使用 form.setFieldsValue 来动态改变表单值。

通过这两条其实就不难发现，我们写在 Form.Item 中的内容可以从 Form.Item 获取到 value 以及 onChange，而我们在写只读表单的时候，可以根据情况从 Form.Item 往自定义组件中传其他值。

### 基础功能

```tsx
// Form.tsx
import { Form as AntForm, FormProps } from "antd";
import FormItem from "./FormItem";
const Form = (props: FormProps & { readOnly?: boolean }) => {
  return <AntForm {...props}>{props.children}</AntForm>;
};
Form.Item = FormItem;
export default Form;
```

```tsx
// FormItem.tsx
import { ReactNode } from "react";
import { Form, FormItemProps } from "antd";
interface Props {
  readOnly?: boolean;
  readOnlyRender?: (v: string) => ReactNode;
  children: any;
}
const RenderText = (props: any) => {
  const { value, readOnlyRender, childrenProps, type } = props;
  if (readOnlyRender) return readOnlyRender(value);
  return <>{value}</>;
};
const FormItem = (props: Props & FormItemProps) => {
  const { readOnly, readOnlyRender, children, ...rest } = props;
  if (readOnly) {
    const type = children?.type ?? {};
    const childrenProps = children?.props ?? {};
    return (
      <Form.Item {...rest}>
        <RenderText
          type={type}
          childrenProps={childrenProps}
          readOnlyRender={readOnlyRender}
        />
      </Form.Item>
    );
  }
  return <Form.Item {...rest}>{children}</Form.Item>;
};
export default FormItem;
```

这两个文件也基本上可以把 readOnly 自定义组件给实现了，但是还有可以完善的地方，比如 Form 上面添加的 readOnly 属性可以使用 context 去写；RenderText 组件可以默认判断一些类型时间等；对于 Select 看下能否实现晚取值行为。

### 添加 context

```ts
// context
import React from "react";
const readOnly = false;
const ReadOnlyContext = React.createContext(readOnly);
export default ReadOnlyContext;
```

```tsx
// Form
// ...
<ReadOnlyContext.Provider value={readOnly}>
  {props.children}
</ReadOnlyContext.Provider>
// ...
```

```tsx
// FormItem
// ...
if (formReadOnly || readOnly) {
  // ...
}
// ...
```

### 对时间进行默认展示

```tsx
// FormItem
const RenderText = (props: any) => {
  const { value, readOnlyRender, childrenProps, type } = props;
  if (readOnlyRender) return readOnlyRender(value)
  if (type === DatePicker) {
    return <>{value.format('YYYY-MM-DD')}</>
  }
  if (type === RangePicker) {
    return <>
      {value[0] ? value[0].format('YYYY-MM-DD') : '--'}
      {' ~ '}
      {value[1] ? value[1].format('YYYY-MM-DD') : '--'}
    </>
  }
  return <>{value}</>;
};
```
