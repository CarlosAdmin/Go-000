# # 作业

1. 我们在数据库操作的时候，比如 dao 层中当遇到一个 `sql.ErrNoRows` 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？

## 我的理解
1. 数据库操作遇到异常则往上抛出
2. 数据操作失败，如更新数据，受影响的行数为0，则往上抛出
3. 数据是否存在，返回true/false 如数据不存在 则返回false
4. 查询单个数据对象 如数据不存在则应抛出异常

## 伪代码现实如下

### dao层
数据操作
```go
package userdao

import (
	"database/sql"
	_ "github.com/go-sql-driver/mysql"
	"github.com/pkg/errors"
)

var (
	db  *sql.DB
	err error
)

func init() {
	db, err = sql.Open("mysql", "root:root@tcp(localhost:3306)/database")

}

type CustomUser struct {
	UserId     int
	UserName string
	Password	string
	Address	string
}

func UpdateUserAddressByUserId(userid int,adddress string) (bool,error) {

	sql:="update customuser set address="+adddress+" where userid="+userid+“;”
	
	execresult, err  := db.Exec(sql)
	if err != nil {
		return false, errors.Wrap(err, "exec error")
	}

	result, err := execresult.RowsAffected()
	if err != nil {
		return false, errors.Wrap(err, "update failed")
	}

	defer db.Close()
	return true, nil
}

```

### service层
业务逻辑操作，
接收异常处理 返回自定义异常
如404 500 等
```go
func UpdateUserAddressByUserId(userid int,adddress string)  (bool,error) {
	return dao.UpdateUserAddressByUserId(userid,adddress)
}

```

### main函数入口
```go
数据接收操作，并处理异常

func main(){
    user, err := service.UserService.UpdateUserAddressByUserId(1，“xxx”)
	if err != nil {
		fmt.Printf("更新失败", err)
	} else {
		fmt.Println("更新成功")
	}
}
```
