# week-1
1.HAFTA ÖDEV LİNKLERİ

https://sqlzoo.net/wiki/SQL_Tutorial

https://selectstarsql.com

https://sqlzoo.net/wiki/Musicians

https://www.kaggle.com/learn/intro-to-sql  


# week-2
2.HAFTA ÖDEV LİNKLERİ

### Soru 1) 1980’den itibaren spor grubu bazında en çok madalya alan 1. 3. 5. ülkeyi bulalım.
### Soru 2) 1980’den itibaren herhangi bir spor grubunda üst üste 3 veya daha fazla madalya almış atletleri bulalım.

Aşağıdaki şekilde

create or replace table `dsmbootcamp.DATASET_ADINIZ.summer_medals`

as

select * from `dsmbootcamp.sample.summer_medals`; 

DATASET_ADINIZ kısmına kendi dataset adınızı yazarak tabloyu create ettikten sonra, soru çözümünüzde bu yarattığınız tabloyu kullanabilirsiniz.

### Soru 3) "sample.pageview": tablosunda 1 gün içerisinde trendyol.com a gelen tüm ziyaretlerin logu var.
--view_ts: ziyaret zamanı

--channel: android,ios,web.

--pagetype: görüntülenen sayfa: homepage, order, boutiquedetail, productdetail.. gibi.

--deviceid: sayfa ziyareti yapan cihaz id'si. Bizim için tekil kullanıcıyı da ifade eder.

tabloya veri akışı dakikada 1 defa gerçekleşir, 23:10:00 anında 23:09:00-23:09:59 kayıtları tabloya eklenir.

### Bu çalışmada çıkarmak istediğimiz bilgi, günün her bir dakikası için aktif kullanıcı sayısının hesaplanması.

Aktif kullanıcı ne demek?

sitede herhangi bir sayfa ziyareti sonrasında 5dk boyunca aktif kullanıcı sayılır.

bir örnek ile:  "2020-03-03 23:10:14" anınında 100farklı cihaz trendyol'u açıp kapattı ise ve sonrasında hiç bir ziyaret gelmedi ise.

--"2020-03-03 23:10" 100 aktif kullanıcı vardır.

--"2020-03-03 23:11" 100 aktif kullanıcı vardır.

--"2020-03-03 23:12" 100 aktif kullanıcı vardır.

--"2020-03-03 23:13" 100 aktif kullanıcı vardır.

--"2020-03-03 23:14" 100 aktif kullanıcı vardır.

--"2020-03-03 23:15" 0 aktif kullanıcı vardır.

#### en basit çözüm ile sadece "2020-03-03 23:14" 'deki aktif kullanıcıları hesaplamak için:

--select timestamp '2020-03-03 23:14:00' view_period

 --   ,count(distinct deviceid) active_user_count
      
-- from sample.pageview
 
--where timestamp_trunc(view_ts,minute) between '2020-03-03 23:10:00' and '2020-03-03 23:14:00'

#### Yazacağınız sorgu/sorguların çıktısında beklediğimiz çıktı sitedeki dakikalık aktif kullanıcı sayısı:

-- view_period            active_user_count

-- 2020-03-03 23:14:00            123123123

-- 2020-03-03 23:13:00            125123127

-- 2020-03-03 23:12:00            126123124

### NOT: Çözümde göz önünde bulundurmanızı istediğimiz koşullar:

- exact sonuç aramıyoruz, %2'ye kadar sapmalar kabul edilebilir, approx fonksiyonları kullanabilirsiniz

- örnek tabloda 289m kayıt var, bu aylar öncesinin verisi, optimizasyonlar için summary, ara tablo oluşturabilirsiniz.

- tek sorguda hesaplayabilirsiniz, temporary tablolar oluşturup, 1den fazla sorgu ile de çözebilirsiniz.

- hll_count.init, hll_count.merge, hll_count.merge_partial, hll_count.extract kullanabilirsiniz.

- https://cloud.google.com/bigquery/docs/reference/standard-sql/hll_functions

## soru 4)
Product database'indeki public.content_category tablosunun dwh ortamına `sample.content_category` isminde her gün 1 defa kopyalandığını/extract edildiğini varsayalım. 

`sample.content_category` tablosunda satışa çıkan productların id'leri ve category bilgileri yer almaktadır.
​
*Tablodaki kolonların açıklamaları aşağıdaki gibidir.*

- cdc_date: İlgili kaydın oluşturulduğu ya da eğer güncellendi ise son güncellendiği timestamp değeri.

- is_deleted: Kaydın silinip silinmediği bilgisi. Default değeri false'dur.

- id: Product id'si. Primary key gibi düşünülebilir. 

- category: Product'ın ait olduğu kategori.
​
`public.content_category` tablosuna belirli aralıklarla delete-update-insert işlemleri uygulanmaktadır.  

Bu durumda tabloya yeni kayıtlar eklenebilir, mevcut kayıtların kategori bilgisi güncellenebilir ya da kayıt silinebilir.  
Mevcut bir kayıt güncellendiği durumda cdc_date alanı güncellenme tarihi ile değiştirilmektedir.  

Yeni bir kayıt eklendiğinde de cdc_date alanı kaydın insert edildiği tarihi göstermektedir.  

is_deleted alanının default değeri ise false'dur. `public.content_category` tablosunda silinen bir kayıt daha sonra tekrar insert edilmemektedir!  
​
Product database'indeki `public.content_category` tablosununun '2020-12-21 00:59' tarihinde `sample.content_category` tablosundaki kayıtları içerdiğini,  

1 gün sonra '2020-12-22 00:59' tarihinde ise `sample.content_category_20201222_00_59` tablosundaki kayıtları içerdiğini varsayalım.  
​
#### Bizim isteğimiz `sample.content_category` ve `sample.content_category_20201222_00_59` tablolarını karşılaştırarak  

#### insert edilen yeni kayıtları `sample.content_category` tablosunada eklemek,  

#### update edilen kayıtlar var ise `sample.content_category` tablosunda da update etmek ve  

#### silinmiş olan kayıtların `sample.content_category` tablosundaki karşılıklarının is_deleted alanını true olarak güncellemek ve silmeden saklamaktır. 

#### Ve bunu yaparken tek bir `create or replace table` ya da `merge` statementı kullanarak yapmak istiyoruz.  

#### Gereken sorguyu çalıştırdıktan sonra `sample.content_category` ve `sample.content_category_target` tablolarının içerikleri birebir aynı olmalıdır!  
​
*Aşağıdaki tabloları kendi datasetiniz altına kopyalamasınız!*

- `sample.content_category`                > `DATASET_ADINIZ.content_category`

- `sample.content_category_20201222_00_59` > `DATASET_ADINIZ.content_category_20201222_00_59`

- `sample.content_category_target`         > `DATASET_ADINIZ.content_category_target`

​
*`create or replace table` statement: https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language#create_table_statement*

​
*`merge` statement: https://cloud.google.com/bigquery/docs/using-dml-with-partitioned-tables#using_a_merge_statement*

​
*2 tabloyu kıyaslamak için her satırın hash'ini alıp bu hash değerleri üzerinden joinleyip bir tabloda olup diğerinde olmayan hash değerleri var mıdır diye kontrol edebilirsiniz.*

*Hash fonksiyonunun örnek kullanımı aşağıdaki gibidir:*

>    select farm_fingerprint(to_json_string(t1)) as _hash1  

    from `dsmbootcamp.sample.content_category_target` t1  
    
    limit 100;  
