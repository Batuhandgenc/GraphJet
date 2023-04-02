# GraphJet

[![Build Status](https://travis-ci.org/twitter/GraphJet.svg?branch=master)](https://travis-ci.org/twitter/GraphJet)

GraphJet, tek bir sunucuda bellekte bir kaydırma zaman penceresi üzerinde tam bir graf dizini tutan Java'da yazılmış gerçek zamanlı bir graf işleme kütüphanesidir. Bu dizin, işbirlikçi filtreleme temelli kişiselleştirilmiş öneri algoritmaları da dahil olmak üzere çeşitli graf algoritmalarını destekler. Bu algoritmalar, özellikle Twitter'da içerik (tweet/URL) önerileri için gereken hızlı değişen bir graf üzerinde işbirlikçi filtreleme yapmaktadır.

GraphJet, dinamik bir graf üzerindeki hızlı kenar veri girişini desteklerken, kompakt kenar kodlaması ve dinamik bellek tahsisi düzenlemesi kombinasyonuyla sorgu sırasında hizmet sunabilme yeteneğine sahiptir. Her GraphJet sunucusu saniyede bir milyon graf kenarını işleyebilir ve kararlı durumda saniyede 500 öneri hesaplar, bu da saniyede birkaç milyon kenar okuma işlemine denk gelir. GraphJet'in iç yapıları hakkında daha fazla bilgiye http://www.vldb.org/pvldb/vol9/p1281-sharma.pdf makalesinde ulaşılabilir.

# Başlangıç ve Örnek

Depoyu klonladıktan sonra, aşağıdaki şekilde derleyin.(Testleri atlamak için ' -DskipTest' seçeneğini kullanın):

```
$ mvn package install
```

GraphJet, [Twitter4j library](http://twitter4j.org/en/) kütüphanesi kullanarak Twitter halka açık örnek akışından okuma yapan ve iki ayrı bellek içi ikili graf tutan bir demo içerir:

+ Kullanıcı-tweet etkileşimlerinin ikili grafiği. Sol taraftaki düğümler kullanıcıları, sağ taraftaki düğümler tweetleri temsil eder ve kenarlar tweet gönderilerini ve yeniden tweetleri temsil eder.
+ Tweet-hashtag içeriklerinin ikili grafiği. Sol taraftaki düğümler tweetleri, sağ taraftaki düğümler hashtag'leri temsil eder ve kenarlar içerik ilişkisini (örneğin, bir tweet bir hashtag içerir) temsil eder.

Demo'yu çalıştırmak için, GraphJet ana dizininde twitter4j.properties adlı bir dosya oluşturun ve Twitter kimlik bilgilerinizi girin (xxxx'yi gerçek kimlik bilgileriyle değiştirin):

```
oauth.consumerKey=xxxx
oauth.consumerSecret=xxxx
oauth.accessToken=xxxx
oauth.accessTokenSecret=xxxx
```
Kimlik bilgilerini almak için, -Twitter OAuth belgelerindeki- (https://dev.twitter.com/oauth/overview/application-owner-access-tokens) talimatları izleyin. Halka açık örnek akışı kayıtlı kullanıcılara açıktır; daha fazla bilgi için -Twitter akış API'leri hakkındaki belgeleri- (https://dev.twitter.com/streaming/overview) inceleyin.
GraphJet'i oluşturduktan sonra, demo'yu aşağıdaki gibi başlatın:

```
$ mvn exec:java -pl graphjet-demo -Dexec.mainClass=com.twitter.graphjet.demo.TwitterStreamReader
```

Demo başlatıldıktan sonra, Twitter genel örnek akışını içe aktarmaya başlar. Program, kullanıcı-tweet grafiğinin ve tweet-hashtag grafiğinin dahili durumunu gösteren bir dizi durum mesajı yazdırır.

Varsayılan olarak 8888 numaralı bağlantı noktasında çalışan bir REST API'si aracılığıyla grafikle etkileşime geçebilirsiniz; farklı bir bağlantı noktası belirtmek için -Dexec.args="-port xxxx" kullanın.

Kullanıcı-tweet etkileşiminin bellek içi bipartit grafiğinin durumunu sorgulamak için aşağıdaki çağrılar kullanılabilir:

+ `userTweetGraph/topTweets`: Etkileşimler (retweet'ler) açısından en popüler tweet'leri sorgulamak için k parametresini kullanarak dönecek sonuç sayısını belirtin (varsayılan olarak on sonuç). Örnek:

```
curl http://localhost:8888/userTweetGraph/topTweets?k=5
```

+ `userTweetGraph/topUsers`: Etkileşimler (retweet'ler) açısından en popüler kullanıcıları sorgulamak için k parametresini kullanarak dönecek sonuç sayısını belirtin (varsayılan olarak on sonuç). Örnek:

```
curl http://localhost:8888/userTweetGraph/topUsers?k=5
```

+ `userTweetGraphEdges/tweets`: Kullanıcı-tweet grafiğinde belirli bir tweet'e bağlı kenarları sorgulamak için, yani tweet ile etkileşimde bulunan kullanıcıları sorgulamak için id parametresini kullanın (yukarıdaki userTweetGraph/topTweets örneğinden tweetId örneği). Örnek:

```
curl http://localhost:8888/userTweetGraphEdges/tweets?id=xxx
```

+ `userTweetGraphEdges/users`: Kullanıcı-tweet grafiğinde belirli bir kullanıcıya bağlı kenarları sorgulamak için, yani kullanıcının etkileşimde bulunduğu tweet'leri sorgulamak için id parametresini kullanın (yukarıdaki userTweetGraph/topUsers örneğinden userId örneği). Örnek:

```
curl http://localhost:8888/userTweetGraphEdges/users?id=xxx
```

Aşağıdaki çağrılar tweet-hashtag içeriklerinin hafızada tutulan çift bölümlü grafiğinin durumunu sorgulamak için kullanılabilir:

+ `tweetHashtagGraph/topTweets`: Hashtaglar açısından en çok etkileşim alan tweetler için sorgular. Sonuç olarak döndürülecek sonuç sayısını belirtmek için k parametresini kullanın (varsayılan ondur). Örnek:

```
curl http://localhost:8888/tweetHashtagGraph/topTweets?k=5
```

+ `tweetHashtagGraph/topHashtags`: En çok tweet içeren hashtag'ler için sorgular. Döndürülecek sonuç sayısını belirtmek için k parametresini kullanın (varsayılan on). Örnek:

```
curl http://localhost:8888/tweetHashtagGraph/topHashtags?k=5
```

+ `tweetHashtagGraphEdges/tweets`: Belirli bir tweet'in tweet-hashtag grafiğindeki, yani tweet'te bulunan hashtag'lerle ilişkili olan kenarları sorgulamak için aşağıdaki çağrılar kullanılabilir. id parametresini belirli tweetId'sini belirtmek için kullanın (yukarıdaki tweetHashtagGraph/topTweets örneğinden). Örnek:

```
curl http://localhost:8888/tweetHashtagGraphEdges/tweets?id=xxx
```

+ `tweetHashtagGraphEdges/hashtags`: Belirli bir hashtag'in tweet-hashtag grafiğindeki kenarlara (yani belirli hashtag'i içeren tweetlere) sorgular. Parametre olarak id kullanarak hashtagId'yi belirtin (yukarıdaki tweetHashtagGraph/topHashtags örneğinden). Örnek:

```
curl http://localhost:8888/tweetHashtagGraphEdges/hashtags?id=xxx
```

Demo program, tweet-hashtag grafiği üzerinde benzerlik sorguları aracılığıyla işbirlikçi filtrelemeyi gösterir. Demo, Twitter'da kullanılan kişiselleştirilmiş öneri algoritmalarını sunmaz çünkü genel örnek akış API'si etkileşimler açısından çok seyrek olduğu için iyi sonuçlar vermez. Benzerlik sorguları için aşağıdaki uç nokta, bir girdi hashtag'ine göre ilgili hashtag'leri sağlar:

+ `similarHashtags`: Gerçek zamanlı verilere dayanarak girdi hashtag'ine benzer hashtag'leri hesaplar. Hashtag belirtmek için parametre olarak hashtag kullanın (ör. yukarıdaki tweetHashtagGraph/topHashtags'den). Örnek:

```
curl http://localhost:8888/similarHashtags?hashtag=trump&k=10
```

# License

Copyright 2016 Twitter, Inc.

Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)
