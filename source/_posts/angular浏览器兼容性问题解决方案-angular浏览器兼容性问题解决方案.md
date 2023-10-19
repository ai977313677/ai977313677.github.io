---
title: angular浏览器兼容性问题解决方案
date: 2020-07-13 16:49:28.267
updated: 2020-12-27 16:28:28.234
url: https://iachieveall.com/archives/angular浏览器兼容性问题解决方案
categories: 
- 学习笔记
tags: 
- 浏览器兼容
---

***问题***：edge浏览器下，固定列的边框消失

***原因***：ng-zorro-antd表格组件使用nzLeft和nzRight指令固定的表格列，这两个指令的实现css3中的标签：

```css
position: -webkit-sticky !important;
position: sticky !important;
```

谷歌、火狐及-webkit-内核的浏览器均支持该属性（css3），IE不支持该属性，所以在IE中，会自动降级，表格无固定列，可滑动的形式。
Edge浏览器在1703之后的版本使用了chromium内核，对css3的属性支持较好，也支持sticky属性，可以使用，可以固定表格列，但边框会消失。 

***解决方案***：
目前可行的解决方案有如下几种：

1. 不使用固定列，若产品没有明确要求使用固定列，可以放弃使用nzLeft及nzRight来固定表格。从而使各个浏览器下的展示效果一致。

   针对Edge浏览器降级处理，与IE浏览器效果一致，无固定列，整体可横向滚动。

2. 自定义实现固定列，不使用组件的固定列实现，通过使用```position: absolute;```这种方式来实现表格的固定列。

第二个方案的详细过程如下：

使用div包裹表格，当表格宽度超过div宽度时，开启滚动：

   

```css
.scroll-table {
  width: 100%;
  overflow-x: scroll;
}
```

针对表格，我们可以指定宽度，让其超过外层div宽度（这样可以看到滚动效果）。

```css
.fixed-table {
  width: 1300px; /* 可由th，td动态扩充，也可指定宽度 */
  border-collapse: collapse;
}
```

最后一个最核心的问题，就是固定列的实现了，非常简单，将表格的一列设置成绝对定位,在设置了绝对定位后，该列会脱离原来的文档流，表格少了一列，所以需要加一个背景板来保证表格能够给这个固定列留出一个位置。

HTML代码大致如下，这个fixed-col可以为固定列的样式，也可以设置成背景板的样式，demo中是用其指定了固定列的样式。

```HTML
<div class="scroll-table">
    <table class="fixed-table">
        <thead>
            <tr>
                <th>无效背景板</th>
                <th class="fixed-col">固定列</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>无效背景板</td>
                <td class="fixed-col">固定列</td>
            </tr>
        </tbody>
    </table>
</div>
```

参考代码：[Ironape](https://www.cnblogs.com/guyg/p/6896847.html)

---

***问题***：Edge浏览器的日历（nz-range-picker）确认按钮需要点两次

***原因***：尚未明确

***解决方案***：

1. **升级组件版本**，目前ng-zorro-antd 8.5之上的版本未见这个问题。
2. **自定义页脚**，加入额外的页脚，来替代确定功能，此时有两种方式来实现：
   只覆盖对应的按钮，如确定按钮，此时按钮的样式与默认的页脚按钮是不一致的，为保持一致，可以自定义样式，也可以直接使用默认页脚中按钮的样式，下例中选择直接使用组件库的样式：ant-calendar-ok-btn，第二步则是覆盖原来的按钮，可以使用绝对定位的方式来实现覆盖：

```html
<nz-range-picker [nzRenderExtraFooter]="renderExtraFooterTpl">
<ng-template #renderExtraFooterTpl>
  <div>
    <button nz-button nzType="primary" class="ant-calendar-ok-btn abs-right">确 定</button>
  </div>
</ng-template>
```

对应css：

```css
.abs-right {
  position: absolute;
  right: 12px;
  top: 7px;
  z-index: 1;
  box-shadow: none;
}
```

删除默认页脚，完全自定义实现页脚。此时需要删除原来的页脚，可通过：

```css
::ng-deep .ant-calendar-footer-btn {
  display: none;
}
```

> 这种方式删除默认页脚，此时额外的页脚不可使用绝对定位。

---

***问题***：IE浏览器下，在多个tab页中切换，echart所在容器高度坍塌

***原因***：IE浏览器下父元素不能动态调整高度（即通过子元素动态改变调整高度）

***解决方案***：固定echart图表所在的容器高度

---

***问题***：IE浏览器下，初始化表单时，触发表单验证

***原因***：这个是IE的问题，IE10+实现了input事件，但是触发的时机却是错误的。比如在placeholder改变时，placeholder的文字不是英语的时候就会触发，Edge15+修复了这个问题，但是IE可能永远都不会修复这个问题。

***解决方案***：

1. 使用表单的reset()重置表单，但是重置的操作需要放在setTimeout中，或者通过其他手段将重置的操作作为表单初始化时的最后一个宏任务执行。这种方式经验证，最终的效果是，初始化表单后，表单输入元素的边框闪烁（红色）一下。
2. 使用自定义的服务商插件（较为推荐），这种方式对原有代码的破坏性小（遵循了OCP原则），该插件是由DerSizeS提供的。只需要在对应的module中增加一个服务商即可

```Javascript
@NgModule({
    providers: [{
	    provide: EVENT_MANAGER_PLUGINS, multi: true,
	    useClass: UniqueInputEventPlugin, deps: [UNIQUE_INPUT_EVENT_PLUGIN_CONFIG],
	}]	
})
class MyModule {}
```

>  需要注意的是，插件需要自己添加到项目文件中（根据angular团队所说，这个插件修复了一个IE10或者IE11的bug，但是提交了太多的代码，这会给增加现有的应用的打包体积，虽然后面关于这个PR讨论了挺久，但是看样子是准备把这个放到FAQ里面，而不会把他并入框架），并在对应的模块中引用。

3. IE的输入框会因为placeholder为中文而触发表单验证，placeholder改变了也会触发表单验证，所以，有一个讨巧的方法，***placeholder里面的内容写成英文形式(推荐）***，但这显然不符合中文产品的需求，而且这显然没有国际化。所以可以想办法绕过这一条，使用 ***HTML实体***（已验证，可行），Unicode编码（不可以）
