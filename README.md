# org-tree
组织架构图
* 支持可缩放
* 支持拖动
* 导出图片





## API

  prop              | descripton                              | type                   | default
  ------------------|-----------------------------------------|:----------------------:|:---------------------------------------------------------:
  data              |         数据源                               | `Object`               |
  props             |  configure props                        | `Object`               | `{label: 'label', children: 'children', expand: 'expand'}`
  label-width        |  node label width                       | `String` \| `Number`   | `auto`
  collapsable       |  children node is collapsable           | `Boolean`              | `true`
  render-content     |  how to render node label               | `Function`             |     -
  label-className    |  node label class                       | `Function` \| `String` |     -
  selected-key       |  The key of the selected node           | `String`               |     -
  selected-class-name |  The className of the selected node     | `Function` \| `String` |     -
  canMove              |  是否支持可以拖动     | `Boolean`|     -
    zoom | 是否支持可以缩放    | `Boolean`|     -
    export | 是否支持可以导出图片    | `Boolean`|     -









