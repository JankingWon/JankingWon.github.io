---
title: 使用Go搭建简单的RESTful服务器之后端数据库实现
urlname: swapi
tags:
  - Go
copyright: true
category: Go
abbrlink: 51980
date: 2018-12-16 16:33:18
---

模仿`swapi`搭建一个前后端协调可用的数据查询服务器

<!-- more -->

![1544949349993](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/swapi/1544949349993.png)

## 建立数据库
调用`github.com/boltdb/bolt`的`Open`函数创建一个容器

类似于这样的语句`root.CreateBucketIfNotExists([]byte("Planets"))`创建一个而带有标号的桶，以后关于`Planet`的数据都放进这个桶里

```go
func setupDB() (*bolt.DB, error) {
	db, err := bolt.Open("swapi.db", 0600, nil)
	if err != nil {
		return nil, fmt.Errorf("could not open db, %v", err)
	}
	err = db.Update(func(tx *bolt.Tx) error {
		root, err := tx.CreateBucketIfNotExists([]byte("DB"))
		if err != nil {
			return fmt.Errorf("could not create root bucket: %v", err)
		}
		_, err = root.CreateBucketIfNotExists([]byte("Planets"))
		_, err = root.CreateBucketIfNotExists([]byte("Species"))
		_, err = root.CreateBucketIfNotExists([]byte("Vehicle"))
		_, err = root.CreateBucketIfNotExists([]byte("Starship"))
		_, err = root.CreateBucketIfNotExists([]byte("Person"))
		_, err = root.CreateBucketIfNotExists([]byte("Film"))
		return nil
	})
	if err != nil {
		return nil, fmt.Errorf("could not set up buckets, %v", err)
	}
	fmt.Println("DB Setup Done")
	return db, nil
}
```

## 插入数据函数

调用数据库的`Update`函数，传递一个参数，而且这个参数也是一个函数

通过`bucketName`找到对应的`bucket`，然后插入一条`id`和`jsonStr`对应的数据

这里有点冗余地调用`*bolt.DB`参数，其实小程序全局用一个桶就好了

```go
func addToBucket(db *bolt.DB, jsonStr string, id string, bucketName string) error {

	planetBytes := []byte(jsonStr)

	err := db.Update(func(tx *bolt.Tx) error {
		err := tx.Bucket([]byte("DB")).Bucket([]byte(bucketName)).Put([]byte(id), []byte(planetBytes))
		if err != nil {
			return fmt.Errorf("could not insert %s: %v", bucketName, err)
		}

		return nil
	})
	fmt.Println("Added " + bucketName)
	return err
}
```

## 添加数据

这里使用了`os/exec`包

通过循环查询`swapi.co`网站的数据写入到我们自己的数据库中

```go
const (
		planet = 1
		species = 2
		vehicle = 3
		starship = 4
		people = 5
		film  = 6
	)
	var tabText = map[int]string{
		planet:  "Planets",
		species: "Species",
		vehicle: "Vehicle",
		starship: "Starship",
		people: "People",
		film:  "Film",
	}
	var nameText = map[int]string{
		planet:  "planets",
		species: "species",
		vehicle: "vehicles",
		starship: "starships",
		people: "people",
		film:  "films",
	}
	var curl []byte
	var cmd *exec.Cmd
	for i := 1; i < 100; i++ {
		for k := 1; k <= 6; k++ {
			// fmt.Println("https://swapi.co/api/" + nameText[k] + "/" + string(i) + "/")
			// fmt.Println(string(i))
			ID := strconv.Itoa(i)
			cmd = exec.Command("curl", ("https://swapi.co/api/" + nameText[k] + "/" + ID + "/"))
			if curl, err = cmd.Output(); err != nil {
			    fmt.Println(err)
			    os.Exit(1)
			}

			// fmt.Println(("https://swapi.co/api/" + nameText[k] + "/" + ID + "/"))
			JSONStr := strings.TrimRight(string(curl), "\n")
			fmt.Println(JSONStr)
			fmt.Println(ID)

			err = addToBucket(db, JSONStr, ID, tabText[k])
			if err != nil {
				log.Fatal(err)
			}
		}
	}
```

## 后端测试

![1544953263611](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/swapi/1544953263611.png)

