---
layout:		post
title:		"C scanf() ile Karakter Okuma Sorunu"
date:		2021-04-21 23:19:15 +0300
categories:	C scanf
tags:		C scanf
---
Eğer bu makaleyi buldunuz ve okuyorsanız yüksek ihtimalle siz de aynı sorundan 
müzdaripsiniz. C ile bir alıştırma veya uygulama yapıyorsunuz, klavyeden birkaç 
kez karakter veya string okumanız gerekiyor, aa, bir bakıyorsunuz ki arada bazı 
girişleri okuyamamışsınız, değişkenlerinizde yalnızca istenmeyen bir `\n` 
karakteri var. Ne olacak şimdi? Gelin örnek bir kodla sorunu irdeleyelim:
<!--more-->
```c
#include <stdio.h>

int main() {   
	char karakter0;
	char karakter1;
	char karakter2;
	char karakter3;
	
	printf("Karakter0: ");
	scanf("%c", &karakter0);
	
	printf("Karakter1: ");
	scanf("%c", &karakter1);
	
	printf("Karakter2: ");
	scanf("%c", &karakter2);
	
	printf("Karakter3: ");
	scanf("%c", &karakter3);
	
	putchar('\n');
	
	printf("Girilen karakterler:\n");
	printf("Karakter0: %c\n", karakter0);
	printf("Karakter1: %c\n", karakter1);
	printf("Karakter2: %c\n", karakter2);
	printf("Karakter3: %c\n", karakter3);
	
	return 0;
}
```

Kodun çıktısı tam olarak şöyle:
> Karakter0: a  
Karakter1: Karakter2: b  
Karakter3:  
Girilen karakterler:  
Karakter0: a  
Karakter1:
>    
Karakter2: b  
Karakter3:  
>  
>  .


Çıktıya baktığımızda ilk girdinin okunduğunu fakat ikinci girdinin sanki 
okunmayıp atlanmış gibi bir havası olduğunu gözlüyoruz. Aynı durum aynı biçimde 
üçüncü ve dördüncü okumalar için de geçerli. Özellikle girilen karakterler 
yazıdırıldığında `Karakter1:` ve `Karakter3:`'ten sonraki boş satırlara dikkat 
edin. Bu boşlukların olmasının bir nedeni var. Çıktıda görünenin özeti şöyle 
yorumlanabilir:

1. scanf Karakter0'ı okudu
2. scanf Karakter1'i atladı
3. scanf Karakter2'yi okudu
4. scanf Karakter3'ü atladı

Öyleyse `scanf` işlevinde bir bug var, hatalı çalışıyor. Peki gerçekte 
böyle mi? Hayır, böyle değil. `scanf` işlevinde hiçbir sorun yok, tıkır tıkır 
çalışıyor. Bu arada söylemek gerekir ki bu sorun *%d* ve *%f* gibi sayısal 
biçimlendiriciler kullanıldığında oluşmuyor, yalnızca *%c* ve *%s* 
biçimlendiricileri kullanılırken oluşuyor (bildiğim kadarıyla). Neden böyle 
olduğunu anlatmadan önce `scanf` işlevinin klavye (veya başka bir girdi aygıtı) 
girdilerini biçimlendirmeye göre nasıl okuduğuna kısaca bakalım.

Klavyeden bir tuş girişi yapıldığında, bu tuş olaylarını tüketen bir uygulama 
veya işlev çalışana dek bu tuşlar geçici bir tampon bellek bölgesinde tutulur. 
`scanf` işlevi bir tuş olayı tüketici işlevdir. Bu işlevlere örnek olarak 
getchar, getch, gets vb. işlevler verilebilir. Kod örneğimiz üzerinden `scanf` 
ile okuma yaparken bu tamponda neler olduğuna adım adım bakalım.

1. `Karakter0: ` ekrana yazdırıldı
2. Tampon <u>boş olduğundan</u> `scanf` bir tuş girdisi beklemeye geçti
3. 'a' karakteri girildi ve `\n` (enter) tuşuna basıldı
4. Bu aşamada tamponda 'a' ve `\n` karakterleri var
5. `scanf` *%c* biçimlendiricisi ile tamponu okuduğundan yalnızca 'a' 
karakterini tüketti ve `karakter0` değişkenine sakladı, `\n` karakteri ise 
tüketilmediği için tamponda kaldı
6. `Karakter1: ` ekrana yazdırıldı
7. İkinci `scanf` çalışınca onun da biçimlendiricisi *%c* olduğundan ve `\n` 
girdisi de sonuçta bir ASC2 karakter olduğundan tampondaki bu karakteri 
tüketip `karakter1` değişkenine sakladı ve tampon yine boşaldı ve işte bu 
aşamada biz `scanf` işlevinin düzgün çalışmadığını düşündük
8. `Karakter2: ` ekrana yazdırıldı
9. Üçüncü `scanf` çalıştırılıdı fakat tampon boş olduğu için beklemeye geçti
10. 'b' karakteri girildi ve `\n` tuşuna basıldı
11. Üçüncü `scanf` 'b' karaketini tüketti ve yine `\n` karakteri tamponda kaldı
12. Dördüncü `scanf` çalıştırıldı ve tamponda kalan `\n` karakterini tüketip 
`karakter3` değişkenine sakladı

Aslında bu durum yalnızca `scanf` işlevine has değil, aynı durum girdi 
tamponundan karakter veya string okuyan `getchar`, `gets` vb. işlevler için de 
geçerlidir. Peki karakter okurken bu sorun oluşuyor da sayısal biçimli olarak 
okurken neden olmuyor? Çünkü `%d` gibi sayısal biçimlendiriciler `\n` ve `\s` 
gibi boşluk karakterlerini otomatikmen tüketiyorlar. Dolayısıyla `scanf` veya 
benzerleriyle ne kadar arka arkaya okuma yapılırsa yapılsın karakter 
biçimlendiricilerde oluşan sorun oluşmuyor.

Güzel, artık bu garip davranışın nedenini anladık. Ancak anlamak yeterli değil,
bunun üstesinden gelmek için ne yapmalıyız? Aslında bu sorunun yanıtı 
yukarıdaki açıklamalarda zaten var. Neydi sorunumuz? `\n` karakterinin tamponda 
kalması ve bir sonraki tüketici tarafından tüketilmesi. Öyleyse ya bu karakteri 
sonraki tüketiciyi çalıştırmadan önce manuel olarak tüketeceğiz, ya da bir 
şekilde bir biçimlendirme numarasıyla boşluk karakterlerinin görmezden 
gelinmesini sağlayacağız.

## Çözüm
Çözüm kodumuzu yazıp çalıştırıp çıktısına bakalım. Sonrasında çözümü 
yöntemlerini irdeleyeceğiz.
```c
#include <stdio.h>

int main() {   
	char karakter0;
	char karakter1;
	char karakter2;
	char karakter3;
	
	printf("Karakter0: ");
	scanf("%c", &karakter0);
	
	while ((getchar()) != '\n'); // Çözüm 1
	printf("Karakter1: ");
	scanf("%c", &karakter1);
	
	printf("Karakter2: ");
	scanf(" %c", &karakter2); // Çözüm 2
	
	printf("Karakter3: ");
	scanf(" %c", &karakter3);
	
	putchar('\n');
	
	printf("Girilen karakterler:\n");
	printf("Karakter0: %c\n", karakter0);
	printf("Karakter1: %c\n", karakter1);
	printf("Karakter2: %c\n", karakter2);
	printf("Karakter3: %c\n", karakter3);
	
	return 0;
}

```

Çözüm uygulanan kodun çıktısı:
> Karakter0: a  
Karakter1: b  
Karakter2: c  
Karakter3: d  
>
Girilen karakterler:  
Karakter0: a  
Karakter1: b  
Karakter2: c  
Karakter3: d  

Çıktı kodu incelenirse istediğimizi elde ettik. Dört karakteri de hiç atlama 
olmadan değişkenlerine atadık. Çözüm kodunda iki ayrı çözüm yöntemi görüyorsunuz. 
Aslında ikisi de aynı işi yapıyor; istenmeyen, artık `\n` karakterlerini 
tüketiyorlar.
```c
while ((getchar()) != '\n'); // Çözüm 1
```
İlk çözümde, `\n` karakterlerinin hepsi tüketilene kadar program döngüde kalıyor.
```c
scanf(" %c", &karakter2); // Çözüm 2
```
İkinci çözümde ise tüketme bizzat scanf biçimlendirme parametresinin içinde 
yapılıyor. Dikkat ettiyseniz %c notasyonundan önce bir boşluk var. Bu boşluk 
tamponda kalan bir önceki boşluk karakterini tüketip asıl almak istediğimiz 
karakteri, karakter2 içerisine alıp onu da tüketmiş olacaktır.

Ancak unutmayın ki istediğimiz karakteri yazıp ardından enter tuşuna 
bastığımızda yine tamponda bir `\n` karakteri tüketilmeyi bekliyor olacaktır. 
Dolayısıyla sonraki stdin tamponundan okuma işlevlerini kullanırken bu durumu 
işlemek yine biz programcıların sorumluluğundadır. Sizin de bu konuda 
bildiğiniz bir çözüm veya öneriniz varsa yorum bölümünden paylaşabilirsiniz. 
Başka bir konuda görüşmek dileğiyle, herkese mutlu kodlamalar.
