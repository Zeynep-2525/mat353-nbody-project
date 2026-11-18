
# MAT353 Nümerik Analiz — 2D N-Cisim (N-Body) Simülasyonu

2026-2027 Güz Dönemi Nümerik Analiz Mini Araştırma Projesi.

**Not:** README dosyasındaki proje yapısı, görev dağılımı, kod sözleşmesi ve çalışma takvimi için bir taslak hazırladım. Henüz ekipçe üzerinden geçip kesinleştirmedik. Müsait olduğunuzda inceleyip eksik veya değiştirilmesini düşündüğünüz noktaları düzenleyebilir ya da grupta paylaşabilirsiniz. Bu format ve içerik üzerinde ortaklaşmamız hâlinde görev dağılımını ve çalışma planını bu doğrultuda ilerletebiliriz. Herkesin görüşünü aldıktan sonra son hâlini belirleyelim.

## Kritik Tarihler ve Kurallar

- **Ekip Bildirimi:** 25 Kasım 2026 Çarşamba (Derste)
- **Proje Teslimi:** 13 Aralık 2026 Pazar, saat 22.00 (Google Classroom)
- **Teslim Dosyaları:**
  - Proje Raporu (8–15 sayfa)
  - Sunum Dosyası (maksimum 10 slayt)
  - Colab Dosyası
- **Sunum Tarihleri:** 16 Aralık / 23 Aralık 2026
- **Sunum Süresi:** 15 dakika (sunum + canlı demo)

---

## Dosya Yapısı ve Sahiplik Tablosu

Çakışma (merge conflict) yaşanmaması için **her üye yalnızca kendi sorumluluğundaki dosyaları** düzenlemelidir.

| Dosya / Klasör | Sorumlu | Açıklama |
|:---|:---:|:---|
| `src/physics.py` | **A** | Başlangıç dağılımları (`seed`), doğrudan $O(N^2)$ kuvvet, enerji ve momentum hesaplamaları |
| `tests/test_physics.py` | **A** | Etki-tepki simetrisi ve enerji/momentum korunumu birim testleri |
| `src/integrators.py` | **D** | İleri Euler, RK4 ve Velocity Verlet zaman integratörleri |
| `experiments/exp1_integrators.py` | **D** | Deney 1: $\Delta t$ yakınsama ve enerji korunumu ölçümleri ve grafikleri |
| `tests/test_integrators.py` | **D** | 2-cisim analitik Kepler yörüngesi ($\epsilon=0$) ve yakınsama derecesi testleri |
| `src/barnes_hut.py` | **T** | 2D Quadtree veri yapısı, kütle merkezi ve Barnes-Hut ($\theta$) kuvvet hesabı |
| `experiments/exp2_bh_accuracy.py` | **T** | Deney 2: Açılma açısı ($\theta$) ve bağıl kuvvet hatası analizi ve grafikleri |
| `experiments/exp3_scaling.py` | **T** | Deney 3: $O(N^2)$ vs. $O(N \log N)$ süre ölçümü (5 `seed`, ortalama ± standart sapma) ve grafikleri |
| `tests/test_barnes_hut.py` | **T** | $\theta = 0$ iken doğrudan hesaplama ile birebir eşleşme birim testi |
| `notebooks/MAT353_NBody_Colab.ipynb` | **ORTAK** | Teslim öncesi tüm modüllerin birleştirileceği tek parça Colab defteri |
| `requirements.txt` ve `README.md` | **ORTAK** | Kütüphane bağımlılıkları ve proje dokümantasyonu |

---

## Ortak Kod Sözleşmesi (Değiştirilmeyecek Arayüzler)

Modüllerin birbirini beklemeden geliştirilebilmesi ve tek seferde birleştirilebilmesi için tüm NumPy dizileri `dtype=np.float64` standardında olmalıdır.

| Değişken | Boyut | Açıklama |
|:---|:---:|:---|
| `positions` | `(N, 2)` | Parçacıkların $(x, y)$ konumları |
| `velocities` | `(N, 2)` | Parçacıkların $(v_x, v_y)$ hızları |
| `masses` | `(N,)` | Parçacık kütleleri |

### 1. Kuvvet Fonksiyonları

`src/physics.py` ve `src/barnes_hut.py` dosyalarındaki fonksiyon imzaları, integratörlerin her iki kuvvet yöntemini de kod değiştirmeden çağırabilmesi için uyumlu tutulmalıdır.

```python
# A tarafından yazılacak (src/physics.py)
def compute_acc_direct(
    positions: np.ndarray,
    masses: np.ndarray,
    G: float = 1.0,
    eps: float = 0.05
) -> np.ndarray:
    """Çıktı: (N, 2) boyutunda float64 ivme dizisi."""


# T tarafından yazılacak (src/barnes_hut.py)
def compute_acc_bh(
    positions: np.ndarray,
    masses: np.ndarray,
    G: float = 1.0,
    eps: float = 0.05,
    theta: float = 0.5
) -> np.ndarray:
    """Çıktı: (N, 2) boyutunda float64 ivme dizisi."""


# D tarafından yazılacak (src/integrators.py)
def step_euler(
    pos: np.ndarray,
    vel: np.ndarray,
    masses: np.ndarray,
    dt: float,
    acc_fn
) -> tuple[np.ndarray, np.ndarray]:
    ...


def step_rk4(
    pos: np.ndarray,
    vel: np.ndarray,
    masses: np.ndarray,
    dt: float,
    acc_fn
) -> tuple[np.ndarray, np.ndarray]:
    ...


def step_verlet(
    pos: np.ndarray,
    vel: np.ndarray,
    masses: np.ndarray,
    dt: float,
    acc_fn
) -> tuple[np.ndarray, np.ndarray]:
    ...
```

---

## Proje Görev Listesi (Checklist)

### Ortak Başlangıç — 25 Kasım'a Kadar

- [ ] GitHub reposuna tüm ekip üyeleri (Collaborators) eklendi.
- [ ] Google Drive ortak klasörü ve 10 başlıklı rapor şablonu oluşturuldu.
- [ ] Her üye kendi modülüyle ilgili en az 2 bilimsel makale bulup Drive'a yükledi.
- [ ] 25 Kasım Çarşamba günü derste ekip üyeleri hocaya bildirildi.

### A — Fizik Çekirdeği ve Giriş/Uygulama

- [ ] `src/physics.py`: Sabit `seed` ile 2-cisim Kepler ve Dönen Disk veri üreticileri yazıldı.
- [ ] `src/physics.py`: `compute_acc_direct`, toplam enerji ($K+U$) ve momentum fonksiyonları yazıldı.
- [ ] `tests/test_physics.py`: Kuvvet simetrisi (etki-tepki) ve korunum birim testleri yazıldı.
- [ ] Rapor:
  - 3. Giriş (en az 3 IEEE kaynak)
  - 4. Yöntem (Fiziksel model ve $O(N^2)$)
  - 5. Uygulama (`float64` seçimi ve parametreler)

### D — ODE İntegratörleri ve Deney 1

- [ ] `src/integrators.py`: `step_euler`, `step_rk4` ve `step_verlet` çözücüleri yazıldı.
- [ ] `tests/test_integrators.py`: 2-cisim analitik yörünge ($\epsilon=0$) ve yakınsama derecesi birim testleri yazıldı.
- [ ] `experiments/exp1_integrators.py`: Farklı $\Delta t$ değerlerinde konum hatası ve enerji kayması ($|\Delta E / E_0|$) grafikleri üretildi.
- [ ] Rapor:
  - 4. Yöntem (ODE formülleri ve akış diyagramları)
  - 6. Deneysel Sonuçlar (Deney 1)
  - 8. Test Süreçleri

### T — Barnes-Hut Quadtree ve Deney 2–3

- [ ] `src/barnes_hut.py`: 2D Quadtree veri yapısı, kütle merkezi hesabı ve `compute_acc_bh` ($\theta$) yazıldı.
- [ ] `tests/test_barnes_hut.py`: $\theta = 0$ iken `compute_acc_direct` ile birebir eşleşme birim testi yazıldı.
- [ ] `experiments/exp2_bh_accuracy.py`: Farklı $\theta$ değerleri için bağıl kuvvet hatası grafiği üretildi.
- [ ] `experiments/exp3_scaling.py`: Parçacık sayısına ($N$) göre $O(N^2)$ ve $O(N \log N)$ süreleri ölçüldü (5 seed, ortalama ± standart sapma).
- [ ] Rapor:
  - 4. Yöntem (Quadtree ve $O(N \log N)$ Big-O analizi)
  - 6. Deneysel Sonuçlar (Deney 2–3)
  - 7. Tartışma
  - 9. Sonuç ve Öneriler

### Teslim Öncesi Son Kontrol — 11–13 Aralık

- [ ] Tüm `.py` modülleri `notebooks/MAT353_NBody_Colab.ipynb` içinde tek parça hâlinde birleştirildi ve **Run All** ile baştan sona hatasız çalıştı.
- [ ] Rapor, şablondaki 10 başlık sırasına uygun ve 8–15 sayfa aralığında tamamlandı.
- [ ] 2. Özet (200–250 kelime, referanssız) ve 10. Kaynakça (IEEE stili) kontrol edildi.
- [ ] Tüm tablo ve grafikler metin içinde yorumlandı.
- [ ] Rastgele deneylerde `seed` ve ortalama ± standart sapma belirtildi.
- [ ] Sunum dosyası en fazla 10 slayt olacak şekilde hazırlandı ve ekip görev dağılımı eklendi.
- [ ] 13 Aralık Pazar günü saat 22.00'ye kadar Rapor, Sunum ve Colab dosyası Google Classroom'a yüklendi.
