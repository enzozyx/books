# flux 与 redux


## flux 设计思路

 1. flux 流程

>单向数据流 

```flow
opStartAction=>operation: startAction
opDispatcherTag=>operation: dispatcher
opStoreTag=>operation: store
opViewTag=>operation: view
opUpdateAction=>operation: updateAction
opStartAction->opDispatcherTag->opStoreTag->opViewTag
opViewTag>opUpdateAction->opDispatcherTag
```


