---
title: "Beego 多数据库支持（主从分离等)"
date: 2020-01-02T14:28:35+08:00
isCJKLanguage: true
draft: false
tags:
  - macos
  - 今天搜了啥
---

 
## 原理

使用 orm.RegisterDataBase 注册多个数据库，其中一个为default，且为主库. 

ORM must register a database with alias default.

<!--more-->
```
orm.RegisterDataBase("default", "mysql", connStr, 20,100)

orm.RegisterDataBase("slav1", "mysql", slav1connStr, 5,50)
orm.RegisterDataBase("slav2", driver2, slav2connStr, 0)
        
```

## 使用 

```
//主库
o1 := orm.NewOrm()
o1.Using("default")
	
//从库
o2 := orm.NewOrm()
o2.Using("slav1")
```

## 同步创建db

在不同的数据库源（db)中创建相同的表:

```
orm.RunSyncdb("default", true, true)
orm.RunSyncdb("db1", true, true)
orm.RunSyncdb("db2", true, true)
```

在不同的数据库中创建不同的表:

```
// register driver.
orm.RegisterDriver(dbDriver, orm.DRMySQL)
//  create table 'tree' for database 'plant'.
orm.RegisterDataBase(dbAliasPlant, dbDriver, datasourcePlant)
orm.RegisterModel(new(Tree))
orm.RunSyncdb(dbAliasPlant, true, false)

// we must clear the mode cache before create another database table.
orm.ResetModelCache()
//  create table 'bird' for database 'animal'.
orm.RegisterDataBase(datasourceAnimal, dbDriver, datasourceAnimal)
orm.RegisterModel(new(Bird))
orm.RunSyncdb(datasourceAnimal, true, false)

// we need register model again.
orm.ResetModelCache()
orm.RegisterModel(new(Bird), new(Tree))
db, _ := orm.GetDB(dbAliasPlant)
OrmObjPlant, _ = orm.NewOrmWithDB(dbDriver, dbAliasPlant, db)
db, _ = orm.GetDB(datasourceAnimal)
OrmObjAnimal, _ = orm.NewOrmWithDB(dbDriver, datasourceAnimal, db)

```

## 原有代码改造

models.go old: 

```
func NewUser(name, nickName string) (*User, error) {
	o := orm.NewOrm()
	...
}
```

调整为 ：  

```
新增后缀_DbAlias方法
func NewnUser_DbAlias(name, nickName string, dbAlias string) (*User, error) {

	o := orm.NewOrm()
	if dbAlias != "" {
		//数据源选择
		o.Using(dbAlias)
	}
	... 
}

func NewUser(name, nickName string) (*User, error) {
	return NewnUser_DbAlias(name, nickName, "")
}
```


