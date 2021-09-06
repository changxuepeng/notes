https://mp.weixin.qq.com/s/p7u0pn4m5b4KhCcDkcrQOA

# 优雅的实现 Excel 导入导出

日常在做后台系统的时候会很频繁的遇到Excel导入导出的问题，正好这次在做一个后台系统，就想着写一个公用工具来进行Excel的导入导出。

一般我们在导出的时候都是导出的前端表格，而前端表格同时也会对应的在后台有一个映射类。

所以在写这个工具的时候我们先理一下我们需要实现的效果：

- 导出方法接收一个list集合，和一个Class类型，和HttpServletResponse 对象
- 导出是可能会有下拉列表，所以需要一个map存储下拉列表数据源，传入参数后只需一行代码即可导出
- 导入方法需要传入file文件，以及一个Class类型，导入之后将会返回一个list集合，里面的对象就是传入类型的对象，传入参数后只需一行代码即可导入

**实现过程：**

首先需要创建三个注解

一个是EnableExport ，必须有这个注解才能导出

```java
/**
 * 设置允许导出
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface EnableExport {
     String fileName();

}
```

然后就是EnableExportField，有这个注解的字段才会导出到Excel里面，并且可以设置列宽

```java
/**
 * 设置该字段允许导出
 * 并且可以设置宽度
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface EnableExportField {
     int colWidth() default  100;
     String colName();
}
```

再就是ImportIndex，导入的时候设置Excel中的列对应的序号

```java
/**
 * 导入时索引
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ImportIndex {
     int index() ;

}
```

注解使用示例

![图片](excel.assets/640)

**三个注解创建好之后就需要开始操作Excel了**

首先，导入方法。在后台接收到前端上传的Excel文件之后，使用poi来读取Excel文件

我们根据传入的类型上面的字段注解的顺序来分别为不同的字段赋值，然后存入集合中，再返回

代码如下：

```java
/**
 * 将Excel转换为对象集合
 * @param excel Excel 文件
 * @param clazz pojo类型
 * @return
 */
public static List<Object> parseExcelToList(File excel,Class clazz){
    List<Object> res = new ArrayList<>();
    // 创建输入流，读取Excel
    InputStream is = null;
    Sheet sheet = null;
    try {
        is = new FileInputStream(excel.getAbsolutePath());
        if (is != null) {
            Workbook workbook = WorkbookFactory.create(is);
            //默认只获取第一个工作表
            sheet = workbook.getSheetAt(0);
            if (sheet != null) {
             //前两行是标题
                int i = 2;
                String values[] ;
                Row row = sheet.getRow(i);
                while (row != null) {
                    //获取单元格数目
                    int cellNum = row.getPhysicalNumberOfCells();
                    values = new String[cellNum];
                    for (int j = 0; j <= cellNum; j++) {
                        Cell cell =   row.getCell(j);
                        if (cell != null) {
                            //设置单元格内容类型
                            cell.setCellType(Cell.CELL_TYPE_STRING );
                            //获取单元格值
                            String value = cell.getStringCellValue() == null ? null : cell.getStringCellValue();
                            values[j]=value;
                        }
                    }
                    Field[] fields = clazz.getDeclaredFields();
                    Object obj = clazz.newInstance();
                    for(Field f : fields){
                        if(f.isAnnotationPresent(ImportIndex.class)){
                            ImportIndex annotation = f.getDeclaredAnnotation(ImportIndex.class);
                            int index = annotation.index();
                            f.setAccessible(true);
                            //此处使用了阿里巴巴的fastjson包里面的一个类型转换工具类
                            Object val =TypeUtils.cast(values[index],f.getType(),null);
                            f.set(obj,val);
                        }
                    }
                    res.add(obj);
                    i++;
                    row=sheet.getRow(i);
                }

            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return res;
}
```

接下来就是导出方法。

导出分为几个步骤：

1. 建立一个工作簿，也就是类型新建一个Excel文件

![图片](excel.assets/640-16308920760882)

1. 建立一张sheet表

![图片](excel.assets/640-16308920794114)

1. 设置标的行高和列宽

![图片](excel.assets/640-16308920883316)

1. 绘制标题和表头

![图片](excel.assets/640-16308921136308)

这两个方法是自定义方法，代码会贴在后面

1. 写入数据到Excel

![图片](excel.assets/640-163089211860910)

1. 创建下拉列表

![图片](excel.assets/640-163089213501812)

1. 写入文件到response

![图片](excel.assets/640-163089213786814)

到这里导出工作就完成了

**下面是一些自定义方法的代码**

```java
/**
 * 获取一个基本的带边框的单元格
 * @param workbook
 * @return
 */
private static HSSFCellStyle getBasicCellStyle(HSSFWorkbook workbook){
    HSSFCellStyle hssfcellstyle = workbook.createCellStyle();
    hssfcellstyle.setBorderLeft(HSSFCellStyle.BORDER_THIN);
    hssfcellstyle.setBorderBottom(HSSFCellStyle.BORDER_THIN);
    hssfcellstyle.setBorderRight(HSSFCellStyle.BORDER_THIN);
    hssfcellstyle.setBorderTop(HSSFCellStyle.BORDER_THIN);
    hssfcellstyle.setAlignment(HSSFCellStyle.ALIGN_CENTER);
    hssfcellstyle.setVerticalAlignment(HSSFCellStyle.VERTICAL_CENTER);
    hssfcellstyle.setWrapText(true);
    return hssfcellstyle;
}

/**
 * 获取带有背景色的标题单元格
 * @param workbook
 * @return
 */
private static HSSFCellStyle getTitleCellStyle(HSSFWorkbook workbook){
    HSSFCellStyle hssfcellstyle =  getBasicCellStyle(workbook);
    hssfcellstyle.setFillForegroundColor((short) HSSFColor.CORNFLOWER_BLUE.index); // 设置背景色
    hssfcellstyle.setFillPattern(HSSFCellStyle.SOLID_FOREGROUND);
    return hssfcellstyle;
}

/**
 * 创建一个跨列的标题行
 * @param workbook
 * @param hssfRow
 * @param hssfcell
 * @param hssfsheet
 * @param allColNum
 * @param title
 */
private static void createTitle(HSSFWorkbook workbook, HSSFRow hssfRow , HSSFCell hssfcell, HSSFSheet hssfsheet,int allColNum,String title){
    //在sheet里增加合并单元格
    CellRangeAddress cra = new CellRangeAddress(0, 0, 0, allColNum);
    hssfsheet.addMergedRegion(cra);
    // 使用RegionUtil类为合并后的单元格添加边框
    RegionUtil.setBorderBottom(1, cra, hssfsheet, workbook); // 下边框
    RegionUtil.setBorderLeft(1, cra, hssfsheet, workbook); // 左边框
    RegionUtil.setBorderRight(1, cra, hssfsheet, workbook); // 有边框
    RegionUtil.setBorderTop(1, cra, hssfsheet, workbook); // 上边框

    //设置表头
    hssfRow = hssfsheet.getRow(0);
    hssfcell = hssfRow.getCell(0);
    hssfcell.setCellStyle( getTitleCellStyle(workbook));
    hssfcell.setCellType(HSSFCell.CELL_TYPE_STRING);
    hssfcell.setCellValue(title);
}

/**
 * 设置表头标题栏以及表格高度
 * @param workbook
 * @param hssfRow
 * @param hssfcell
 * @param hssfsheet
 * @param colNames
 */
private static void createHeadRow(HSSFWorkbook workbook,HSSFRow hssfRow , HSSFCell hssfcell,HSSFSheet hssfsheet,List<String> colNames){
    //插入标题行
    hssfRow = hssfsheet.createRow(1);
    for (int i = 0; i < colNames.size(); i++) {
        hssfcell = hssfRow.createCell(i);
        hssfcell.setCellStyle(getTitleCellStyle(workbook));
        hssfcell.setCellType(HSSFCell.CELL_TYPE_STRING);
        hssfcell.setCellValue(colNames.get(i));
    }
}
/**
 * excel添加下拉数据校验
 * @param sheet 哪个 sheet 页添加校验
 * @return
 */
public static void createDataValidation(Sheet sheet,Map<Integer,String[]> selectListMap) {
    if(selectListMap!=null) {
        selectListMap.forEach(
                // 第几列校验（0开始）key 数据源数组value
                (key, value) -> {
                    if(value.length>0) {
                        CellRangeAddressList cellRangeAddressList = new CellRangeAddressList(2, 65535, key, key);
                        DataValidationHelper helper = sheet.getDataValidationHelper();
                        DataValidationConstraint constraint = helper.createExplicitListConstraint(value);
                        DataValidation dataValidation = helper.createValidation(constraint, cellRangeAddressList);
                        //处理Excel兼容性问题
                        if (dataValidation instanceof XSSFDataValidation) {
                            dataValidation.setSuppressDropDownArrow(true);
                            dataValidation.setShowErrorBox(true);
                        } else {
                            dataValidation.setSuppressDropDownArrow(false);
                        }
                        dataValidation.setEmptyCellAllowed(true);
                        dataValidation.setShowPromptBox(true);
                        dataValidation.createPromptBox("提示", "只能选择下拉框里面的数据");
                        sheet.addValidationData(dataValidation);
                    }
                }
        );
    }
}
```

**使用实例**

导出数据

![图片](excel.assets/640-163089218083516)

导入数据（返回对象List）

![图片](excel.assets/640-163089219077618)

源码地址：

https://github.com/xyz0101/excelutils

```
来源：blog.csdn.net/youzi1394046585/article/details/86670203
```