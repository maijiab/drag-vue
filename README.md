## 一个**用vue实现的拖拽排列卡片组件**


**这是vue实现的拖动卡片组件，主要实现了**：
- 拖动卡片与其他卡片的位置更换，并且其他卡片根据拖动的位置自动顺移，位置数据实时更新
- 拖动的时候可使用鼠标滚动
- 卡片根据数据生成，所有参数和内容都是可以自定义的，方便应用于不同场景
- 不同操作的事件都可获取到，拖动后的位置数据会实时更新
- 可以全局安装和按需加载 