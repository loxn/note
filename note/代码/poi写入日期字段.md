```java
CellStyle dataCellStyle = workbook.createCellStyle();
dataCellStyle.setDataFormat(workbook.createDataFormat().getFormat("yyyy-MM-dd HH:mm:ss"));
Cell cell = row.createCell(j);
cell.setCellValue(new Date());
cell.setCellStyle(dataCellStyle);
```

