# week-1
1.HAFTA ÖDEV LİNKLERİ

https://sqlzoo.net/wiki/SQL_Tutorial

https://selectstarsql.com

https://sqlzoo.net/wiki/Musicians

https://www.kaggle.com/learn/intro-to-sql  


# week-2
2.HAFTA ÖDEV LİNKLERİ

## Soru 1) 1980’den itibaren spor grubu bazında en çok madalya alan 1. 3. 5. ülkeyi bulalım.
## Soru 2) 1980’den itibaren herhangi bir spor grubunda üst üste 3 veya daha fazla madalya almış atletleri bulalım.

Aşağıdaki şekilde

create or replace table `dsmbootcamp.DATASET_ADINIZ.summer_medals`

as

select * from `dsmbootcamp.sample.summer_medals`; 

DATASET_ADINIZ kısmına kendi dataset adınızı yazarak tabloyu create ettikten sonra, soru çözümünüzde bu yarattığınız tabloyu kullanabilirsiniz.

## Soru 3) "sample.pageview": tablosunda 1 gün içerisinde trendyol.com a gelen tüm ziyaretlerin logu var.
--view_ts: ziyaret zamanı

--channel: android,ios,web.

--pagetype: görüntülenen sayfa: homepage, order, boutiquedetail, productdetail.. gibi.

--deviceid: sayfa ziyareti yapan cihaz id'si. Bizim için tekil kullanıcıyı da ifade eder.

--tabloya veri akışı dakikada 1 defa gerçekleşir, 23:10:00 anında 23:09:00-23:09:59 kayıtları tabloya eklenir.

Bu çalışmada çıkarmak istediğimiz bilgi, günün her bir dakikası için aktif kullanıcı sayısının hesaplanması.

## Aktif kullanıcı ne demek?
--sitede herhangi bir sayfa ziyareti sonrasında 5dk boyunca aktif kullanıcı sayılır.

--bir örnek ile, 

---- "2020-03-03 23:10:14" anınında 100farklı cihaz trendyol'u açıp kapattı ise ve sonrasında hiç bir ziyaret gelmedi ise.

--"2020-03-03 23:10" 100 aktif kullanıcı vardır.

--"2020-03-03 23:11" 100 aktif kullanıcı vardır.

--"2020-03-03 23:12" 100 aktif kullanıcı vardır.

--"2020-03-03 23:13" 100 aktif kullanıcı vardır.

--"2020-03-03 23:14" 100 aktif kullanıcı vardır.

--"2020-03-03 23:15" 0 aktif kullanıcı vardır.

--en basit çözüm ile sadece "2020-03-03 23:14" 'deki aktif kullanıcıları hesaplamak için:

select timestamp '2020-03-03 23:14:00' view_period

      ,count(distinct deviceid) active_user_count
      
 from sample.pageview
 
where timestamp_trunc(view_ts,minute) between '2020-03-03 23:10:00' and '2020-03-03 23:14:00'

--Yazacağınız sorgu/sorguların çıktısında beklediğimiz çıktı sitedeki dakikalık aktif kullanıcı sayısı:
-- view_period            active_user_count

-- 2020-03-03 23:14:00            123123123

-- 2020-03-03 23:13:00            125123127

-- 2020-03-03 23:12:00            126123124

## NOT: Çözümde göz önünde bulundurmanızı istediğimiz koşullar:

-- exact sonuç aramıyoruz, %2'ye kadar sapmalar kabul edilebilir, approx fonksiyonları kullanabilirsiniz

-- örnek tabloda 289m kayıt var, bu aylar öncesinin verisi, optimizasyonlar için summary, ara tablo oluşturabilirsiniz.

-- tek sorguda hesaplayabilirsiniz, temporary tablolar oluşturup, 1den fazla sorgu ile de çözebilirsiniz.

-- hll_count.init, hll_count.merge, hll_count.merge_partial, hll_count.extract kullanabilirsiniz.

-- https://cloud.google.com/bigquery/docs/reference/standard-sql/hll_functions
