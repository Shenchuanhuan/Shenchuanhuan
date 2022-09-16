---
title: IndexedDB笔记
---

# 什么是indexdb?
IndexedDB 是一种可以让你在用户的浏览器内持久化存储数据的方法。IndexedDB 为生成 Web Application 提供了丰富的查询能力，使我们的应用在在线和离线时都可以正常工作。

‼️ 显然，浏览器不希望允许某些广告网络或恶意网站来污染你的计算机，所以浏览器会在任意给定的 web app 首次尝试打开一个 IndexedDB 存储时对用户进行提醒。用户可以选择允许访问或者拒绝访问。还有，IndexedDB 在浏览器的隐私模式（Firefox 的 Private Browsing 模式和 Chrome 的 Incognito 模式）下是被完全禁止的。 隐私浏览的全部要点在于不留下任何足迹，所以在这种模式下打开数据库的尝试就失败了。

‼️ 在打开数据库时常见的可能出现的错误之一是 VER_ERR。这表明存储在磁盘上的数据库的版本高于你试图打开的版本。这是一种必须要被错误处理程序处理的一种出错情况。

‼️ 如果 onupgradeneeded事件成功执行完成，打开数据库请求的 onsuccess 处理函数会被触发。

‼️ 尝试创建一个与已存在的对象仓库重名（或删除一个不存在的对象仓库）会抛出错误。

‼️ 错误事件遵循冒泡机制。错误事件都是针对产生这些错误的请求的，然后事件冒泡到事务，然后最终到达数据库对象。如果你希望避免为所有的请求都增加错误处理程序，你可以替代性的仅对数据库对象添加一个错误处理程序

‼️ Note: As of Firefox 40, IndexedDB transactions have relaxed durability guarantees to increase performance (see bug 1112702.) Previously in a readwrite transaction, a complete event was fired only when all data was guaranteed to have been flushed to disk. In Firefox 40+ the complete event is fired after the OS has been told to write the data but potentially before that data has actually been flushed to disk. The complete event may thus be delivered quicker than before, however, there exists a small chance that the entire transaction will be lost if the OS crashes or there is a loss of system power before the data is flushed to disk. Since such catastrophic events are rare most consumers should not need to concern themselves further. If you must ensure durability for some reason (e.g. you're storing critical data that cannot be recomputed later) you can force a transaction to flush to disk before delivering the complete event by creating a transaction using the experimental (non-standard) readwriteflush mode (see IDBDatabase.transaction).



# 目录
* [window.indexedDB](#window.indexedDB)
* IndexedDB API
    * [IDBFactory](#IDBFactory)
        * IDBFactory.open-打开数据库
    * [IDBOpenDBRequest](#IDBOpenDBRequest)
    * [IDBDatabase](#IDBDatabase)
    * [IDBTransaction](#IDBTransaction)
    * IDBIndex
    * [IDBRequest](#IDBRequest)
    * [IDBObjectStore](#IDBObjectStore)
* 使用集
    * [创建indexedDB数据库](#创建indexedDB数据库)
* examples
    * [toDoList](https://github.com/mdn/to-do-notifications)


[indexedDB doc](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)

### window.indexedDB
    `indexedDB`是一个只读的全局对象属性，它提供了异步操作<u>*indexed database*</u>的方法，比如：`window.indexedDb.open('database-name')`。它的值是[IDBFactory](https://developer.mozilla.org/en-US/docs/Web/API/IDBFactory)。
--------------------------------

### IDBFactory
[mdn doc](https://developer.mozilla.org/en-US/docs/Web/API/IDBFactory)
    它是`IndexedDB API`的接口，`window.indexedDB`是它的实现。

### IDBFactory.open
[mdn doc](https://developer.mozilla.org/en-US/docs/Web/API/IDBFactory/open)
* 语法: open(name) or open(name, version)。接收两个参数，数据库名和数据库版本，数据库版本为可选参数。如果不提供`version`参数，如果要打开的数据库存在，那么打开数据库不会改变数据库版本；如果数据库不存在，数据库会被创建，并且版本为1。
* 返回：IDBOpenDBRequest
    >e.g.
    const request = window.indexedDB.open("toDoList", 4);

‼️ 数据库的版本决定了数据库架构。如果打开的数据库不存在则会创建数据库，同时`onupgradeneeded`事件会被触发，需要在这个事件中创建数据库模式；如果数据库存在，但打开时指定了更高的版本，`onupgradeneeded`事件也会被触发，可以在事件中更新数据库模式。<u>如果 onupgradeneeded事件成功执行完成，打开数据库请求的 `onsuccess` 处理函数会被触发。</u>

⚠️警告： 版本号是一个 unsigned long long 数字，这意味着它可以是一个特别大的数字，但不能使用浮点数，否则它将会被转变成离它最近的整数，这可能导致 upgradeneeded 事件不会被触发。例如，不要使用 2.4 作为版本号。

    var request = indexedDB.open("MyTestDatabase", 2.4); // 不要这么做，因为版本会被置为 2。

--------------------------------
### IDBOpenDBRequest
[mdn doc](https://developer.mozilla.org/en-US/docs/Web/API/IDBOpenDBRequest)
    indexedDB API的接口之一。是打开、删除等请求的返回结果，继承了父亲(IDBRequest)和祖辈(EventTarget)的所有属性及方法。
    
* method
    * onsuccess: (event) => {}
        > event.target.result是一个`IDBDatabase`实例，
    * onerror: (event) => {}

--------------------------------
### IDBDatabase
[mdn doc](https://developer.mozilla.org/en-US/docs/Web/API/IDBDatabase)
* Define
>已连接的数据库的实例。可以创建事务(*transaction*)用来增删改数据库数据。这是唯一可以获取、管理数据库版本的接口。

‼️ Note: Everything you do in IndexedDB always happens in the context of a transaction, representing interactions with data in the database. All objects in IndexedDB — including object stores, indexes, and cursors — are tied to a particular transaction. Thus, you cannot execute commands, access data, or open anything outside of a transaction.
>基本上你在indexedDB里做的操作都是基于事务的。

* Properties
    * (只读)name-数据库名称
    * (只读)vertion-数据库版本（64位整形）
    * (只读)objectStoreName-数据库里`objectStore`名称字符串集合(DOMStringList)
* Methods
    * 继承`EventTarget`所有的方法
    * close
        >Returns immediately and closes the connection to a database in a separate thread.
    * createObjectStore
        >Creates and returns a new object store or index.
    * deleteObjectStore
        >Destroys the object store with the given name in the connected database, along with any indexes that reference it.
    * transaction
        >Immediately returns a transaction object (IDBTransaction) containing the IDBTransaction.objectStore method, which you can use to access your object store. Runs in a separate thread.

--------------
### IDBTransaction
[mdn doc](https://developer.mozilla.org/en-US/docs/Web/API/IDBTransaction)

--------------
### IDBRequest
[mdn doc](https://developer.mozilla.org/en-US/docs/Web/API/IDBRequest)

--------------



### IDBObjectStore
--------------
[mdn doc](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore)

---------------




### 创建indexedDB数据库
```javascript
    var db;
    const request = window.indexedDB.open(name, version);
    request.onerror = function(evt) {
        // error on open
    };
    request.onsuccess = function(evt) {
        // db = evt.target.result;
        db = this.result;
    };
```