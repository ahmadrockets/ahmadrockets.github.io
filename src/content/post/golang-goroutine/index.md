---
title: "Golang Goroutine"
description: "This post will explain golang goroutine and channels and also wait group"
publishDate: "01 Nov 2024"
updatedDate: "01 Nov 2024"
# coverImage:
#   src: "./cover.png"
#   alt: "Astro build wallpaper"
tags: ["golang", "tech", "goroutine", "teknologi"]
---

# GOROUTINE & CHANNELS
## Goroutine
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
Jika program dijalankan maka akan memunculkan output seperti berikut :
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

## Channels
Channel adalah penghubung antara goroutine satu dengan goroutine lainnya. Channel bisa dibuat dengan menggunakan fungsi `make()` dengan menentukan tipe data yang akan dikirim melalui channel.

``` go
ch := make(chan int)
```
kode diatas adalah contoh kode untuk membuat channel.

``` go
ch <- v    // Mengirim variable v ke channel ch.
v := <-ch  // Menerima dari ch ch, dan assign value ke v
```
Kode diatas adalah contoh cara untuk mengirim dan menerima channel.

```go
package main

import "fmt"

func cetak(ch chan int, angka int) {
	fmt.Println("ini dari goroutine cetak...")
	ch <- angka
}

func main() {
	angka := make(chan int)
	go cetak(angka, 20)
	go cetak(angka, 30)
	nilai1, nilai2 := <-angka, <-angka
	fmt.Println("nilai channel integer 1 :", nilai1)
	fmt.Println("nilai channel integer 2 :", nilai2)
	fmt.Println("ini dari function main...")
}
```
Kode diatas adalah contoh penggunaan channel, jika dijalankan 2x maka akan menghasilkan output:
```bash
fahrudin@belajar-goroutine $ go run channel/channel.go
ini dari goroutine cetak... 30
ini dari goroutine cetak... 20
nilai channel integer 1 : 30
nilai channel integer 2 : 20
ini dari function main...

fahrudin@belajar-goroutine $ go run channel/channel.go
ini dari goroutine cetak... 20
ini dari goroutine cetak... 30
nilai channel integer 1 : 20
nilai channel integer 2 : 30
ini dari function main...
```
Output pertama dan kedua berbeda urutan tergantung goroutine mana yang lebih dulu di eksekusi. Hal ini dikarenakan, pengiriman data adalah dari 2 goroutine yang berbeda, yang kita tidak tau mana yang prosesnya selesai lebih dulu. Goroutine yang dieksekusi lebih awal belum tentu selesai lebih awal, yang jelas proses yang selesai lebih awal datanya akan diterima lebih awal.

## Blocking Channels
Pengiriman dan penerimaan data pada channel bersifat `blocking` atau synchronous. Artinya, statement di-bawah syntax pengiriman dan penerimaan data via channel hanya akan dieksekusi setelah proses serah terima berlangsung dan selesai.
```go
package main

import "fmt"

func main() {
	ch := make(chan int) // Unbuffered channel

	fmt.Println("Mengirim data ke channel...")
	ch <- 42                     // Terblokir karena tidak ada penerima
	fmt.Println("Data terkirim") // Tidak akan dieksekusi
}

```
Contoh kode diatas jika dijalankan maka baris kode terakhir tidak akan pernah di eksekusi, akan terjadi proses deadlock dan program akan excited ```fatal error: all goroutines are asleep - deadlock!``` karena tidak ada proses penerimaan channel dari proses diatas.


## Buffered Channels
Secara default channel bersifat *unbufferd*. buffered channel sama seperti channel biasa, tetapi buffered channel memiliki kapasitas penyimpanan data. Selama jumlah data yang dikirim tidak melebihi jumlah buffer, maka pengiriman akan berjalan ***asynchronous*** (tidak blocking). Untuk membuat buffered channel kita bisa menggunakan fungsi ```make(chan string, 2)```, artinya channel dibuat dengan kapasitas 2. <br />
Kelabihan dari buffered channel antara lain bisa mengurangi blocking pada goroutine, selain itu data bisa ditampung sementara tanpa menunggu goroutine penerima. Untuk kekurangan dari buffered channel adalah jika channel penuh, goroutine akan diblock sampai ada tempat kosong.

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```
Kode diatas artinya kita membuat channel dengan kapasitas 2 buffers. Jika dijalankan akan menampilkan.
```bash
fahrudin@belajar-goroutine $ go run buffered_channel/buffered_channel.go 
1
2
```

Berikut ini contoh code channel lainnya.
```go
package main

import "fmt"

func main() {
	ch2 := make(chan int, 2)
	ch2 <- 1
	ch2 <- 2
	ch2 <- 3
	fmt.Println(<-ch2)
	fmt.Println(<-ch2)
	fmt.Println(<-ch2)
}
```

Jika dijalankan maka akan terjadi error deadlock karena kapasitas channel yang didefinisikan adalah 2 buffers, tetapi ada 3 kali value dikirimkan ke channel tersebut.
```bash
fahrudin@belajar-goroutine $ go run buffered_channel/buffered_channel.go
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
        /golang-goroutine/buffered_channel/buffered_channel.go:9 +0x58
exit status 2
```

## Channel Select
`select` digunakan untuk menunggu dan menangani banyak opearasi pada `channel` secara bersamaan. Penggunaan / konsepnya sama seperti `switch` untuk seleksi kondisinya.
Select akan menunggu sampai salah satu case select bisa dieksekusi. Jika lebih dari 1 channel siap, salah satunya dipilih acak. Jika ada default, maka blok default akan dijalankan jika tidak ada channel yang siap. 
Berikut ini adalah contoh penerapannya:
### a. menunggu dari banyak channel
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)

	go func() {
		time.Sleep(1 * time.Second)
		ch1 <- "Message dari ch1"
	}()

	go func() {
		time.Sleep(1 * time.Second)
		ch2 <- "Message dari ch2"
	}()

	for i := 0; i < 2; i++ {
		select {
		case msg := <-ch1:
			fmt.Println("Dapat:", msg)
		case msg := <-ch2:
			fmt.Println("Dapat:", msg)
		}
	}
}
```
Dari kode diatas terdapat `2 channel` dengan tipe string. Keduanya dijalankan dengan goroutine, kemudian terdapat loop 2x sesuai dengan jumlah channel. `Select` akan menunggu sampai salah satu channel punya data. Kalau 2 channel tersebut mendapatkan data secara `bersamaan` maka select akan memilih secara `acak` goroutine mana yang akan dijalankan terlebih dahulu.
jika dijalankan maka akan menampilkan:
```bash
fahrudin@belajar-goroutine $ go run channel_select/channel_select.go
Dapat: Message dari ch1
Dapat: Message dari ch2
fahrudin@belajar-goroutine $ go run channel_select/channel_select.go
Dapat: Message dari ch2
Dapat: Message dari ch1
```
 
### b. Menggunakan default
Default digunakan agar select tidak blocking. Berikut ini contoh penerapannya
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)

	go func() {
		time.Sleep(1 * time.Second)
		ch1 <- "Message dari ch1"
	}()

	for i := 0; i < 4; i++ {
		select {
		case msg := <-ch1:
			fmt.Println("Dapat:", msg)
		case msg := <-ch2:
			fmt.Println("Dapat:", msg)
		default:
			fmt.Println("Menunggu pesan...")
			time.Sleep(500 * time.Millisecond)
		}
	}
}

```
Dari kode diatas jika tidak ada channel yang siap `default` dijalankan sehingga tidak terjadi blocking channel. Jika kode dijalankan outputnya adalah:
```bash
fahrudin@belajar-goroutine $ go run channel_select/channel_select.go
Menunggu pesan...
Menunggu pesan...
Dapat: Message dari ch1
Menunggu pesan...
```

### c. Timeout dengan select + time.After
Timeout digunakan untuk mengantisipasi proses yang dijalankan terlalu lama.
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)

	go func() {
		time.Sleep(5 * time.Second) // simulasi proses berlangsung selama 5 detik
		ch1 <- "Message dari ch1"
	}()

	go func() {
		time.Sleep(5 * time.Second) // simulasi proses berlangsung selama 5 detik
		ch2 <- "Message dari ch2"
	}()

	for i := 0; i < 2; i++ {
		select {
		case msg := <-ch1:
			fmt.Println("Dapat:", msg)
		case msg := <-ch2:
			fmt.Println("Dapat:", msg)
		case <-time.After(2 * time.Second):
			fmt.Println("Timeout! tidak ada pesan masuk")
		}
	}
}
```
Dari kode diatas maka tiap proses akan mendapatkan timeout karena batas timeout yang kita tentukan adalah 2 detik sedangkan kita mensimulasikan tiap proses berjalan selama 5 detik. Jika kode dijalankan outputnya adalah:
```bash
fahrudin@belajar-goroutine $ go run channel_select.go
Timeout! tidak ada pesan masuk
Timeout! tidak ada pesan masuk
```

### d. Infinite loop dengan goroutine
Infinite loop biasanya digunakan jika kita ingin menjalankan goroutine secara terus menerus di background untuk menghandle sesuatu.
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan string)

	// Goroutine dengan infinite loop
	go func() {
		for {
			msg := <-ch
			fmt.Println("Dapat:", msg)
		}
	}()

	// Kirim data ke goroutine tiap 1 detik
	for i := 1; i <= 5; i++ {
		ch <- fmt.Sprintf("Pesan %d", i)
		time.Sleep(1 * time.Second)
	}

	fmt.Println("Selesai")
}

```
Jika kode dijalankan maka outputnya:
```bash
fahrudin@belajar-goroutine $ 
Dapat: Pesan 1
Dapat: Pesan 2
Dapat: Pesan 3
Dapat: Pesan 4
Dapat: Pesan 5
Selesai
```

## Channel Range & Close

```go
package main

import (
	"fmt"
	"time"
)

func worker(id int, jobs <-chan int) {
	for job := range jobs {
		fmt.Printf("Worker %d memproses job %d di waktu %s \n", id, job, time.Now().String())
		time.Sleep(500 * time.Millisecond)
	}
	fmt.Printf("Worker %d selesai (channel ditutup)\n", id)
}

func main() {
	jobs := make(chan int)

	// jalankan 2 worker
	go worker(1, jobs)
	go worker(2, jobs)

	// kirim 5 job
	for j := 1; j <= 5; j++ {
		jobs <- j
	}
	close(jobs) // sinyal tidak ada job lagi

	time.Sleep(3 * time.Second)
	fmt.Println("Program selesai")
}

```

Jika kode dijalankan maka outputnya:
```bash
fahrudin@belajar-goroutine $ go run channel_range_close/channel_range_close.go 
Worker 2 memproses job 1 di waktu 2025-10-01 22:23:54.987473 +0700 WIB m=+0.000078459 
Worker 1 memproses job 2 di waktu 2025-10-01 22:23:54.987483 +0700 WIB m=+0.000088209 
Worker 1 memproses job 4 di waktu 2025-10-01 22:23:55.488685 +0700 WIB m=+0.501305251 
Worker 2 memproses job 3 di waktu 2025-10-01 22:23:55.488636 +0700 WIB m=+0.501256501 
Worker 2 memproses job 5 di waktu 2025-10-01 22:23:55.989838 +0700 WIB m=+1.002472918 
Worker 1 selesai (channel ditutup)
Worker 2 selesai (channel ditutup)
Program selesai
```


## Channel Timeout
## WaitGroup


## Referensi Tulisan
- https://dasarpemrogramangolang.novalagung.com/A-goroutine.html
- https://dasarpemrogramangolang.novalagung.com/A-channel.html
- https://medium.com/@jamal.kaksouri/goroutines-in-golang-understanding-and-implementing-concurrent-programming-in-go-600187bcfaa2
- https://buildwithangga.com/tips/penggunaan-channel-dalam-go-komunikasi-antar-go-routines
