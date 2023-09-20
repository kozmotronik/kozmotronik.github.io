---
layout:		post
title:		"Android İçin Düzey Göstergesi"
date:		2021-04-18 21:54:51 +0300
categories:	android java
tags:		android java
assets_path: /assets/images/android-duzey-gostergesi
---

Mobil uygulamalar geliştirirken bazı durumlarda kullanıcıya değişen düzey 
değerlerini uygulamamıza özel görseller kullanarak sunmamız gerekebilir. 
Bu, bir sürecin ilerleme düzeyi, bir pilin doluluğu, şebeke, wifi veya 
bluetooth gibi bir radyo sinyalinin gücü olabilir. Elbette bunu yapmanın birden 
çok yöntemi olabilir. Fakat burada gerçekleyeceğimiz yöntem Android 
ekosisteminin doğal gücünden yararlandığı için şu ana dek denediklerim arasında 
en verimli ve iyi bir görsel deneyim sağlayan yöntemdir. Bu yüzden bu yöntemi 
ileride böyle bir tasarım yapmak isteyecekler için paylaşıyorum.<!--more-->

Bu örneği kavrayabilmek için yeterli düzeyde Java bilgisi ve Android Studio 
deneyimine sahip olmanız gerekir. Yeni öğrenenler için uygun bir çalışma 
materyali değildir.

Bu örnekte bir pilin doluluk değerini gösterecek, doluluk değerini bir 
[`SeekBar`][1]{:target="_blank"} ile, ayrıca iki adet tuş ile dinamik olarak 
değiştirebilecek ve bu düzey değişimlerini düzey göstergemizde 
gözlemleyebileceğiz. Göstergemize düzey değerine göre değişim dinamiğini veren 
[`Drawable`][2]{:target="_blank"} sınıfıdır.  

Hazırsanız başlayalım. Tasarımımıza başlamak için yeni bir Android Studio 
projesi oluşturun, boş bir aktivite seçip programlama dilini Java seçin. 
Android Studio'nun dosyaları indekslemesi için biraz bekleyin. Hazırsa, 
adım adım düzey göstergesi için kullanacağımız çizimleri ekleyip kullanıma 
hazır hale getirelim.  

***
<br>
### 1. Arka Plan Çizimini Ekleme
Studio'nun sol yanındaki Project bölümünü açıp **res > drawable** dizini 
üzerine sağ tıklayın. Açılan içerik menüsünden **New > Vector Asset** 
seçeneğini seçin. Google'ın sağladığı vektörel çizimleri uygulamanıza 
ekleyebileceğiniz bir pencere gelecektir. Bu pencerede **Clip Art** yazısının 
karşısındaki simgeye tıklayıp çizim galerisini açın.

![Pil çizimi seçimi]({{page.assets_path}}/1-galeri_battery_sec.png)

**Görsel 1: Vektörel çizim seçimi**
 
Arama kutusuna *battery* yazın ve bulunanlardan *battery std* olanı seçin ve 
OK tuşuna basın. İsterseniz *battery full* olanı da seçebilirsiniz.

![Arka planı ekle]({{page.assets_path}}/2-arkaplan_ekle.png)
**Görsel 2: Göstergenin arkaplanının eklenmesi**

Bu çizimi arkaplan olarak kullanacağımız için *pil_arkaplan* olarak 
adlandırdım. **Opacity** yani saydamlığı %25'e ayarlayın ve NEXT'i tıklayıp 
eklemeyi bitirin.  

***
<br>
### 2. Ön Plan Çizimini Ekleme
İlk adımdaki ekleme işlemini tekrarlayın yalnız **saydamlığı değiştirmeyin**.
 
![Ön plan pil çizimini seç]({{page.assets_path}}/3-duzey_ekle.png)
**Görsel 3: Önplan çiziminin eklenmesi**

Bunu da *pil_duzey* olarak adlandırdım çünkü opak olan bu çizim düzey 
değerinin % (yüzde) olarak görsel temsili olacaktır. Bu çizimin rengini 
sistemin *colorPrimary* rengine ayarladım, siz istediğiniz soluk olmayan 
bir renk seçebilirsiniz. Bu noktada, bu çizimi doğrudan değil dolaylı olarak 
kullanacağımızı söylemekte fayda var. Nasıl ve neden olduğunu ilerleyen 
adımlarda açıklayacağım.  

***
<br>
### 3. Clip Drawable Oluşturma
Arka ve ön plan olarak kullanacağımız resimleri / çizimleri ekledikten sonra 
sıra geldi  önemli olanlardan birine. Bu adımda bir 
[`ClipDrawable`][3]{:target="_blank"} dosyası oluşturacağız. Clip drawable 
ön plandaki resmin düzey miktarı kadarının görülmesini sağlayan bir 
[`DrawableResource`][4]{:target="_blank"} nesnesidir. Bilmeyenler için; 
yaptığı işten anlayacağınız gibi *clip* sözcüğünün anlamı *kırpmak*tır.

![Clip drawable oluşturma]({{page.assets_path}}/4-duzey_clip_ekle.png)
**Görsel 4: Clip Drawable oluşturma**

*duzey_clip* olarak adlandırıp **Root element** olarak clip yazın ve ardından 
OK tuşuna basın. Ekledikten sonra dosyayı açın ve aşağıdaki kodları dosyaya 
girin.
```xml
<?xml version="1.0" encoding="utf-8"?>
<clip xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/pil_duzey"
    android:gravity="bottom"
    android:clipOrientation="vertical"/>
```

 Kodu kısaca açıklarsak:
 
- Kırpılacak çizim olarak `@drawable/pil_duzey` tanımladık.
- Çizimimiz dikey olduğu ve pilin dolumunu aşağıdan yukarıya olacak şekilde 
yapacağımız için `gravity` niteliğini `bottom` tanımladık.
- Kırpma yönelimi `clipOrientation`niteliğini de çizimimiz dikey olduğu için 
`vertical` tanımladık.

***
<br>
### 4. Katman Listesini Oluşturma
Artık gösterge için çizim kaynaklarını bir araya getirip bir katman listesi 
oluşturabiliriz. Bunun için `duzey_layer_list` adında bir 
[`LayerList`][5]{:target="_blank"} drawable dosyası oluşturacağız.

![Layer list oluşturma]({{page.assets_path}}/5-duzey_layer-list_ekle.png)
**Görsel 5: Layer List oluşturma**

Dosyayı ekledikten sonra açıp aşağıdaki kodu girin.
```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/pil_arkaplan"/>
    <item android:drawable="@drawable/duzey_clip"/>
</layer-list>
```
XML kodunda bir şeye dikkat ettiniz mi? İlk katman olarak doğrudan arka plan 
çizimini kullandık ancak ikinci katmanda ön plan çizimini doğrudan kullanmadık. 
Neden? Çünkü ön plan çiziminin yalnızca düzey değeri oranında görünmesini geri 
kalanının da kırpılmasını, yani görünmemesini istiyoruz. Bu yüzden üçüncü 
adımda oluşturduğumuz düzeye göre kırpma işini yapacak *duzey_clip*'i 
tanımladık.

***
<br>
### 5. Arayüz Tasarımı
Buraya kadar resource dosyalarını hazırlamayı tamamladık. Şimdi basit bir 
arayüz oluşturup düzey göstergemizi işlevsel hale getireceğiz. Bunun için 
Aktivitenizin layout dosyasını açın ve aşağıdaki kodu girin.
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".DuzeyGostergesiActivity">

    <View
        android:id="@+id/view_gosterge"
        android:layout_width="128dp"
        android:layout_height="128dp"
        android:layout_marginTop="32dp"
        android:background="@drawable/duzey_layer_list"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/textView_yuzde"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="%0"
        app:layout_constraintBottom_toBottomOf="@+id/view_gosterge"
        app:layout_constraintEnd_toEndOf="@+id/view_gosterge"
        app:layout_constraintStart_toStartOf="@+id/view_gosterge"
        app:layout_constraintTop_toTopOf="@+id/view_gosterge" />

    <TextView
        android:id="@+id/textView_bilgi"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:text="Pili doldur / boşalt"
        android:textAppearance="@style/TextAppearance.AppCompat.Body1"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/view_gosterge" />

    <SeekBar
        android:id="@+id/seekBar_ayar"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:max="100"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="1.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textView_bilgi" />

    <Button
        android:id="@+id/button_doldur"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Doldur"
        app:layout_constraintEnd_toEndOf="@+id/editText_miktar"
        app:layout_constraintHorizontal_bias="1.0"
        app:layout_constraintStart_toEndOf="@+id/button_bosalt"
        app:layout_constraintTop_toBottomOf="@+id/editText_miktar" />

    <Button
        android:id="@+id/button_bosalt"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Boşalt"
        app:layout_constraintStart_toStartOf="@+id/editText_miktar"
        app:layout_constraintTop_toBottomOf="@+id/editText_miktar" />

    <TextView
        android:id="@+id/textView_kademeBilgi"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:text="Artırma / azaltma değeri"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/seekBar_ayar" />

    <EditText
        android:id="@+id/editText_miktar"
        style="@android:style/Widget.Material.EditText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="5"
        android:selectAllOnFocus="true"
        android:singleLine="true"
        android:textAlignment="center"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textView_kademeBilgi" />
</androidx.constraintlayout.widget.ConstraintLayout>
```
Kodu girdikten sonra tasarım kipine geçin. Tasarımın yapısal olarak böyle 
görünmesi gerekiyor ama renkler farklı olabilir:

![Arayüz oluşturma]({{page.assets_path}}/6-tasarim_ekrani.png)

**Görsel 6: Uygulamanın tasarım ekranı**
 
Arayüzümüz kısaca; tasarladığımız düzey göstergesi, bir SeekBar, bir 
[`EditText`][6]{:target="_blank"}, açıklama içeren birkaç 
[`TextView`][7]{:target="_blank"} ve iki adet [`Button`][8]{:target="_blank"}dan 
oluşuyor. Tuşları ve kaydırma çubuğunu kullanarak dinamik olarak bir düzey 
değeri üretip bu değerin temsilini göstergemizde göstermeyi planlıyoruz.
`EditText` denetimini tuşların her bir basmada ne kadar doldurma veya boşaltma 
yapacağını belirlemek için kullanıyoruz.

***
<br>
### 6. Java Kodu
Son aşamamızda uygulamanın mantığını işleyecek kodları yazacağız. Aktivitenin 
kaynak kodu dosyasını açın ve aşağıdaki kodları girin:
```java
public class DuzeyGostergesiActivity extends AppCompatActivity {
    private static final String ETIKET = DuzeyGostergesiActivity.class.getSimpleName();

    View gosterge;
    TextView yuzde;
    SeekBar ayar;
    EditText editTextMiktar;
    Button doldur, bosalt;

    // Tuşlarla yapılacak artırma ve azaltma için miktar. EditText ile alınacak. Varsayılan 5.
    int miktar = 5;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_duzey_gostergesi);

        // UI denetim öğelerini ilkleyelim
        gosterge = findViewById(R.id.view_gosterge);
        yuzde = findViewById(R.id.textView_yuzde);
        ayar = findViewById(R.id.seekBar_ayar);
        editTextMiktar = findViewById(R.id.editText_miktar);
        doldur = findViewById(R.id.button_doldur);
        bosalt = findViewById(R.id.button_bosalt);

        /*
        SeekBar'ın ilerleme (progress) değişimini dinleyip pil göstergemiz üzerinde gereken
        güncellemeyi yapacağız.
         */
        ayar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                /**
                 * Bir {@link Drawable} nesnesinin düzeyi (level)
                 * 0 ile 10000 arasında bir değere kurulabilir. Fakat {@link SeekBar} aracının
                 * maksimum düzeyini okunabilirlik açısından 100 yaptık. Bu yüzden gelen değeri
                 * 10000 değerine ölçeklemek için 10000 / 100 = 100 ile çarpacağız.
                 */
                int duzey = progress * 100;
                String sYuzde = "%" + progress;
                gosterge.getBackground().setLevel(duzey);
                yuzde.setText(sYuzde);
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {

            }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {

            }
        });

        /*
        Burada artırma ve azaltma miktarını EditText yoluyla alacağız. Bunun için yazı değişimini
        bir TextWatcher sınıfı ile dinlememiz gerekir.
         */
        editTextMiktar.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {

            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                /*
                Integer sınıfının statik yordamı parseInt kullanarak String sayı girdisini int değere dönüştürüyoruz.
                Bu yordam geçersiz bir sayı stringi durumunda NumberFormatException hatası atabilir.
                Bu yüzden dönüştürme işlemini try-catch bloğu içinde yapacağız.
                 */
                try {
                    miktar = Integer.parseInt(s.toString(), 10); // Decimal radixte string girdiyi sayıya dönüştür
                } catch (NumberFormatException numberFormatException) {
                    // Girilen string verisinde sayı olarak değerlendirilecek bir girdi yok, uyarı ver
                    Snackbar.make(editTextMiktar, s.toString()+" geçerli bir sayı değil!", 1500).show();
                    miktar = 5; // hata durumunda miktarı varsayılan değere kur
                    numberFormatException.printStackTrace(); // Hatayı loga yazdır
                }
                Log.d(ETIKET,"miktar: "+miktar);
            }

            @Override
            public void afterTextChanged(Editable s) {

            }
        });

        // Tuşların görevlerini kuralım
        bosalt.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // Önce zaten boş olmadığından emin olmalıyız
                int duzey = ayar.getProgress() - miktar;
                if(duzey < 0) duzey = 0; // Sıfırın altına düştüyse sıfırda tut.
                // Seekbar progress değerini kurunca pil düzeyi seekbar onProgressChanged içinde güncellenir
                ayar.setProgress(duzey);
            }
        });

        doldur.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // Değerin 100 ü aşmadığına emin olmalıyız
                int duzey = ayar.getProgress() + miktar;
                if(duzey > 100) duzey = 100;
                ayar.setProgress(duzey);
            }
        });

    }
}
```
Kod içerisinde gerekli açıklamaları yaptım. Ancak uygulamamızın en önemli 
noktalarına burada da kısaca değineyim:
 
* Gösterge olarak kullanmak istediğimiz [`View`][9]{:target="_blank"} veya 
türevi nesnelerin `background` niteliğine 4. adımda hazırladığımız 
`duzey_layer_list` dosyasını tanımlıyoruz.
* Düzeyi değiştirmek istediğimizde Drawable sınıfının 
[`setLevel()`][10]{:target="_blank"} yordamını kullanıyoruz.
* Drawable sınıfında düzey (level) değeri *0 - 10.000* arasında bir değer 
almakta, *10.000* değeri *%100*'ü temsil etmektedir. O yüzden oldukça iyi düzey 
görüntüsü oluşturma hassasiyetine sahiptir.
* Bu uygulamada maksimum değerimiz *100* olduğu için; *10.000 / 100 = 100* 
hesabına göre, *0-100* arası elde ettiğimiz düzey değerini *100* ile çarpmamız 
gerekir. Bu işlemi kod içerisinde de görebilirsiniz.

***
<br>
Aşağıda uygulamanın çalışan bir demosunu görebilirsiniz.

[![Düzey göstergesi demo videosu][11]][12]{:target="_blank"}

**Video: Uygulamanın demosu**  
<br>
Uygulamanın github reposuna [buradan][13] ulaşabilir ve proje olarak 
indirebilirsiniz. Başka bir makalede görüşmek üzere herkese iyi çalışmalar.


[1]: https://developer.android.com/reference/android/widget/SeekBar
[2]: https://developer.android.com/reference/android/graphics/drawable/Drawable
[3]: https://developer.android.com/guide/topics/resources/drawable-resource.html#Clip
[4]: https://developer.android.com/guide/topics/resources/drawable-resource.html
[5]: https://developer.android.com/guide/topics/resources/drawable-resource.html#LayerList
[6]: https://developer.android.com/reference/android/widget/EditText
[7]: https://developer.android.com/reference/android/widget/TextView
[8]: https://developer.android.com/reference/android/widget/Button
[9]: https://developer.android.com/reference/android/view/View
[10]: https://developer.android.com/reference/android/graphics/drawable/Drawable#setLevel\(int\)
[11]: http://img.youtube.com/vi/dux1HbJaDF0/0.jpg
[12]: https://youtube.com/shorts/dux1HbJaDF0?si=URSt78EuUS_HHyz8
[13]: https://github.com/kozmotronik/DuzeyGostergesi
