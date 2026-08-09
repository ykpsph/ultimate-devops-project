# CI / CD
CI = kodu otomatik olarak test etme ve build etme süreci. CI = Build -> Test -> Image oluşturma
CD = başarılı olan yeni versiyonu otomatik olarak deploy etme süreci. CD = Deployment

### CI tam olarak ne yapıyor? 
- Diyelim ki bir developer, Pull Request oluşturdu
```
Pull Request
     ↓
Checkout code
     ↓
Unit tests
     ↓
Static code analysis
     ↓
Build application
     ↓
Create Docker image
     ↓
Scan Docker image
     ↓
Push Docker image
     ↓
Update Kubernetes manifest
```

1. **Checkout code :** CI sistemi önce kodu Git repository'den alıyor. CI sistemi yeni bir VM üzerinde kodu çalıştırıyor. GitHub Actions bunu otomatik olarak oluşturabilir; Jenkins'te ise önceden hazırlanmış makineler kullanılabilir.
2. **Unit tests :** Kod checkout edildikten sonra testler çalıştırılıyor:
3. **Static code analysis :** Burada programı çalıştırmadan kodu analiz ediyoruz.
4. **Build application :** Sonra application build ediliyor:
5. **Create Docker image :** Docker Image oluşturma
6. **Scan Docker image :** Sonra image security açısından taranıyor.
7. **Push Docker image :** Image başarılıysa registry'ye gönderiliyor.
8. **Update Kubernetes manifest :** CI son aşamada Git repository'deki Kubernetes manifestini güncelliyor.

### Peki CD ne yapıyor?
- CD'nin amacı: Yeni image'ı Kubernetes cluster'a deploy etmek.
```
Kubernetes manifest
        ↓
        CD
        ↓
      ArgoCD
        ↓
Kubernetes Cluster
```