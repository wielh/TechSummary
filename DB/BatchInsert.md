# BatchInsert 效率紀錄

## 前置條件

+ 使用 golang, go-pg, postgreSQL

+ 插入 10w 筆資料

+ 三條件分別為單筆插入，預編譯單筆插入和批量插入

## 結果

分別花費 11s, 9s 與 1s

## example code 

```
func timeTrack(name string, f func([]int32)) {
	r := generateRandomNumbers()
	start := time.Now()
	f(r)
	elapsed := time.Since(start)
	fmt.Printf("%s took %s\n", name, elapsed)
}

func generateRandomNumbers() []int32 {
	rand.Seed(time.Now().UnixNano())
	answer := make([]int32, 100000)
	for i := 0; i < len(answer); i++ {
		answer[i] = 100 + rand.Int31n(100)
	}
	return answer
}

func direect(ramdonNums []int32) {
	for _, num := range ramdonNums {
		_, err := DB.Model(&user{Gold: num}).Insert()
		if err != nil {
			fmt.Println(err)
		}
	}
}

func prepared(ramdonNums []int32) {
	stmt, err := DB.Prepare("insert into users(gold) values($1)")
	if err != nil {
		fmt.Println(err)
		return
	}
	defer stmt.Close()

	for _, num := range ramdonNums {
		_, err := stmt.Exec(num)
		if err != nil {
			fmt.Println(err)
		}
	}
}

func batch(ramdonNums []int32) {
	users := []user{}
	for _, num := range ramdonNums {
		users = append(users, user{Gold: num})
	}
	_, err := DB.Model(&users).Insert()
	if err != nil {
		fmt.Println(err)
	}
}
```