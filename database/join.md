# join 语法
![image](https://user-images.githubusercontent.com/42630862/179871229-b081f9ad-ca4d-448e-9154-8d34d64d759d.png)



# 1 hash join 原理
## 1.1 mysql的hash join
### 1.1.1 内存能放下所有的hash row

![image](https://user-images.githubusercontent.com/42630862/179874656-4c17047b-410f-4d0c-81a8-857553038565.png)

![image](https://user-images.githubusercontent.com/42630862/179874670-95ed6ddb-7bdb-4219-8059-e358f60bd618.png)

### 1.1.2 内存放不下， 利用disk 分治
![image](https://user-images.githubusercontent.com/42630862/179877191-e684d448-8fba-4e9e-9afc-20871cd5e499.png)

![image](https://user-images.githubusercontent.com/42630862/179877156-da4e9228-657b-41a8-9bc2-a2130f91fb3a.png)
![image](https://user-images.githubusercontent.com/42630862/179877165-cc0013cb-5a7c-4a7f-aa8b-fe6fcef7f2af.png)



