# 渲染机制与优化

> 如果你只是为了学习如何使用 Vue，这个页面不是必须阅读的，但如果您对渲染的底层工作方式感到好奇的话，它提供了更多信息为您解答。

## Virtual DOM

我们已经知道侦听器是如何更新组件的了，但你可能会想问这些变更是如何应用到 DOM 上的。或许你曾听说过 Vitrual DOM，包括 Vue 在内的许多框架都是采用了这一范式来确保用户界面响应式地展示出 JavaScript 中的变更。

<div class="reactivecontent">
  <iframe height="557" style="width: 100%;" scrolling="no" title="虚拟DOM是如何工作的？" src="https://codepen.io/shenqingchuan/embed/oNxowzw?height=557&theme-id=light&default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/shenqingchuan/pen/oNxowzw'>虚拟DOM是如何工作的？</a> by shenqingchuan
  (<a href='https://codepen.io/shenqingchuan'>@shenqingchuan</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
</div>

我们将其用 JavaScript 做一份拷贝，即生成一个 Virtual DOM，这样做是因为在 JavaScript 中直接操作 DOM 代价很高，然而只是 JavaScript 中的更新操作却代价很低。用 JS 找到所需的 DOM 节点并更新，代价很高。所以我们集合所有的更改，只调用一次。

虚拟 DOM 是一个轻量级的 JavaScript 对象，由渲染函数创建。它需要三个参数：目标元素，一个带有数据、props，attrs 和其他更多内容的对象，最后是一个数组。我们在该数组中传入子节点，它们也同样需要这三个参数，也可以有自己的子节点，子子孙孙层层递进，直到建成完整元素的树形结构。

如果我们需要更新列表内元素，我们会在 Javascript 中这样做：使用我们之前讲过的响应式机制，我们可以将所有更改都应用到这份 JavaScript 拷贝的虚拟 DOM 上，执行一次与真实 DOM 的差异对比（diff），因而我们只会操作那部分实际变更的元素，虚拟 DOM 让我们能在用户界面上做出高性能的更新！
