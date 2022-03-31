# Argo CD

[GitHub Homepage](https://argoproj.github.io/cd/) | [Alternative Homepage](https://argo-cd.readthedocs.io/en/stable/) | [Argo CD Example Apps](https://github.com/Scope-Infrastructure-Calismalari/argocd-example-apps)

*"Argo CD is a declerative, **GitOps** continuous delivery tool for Kubernetes."*

**Dikkat! GitOps aracı olan Argo CD'nin ne olduğunu okumadan önce eğer GitOps'un ne olduğunu bilmiyorsanız ilk olarak [GitOps yazımızı](GitOps.md) okumanızı rica ederiz, sonrasında bu yazı daha anlaşılır olacaktır.**

Yazının devamında daha detaylı olarak anlatılacak olan Argo CD mimarisinin bir örneğini aşağıdaki görselde görebilmekteyiz. GitOps yazısını okuduktan sonra "GitHub Repo" ifadesinin neden mimaride yer aldığını, alması gerektiğini hemen anlayabilmiş olmalıyız.

<p align="center">
 <img src="images/Argo-CD/image-1.png">
</p>

## Argo CD Nedir?
Argo CD, adından da anlaşılabileceği üzere aslında bir **C**ontinuous **D**elivery (Sürekli Teslim) aracıdır. Argo CD'yi anlamadan önce CD'nin projelere nasıl eklenip uygulandığını, ve birçok projenin ortak tercihi olan Jenkins ve GitLab gibi CI/CD araçlarının anlayıp sonrasında Argo CD'yi bunlarla kıyaslayarak devam edeceğiz.

### Birçok projede CD nasıl kullanılmaktadır?

Aşağıdaki görsellerin solundaki gibi birçok mikro servisimizin olduğunu ve sağ görseldeki gibi bunları Kubernetes(k8s) cluster'ına taşıdığımızı düşünelim.

<p float="center">
  <img src="images/Argo-CD/image-2.png" width="49%"/>
  <img src="images/Argo-CD/image-3.png" width="49%"/>
</p>

Uygulamamızın kaynak kodunda değişiklikler yapıp (yeni özellilk eklemek veya bugfix yapmak gibi), Git repo'suna push'ladığımızı düşünelim.

<p align="center">
  <img src="images/Argo-CD/image-4.png"/>
</p>

Bu değişiklik ve push'lama işleminden sonra sistemimizde kurulu olan ve bu Git repos'su ile eşlenmiş olan Jenkins vb. uygulamalarımızın CI pipeline süreci otomatik olarak tetiklenecektir.

<p align="center">
  <img src="images/Argo-CD/image-5.png"/>
</p>

Tetiklenen bu pipeline süreci otomatik adımlarla önce uygulamamızı test sürecinden geçirecek, hata ile karşılaşmazsa yeni bir Docker imajı oluşturacak ve bunu Docker Repo'ya push'layarak CI sürecini tamamlayacaktır.

<p align="center">
  <img src="images/Argo-CD/image-6.png"/>
</p>

Artık önümüzde yeni bir soru var: *"Yeni oluşturulan bu Docker imajı k8s cluster'ına nasıl deploy edilecek?"*

  1. k8s deployment YAML dosyasını Docker imajının yeni versiyon numarasını yazarak güncelleriz.

<p align="center">
  <img src="images/Argo-CD/image-7.png"/>
</p>

  2. Değiştirilen YAML dosyasını k8s'e uygularız (apply).

Docker imajını Docker Repo'ya push'lama işlemine kadar olan basamakların tümü CI sürecini oluştururken güncellenmiş YAML dosyasını k8s cluster'ına apply etmek ise CD sürecini oluşturmaktadır.

<p align="center">
  <img src="images/Argo-CD/image-8.png"/>
</p>

**Bu CI/CD sürecinin zorlukları ve sıkıntıları:**

 - K8s cluster'ına erişebilmek ve değişiklikler yapabilmek için kubectl, helm gibi araçların bu örnekte Jenkins olarak varsaydığımız ve sistemimizde kullandığımız *Build Automation Tool* 'larına yüklenmesi gerekliliği

 - K8s'e erişimin sağlanması da gerekmekte çünkü kubectl yalnızca k8s client aracıdır ve k8s'e erişebilmesi için "credential"ların tanımlanması gerekmektedir. Eğer AWS gibi bulut sunucuları kullanıyorsak bunlara da erişim için ayrıca credentials tanımlamamız gerekmektedir.

 - Bu credential tanımlamaları yalnızca konfigürasyona emek vermek değil bunun yanı sıra güvenlik konusunu da gündeme getirmektedir. Cluster credential'larını external servislere ve araçlara da vermemiz gerekmektedir. Örneğin 33 adet uygulamamız varsa her uygulama kendisi için ayrıca credential talep etmektedir. Ancak bu şekilde her uygulama cluster'da kendisi için tanımlı uygulama kaynaklarına erişebilmiş olacaktır. Eğer tek bulut servisimiz değil de başka cluster'larımız da varsa bunların herbiri için de yine credential tanımlamaları gerekecektir.

 - **En önemli sorun** ise k8s'e uygulama deploy eden veya k8s konfigürasyonlarında bir değişiklik yapan Jenkins, bu deployment'ların durumları ile ilgili bilgi sahibi olamamaktadır. Bir defa "`kubectl apply ...`" komutu çalıştırıldığında Jenkins aslında bu execution'ın durumu hakkında bilgi sahibi olamamaktadır. "Uygulama kuruldu mu?", "Uygulama durumu healty mi?", veya "Uygulama başlama aşamasında hata mı verdi?" gibi soruların yanıtını Jenkins takip edememektedir.

**Bu durumlardan dolayı CI/CD sürecinin CD kısmının iyileştirilebileceğini görmekteyiz.**

İşte Argo CD bu özel durumlar göz önüne alınarak K8s cluster'larına daha efektif bir şekilde *"delivery"* yapılabilmesi için GitOps prensipleri baz alınarak geliştirildi. Argo CD için **CD for Kubernetes** diyebiliriz.

<br>
<br>
<br>

### Argo CD'nin kıyaslanması

<br>

#### Argo CD yalnızca başka bir CD aracı mıdır?

<br>

#### Diğer CI/CD araçlarının yerine kullanılabilir mi?

<br>
<br>
<br>

## Neden Argo CD? Kullanım Alanları Nelerdir?

<br>
<br>
<br>

## Nasıl Çalışır, Faydaları Nelerdir?
























































## Kurulum (Yerel makina)

Argo CD uygulaması "minikube" uygulaması üzerinde çalışan Kubernetes cluster'ına kurulmuş ve orada çalıştırılmıştır.

1. Minikube uygulaması başlatılır:

    `minikube start`

2. Argo CD uygulaması için namespace tanımlanır:

    `kubectl create namespace argocd`

3. Argo CD kurulumu yapılır:

    `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

    Not: Load Balancer yapı için farklı kurulum URL'i gerekmektedir.

4. Kurulumun doğru yapılıp yapılmadığını şu şekilde kontrol edebiliriz:

    `kubectl -n argocd get all`

    Eğer kurulum doğru şekilde tamamlanmışsa karşımıza şuna benzer sonuçlar gelecektir:

        NAME                                                    READY   STATUS    RESTARTS   AGE
        pod/argocd-application-controller-***                   1/1     Running   0          3h32m
        pod/argocd-applicationset-controller-***                1/1     Running   0          3h32m
        pod/argocd-dex-server-***-djnpt                         1/1     Running   0          3h32m
        pod/argocd-notifications-controller-***                 1/1     Running   0          3h32m
        pod/argocd-redis-***                                    1/1     Running   0          3h32m
        pod/argocd-repo-server-***                              1/1     Running   0          3h32m
        pod/argocd-server-***                                   1/1     Running   0          120m

        NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
        service/argocd-applicationset-controller          ClusterIP   XX.XX.XX.XX      <none>        7000/TCP                     3h32m
        service/argocd-dex-server                         ClusterIP   XX.XX.XX.XX      <none>        5556/TCP,5557/TCP,5558/TCP   3h32m
        service/argocd-metrics                            ClusterIP   XX.XX.XX.XX      <none>        8082/TCP                     3h32m
        service/argocd-notifications-controller-metrics   ClusterIP   XX.XX.XX.XX      <none>        9001/TCP                     3h32m
        service/argocd-redis                              ClusterIP   XX.XX.XX.XX      <none>        6379/TCP                     3h32m
        service/argocd-repo-server                        ClusterIP   XX.XX.XX.XX      <none>        8081/TCP,8084/TCP            3h32m
        service/argocd-server                             ClusterIP   XX.XX.XX.XX      <none>        80/TCP,443/TCP               3h32m
        service/argocd-server-metrics                     ClusterIP   XX.XX.XX.XX      <none>        8083/TCP                     3h32m

        NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/argocd-applicationset-controller   1/1     1            1           3h32m
        deployment.apps/argocd-dex-server                  1/1     1            1           3h32m
        deployment.apps/argocd-notifications-controller    1/1     1            1           3h32m
        deployment.apps/argocd-redis                       1/1     1            1           3h32m
        deployment.apps/argocd-repo-server                 1/1     1            1           3h32m
        deployment.apps/argocd-server                      1/1     1            1           3h32m

        NAME                                                       DESIRED   CURRENT   READY   AGE
        replicaset.apps/argocd-applicationset-controller-***       1         1         1       3h32m
        replicaset.apps/argocd-dex-server-***                      1         1         1       3h32m
        replicaset.apps/argocd-notifications-controller-***        1         1         1       3h32m
        replicaset.apps/argocd-redis-***                           1         1         1       3h32m
        replicaset.apps/argocd-repo-server-***                     1         1         1       3h32m
        replicaset.apps/argocd-server-***                          0         0         0       3h32m
        replicaset.apps/argocd-server-***                          1         1         1       120m

        NAME                                             READY   AGE
        statefulset.apps/argocd-application-controller   1/1     3h32m

5. Cluster'da pod'da çalışan uygulamaya erişebilmek için servis tanımlanması gerekmektedir.Load Balancer, ingress gibi farklı seçenekler mevcut. Biz bu anlatımda *Port Forwarding* kullanacağız:

    `kubectl port-forward svc/argocd-server -n argocd 8080:443`

    Bu komut ile Argo CD'nin API sunucusuna https://localhost:8080 bağlantısını kullanarak erişebilmiş olacağız.

    - LoadBalancer için:

        argocd-server servisinin tipini LoadBalancer olarak değiştirmeliyiz:

        `kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'`

    - Ingress için:

        Argo CD'nin ingress ile konfigüre edilmesini anlatan doküman için bağlantı: https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/ingress.md

    **Eğer bu adıma kadar sorunsuz ilerleyebildiysek localhost bağlantısına tıkladığımızda ekranımıza aşağıdaki gibi bir sayfa gelecektir.**

<p align="center">
 <img src="images/Argo-CD/image-111.png">
</p>

6. Varsayılan kullanıcı adı olarak "admin" ile gelen Argo CD uygulaması, kullanıcı şifresini ise kurulum anında oluşturmaktadır. Initial password'e ise şu şekilde ulaşılabilir:

    - Aşağıdaki komut ile argocd namespace'indeki tüm kurulumları listeyelim pod'lardan *argocd-server* olanını filtreliyoruz.

        `kubectl -n argocd get all | grep pod/argocd-server`

        Çıktı olarak bize dönecek sonuç:

          pod/argocd-server-77b597bc68-w52zp      1/1     Running     0     148m

        Burada "pod/" kısmından sonraki kısım bizim initial password'ümüzdür.

    **Dikkat!!**

      Muhtemelen versiyon farkından kaynaklanan nedenlerden dolayı artık initial password'e bu şekilde ulaşılamamaktadır, onun yerine direkt bu şifreyi kendi istediğimiz şifre ile değiştirebiliriz.

      `kubectl -n argocd patch secret argocd-secret \\n  -p '{"stringData": {\n    "admin.password": "XXXXXXXXXXXXX",\n    "admin.passwordMtime": "'$(date +%FT%T%Z)'"\n  }}'`

      Buradaki *admin.password* 'den sonra *XXX* ile belirtilen yere kendi **encrypt edilmiş** şifremizi yazmamız gerekmektedir.

      Şifrenin **Bcrypt** şifreleme yöntemi kullanılarak şifrelenmesi gerekmektedir. Bunu yapabilmek için [browserling](https://www.browserling.com/tools/bcrypt) adresine gidip Password kısmına belirlediğimiz şifreyi yazıp **Bcrypt** butonuna basarak şifreleme işlemini gerçekleştirebiliriz.

<p align="center">
    <img src="images/Argo-CD/image-112.png">
</p>

      Oluşturulan bu şifreyi kopyalayıp yukarıdaki komuttaki "XXXXXXXXXXXXX" kısmının yerine yazıp çalıştırdığımızda şifre değiştirme işlemi başarıyla tamamlanmış olacaktır.

7. Şifre değiştiği için Argo CD uygulamasının servide deployment'ını yeniden başlatmamız gerekmektedir:

    `kubectl -n argocd rollout restart deployment argocd-server`

8. Yeniden başlatma nedeniyle durmuş olan *port-forwarding* işlemini yeniden başlatıyoruz:

    `kubectl port-forward svc/argocd-server -n argocd 8080:443`

    Bu komuttan sonra şu şekilde bir çıktı oluşacaktır:

       deployment.apps/argocd-server restarted


9. Bu aşamaları sıkıntısız geçebildiysek artık localhost:8080 ile yeniden uygulamamıza erişip kendi belirlediğimiz ve bcrypt ettiğimiz şifre ile giriş yapabilmemiz gerekmekte. Giriş işlemi başarıyla sonuçlanırsa karşımıza şu şekilde bir ekran gelecektir:

<p align="center">
    <img src="images/Argo-CD/image-113.png">
</p>

10. Tebrikler. Kurulum işlemini başarıyla tamamladınız. Bir sonraki adımda bu boş ekranı uygulamalarla doldurup biraz hareketlendireceğiz :)

<br>
<br>
<br>

## Nedir, Ne İşe Yarar, Kullanım (Detaylı Anlatım)

<br>
<br>
<br>

## Kullanım Örneği