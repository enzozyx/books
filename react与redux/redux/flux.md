# flux

## 单项数据流设计思路

```flow
st=>start: startAction
opDistpatch=>operation: dispatch
opStore=>operation: store
opView=>operation: view
opUpdateAction=>operation: updateAction
opNeedUpdate=>condition: needUpdate
et=>end: display
st->opDistpatch->opStore->opView->opNeedUpdate
opNeedUpdate(no)->et
opNeedUpdate(yes)->opUpdateAction->opDistpatch


```