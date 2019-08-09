# Excel

## 注意事项

* 当读取ClassPath中的Excel时，报类似错误：`NotOLE2FileException: Invalid header signature; read 0xE011BDBFEFBDBFEF, expected 0xE11AB1A1E011CFD0 - Your file appears not to be a valid OLE2 document`。
请检查`pom`文件配置`resource`时是否`filtering` Excel文件。正确配置参考如下：

    ```xml
    <resources>
      <resource>
        <filtering>true</filtering>
        <directory>src/main/resources</directory>
        <excludes>
          <exclude>excel/*</exclude>
        </excludes>
      </resource>
      <resource>
        <directory>src/main/resources</directory>
        <includes>
          <include>excel/*</include>
        </includes>
      </resource>
    </resources>
    ```

* `EasyPoi` `4.1.0` 版本在使用模板导出时，循环表达式下面不能只有一行内容，否则最终生成的Excel文件会缺失最后一行。
所以要么没有内容，要么为多行内容。如果你确实只需要一行内容，你可以把Excel的多行合并成一行，在合并后的行里填充模板。
产生此BUG的代码在`ExcelExportOfTemplateUtil#addListDataToExcel()`：

    ```java
    if (isShift && datas.size() * rowspan > 1 && cell.getRowIndex() + rowspan < cell.getRow().getSheet().getLastRowNum()) {
        int lastRowNum = cell.getRow().getSheet().getLastRowNum();
        cell.getRow().getSheet().shiftRows(cell.getRowIndex() + rowspan, lastRowNum, (datas.size() - 1) * rowspan, true, true);
        mergedRegionHelper.shiftRows(cell.getSheet(), cell.getRowIndex() + rowspan, (datas.size() - 1) * rowspan);
        templateSumHandler.shiftRows(cell.getRowIndex() + rowspan, (datas.size() - 1) * rowspan);
        PoiExcelTempUtil.reset(cell.getSheet(), cell.getRowIndex() + rowspan + (datas.size() - 1) * rowspan, cell.getRow().getSheet().getLastRowNum());
    }
    ```
    
    > `cell.getRowIndex() + rowspan < cell.getRow().getSheet().getLastRowNum()`比较会出现相等的情况。`cell.getRowIndex()` 当前遍历标签
    的行索引，`rowspan`遍历标签占用的行数，`cell.getRow().getSheet().getLastRowNum()`当前模板的最大行索引数。

## 组件

* [POI](http://poi.apache.org/components/spreadsheet/quick-guide.html)
* [Java Excel API](http://.sourceforge.net/)
* [EasyPoi -【推荐】- 对POI的完整封装，支持注解和模板](https://easypoi.mydoc.io/)
* [JXLS -【推荐】- 使用模板导出，支持`POI`及`Java Excel API`](http://jxls.sourceforge.net/index.html)
    * [jxls-demo](https://bitbucket.org/leonate/jxls-demo/src/master/)
* [easyexcel - 对POI的完整封装](https://github.com/alibaba/easyexcel)

## 参考文档

* [java 导出 excel 最佳实践，java 大文件 excel 避免OOM(内存溢出) excel 工具框架](https://juejin.im/post/5bfbfbebe51d4505e64bb392)
* [docx4j](https://www.docx4java.org/trac/docx4j)
