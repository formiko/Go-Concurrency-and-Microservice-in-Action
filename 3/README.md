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