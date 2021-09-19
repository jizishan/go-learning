1. 我们在数据库操作的时候，比如 dao 层中当遇到一个 sql.ErrNoRows 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？
答：不用，在dao层，一般会定义自己的错误，以便进行业务逻辑处理


```go
package main

import (
	"database/sql"
	"errors"
	"fmt"

	_ "github.com/go-sql-driver/mysql"
)

var db *sql.DB

type user struct {
	id   int
	age  int
	name string
}

var (
	ErrorUserExist       = errors.New("用户已存在")
	ErrorUserNotExist    = errors.New("用户不存在")
	ErrorInvalidPassword = errors.New("用户名或密码错误")
	ErrorInvalidID       = errors.New("无效的ID")
)

// 定义一个初始化数据库的函数
func initDB() (err error) {
	// DSN:Data Source Name
	dsn := "root:123456@tcp(127.0.0.1:3306)/mygo?charset=utf8mb4&parseTime=True"
	// 不会校验账号密码是否正确
	// 注意！！！这里不要使用:=，我们是给全局变量赋值，然后在main函数中使用全局变量db
	db, err = sql.Open("mysql", dsn)

	if err != nil {
		return err
	}
	// 尝试与数据库建立连接（校验dsn是否正确）
	err = db.Ping()
	if err != nil {
		return err
	}
	return nil
}


// 查询单条数据
func queryRowDemo(myName string) (err error) {
	sqlStr := "select id, name  from stu where name=?"
	var u user

	if err := db.QueryRow(sqlStr, myName).Scan(&u.id, &u.name); err != nil {
		//fmt.Printf("%s not found, %v\n", myName, err)
		return ErrorUserNotExist
	}
	//fmt.Printf("id:%d name:%s", u.id, u.name)
	return nil
}

func main() {
	if err := initDB(); err != nil {
		fmt.Printf("init db failed,err:%v\n", err)
		return
	}
	if err := queryRowDemo("Pony"); err != nil {
		fmt.Printf("%v\n", err)
	}
}
```
