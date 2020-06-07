---
path: "/2019-01-11"
date: "2019-01-11T10:57:02.804Z"
title: Menikmati Powershell
editor: typora with gitlab theme
---

# Menikmati Powershell

[TOC]

Minggu lalu gue ada kerjaan, task, ya kerjaan deh, yang butuh `script` buat ngotomatisasi penghapusan *file-file* *gaguns* (baca: gak guna) dari *Production server*


Dari dulu sebenernya gue rada males bikin `script-script-an` pake `Batch` atau `Powershell`
Karena gue kalo mau apa-apa pasti pake `C#`. Udahlah cinta mati gueh lah itu. Etapi kan ya, kalo mau nyiapin *environment*-nya lagi, di *Production* pula, keknya bakalan ditolak *Infra*
Jadi pada akhirnya yaa balik lagi kalo gak `Batch` atau `Powershell`. Sebenernya ada satu lagi sih, `WSH` (`Windows Scripting Host`). Cuma saia rada *reluctant* karena semacam *unsupported* lagi karena pake `VBScript` dan `JScript`


Pilihan-pilihan di atas yang akhirnya membuat gue ngerasa udahlah, udah saatnya pake `Powershell` mungkin. Dan gue pun meriset (baca: meng-*google* / mem-*bing*) beberapa *command* `Powershell`
Naah, ternyata si `Powershell` ini tuh memang di-*build on top of `.Net`*. Jadi dia bisa jalanin `C#`, dia bisa pake `dll .Net` juga
Wow?!?! baru tau saia. 😄


Kalo gue belajar bahasa pemrograman baru. Apapun itu, pasti mulainya dari manipulasi `Array`

Pertama-tama, nyiapin data *dummy* nih. Ngambil dari *[Online generator](http://www.convertcsv.com/generate-test-data.htm)* ini.
Udah gitu masukin ke `$data`. Terus tampilin ke layar dengan manggil `$data` lagi


```powershell
 $data = @(
     "Clifford", "Lewis", "Ollie", "Leah", "Kathryn", "Carolyn",
     "Genevieve", "Adam", "Milton", "Eleanor", "Maurice", "Ethel",
     "Charles", "Danny", "Stephen", "Gabriel", "Susan", "Donald",
     "Isabella", "Patrick"
 )

 $data
```


Btw, karena bakalan di-*run* di *Server*, jadinya gue *save* ke `Script.ps1`. Terus *run*!!


Nanti jadi gini.


```powershell
 Clifford
 Lewis
 Ollie
 Leah
 Kathryn
 Carolyn
 Genevieve
 Adam
 Milton
 Eleanor
 Maurice
 Ethel
 Charles
 Danny
 Stephen
 Gabriel
 Susan
 Donald
 Isabella
 Patrick
```


Sedikit review nih. Operasi `Array` itu umumnya ada 4. Yaitu adalah jeng-jeng-jeng-jeng!!!!! `Map`, `Filter`, `Sort`, `Aggregate/Reduce`
Kita mulai dari yang pertama dulu yaa.
​


#### 1. Map


`Map` di `Powershell` mirip kaya yang ada di `.Net`. *Keyword*-nya `Select-Object`
Sedangkan kalo `.Net` pake `.Select()`
​
```powershell
 $counter = 1;
 $mapped = $data |
     Select-Object { ("{0}. {1}" -f $counter++,$_) }

 $mapped
```


Hasilnya kira-kira kaya di bawah ini nih.

```powershell
 ("{0}. {1}" -f $counter++,$_)
 -------------------------------
 1. Clifford
 1. Lewis
 1. Ollie
 1. Leah
 1. Kathryn
 1. Carolyn
 1. Genevieve
 1. Adam
 1. Milton
 1. Eleanor
 1. Maurice
 1. Ethel
 1. Charles
 1. Danny
 1. Stephen
 1. Gabriel
 1. Susan
 1. Donald
 1. Isabella
 1. Patrick
```


Ada yang aneh gak? Perhatiin gak angkanya? 1 semua kan?


Ternyata itu karena di `Powershell` ada *scope* di *scripting*-nya. Yang mana `$counter` di dalem *block* `Select-Object` sewaktu di-`++` gakkan pengaruh ke `$counter` yang di luar
Karena pada dasarnya `Select-Object` ini adalah *function* yang mana *function* punya *scope* sendiri
Bisa dibaca [disini](https://ss64.com/ps/syntax-scopes.html) ya.


Nah, karena kita pake *file* buat ngejalanin `script`-nya jadi kit pake *scope* `$script:`


Jadi gini nih.


```powershell
 $counter = 1;
 $mapped = $data |
     Select-Object { ("{0}. {1}" -f $script:counter++,$_) }

 $mapped
```


Hasilnya jadinya bener kaya di bawah ini.


```powershell
 ("{0}. {1}" -f $script:counter++,$_)
 --------------------------------------
 1. Clifford
 2. Lewis
 3. Ollie
 4. Leah
 5. Kathryn
 6. Carolyn
 7. Genevieve
 8. Adam
 9. Milton
 10. Eleanor
 11. Maurice
 12. Ethel
 13. Charles
 14. Danny
 15. Stephen
 16. Gabriel
 17. Susan
 18. Donald
 19. Isabella
 20. Patrick
```



#### 2. Filter


Dan lagi-lagi `Filter` di `Powershell` mirip kaya yang ada di `.Net`
Kalo di `.Net` itu `.Where()`. Kalo di `Powershell`-nya itu `Where-Object`


```powershell
 $counter = 1;
 $mapped = $data |
     Where-Object { $_.ToLower().Contains("an") } |
     Select-Object { ("{0}. {1}" -f $script:counter++,$_) }

 $mapped
```


*Output*-nya gini.


```powershell
("{0}. {1}" -f $script:counter++,$_)
--------------------------------------
1. Eleanor
2. Danny
3. Susan
```


Nyadar gak kalo di atas gue pake `.ToLower()` sama `.Contains()`-nya `.Net`
Yuhuuu. Cadas ye gak? 👍



#### 3. Sort


Nah, kali ini `Sort` di `Powershell` beda sama yang ada di `.Net`
`.Net` punya = `.OrderBy()` atau `.OrderByDescending()`. Sedangkan `Powershell` punya = `Sort-Object` atau `Sort-Object -Descending`
Contohnya di bawah ini.


```powershell
 $counter = 1;
 $mapped = $data |
     Where-Object { $_.ToLower().StartsWith("le") -Or \`
        $_.ToLower().EndsWith("el") -Or \`
        ($_[0].ToString().ToLower() -Eq "d") } |
     Sort-Object -Descending { $_ } |
     Select-Object { ("{0}. {1}" -f $script:counter++,$_) }

 $mapped
```


Yang mana meng-*output*-kan inih.


```powershell
 ("{0}. {1}" -f $script:counter++,$_)
 --------------------------------------
 1. Lewis
 2. Leah
 3. Gabriel
 4. Ethel
 5. Donald
 6. Danny
```


Setelah beberapa kali *output*, liat gak *header*-nya? `("{0}. {1}" -f $script:counter++,$_)` kan


Itu karena `Select-Object` sejatinya memang untuk `Object`. Apapun yang di *output*-in `Select-Object` pasti `type`-nya `Object`
Apa hubungannya sama *header* yang *suneh* (baca: suka aneh) gitu? Itu karena `Select-Object`-nya *projecting anonymous object*
Lalu gimana biar gak *anonymous*? Kita bisa pake yang namanya *computed property* atau *calculated property*.
Contohnya gini.


```powershell
 $counter = 1;
 $mapped = $data |
     Where-Object { $_.ToLower().StartsWith("le") -Or \`
        $_.ToLower().EndsWith("el") -Or \`
        ($_[0].ToString().ToLower() -Eq "d") } |
     Sort-Object -Descending { $_ } |
     Select-Object @{ Name = "Mapped";Expression = {"{0}. {1}" -f $script:counter++,$_} }

 $mapped
```


Nanti hasilnya jadi gini.


```powershell
 Mapped
 --------------------------------------
 1. Lewis
 2. Leah
 3. Gabriel
 4. Ethel
 5. Donald
 6. Danny
```



#### 4. Aggregate / Reduce


`Reduce` di `Powershell` rada beda ya. Soalnya dia pake keyword `ForEach-Object`. Padahal kan ya `foreach` itukan buat `iterate / looping` pada umumnya
Tapi di `Powershell` sendiri ada `ForEarch-Object` ada `ForEach` *statement*. Gue gakkan bahas disini biar bahasannya gak meluas


Nah, `ForEach-Object` punya `Begin`, `Process`, dan `End`. 3 fitur ini yang bisa dipake buat melakukan `Reduce`


Karena ini fungsi terakhir jadi gue *combine* aja semua fungsi-fungsi di atas. *Here goes!*


```powershell
 $counter = 1;
 $mapped = $data |
     Sort-Object -Descending { $_ } |
     Select-Object \`
         @{ Name = "Index";Expression = {($script:counter++)} }, \`
         @{ Name = "Name";Expression = {$_} }

 $even = $mapped |
     Where-Object { $_.Index % 2 -Eq 0 } |
     ForEach-Object \`
         -Begin { $start = "" } \`
         -Process { $start = $start + $_.Index + ": " + $_.Name + ", " } \`
         -End { $start.Substring(0, $start.Length -2) }

 $odd = $mapped |
     Where-Object { $_.Index % 2 -Ne 0 } |
     ForEach-Object \`
         -Begin { $start = "" } \`
         -Process { $start = $start + $_.Index + ": " + $_.Name + ", " } \`
         -End { $start.Substring(0, $start.Length -2) }

 $even
 $odd
```


Menghasilkan iniih.


```powershell
 2: Stephen, 4: Ollie, 6: Maurice, 8: Leah, 10: Isabella, 12: Gabriel, 14: Eleanor, 16: Danny, 18: Charles, 20: Adam
 1: Susan, 3: Patrick, 5: Milton, 7: Lewis, 9: Kathryn, 11: Genevieve, 13: Ethel, 15: Donald, 17: Clifford, 19: Carolyn
```


Wuaaahhhh, gue gak nyangka ternyata `Powershell` se-*mancay* iniihh. Kadang hal-hal kecil semacam ini yang ngasi motivasi buat *explore* lebih jauh lagi
