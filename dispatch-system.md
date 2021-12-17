## 派遣系统

### el-pagination 报错

```html
        <el-pagination
            v-show="data.length > 0"
            @size-change="handlePagesizeChange"
            @current-change="handleCurrentpageChange"
            :current-page="currentPage"
            :page-sizes="pagesizes"
            :page-size="pagesize"
            layout="total, sizes, prev, pages, next, jumper"
            style="margin-top: 20px"
            :total="data.length">
        </el-pagination>
```

以上代码会报下面的错：

```
[Vue warn]: Error in nextTick: "TypeError: Cannot read properties of undefined (reading 'key')"

TypeError: Cannot read properties of undefined (reading 'key')

```

问题出在 layout 的值里面有一个 `pages`，但 layout 的可选值只  `sizes, prev ,pager ,  next, jumper, ->, total, slot` 。

