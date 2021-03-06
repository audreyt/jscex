---
layout: manual-zh-cn
title: 评论管理（异步编程示例）
---

## <a name="description"></a>描述

如今前端应用中的信息管理场景并不罕见，例如评论管理便是常见的一例：用户可以在页面上通过页面中的一系列行为来添加，删除或是修改评论，同时页面中使用AJAX异步操作与服务器端进行交互，避免页面刷新，以此带来优秀的用户体验。

由于JavaScript和HTML的限制，各种用户交互的参与都必须以回调函数（或是事件，而事件其实也可以认为是回调函数的一种）。但实际上，这种做法在很多情况下不能说是最直接且最易用的表达方式。例如，一个统一的事务原本可以连贯的表现，却被异步操作将原本连贯一致的逻辑拆分成小段方法来处理。

此外，由于异步操作都会消耗一定时间，开发人员还必须防备一些意外的情况。例如，一个AJAX请求发起之后尚未返回，此时用户继续在页面上进行操作，如果缺少有效的防护措施，页面便会进入不一致的状态，甚至必须重新加载页面才能恢复过来。自然，错误处理也是一个不可忽视的环节。

本文将通过实现一个评论管理功能，演示如何使用[Jscex异步模块](../)来轻松直接地表达一些关系紧密的事务操作，并解决以上所提到的各类问题。

## <a name="requirement"></a>需求

评论管理是如今前端应用中典型的案例，用户可以添加评论，删除或是更新现有的评论。其中整个过程都是AJAX及DOM操作，页面不会刷新，用户体验良好。

打开页面后，页面上显示若干已有评论。页面下方有一个文本框，可以由用户输入评论内容。还有一个“Add”按钮，点击后将会发起一个AJAX操作向服务器端添加数据。此时可能出现两种情况：

* **成功：**将评论内容显示在上方列表中，并清空文本框。
* **失败：**将错误信息提示给用户，其余不变，并等待用户再次提交。

每条评论均包含一个“Edit”按钮，点击之后将发起一个AJAX请求获取服务器端的评论内容，此时也可能发生两种情况：

* **成功：**将评论内容显示在下方文本框内，隐藏“Add”按钮，显示“Update”和“Cancel”两个按钮（即进入“编辑模式”）。
* **失败：**将错误信息提示给用户，其余不变。

用户可以点击“Cancel”按钮取消这次编辑，此时需要清空文本框，隐藏“Update”和“Cancel”按钮，并重新显示“Add”按钮。当用户编辑评论内容之后，也可以点击“Update”按钮发送AJAX请求至服务器端提交更改，此时还是可能发生两种情况：

* **成功：**将评论更新至页面上对应的评论项，清空文本框，隐藏“Update”和“Cancel”按钮，并重新显示“Add”按钮。
* **失败：**将错误信息提示给用户，其余不变，并等待用户再次提交（或是取消）。

最后，每条评论也都包含一个“Remove”按钮，点击后将要求用户确认删除，确认后发送AJAX请求至服务器端删除数据，同样可能发生两种情况：

* **成功：**将页面上的评论项删除。
* **失败：**将错误信息提示给用户，其余不变。

还有一些额外的要求：

1. 发起AJAX请求时，需要显示“Loading”提示信息，AJAX操作完毕后（无论成功失败）隐藏。
2. 在AJAX请求过程中，用户其他任何操作都无效，直至AJAX操作完毕，例如：
    * 用户反复点击“Update”按钮。
    * 用户点击“Cancel”按钮。
    * 用户点击某条评论的“Edit”或“Remove”按钮。
3. 只要不是在AJAX请求过程中，用户可以随时点击某条评论项的“Edit”按钮进入“编辑模式”，即使已经在编辑某条评论了。

## <a name="page"></a>相关页面元素及操作

页面上已经准备了一系列DOM元素及内置的方法，您可以直接用其进行开发。

### <a name="page-ajax"></a>AJAX模拟操作

页面上定义了一系列模拟AJAX的异步操作，它们均有一定几率失败。

    // 向服务器端提交评论，text为评论内容
    addServerComment(text, function (error, id, text) {
        if (error) {
            // 失败，error为错误信息
        } else {
            // 成功，id为服务器端生成的评论标识，text为内容
        }
    });

    // 删除服务器端评论，id为评论标识
    removeServerComment(id, function (error) {
        if (error) {
            // 失败，error为错误信息
        } else {
            // 成功
        }
    });
    
    var updateServerComment(id, function (error) {
        if (error) {
            // 失败，error为错误信息
        } else {
            // 成功
        }
    });

    var loadServerComment(id, function (error, text) {
        if (error) {
            // 失败，error为错误信息
        } else {
            // 成功，text为评论内容
        }
    });

### <a name="page-dom"></a>DOM操作

页面上同样定义了一系列操作DOM的方法。

    // 在页面上添加评论，id为标识，text为内容
    addClientComment(id, text);

    // 从页面上删除评论，id为标识
    removeClientComment(id);

    // 更新页面上的评论，id为标识，text为内容
    updateClientComment(id, text);

### <a name="page-helpers"></a>辅助操作

页面上还定义了一些其他辅助方法。

    // 显示Loading提示信息
    showLoadingMessage();

    // 隐藏Loading提示信息
    hideLoadingMessage();

## <a name="related-links"></a>相关链接

* [在线演示](http://repository.jscex.info/samples/async/comments.html)
* [完整代码](https://github.com/JeffreyZhao/jscex/blob/master/samples/async/comments.html)
* [Jscex异步模块](../)
* [Jscex异步增强模块](../powerpack.html)