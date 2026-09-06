# Windows 11 Electron GPU process hatası: gözlemlenen ACL çözümü

[English](README.md) | [Vaka kaydı](docs/case-study.md) | [Sorun giderme](docs/troubleshooting.md) | [Kaynaklar](docs/sources.md)

Windows 11 25H2 / 26200.x üzerinde Electron veya Chromium tabanlı bir uygulama `exit_code=-2147483645`, `0x80000003` ve `GPU process isn't usable. Goodbye.` hatasıyla kapanıyorsa bu repo benzer bir sistemde işe yarayan yöntemi belgeliyor. Antigravity, uygulama klasörüne sınırlı izin eklendikten sonra **hiçbir başlatma bayrağı olmadan açıldı**.

Bu, etkilenen tek bir sistemde gözlemlenmiş çözüm/geçici çözüm kaydıdır. Microsoft, Electron, Chromium, Google veya Notion tarafından yayımlanmış resmi düzeltme değildir. Aynı hata kodunun her bilgisayarda aynı nedenden kaynaklandığı ya da sorunu Windows 25H2'nin başlattığı kanıtlanmış değildir.

## Çalışan komut

Kullanıcı aşağıdaki komutu **Komut İstemi'nde (CMD)** uyguladı:

```cmd
icacls "%LOCALAPPDATA%\Programs\antigravity" /grant *S-1-15-2-2:(OI)(CI)(RX)
```

Önce aşağıdaki kapsam, yedekleme ve geri alma adımlarını okuyun. Bu CMD sözdizimini doğrudan PowerShell'e yapıştırmayın. Komut, kurulum klasöründe ALL RESTRICTED APPLICATION PACKAGES grubuna kalıtılabilir okuma/çalıştırma izni ekler. Uygulamanın varsayılan sandbox ayarlarıyla açılmasına olanak tanımıştır; tek başına başarılı açılış, bütün süreçlerin sandbox durumunun bağımsız olarak denetlendiği anlamına gelmez.

## Belirtiler ve sistem bilgileri

Normal açılışta GPU alt süreci tekrar tekrar kapanıyor, ardından uygulama sonlanıyordu:

```text
GPU process exited unexpectedly: exit_code=-2147483645
GPU process isn't usable. Goodbye.
```

`-2147483645`, 32 bit işaretli gösterimde `0x80000003` değeridir. Microsoft bu değeri `STATUS_BREAKPOINT` olarak adlandırır. Tek başına bozuk ekran kartını veya izin reddini kanıtlamaz. [Microsoft kod açıklaması](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/bug-check-0x3b--system-service-exception) yalnızca kod eşlemesi için kullanılmıştır; olayın mavi ekran olduğu iddia edilmez.

Konuşmadaki sistem bağlamı Windows 11 25H2, `26200.9278` build ve Intel Iris Xe örneğidir. Loglarda Antigravity `2.12.2` görünür; paketlenmiş Electron/Chromium sürümleri alınmamıştır. Teşhiste Intel `32.0.101.7088` sürücüsü konuşulmuştur; bu bir sürücü yükleme önerisi değildir. Yöntem Intel'e özgü anlatılmamıştır.

Notion ve Notion Calendar için de aynı makinede açılış sorunu bildirilmiştir. Bu izin değişikliğinin başarısı yalnızca Antigravity için doğrulanmıştır.

Microsoft'un [sürüm tablosu](https://learn.microsoft.com/en-us/windows/release-health/windows11-release-information) 25H2'yi 26200 serisinde ve Genel Kullanılabilirlik kanalında da listeler. 26200 sayısı tek başına Insider demek değildir. Tam build, KB ve Insider üyeliğini ayrı kaydedin. Bu repo belirli bir güncellemeyi kesin neden veya evrensel çözüm olarak göstermiyor.

## Gerçek teşhis akışı

| İzin değişikliğinden önceki deneme | Gözlemlenen sonuç |
| --- | --- |
| Normal açılış | GPU process hatası ve kapanma |
| `--disable-gpu` | Hata devam etti |
| `--disable-gpu-sandbox` | Kapanma durdu fakat gri ekran kaldı |
| `--disable-gpu --disable-gpu-sandbox --in-process-gpu` | Gri/koyu ekran |
| `--no-sandbox --disable-gpu` | Uygulama kullanılabilir şekilde açıldı |
| Klasör izni eklendi, flagsiz açıldı | Antigravity açıldı |

Bu tablo geçmiş gözlemleri aktarır; tüm riskli denemeleri tekrarlama talimatı değildir. `--no-sandbox` tek başına test edilmiş gibi sunulmaz. İki ayar birden değiştiği için birleşik deneme tek bir mekanizmayı izole etmez. Gri ekran, destekleyen log olmadan renderer çökmesi diye yorumlanmamalıdır.

[Electron belgeleri](https://www.electronjs.org/docs/latest/tutorial/sandbox), `--no-sandbox` seçeneğinin Chromium sandbox korumasını tüm süreçlerde kapattığını ve yalnızca test için kullanılmasını belirtir. Kalıcı kısayola veya başlangıç dosyasına eklemeyin. Kısa bir teşhis denemesi gerekiyorsa güvenilmeyen içerik ve hassas oturumlardan kaçının, uygulamayı kapatıp bayrağı kaldırın.

## Neden ACL şüphesi oluştu?

Sandbox ayarları sonucu değiştirdi; klasör izni eklendikten sonra normal açılış çalıştı. Bu gözlemler, kurulum dosyalarına erişim ile sandbox başlatma işlemleri arasındaki bir etkileşimi destekler. Hangi dosyanın veya erişim kontrolünün başarısız olduğu kanıtlanmamıştır.

İzin listesinde uzun bir `S-1-15-2-...` girdisi görülmüştü. Böyle AppContainer SID'leri meşru olabilir; çözümlenmeyen isim veya "bilinmeyen hesap" görünümü tek başına bozulma ya da zararlı yazılım kanıtı değildir. Sadece tanımadığınız için silmeyin. [Microsoft SID açıklaması](https://devblogs.microsoft.com/oldnewthing/20220502-00/?p=106550).

[Electron #51761](https://github.com/electron/electron/issues/51761), benzer DACL davranışı ve aynı izin ekleme yöntemini bildiren bir kullanıcı raporudur. Bu bilgisayar için üreticinin doğruladığı kök neden değildir. İzni bir uygulamanın, agent sandbox'ın, temizleyicinin, kurucunun, antivirüsün veya Windows güncellemesinin değiştirdiğine ilişkin kesin kayıt yoktur.

Uygulamayı yeniden kurmak klasör izinlerini veya üst klasörden kalıtılan girdileri koruyabilir. GPU sürücüsünü güncellemek kurulum klasörünün DACL'sini normalde düzeltmez. İzin kaynaklı bir durumda bu işlemlerin neden yetersiz kalabileceğini açıklar; başka sistemlerde sürücü sorunu olmadığını kanıtlamaz. Sırf bu hata var diye profil ve önbellekleri tekrar tekrar silmeyin.

## Adım adım sınırlı düzeltme

### 1. Uygulamayı kapatın, hedefi kontrol edin

Çalışmanızı kaydedin; arka plan süreçleri dahil Antigravity'yi tamamen kapatın. Kısayol Özellikleri veya Görev Yöneticisi'ndeki Dosya konumunu aç ile gerçek kurulum klasörünü bulun. CMD'de:

```cmd
echo "%LOCALAPPDATA%\Programs\antigravity"
dir "%LOCALAPPDATA%\Programs\antigravity\Antigravity.exe"
icacls "%LOCALAPPDATA%\Programs\antigravity"
```

Yol uyuşmuyorsa durun. Hedef `%APPDATA%\Antigravity` veri klasörü veya tüm kullanıcı profili değildir. Alt klasörlü yedeklemeden önce dışarı yönlenen junction/sembolik bağlantıları kontrol edin; varsa bu genel prosedüre devam etmeyin. WindowsApps sahipliğini almayın ve kurumsal izin politikasını aşmayın.

### 2. Değişiklikten önce DACL yedeği alın

Her deneme için ayrı yedek kullanın. CMD'de satırları tek tek çalıştırın:

```cmd
set "ACL_BACKUP=%TEMP%\antigravity-acl-%RANDOM%-%RANDOM%"
mkdir "%ACL_BACKUP%"
echo Backup location: "%ACL_BACKUP%"
pushd "%LOCALAPPDATA%\Programs"
icacls "antigravity" /save "%ACL_BACKUP%\before.dacl" /t
echo Exit code: %ERRORLEVEL%
popd
```

Çıkış kodu 0 ve başarısız dosya sayısı 0 olmadan ilerlemeyin. Yazdırılan yedek yolunu kaydedin; geçici dosya temizliğinden önce yedeği özel ve kalıcı bir konuma kopyalayın. Bu yedek DACL'leri kapsar; dosya içeriğinin, sahipliğin veya tüm güvenlik bilgilerinin yedeği değildir. Yedeği GitHub'a yüklemeyin.

Yetki hatası çıkarsa aynı Windows hesabıyla yönetici CMD açıp yolları kontrol ederek hazırlığı tekrarlayın. Farklı yönetici hesabı `%LOCALAPPDATA%` hedefini değiştirir. Kaydetme/izin ekleme çalışsa bile geri yükleme yönetici yetkisi isteyebilir. UAC'yi kapatmayın, sahiplik sıfırlamayın.

### 3. İzni ekleyin

```cmd
icacls "%LOCALAPPDATA%\Programs\antigravity" /grant *S-1-15-2-2:(OI)(CI)(RX)
echo Exit code: %ERRORLEVEL%
```

Gerçek vakada bir öğe başarıyla işlendi, hata olmadı. Komut kök uygulama klasörünü hedefler; kalıtım alt öğeleri de etkileyebilir. Ek `/t`, `/reset`, `/grant:r` veya Full Control kullanmayın.

PowerShell tercih ediyorsanız, yedeği aldıktan sonra CMD komutu yerine şu eşdeğeri kullanın:

```powershell
icacls "$env:LOCALAPPDATA\Programs\antigravity" /grant '*S-1-15-2-2:(OI)(CI)(RX)'
$LASTEXITCODE
```

### 4. Flagsiz doğrulayın

```cmd
icacls "%LOCALAPPDATA%\Programs\antigravity"
icacls "%LOCALAPPDATA%\Programs\antigravity\Antigravity.exe"
"%LOCALAPPDATA%\Programs\antigravity\Antigravity.exe"
```

Klasörde `S-1-15-2-2` veya yerelleştirilmiş grup adını ve RX/kalıtım bilgisini kontrol edin. EXE ve ilgili kaynak dosyalarda kalıtılmış erişimi inceleyin. Korumalı kalıtım veya deny girdisi varsa kapsamı büyütmeyin; nedenini araştırın.

Kısayol, sarmalayıcı ve başlangıç girdilerindeki teşhis bayraklarını kaldırın. Daha önce eklediğiniz sandbox kapatan ortam değişkenlerini kontrol edin. Arayüzü ve sıradan bir iş akışını deneyin; tam kapatıp yeniden açın. Yeniden başlatma ve uygulama güncellemesi sonrasında tekrar kontrol edin. Bu uzun süreli kontroller vaka kaydında doğrulanmış değildir; doğrulanan sonuç ilk flagsiz açılıştır.

### 5. Gerekirse geri alın

Değişiklik öncesinde hedef klasörde bu SID için **açık bir grant yoksa** ve daha sonra korunması gereken izin değişiklikleri yapılmadıysa:

```cmd
icacls "%LOCALAPPDATA%\Programs\antigravity" /remove:g *S-1-15-2-2
echo Exit code: %ERRORLEVEL%
```

Bu işlem yalnızca eklenen RX bitlerini çıkarmaz; hedef nesnedeki SID grant'lerini kaldırır. Önceden aynı SID'ye izin verilmişse DACL yedeğini geri yükleyin. Kaldırma komutuna `/t` eklemeyin.

`ACL_BACKUP` değişkeninin tanımlandığı aynı CMD oturumunda:

```cmd
icacls "%LOCALAPPDATA%\Programs" /restore "%ACL_BACKUP%\before.dacl"
echo Exit code: %ERRORLEVEL%
```

CMD'yi yeniden veya yönetici olarak açtıysanız önce `ACL_BACKUP` değişkenini kaydettiğiniz gerçek yedek yoluna ayarlayın. Hedef kullanıcının kurulum yolunu da doğrulayın. Yedek `antigravity` göreli yoluyla alındığından geri yükleme hedefi üst klasör `Programs` olur.

Çıkış kodu 0 olmalı; tüm çıktıyı okuyup klasör ve alt dosya izinlerini karşılaştırın. Geri yükleme, özetinde sıfır başarısız dosya yazsa bile ayrıcalık eksikliğiyle başarısız olabilir. Yedekten sonra kurucu/politika izinleri değiştirdiyse geri yükleme bunları ezebilir. Sonradan oluşan dosyalar yedekte tek tek yer almaz; güncelleme sonrası tam ağaç geri dönüşü garantisi vermeyin. [icacls referansı](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls).

## ACL açıklaması ve risk

`S-1-15-2-2`, ALL RESTRICTED APPLICATION PACKAGES grubudur; `S-1-15-2-1` ile aynı değildir. Baştaki `*` sayısal SID içindir. `(OI)(CI)` dosya ve klasör kalıtımı, `(RX)` okuma/çalıştırma anlamına gelir. `/grant` önceki açık izinlere ekler.

İzin sadece Antigravity kimliğine verilmez; bir paket grubunun seçilen klasöre erişimi genişler. Klasörde kişisel belgeler, anahtarlar veya oturum sırları varsa bu genel yöntemi uygulamayın. Bu işlem risksiz değildir; yazma veya Full Control vermediği halde okuma kapsamını artırır. Deny girdileri, korumalı alt DACL'ler ve üst klasör geçiş izinleri ayrıca değerlendirilmelidir.

Komutu sürücü köküne, `C:\Windows` klasörüne, bütün kullanıcı profiline, `%LOCALAPPDATA%`, `%APPDATA%` veya tüm Electron uygulamalarına topluca uygulamayın. Başarılı izin komutu, etkili erişimin tam denetimi değildir.

## Diğer uygulamalar ve sorun giderme

Notion/Notion Calendar için kurulum klasörünü ayrıca bulun; Antigravity yolunu körlemesine değiştirmeyin. Sürümlü `app-*` dizinleri ve güncelleyici yapısı farklı olabilir. Microsoft Store/WindowsApps veya kurum tarafından yönetilen kurulumlarda destek kanalını kullanın. Adımlar [uyarlama rehberinde](docs/other-apps.md) ayrıntılıdır.

Gri ekran sürüyorsa, yol bulunamıyorsa, izin reddediliyorsa veya güncelleme sonrası sorun dönüyorsa [sorun giderme rehberini](docs/troubleshooting.md) kullanın. Full Control, toplu ACL reset, SID silme ve antivirüsü kalıcı kapatma yoluna gitmeyin. İşe yaramayan denemeleri de sürüm bilgileri ve temizlenmiş loglarla bildirin. İngilizce ve Türkçe katkılar kabul edilir; [CONTRIBUTING](CONTRIBUTING.md), [SECURITY](SECURITY.md) ve issue formunu kullanın.

Önerilen repo adı `electron-windows-gpu-sandbox-acl-fix`. [PUBLISHING.md](PUBLISHING.md) açıklamayı, konuları ve yayın adımlarını içerir. Belgeler ve örnekler [MIT lisanslıdır](LICENSE); üreticilerin uygulama dosyaları dahil değildir.

Arama terimleri: Windows 11 25H2 Electron açılmıyor; 26200 GPU process hatası; Antigravity gri ekran; Notion açılmıyor; Notion Calendar çöküyor; Chromium sandbox izin sorunu; 0x80000003; exit_code=-2147483645; GPU process isn't usable Goodbye; ALL RESTRICTED APPLICATION PACKAGES; S-1-15-2-2; icacls RX; Intel Iris Xe. Bu ifadeler bulunabilirlik içindir, her uygulamada çözüm doğrulandığı anlamına gelmez.
