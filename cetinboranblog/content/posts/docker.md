---
title: "Docker 101"
date: 2024-05-06T17:55:28+08:00
description: 'This is Docker 101'
tags: ["Docker", "Dockerfile", "Docker Compose", "Container"]
type: post
weight: 50
showTableOfContents: true
---

# 1 - Docker

## 1.1 Konteyner

### Nedir?

Konteyner, bir uygulamanın veya bir hizmetin kendi kütüphaneleri ve bağımlılıkları ile birlikte dış ortam ve uygulamalardan izole bir şekilde tutulmasına denir. Bu, bir tür sanallaştırma teknolojisidir.

### Konteyner != Sanal Makine (VM)

* Sanal makineler genellikle daha fazla sistem kaynağına ihtiyaç duyar ve daha uzun başlatma sürelerine sahiptir.
* Konteynerlara kıyasla, hafif ve hızlı dağıtım gerektiren senaryolarda sanal makineler daha ağır bir seçenek olabilir.
* Sanal makineler, daha fazla izolasyon içeren yerlerde tercih edilebilirler.

Aşağıdaki resimde görüldüğü gibi, sanal makinelerin hepsi işletim sistemi üzerine kurulmuştur. Dolayısıyla izolasyonları konteynerlara kıyasla daha fazladır, ancak bu hızlı dağıtım ve taşınabilirliği azaltır.

Ancak konteynerlarda fazladan işletim sistemi yoktur, dolayısıyla sanal makinelerden daha az yer kaplarlar.

![Difference](https://i.hizliresim.com/87v482l.jpg)

### Örnek

Diyelim ki backend ve database olarak PostgreSQL kullanacağız. Bu işi konteynerlarda yapmak hızlı ve minimum yer kaplarken, sanal makinelerde yapmak için iki ayrı işletim sistemi kurmak ve gerekli programları eklemek gerekmektedir; bu da gereksiz yer ve zaman harcamaya neden olur.

## 1.2 Docker

### Nedir?

Docker, konteyner'ları oluşturmak, yönetmek ve çalıştırmak için bir dizi araç ve hizmet sunan bir platformdur.

### Neden Kullanılır?

* İzolasyon
* Taşınabilirlik
* Hızlı dağıtım
* Kaynak Verimliliği

Docker, hızlı dağıtım ve taşınabilirliği image'ler yardımı ile yapar. Image'ler sayesinde bir satır komut ile istediğimiz uygulamayı indirip kullanabiliriz. İleride daha çok değineceğiz.

### Örnek

Diyelim ki bir Backend ve bir Web Server'imiz var. Bu iki program da A paketinin x.x.x sürümünü kullansın. Şimdi eğer bizim Backend'e bir güncelleme geldiğinde ve artık A paketinin x.x.y sürümünü istediğinde ne yapacağız? Bizim Backend'imiz A pakettenin x.x.y sürümünü istiyor ancak Web Server'imiz A paketinin x.x.x sürümünü istiyor. Yani kısaca sürüm uyuşmazlığı yaşanıyor. Bu gibi durumlarda Docker kullanılabilir. Biz gidip Backend'i bir konteyner içine alacağız ve içine A paketinin x.x.y sürümünü yükleyeceğiz, aynı şekilde Web Server'i konteyner içine alacağız ve içine A paketinin x.x.x sürümünü yükleyeceğiz. Böylelikle bu iki uyuşmayan program, artık birlikte konteyner'lar yardımıyla kendi kütüphaneleri ve bağımlılıklarını kullanarak çalışabiliyorlar.

### Nasıl Kullanılır?

Ben burada Docker Desktop uygulamasını anlatmak yerine biraz command line üzerinden en çok kullanılan komutları anlatmayı daha uygun gördüm.

**Global**

* **`docker search image:tag`**: [Docker Hub'tan](https://hub.docker.com/) bir imaj arar.
* **`docker pull image:tag`**: [Docker Hub'tan](https://hub.docker.com/) bir imajı indirir.

**Network**

* **`docker network create name`**: Kendi ağınızı oluşturur.
* **`docker network ls`**: Oluşturulan tüm ağları listeler.
* **`docker network rm network_id`**: Belirtilen ağı siler.

**Image**

* **`docker images` / `docker image ls`**: Bilgisayarınıza yüklenmiş tüm imajları listeler.
* **`docker image rm image_id`**: Belirtilen imajı siler.
* **`docker image inspect <image>`**: İmaj hakkında detaylı bilgi sağlar.
* **`docker image build dockerFilePath` / `docker build dockerFilePath`**: Docker Dosyası kullanarak özel bir imaj oluşturmanızı sağlar.
* **`docker run [OPTIONS] image [COMMANDS]`**: Bir imajı kullanarak bir konteyner oluşturur. \[COMMANDS] tamamlandıktan sonra kapanır.
  * **OPTIONS**:
    * `--name`: Konteynera özel bir isim atar.
    * `--rm`: Konteyner işini bitirdikten sonra otomatik olarak kendini siler.
    * `-it`: Konteyneri interaktif modda başlatır ve terminali size bağlar.
    * `-p`: Konteynerin belirli bir portunu ana makineye yönlendirmek için kullanılır.
    * `-d`: Konteynerı arkaplanda çalıştırır.
    * `-e`: Çevresel değişkenleri belirlemek için kullanılır.
    * `-v`: Ana makinedeki bir dizini veya dosyayı konteynere bağlamak için kullanılır.
    * `--network <name>`: Konteynerin bağlı olduğu ağı belirtir.
    * `--secuity-opt PROFILES`: Konteynerin bağlıolduğu ağı belirtir.
    * `--privileged`: Konteyneri ana makinenin kernel düzeyinde ayrıcalıklı bir modda çalıştırır ve tüm yetkilere sahip olur.
    * `–-cap-add capability`: Konteyner’a yetenek ekler.
    * `-–cap-drop capability`: Konteyner'dan yetenek çıkarır.
      * `ALL`: yazılırsa eğer capabilitiy yerine bütün belirli yetenekleri ekler/çıkarır.
* **COMMANDS**
  * Aslında buradaki komutlar terminalde yazabileceğiniz her şey olabilir.
  * `ls -l`
  * `sleep 3600`

**Containers**

* **`docker ps`**: Çalışan konteyner’ları listeler.
  * `-a`: Çalışan çalışmayan tüm konteyner’ları listeler.
* **`docker start container_id/name`**: Belirli bir Konteyner’ı çalıştırır.
* **`docker stop container_id/name`**: Belirli bir Konteyner’ı durdurur.
* **`docker restart container_id/name`**: Belirli bir konteyner’ı yeniden başlatır.
* **`docker rm container_id/name`**: Belirli bir konteyner’ı siler.
* **`docker exec [OPTIONS] <container_id> [COMMAND]`**: Çalışan bir konteyner’da komut çalıştırmamızı sağlar. Kullanımı `docker run ...` ile benzerdir.
  * **`docker exec my_container ls -l`**: my\_container içinde `ls –l` komutunu çalıştırır.

### Uygulama

Şimdi sizden basit bir uygulama yapmanızı isteyeceğim, docker'ı açın hello-world image'ini Docker Hub'tan indirin. Kontrol edin bakalım inmiş mi? Bu image'i kullanarak düz bir konteyner oluşturun işlevi nedir?. Çalışıp çalışmayan bütün konteyner'ları listeleyin ve ardından o açtığınız konteyner'ı silmeyi deneyin. Konteyner oluştururken yukarıda öğrendiğiniz bütün OPTION ları deneyip biraz zaman geçirmenizi tavsiye ediyorum.

## 1.3 Dockerfile

### Nedir?

Dockerfile, kendi özelleştirdiğimiz imajları oluşturmak için kullanılan bir dosyadır.

### Nasıl Kullanılır?

* **`FROM image:tag`**: Konteyner’ın hangi image’ı temel alacağını beliritir.
* **`RUN command`:** İmaj üzerinde komutları çalıştırmak için kullanılır.
* \*\*`COPY hostPath containerPath**`: Ana makinadan konteyner içine dosya kopyalamamızı sağlar.
* **`WORKDIR`**: Çalışma dizinini belirtir.
* **`EXPOSE`**: Konteyner'ın hangi portlarını dış dünyaya açacağını belirtir.
* **`CMD`**: Konteyner çalıştırıldığında otomatik olarak çalışacak komutu belirtir.

### Örnek

Alttaki örnekte ubuntu'nun 18.04 sürümünü temel alacağımı belirttim. Ardından bu image'de olması gereken yükleme işlemlerini run komutu ile gerçekleştirdim. Ana dizinimi /app yaptıktan sonra kendi makinemden ./script.sh 'ı konteyner içine kopyaladım. Son olarak konteyner çalıştığında çalışmasını istediğim komutu yazdım.

```Dockerfile
FROM ubuntu:18.04

RUN apt update -y && apt install nano

WORKDIR /app
COPY ./script.sh .

CMD ["./script.sh"]
```

Bu üstteki Dockerfile ı image yapmak için `docker build -t image_tag dockerfilePath` yazarız.

## 1.4 Docker Compose

### Neden Lazım?

Biz `docker run` kullanarak bir tane konteyner çalıştırıyoruz. Peki birden fazla konteyner'ı çalıştırmayı ayrıca bunları birbirine bağlamayı istersek ne yapacağız?

Modern bir site düşünelim. _Backend: golang_, _Web Server: ngnix_, _Frontend: reactJS_ ve _Database: PostgreSQL_ olsun. Ben bu siteyi docker'da kaldırmak için yapmam gereken 9 satır komut vardır.

Ilk olarak hepsi için `docker build` komutunu kullanıp **Dockerfile** larını kullanarak image'lerini oluşturmak. Sonra bu konteyner'ların birbirleri ile haberleşmesini sağlamak için `docker network create name` ile bir ağ oluşturmak. En son olarakta bu image'lerin hepsi için `docker run` yazıp `--network` tagı veya spesifik konteyner için spesifik option'ları eklemeliyim ki düzgün bir şekilde çalışabilsin.

Bu işlemi açıkça zor ve zaman alıcıdır. Bunu daha kolay yapmak için **docker compose** kullanıyoruz.

### Nedir?

Docker Compose, Docker tarafından sağlanan ve çoklu konteynerlı uygulamaları tanımlamak, yönetmek ve çalıştırmak için kullanılan bir araçtır.

### Neden İhtiyacımız Var?

Modern bir site düşünelim. Frontend olarak React, Backend olarak Go, Web sunucusu olarak Nginx, Veritabanı olarak PostgreSQL kullanalım.

Bu senaryoda en az 4 konteyner bulunmaktadır. Konteyner'ları build edeceğiz ve çalıştıracağız aynı zamanda bazı konteynerların birbirine bağlanması için bir ağ oluşturmamız gerekmektedir.

Şimdi minimal bir modern siteyi ayağa kaldırmak için kaç komut kullanmamız gerektiğine bakalım. İlk olarak konteynerları oluşturmamız gerekmektedir, bu nedenle 4 kez `docker build ...` komutunu kullanırız. Ardından, konteynerlar arasında bir ağ oluşturmak için `docker network create ...` komutunu kullanırız ve son olarak konteynerları çalıştırmak için `docker run ...` komutunu kullanırız. Ancak bu şekilde her bir konteyner için komutları hatırlamak ve yazmak zor olabilir.

Yukarıda da belirtildiği gibi toplamda 9 satır komut yazdık ve yalnızca bir minimal modern siteyi çalıştırdık. İşte bu nedenle Docker Compose'u kullanıyoruz.

### Nasıl Yapılır?

Docker Compose, YAML formatında bir dosya kullanarak birden fazla Docker konteynerini tek bir uygulama olarak tanımlamanıza ve yönetmenize olanak tanır.

Bu dosyayı birden fazla Dockerfile için yazacağınız `docker run ...` komutu olarak düşünebilirsiniz.
Her Dockerfile için ayrı ayrı `docker run` bilgilerini giriyorsunuz ve tek komutla bütün Dockerfile'ları build edip çalıştırabiliyorsunuz.

Burada kısa olsun diye backend ve database olarak olarak anlatacağım. Bu kısa bir örnek olacak daha fazla bilgi için kaynakça'ya bakabilirsiniz.

İlk olarak docker-compose.yml dosyası oluşturun.

```docker-compose
version: '3.9'

services:
  backend:
    build: ./backend
    container_name: backend
    restart: always
    ports:
      - "8081:8081"
    depends_on:
      - postgres
    networks:
      - impact

  postgres:
    image: postgres:latest
    container_name: postgres
    environment:
      POSTGRES_USER: "test"
      POSTGRES_PASSWORD: "test"
      POSTGRES_DB: "test"
    ports:
      - "5432:5432"
    restart: on-failure
    volumes:
      - ./backend/script:/docker-entrypoint-initdb.d
      - ./postgres-data:/var/lib/postgresql/data
    networks:
      - impact

networks:
  impact:
    driver: bridge

```

+ **version**: Docker Compose versionunu belirtir.
+ **services**: Docker Compose dosyasında tanımlanan servislerin başladığı bölümü başlatır.
+ **backend**: Burada backend servisinin başladığını gösteriyor. Farklı şeylerde yazılabilir, örneğin web server, frontend gibi.
+ **image**: Bu komut, kullanılacak Docker imajını belirtir. Genellikle resmi veya özel Docker imajları kullanılır.
+ **build**: Dockerfile'ın yerini belirtir.
+ **container_name**: Oluşturulacak olan konteyner'ın ismini verebilirsiniz.
+ **restart**: Bu komut, bir konteynerin ne zaman yeniden başlatılacağını belirtir. `always`, `on-failure` veya `unless-stopped` gibi değerler alabilir.
+ **environment**: Bu komut, bir konteynerin çalışma zamanı ortam değişkenlerini belirtir. Örneğin, veritabanı kullanıcı adı ve parolası gibi.
+ **volumes**: Bu komut, host makinadaki dizinlerin veya dosyaların konteynere bağlanmasını sağlar. Bu, veri kalıcılığı ve paylaşımı için kullanılır.
+ **ports**: Bu komut, bir konteynerin hangi portlarının host makinadaki portlara bağlanacağını belirtir. Genellikle `hostPort:containerPort` formatında kullanılır.
+ **depends_on**: Bu komut, bir servisin başlaması için gereken diğer servisleri belirtir. Ancak, bu komut belirli bir servisin hazır olduğunu garanti etmez.
+ **networks**: Bu komut, bir servisin hangi Docker ağlarında çalışacağını belirtir. Birden fazla ağa bağlanabilir ve ağlar arası iletişim sağlayabilir.

Yukarıda temel olarak kullanılan docker-compose tagları yer alıyor. Tabikii daha fazla var bunun için kaynakça'dan daha fazla bilgi edinebilirsiniz.

Bu örnekte backend servisi alttaki Postgres servisini kullanıyor ve network aracılığı ile birbirleri ile etkileşim halindeler. Şimdi bu servisleri alttaki komutlar ile çalıştırıp kullanıma hazır hale getirebilirsiniz.

+ `docker compose build`: Bu komut, belirtilen docker dosyalarını (Dockerfile'ları) kullanarak tüm hizmetlerin imajlarını oluşturur.
+ `docker compose up`: Bu komut, diğer parametrelere göre belirtilen tüm hizmetler için bir `docker run` komutu oluşturur.
+ `docker compose up --build`: Yazarak tek seferde iki işlemi yapabilirsiniz.

# Kaynakça

<details>
<summary>Docker için Kaynakça</summary>

- [Intro to Docker LAB](https://tryhackme.com/room/introtodockerk8pdqk)
- [Kablosuz Kedi](https://www.youtube.com/watch?v=4XVfmGE1F_w&t=922s&ab_channel=kablosuzkedi)
- [Mosh](https://www.youtube.com/watch?v=HG6yIjZapSA&t=81s&ab_channel=ProgrammingwithMosh)
- [Compose Samples](https://github.com/docker/awesome-compose)

</details>