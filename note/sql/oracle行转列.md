![img](D:\笔记\img\12) 

SELECT t.CUBE_TYPE as type,

max(decode(t.DIMENSIONS,'X',DEMARCATION,null)) as xValue,

max(decode(t.DIMENSIONS,'Y',DEMARCATION,null)) as yValue 

from d_pos_cube_config t GROUP BY t.CUBE_TYPE

![img](D:\笔记\img\13)

