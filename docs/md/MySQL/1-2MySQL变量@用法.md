# 1.2 MySQL变量@用法

## 1.增加临时表，实现变量的自增

```
SELECT (@i:=@i+1),A.* FROM A,(SELECT @i:=0) AS B
```

> (@i:=@i+1)代表定义一个变量，每次叠加1,(SELECT @i:=0) AS B 代表建立一个临时表

## 2.实现排序递增

```
SELECT (@i:=@i+1),C.*  FROM ( SELECT * FROM A ORDER BY A.id DESC )C,(SELECT@i:=0)B
```

## 3.实现分组递增

```
SELECT  (case when @bq = A.name then @i:=@i when @bq != A.name then @i:=@i+1 end),(@bq:= A.name),A.* 
FROM A ,(SELECT @i:=0,@bq:='') AS B
```

