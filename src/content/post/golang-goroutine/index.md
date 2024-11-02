---
title: "Golang Goroutine"
description: "This post will explain golang goroutine and channels and also wait group"
publishDate: "01 Nov 2024"
updatedDate: "01 Nov 2024"
# coverImage:
#   src: "./cover.png"
#   alt: "Astro build wallpaper"
tags: ["golang", "tech", "goroutine"]
---

# GOROUTINE & CHANNELS
## 1. Goroutine
Goroutine adalah lightweight thread yang dimanage oleh Go runtime. Goroutine sangat ringan, hanya dibutuhkan sekitar **2kB** memori saja untuk satu buah Goroutine. Goroutine memiliki sifat yang asynchronous jadi tidak saling menunggu dengan Goroutine yang lain. 

Untuk membuat Goroutine baru caranya cukup mudah, yaitu dengan menambahkan awalan `go` yang diikuti dengan nama method yang akan dijalankan secara Goroutine. Berikut ini contoh penggunaannya.
```go
package main

import (
  "fmt"
  "time"
)

func say(s string) {
  for i := 0; i < 5; i++ {
   time.Sleep(100 * time.Millisecond)
   fmt.Println(s)
  }
}

func main() {
  go say("world")
  say("hello")
}
```
Jika program dijalankan maka akan muncul :
```bash
fahrudin@belajar-goroutine $ go run main.go
world
hello
hello
world
world
hello
hello
world
world
hello    
```
Dapat dilihat output yang dihasilkan adalah tulisan `hello` dan `world` muncul secara selang seling, ini dikarenakan `say("world")` dijalankan sebagai Goroutine, sehingga tidak terjadi saling menunggu.

## 2. Channels
## 3. Buffered Channels
## 4. Select


### Referensi Tulisan
- https://dasarpemrogramangolang.novalagung.com/A-goroutine.html
