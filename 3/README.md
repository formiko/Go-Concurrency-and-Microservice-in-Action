# 第3章 Go 语言基础
## 3.2 环境安装
### 3.2.2 第一个Go语言程序
``` Go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"math/rand"
	"net/http"
)

// 请求结构体
type requestBody struct {
	Key string `json:"key"`
	Info string `json:"info"`
	UserInfo string `json:"userid"`
}
// 结果结构体
type responseBody struct {
	Code int `json:"code"`
	Text string `json:"text"`
	List []string `json:"list"`
	Url string `json:"url"`
}
// 请求机器人
func process(inputChan <-chan string, userid string) {
	for {
		// 从通道中接受输入
		input := <- inputChan
		if input == "EOF" {
			break
		}
		// 构建请求体
		reqData := &requestBody {
			Key: "9834709b6c4046f1baa03cc62f83f1bd",
			Info: input,
			UserInfo: userid,
		}
		// 转义为 json
		byteData, _ := json.Marshal(&reqData)
		req, err := http.NewRequest("POST", "http://www.tuling123.com/openapi/api", bytes.NewReader(byteData))
		req.Header.Set("Content-Type", "application/json;charset=UTF-8")
		client := http.Client{}
		resp, err := client.Do(req)
		if err != nil {
			fmt.Println("Network Error!", err)
		} else {
			body, _ := ioutil.ReadAll(resp.Body)
			var respData responseBody
			json.Unmarshal(body, &respData)
			fmt.Println("AI: " + respData.Text)
		}
		if resp != nil {
			resp.Body.Close()
		}
	}
}
func main() {
	var input string
	fmt.Println("Enter 'EOF' to shut down: ")
	// 创建通道
	channel := make(chan string)
	// main 结束时关闭通道
	defer close(channel)
	// 启动 goroutine 运行机器人回答线程
	go process(channel, string(rand.Int63()))
	for {
		fmt.Scanf("%s", &input)
		channel <- input
		if input == "EOF" {
			fmt.Println("Bye!")
			break
		}
	}
}
```

## 3.3 基本语法
### 3.3.3 指针
* 使用flag从命令行读取数据
    ``` Go
    package main

    import "flag"
    import "fmt"
    func main() {
        surname := flag.String("surname", "王", "你的姓")
        var personalname string
        flag.StringVar(&personalname, "personalname", "小二", "你的名")
        id := flag.Int("id", 0, "你的ID" )
        flag.Parse()
        fmt.Printf("我叫%v%v，我的ID是%v", *surname, personalname, *id)
    }
    ```
## 3.5 函数与接口
### 3.5.5 函数体实现接口
``` Go
package main

import "fmt"

type Printer interface {
	Print(interface{})
}
type FuncCaller func(p interface{})
func (funcCaller FuncCaller) Print(p interface{}) {
	funcCaller(p)
}
func main() {
	var printer Printer
	printer = FuncCaller(func(p interface{}){
		fmt.Println(p)
	})
	printer.Print("Golang is Good!")
}
```
## 3.6 结构体和方法
### 3.6.4 结构体实现接口
``` Go
package main
import "fmt"
type Cat interface {
	CatchMouse()
}
type Dog interface {
	Bark()
}
type CatDog struct {
	Name string
}
func (catDog *CatDog) CatchMouse() {
	fmt.Printf("%v caught the mouse and ate it!\n", catDog.Name)
}
func (catDog *CatDog) Bark() {
	fmt.Printf("%v barked loudly!\n", catDog.Name)
}
func main() {
	catDog := &CatDog{
		"Lucy",
	}
	var cat Cat
	cat = catDog
	cat.CatchMouse()

	var dog Dog
	dog = catDog
	dog.Bark()
}
```
### 3.6.5 内嵌和组合
* 组合
    ``` Go
    package main
    import "fmt"
    type Swimming struct {
    }
    func (swim *Swimming) swim() {
        fmt.Println("swimming is my ability")
    }
    type Flying struct {
    }
    func (fly *Flying) fly() {
        fmt.Println("flying is my ability")
    }
    type WildDuck struct {
        Swimming
        Flying
    }
    type DomesticDuck struct {
        Swimming
    }
    func main() {
        wild := WildDuck{}
        wild.fly()
        wild.swim()

        domestic := DomesticDuck{}
        domestic.swim()
    }
    ```