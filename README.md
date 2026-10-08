Video Game Sales Dataset — Data Cleaning
Bu proje, video oyun satış verilerinden oluşan bir dataset üzerinde veri temizleme ve keşifsel veri analizi çalışmalarını kapsamaktadır.

Çalışmanın temel amacı, modelleme veya analiz aşamasına geçmeden önce dataset içerisindeki eksik değerleri, tekrar eden kayıtları ve tutarsızlıkları incelemek ve mümkün olduğunca güvenilir yöntemlerle temizlemektir.

Not: Bu çalışma adım adım ilerlemektedir. Veri temizleme sırasında değerleri doğrudan doldurmak yerine, her kararın veri üzerindeki etkisi incelenmektedir.
Dataset
Dataset toplam 16.598 satır ve başlangıçta 11 sütundan oluşmaktadır.

Sütun Açıklama
Rank Oyunun satış sıralaması
Name Oyun adı
Platform Oyunun platformu
Year Oyunun çıkış yılı
Genre Oyun türü
Publisher Oyunun yayıncısı
NA_Sales Kuzey Amerika satışları
EU_Sales Avrupa satışları
JP_Sales Japonya satışları
Other_Sales Diğer bölgelerdeki satışlar
Global_Sales Global satışlar
Satış sütunları milyon adet satış cinsindendir.

1. Dataset'in İlk İncelenmesi
   İlk olarak dataset'in yapısı ve boyutu incelendi.

df.shape
Sonuç:

(16598, 12)
Çalışma sırasında oluşturulan geçici None_Year sütunu daha sonra gereksiz olduğu için datasetten çıkarıldı.

Son durumda dataset:

16598 satır
11 sütun
halindedir.

2. Veri Tiplerinin ve Eksik Değerlerin İncelenmesi
   Dataset'in genel yapısını görmek için:

df.info()
kullanıldı.

Başlangıçta eksik değerler:

Sütun Eksik Değer
Year 271
Publisher 58
Diğer sütunlar 0
Eksik değerlerin genel dağılımı:

df.isna().sum()
ile kontrol edildi.

3. Eksik Year Değerlerinin İncelenmesi
   Öncelikle Year sütununda eksik olan kayıtlar incelendi.

df.loc[df["Year"].isna(), ["Name", "Year"]]
Toplam 271 adet eksik Year değeri olduğu görüldü.

3.1. Ortalama ve Medyanın İncelenmesi
Eksik yılları doğrudan doldurmadan önce Year sütununun merkezi eğilimleri incelendi.

Ortalama:

df["Year"].mean()
Yaklaşık:

2006
Medyan:

df["Year"].median()
Sonuç:

2007
Ancak yıl değişkeni bir zaman bilgisi olduğu için tüm eksik değerleri ortalama veya medyan ile doldurmanın veri açısından doğru olmayabileceği değerlendirildi.

Bu nedenle önce oyun isimlerinden eksik yıl bilgisini çıkarıp çıkaramayacağımız araştırıldı.

4. Oyun İsimlerinde Yıl Bilgisi Aranması
   Year değeri eksik olan oyunların Name sütunları incelendi.

İsim içerisinde rakam bulunan oyunları bulmak için:

df.loc[
(df["Year"].isna()) &
(df["Name"].str.contains("\\d", regex=True)),
["Name", "Platform"]
]
kullanıldı.

Daha sonra özellikle 4 basamaklı yıl ifadeleri araştırıldı.

Örneğin:

Madden NFL 2004
FIFA Soccer 2004
wwe Smackdown vs. Raw 2006
NASCAR Thunder 2003
PES 2009: Pro Evolution Soccer
Madden NFL 2002
NFL GameDay 2003
NBA Live 2003
Tomb Raider (2013)
All-Star Baseball 2005
Tour de France 2011
Football Manager 2007
PDC World Championship Darts 2008
gibi kayıtlar bulundu.

Regex kullanılarak 4 basamaklı yıl değerleri çıkarıldı:

nameList.str.extract("(\\d\\d\\d\\d)")
Bu yöntemle güvenilir olduğu düşünülen 17 adet Year değeri oyun isimlerinden elde edildi.

Sonuç olarak:

Year eksikliği:
271 → 254
seviyesine düşürüldü.

5. Publisher Eksiklerinin İncelenmesi
   Başlangıçta Publisher sütununda 58 eksik değer bulunuyordu.

Eksik Publisher kayıtlarını incelemek için:

df.loc[
df["Publisher"].isna(),
["Name", "Platform", "Genre"]
]
kullanıldı.

Bu incelemede eksik Publisher değerlerinin farklı platform ve oyun türlerine yayıldığı görüldü.

Bu nedenle eksik Publisher değerlerini doğrudan mode() ile doldurmak yerine daha kontrollü bir yöntem kullanılmasına karar verildi.

6. Duplicate Kayıtların İncelenmesi
   Veri temizleme sırasında duplicate kayıtların modelleme açısından problem oluşturabileceği değerlendirildi.

Öncelikle bütün sütunları tamamen aynı olan kayıtları bulmak için:

df.duplicated(keep=False)
kullanıldı.

Burada keep=False kullanılmasının nedeni, duplicate grubundaki ilk kayıt dahil tüm tekrarları görmek istememizdir.

Ayrıca:

df.duplicated(subset=["Name", "Platform"], keep=False)
ile yalnızca Name + Platform kombinasyonlarının tekrar edip etmediği incelendi.

Bu iki yaklaşımın birbirinden farklı olduğu görüldü:

Bütün sütunların aynı olması → gerçek anlamda tamamen aynı kayıt
Name + Platform değerlerinin aynı olması → aynı oyun-platform kombinasyonu, fakat diğer bilgiler farklı olabilir
Bu nedenle duplicate kayıtların otomatik olarak silinmemesine ve önce veri yapısının incelenmesine karar verildi.

7. Name + Platform Üzerinden Publisher Araştırması
   Eksik Publisher değerlerini doldurmak için daha güvenilir bir yöntem araştırıldı.

Öncelikle dataset:

df.groupby(["Name", "Platform"])["Publisher"].count()
ile Name + Platform gruplarına ayrıldı.

Burada count() fonksiyonunun NaN değerleri saymadığı öğrenildi.

Örneğin:

Sonic the Hedgehog | PS3 | Sega
Sonic the Hedgehog | PS3 | NaN
grubunda:

Publisher count = 1
olmaktadır.

Bu durum, eksik Publisher'ın aynı gruptaki mevcut Publisher bilgisi kullanılarak doldurulabileceğine dair güçlü bir kanıt sağlamaktadır.

8. transform() Kullanımı
   groupby() sonucunun doğrudan DataFrame satırlarıyla kullanılabilmesi için transform() yöntemi incelendi.

Örneğin:

df.groupby(
["Name", "Platform"]
)["Publisher"].transform("count")
her satıra kendi grubundaki dolu Publisher sayısını aktarır.

Benzer şekilde:

df.groupby(
["Name", "Platform"]
)["Publisher"].transform("first")
her satıra kendi grubundaki ilk dolu Publisher değerini aktarır.

Bu yaklaşım, grup seviyesindeki bilgiyi tekrar orijinal DataFrame'in satır seviyesine taşımamızı sağlar.

9. Güvenli Publisher Doldurma Kuralı
   Publisher doldururken şu kural oluşturuldu:

Publisher değeri eksik olacak ve aynı Name + Platform grubunda tam olarak 1 adet dolu Publisher bulunacak.
Bu koşul:

mask = (
(df.groupby(["Name", "Platform"])["Publisher"].transform("count") == 1)
& (df["Publisher"].isna())
)
ile oluşturuldu.

Ardından bu maskeye uyan kayıtların Publisher değerleri:

df.loc[mask, "Publisher"] = df.groupby(
["Name", "Platform"]
)["Publisher"].transform("first")
kullanılarak dolduruldu.

10. Publisher Sonucu
    İşlem öncesinde:

Publisher eksik = 58
İşlem sonrasında:

df["Publisher"].isna().sum()
sonucu:

57
oldu.

Yani bu yöntemle:

58 → 57
şeklinde 1 adet Publisher değeri güvenli şekilde dolduruldu.

Bu kayıt, Sonic the Hedgehog + PS3 örneğiydi.

11. Neden Tüm Eksik Publisher'lar Doldurulmadı?
    Amaç eksik değerleri mümkün olduğunca çok doldurmak değil, güvenilir bilgiyle doldurmak olarak belirlendi.

Örneğin şu durumda:

Game A | PS2 | Sega
Game A | PS2 | Nintendo
Game A | PS2 | NaN
first() kullanılarak doğrudan Sega seçmek doğru olmayabilir.

Çünkü aynı grup içerisinde farklı Publisher bilgileri bulunmaktadır.

Bu nedenle yalnızca:

Dolu Publisher sayısı = 1
olan gruplar güvenli kabul edildi.

12. Şu Anki Durum
    Şu anda:

## Year

Başlangıç eksik: 271
İsimlerden güvenilir şekilde doldurulan: 17
Kalan eksik: 254
ve:

## Publisher

Başlangıç eksik: 58
Name + Platform yöntemiyle doldurulan: 1
Kalan eksik: 57
durumundayız.

13. Bir Sonraki Adım
    Bir sonraki aşamada kalan 57 eksik Publisher incelenecek.

İlk olarak:

df.loc[
df["Publisher"].isna(),
["Name", "Platform", "Year", "Genre"]
]
ile kalan kayıtlar tekrar incelenecek.

Ardından şu soru araştırılacak:

Eksik Publisher'a sahip bir oyunun Name değeri, başka platformlarda Publisher bilgisiyle mevcut mu?
Örneğin:

Game A | PS2 | NaN
Game A | X360 | Sega
Game A | Wii | Sega
gibi bir yapı varsa, yalnızca Name üzerinden Publisher bilgisinin güvenilir şekilde çıkarılıp çıkarılamayacağı araştırılacak.

Burada özellikle Publisher bilgilerinin tutarlı olup olmadığı kontrol edilecek.

Öğrenilen Pandas Kavramları
Bu çalışma sırasında aşağıdaki Pandas kavramları uygulamalı olarak öğrenildi:

df.info()
df.shape
df.isna().sum()
df.loc[]
Boolean mask
str.contains()
Regex
str.extract()
duplicated()
groupby()
count()
first()
transform()
fillna()
MultiIndex mantığı
& ile birden fazla koşul oluşturma
Önemli prensip:

Veri temizleme işlemlerinde mümkün olduğunca veriyi değiştirmeden önce problemin kaynağı araştırılmalı ve doldurma/silme kararları kanıta dayalı verilmelidir.
Projenin Sonraki Aşamaları
Kalan 57 Publisher eksikliğini incelemek
Name bazında Publisher bilgisi araştırmak
Çelişkili Publisher kayıtlarını tespit etmek
Kalan eksik Publisher değerleri için uygun strateji belirlemek
Kalan Year eksikliklerini değerlendirmek
Duplicate kayıtları detaylı incelemek
Veri tiplerini ve kategorik değişkenleri kontrol etmek
Aykırı değerleri incelemek
Modelleme öncesi veri sızıntısı (data leakage) risklerini kontrol etmek
Train/test ayrımını doğru şekilde yapmak
