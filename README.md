# Укрощение Windows. Устанавливаем систему правильно: debloating, загрузка из VHDX, OPNsense firewall, ProtonVPN

Более 15 лет в качестве основной операционной системы на моих ПК были различные дистрибутивы Linux. В 2020 году я приобрёл Surface Pro 7 - полноценный компьютер в форм-факторе планшета, и, благодаря пассивной системе охлождения, совершенно бесшумный. С момента выпуска Surface Pro 7 прошло 4 года, но в официальном ядре Linux до сих пор нет поддержки сенсорного экрана и веб-камеры для этого устройства. По этой причине на моём компьютере установлено две системы: Fedora и Windows. Если установить Windows с параметрами по умолчанию, то Microsoft собирает информацию о каждой нажатой клавише, каждой просмотренной странице и кто знает что ещё. По этой причине использование Windows всегда вызывало у меня дискомфорт и внутреннее сопротивление. Недавно мне потребовалось освободить дисковое пространство и я решил использовать эту возможность чтобы переустановить Windows правильно. В моём случае правильно означает:

1. [Удаление предустановленных компонентов](https://github.com/ruslanbay/linuxoid-vs-windows/blob/main/README.md#%D0%BF%D0%BE%D0%B4%D0%B3%D0%BE%D1%82%D0%BE%D0%B2%D0%BA%D0%B0-%D0%B7%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BE%D1%87%D0%BD%D0%BE%D0%B3%D0%BE-%D0%BE%D0%B1%D1%80%D0%B0%D0%B7%D0%B0), которыми я не пользуюсь: игры, программы для рисования и прочее
2. [Отключение сбора и отправки телеметрии](https://github.com/ruslanbay/linuxoid-vs-windows/blob/main/README.md#%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D0%BE%D0%BF%D0%B5%D1%80%D0%B0%D1%86%D0%B8%D0%BE%D0%BD%D0%BD%D0%BE%D0%B9-%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D1%8B)
3. [Наличие воспроизводимого образа системы](https://github.com/ruslanbay/linuxoid-vs-windows/blob/main/README.md#%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D1%8B-%D0%BD%D0%B0-%D0%B2%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B9-%D0%B4%D0%B8%D1%81%D0%BA-vhdx)
4. [Блокировка нежелательного трафика с помощью firewall](https://github.com/ruslanbay/linuxoid-vs-windows/blob/main/README.md#opnsense-firewall)
5. [Туннелирование всего трафика через VPN](https://github.com/ruslanbay/linuxoid-vs-windows/blob/main/README.md#%D1%82%D1%83%D0%BD%D0%BD%D0%B5%D0%BB%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D1%82%D1%80%D0%B0%D1%84%D0%B8%D0%BA%D0%B0-%D1%87%D0%B5%D1%80%D0%B5%D0%B7-vpn)
6. [Предотвращение утечек DNS запросов](https://github.com/ruslanbay/linuxoid-vs-windows/blob/main/README.md#%D0%BF%D1%80%D0%B5%D0%B4%D0%BE%D1%82%D0%B2%D1%80%D0%B0%D1%89%D0%B5%D0%BD%D0%B8%D0%B5-%D1%83%D1%82%D0%B5%D1%87%D0%B5%D0%BA-dns-%D0%B7%D0%B0%D0%BF%D1%80%D0%BE%D1%81%D0%BE%D0%B2)

### Подготовка загрузочного образа

Первое на что я обратил внимание - официальный установочный образ Windows 11 не обновлялся более года. В классическом случае пользователь скачивает образ, устанавливает систему, затем подключается к Сети и загружает обновления. Проблема в том, что за прошедший год Microsoft выпустила множество заплаток, которые закрывают серьёзные уязвимости. Подключаться к Сети без этих заплаток небезопасно. Мне не удалось найти альтернативу Fedora Netinstall для Windows. Изначально я планировал заранее скачать Cumulative Update Package с [сайта Microsoft](https://catalog.update.microsoft.com), установить систему и обновить её offline. Но затем я обнаружил более интересный способ установки обновлений. Windows поставляется с инструментом Deployment Image Servicing and Management (DISM.exe), который позволяет модифицировать непосредственно установочный образ. Помимо обновлений, DISM позволяет установить драйвера, удалить несипользуемые приложения и "фичи", изменить настройки системы через редактирование регистра. Но обо всём по порядку.

Для начала скачаем установочный iso с [официального сайта Microsoft](https://www.microsoft.com/software-download/windows11). Я буду работать со следующим файлом:

```
Win11_23H2_EnglishInternational_x64.iso
SHA256: 0EEA13C537E39CD1631B1758CD98847CF0D88BD0C32A8D69D9A02723AE79A612
SIZE: 6.22 GB
```

Далее смонтируем iso-образ и из каталога `sources` скопируем файл `install.wim`. С этой копией мы и будем работать далее. В моём случае полный путь выглядит так: `E:\user_files\install.wim`. Для работы с утилитой DISM нам необходимо открыть Windows Terminal или консоль PowerShell с правами Администратора. Первое что мы сделаем - посмотрим список редакций Windows, которые доступны для установки:

```PowerShell
Dism /Get-ImageInfo /ImageFile:"E:\user_files\install.wim"

Deployment Image Servicing and Management tool
Version: 10.0.22621.1

Details for image : F:\sources\install.wim

Index : 1
Name : Windows 11 Home
Description : Windows 11 Home
Size : 18,358,026,779 bytes

Index : 2
Name : Windows 11 Home N
Description : Windows 11 Home N
Size : 17,679,716,517 bytes

Index : 3
Name : Windows 11 Home Single Language
Description : Windows 11 Home Single Language
Size : 18,342,740,416 bytes

Index : 4
Name : Windows 11 Education
Description : Windows 11 Education
Size : 18,626,334,585 bytes

Index : 5
Name : Windows 11 Education N
Description : Windows 11 Education N
Size : 17,939,693,652 bytes

Index : 6
Name : Windows 11 Pro
Description : Windows 11 Pro
Size : 18,641,679,997 bytes

Index : 7
Name : Windows 11 Pro N
Description : Windows 11 Pro N
Size : 17,974,447,399 bytes

Index : 8
Name : Windows 11 Pro Education
Description : Windows 11 Pro Education
Size : 18,626,284,795 bytes

Index : 9
Name : Windows 11 Pro Education N
Description : Windows 11 Pro Education N
Size : 17,939,642,962 bytes

Index : 10
Name : Windows 11 Pro for Workstations
Description : Windows 11 Pro for Workstations
Size : 18,626,309,690 bytes

Index : 11
Name : Windows 11 Pro N for Workstations
Description : Windows 11 Pro N for Workstations
Size : 17,939,668,307 bytes
```

Я собираюсь установить Windows 11 Pro. Чтобы посмотреть дополнительную информацию об этой редакции выполним ту же самую команду с параметром `/index:6`:

```PowerShell
Dism /Get-ImageInfo /ImageFile:"E:\user_files\install.wim" /index:6
Index : 6
Name : Windows 11 Pro
Description : Windows 11 Pro
Size : 18,641,679,997 bytes
WIM Bootable : No
Architecture : x64
Hal : <undefined>
Version : 10.0.22621
ServicePack Build : 2428
ServicePack Level : 0
Edition : Professional
Installation : Client
ProductType : WinNT
ProductSuite : Terminal Server
System Root : WINDOWS
Directories : 26826
Files : 113746
Created : 01/10/2023 - 13:05:52
Modified : 01/10/2023 - 13:54:01
Languages :
        en-GB (Default)
```

Далее я в файловом менеджере создам каталог `D:\image` и с помощью следующей команды смонтирую в него образ `E:\user_files\install.wim`:

```PowerShell
Dism /Mount-Image /ImageFile:"E:\user_files\install.wim" /index:6 /MountDir:"D:\image"
```

#### Установка обновлений
Первое что мы сделаем - добавим в образ обновления операционной системы. Каждый месяц Microsoft выпускает пакеты Cumulative Update, которые содержат обновления безопасности и новые фичи. Чтобы обновить систему до актуального состояния необходимо установить только самый последний Cumulative Update Package. Перейдём в [Microsoft Update Catalog](https://www.catalog.update.microsoft.com/Search.aspx?q=Cumulative+Update+for+Windows+11+for+x64) и скачаем обновления для Windows и .NET Framework. Для установки обновлений выполним следующую команду:

```PowerShell
Dism /Image:"D:\image" /Add-Package /PackagePath:"E:\user_files\2023-10 Cumulative Update for Windows 11 Version 23H2 for x64-based Systems (KB5031354).msu"
Dism /Image:"D:\image" /Add-Package /PackagePath:"E:\user_files\2023-10 Cumulative Update for .NET Framework 3.5 and 4.8.1 for Windows 11, version 23H2 for x64 (KB5030651).msu"
```

#### Получаем список активированных фич
```PowerShell
Dism /Image:"D:\image" /Get-Features /English /Format:Table | Select-String -pattern ".+Enabled.*"
Windows-Defender-Default-Definitions        | Enabled
Printing-PrintToPDFServices-Features        | Enabled
MSRDC-Infrastructure                        | Enabled
MicrosoftWindowsPowerShellV2Root            | Enabled
MicrosoftWindowsPowerShellV2                | Enabled
NetFx4-AdvSrvs                              | Enabled
WCF-Services45                              | Enabled
WCF-TCP-PortSharing45                       | Enabled
MediaPlayback                               | Enabled
SearchEngine-Client-Package                 | Enabled
WorkFolders-Client                          | Enabled
Printing-Foundation-Features                | Enabled
Printing-Foundation-InternetPrinting-Client | Enabled
SmbDirect                                   | Enabled
```

#### Удаляем неиспользуемые фичи:
```PowerShell
Dism /Image:"D:\image" /Disable-Feature /FeatureName:Windows-Defender-Default-Definitions
Dism /Image:"D:\image" /Disable-Feature /FeatureName:Printing-PrintToPDFServices-Features
Dism /Image:"D:\image" /Disable-Feature /FeatureName:MSRDC-Infrastructure
Dism /Image:"D:\image" /Disable-Feature /FeatureName:MicrosoftWindowsPowerShellV2Root
Dism /Image:"D:\image" /Disable-Feature /FeatureName:MicrosoftWindowsPowerShellV2
Dism /Image:"D:\image" /Disable-Feature /FeatureName:NetFx4-AdvSrvs
Dism /Image:"D:\image" /Disable-Feature /FeatureName:WCF-Services45
Dism /Image:"D:\image" /Disable-Feature /FeatureName:WCF-TCP-PortSharing45
Dism /Image:"D:\image" /Disable-Feature /FeatureName:MediaPlayback
Dism /Image:"D:\image" /Disable-Feature /FeatureName:SearchEngine-Client-Package
Dism /Image:"D:\image" /Disable-Feature /FeatureName:WorkFolders-Client
Dism /Image:"D:\image" /Disable-Feature /FeatureName:Printing-Foundation-Features
Dism /Image:"D:\image" /Disable-Feature /FeatureName:Printing-Foundation-InternetPrinting-Client
Dism /Image:"D:\image" /Disable-Feature /FeatureName:SmbDirect
```

#### Получаем список предустановленных системных компонентов
```PowerShell
Dism /Image:"D:\image" /Get-Capabilities /Format:Table | Select-String "Installed"

App.StepsRecorder~~~~0.0.1.0                                   | Installed
Browser.InternetExplorer~~~~0.0.11.0                           | Installed
DirectX.Configuration.Database~~~~0.0.1.0                      | Installed
Hello.Face.20134~~~~0.0.1.0                                    | Installed
Language.Basic~~~en-GB~0.0.1.0                                 | Installed
Language.Handwriting~~~en-GB~0.0.1.0                           | Installed
Language.OCR~~~en-GB~0.0.1.0                                   | Installed
Language.Speech~~~en-GB~0.0.1.0                                | Installed
Language.TextToSpeech~~~en-GB~0.0.1.0                          | Installed
MathRecognizer~~~~0.0.1.0                                      | Installed
Media.WindowsMediaPlayer~~~~0.0.12.0                           | Installed
Microsoft.Wallpapers.Extended~~~~0.0.1.0                       | Installed
Microsoft.Windows.Ethernet.Client.Intel.E1i68x64~~~~0.0.1.0    | Installed
Microsoft.Windows.Ethernet.Client.Intel.E2f68~~~~0.0.1.0       | Installed
Microsoft.Windows.Ethernet.Client.Realtek.Rtcx21x64~~~~0.0.1.0 | Installed
Microsoft.Windows.Ethernet.Client.Vmware.Vmxnet3~~~~0.0.1.0    | Installed
Microsoft.Windows.Notepad.System~~~~0.0.1.0                    | Installed
Microsoft.Windows.PowerShell.ISE~~~~0.0.1.0                    | Installed
Microsoft.Windows.Wifi.Client.Broadcom.Bcmpciedhd63~~~~0.0.1.0 | Installed
Microsoft.Windows.Wifi.Client.Broadcom.Bcmwl63al~~~~0.0.1.0    | Installed
Microsoft.Windows.Wifi.Client.Broadcom.Bcmwl63a~~~~0.0.1.0     | Installed
Microsoft.Windows.Wifi.Client.Intel.Netwbw02~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Intel.Netwew00~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Intel.Netwew01~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Intel.Netwlv64~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Intel.Netwns64~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Intel.Netwsw00~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Intel.Netwtw02~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Intel.Netwtw04~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Intel.Netwtw06~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Intel.Netwtw08~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Intel.Netwtw10~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Marvel.Mrvlpcie8897~~~~0.0.1.0   | Installed
Microsoft.Windows.Wifi.Client.Qualcomm.Athw8x~~~~0.0.1.0       | Installed
Microsoft.Windows.Wifi.Client.Qualcomm.Athwnx~~~~0.0.1.0       | Installed
Microsoft.Windows.Wifi.Client.Qualcomm.Qcamain10x64~~~~0.0.1.0 | Installed
Microsoft.Windows.Wifi.Client.Ralink.Netr28x~~~~0.0.1.0        | Installed
Microsoft.Windows.Wifi.Client.Realtek.Rtl8187se~~~~0.0.1.0     | Installed
Microsoft.Windows.Wifi.Client.Realtek.Rtl8192se~~~~0.0.1.0     | Installed
Microsoft.Windows.Wifi.Client.Realtek.Rtl819xp~~~~0.0.1.0      | Installed
Microsoft.Windows.Wifi.Client.Realtek.Rtl85n64~~~~0.0.1.0      | Installed
Microsoft.Windows.Wifi.Client.Realtek.Rtwlane01~~~~0.0.1.0     | Installed
Microsoft.Windows.Wifi.Client.Realtek.Rtwlane13~~~~0.0.1.0     | Installed
Microsoft.Windows.Wifi.Client.Realtek.Rtwlane~~~~0.0.1.0       | Installed
Microsoft.Windows.WordPad~~~~0.0.1.0                           | Installed
OneCoreUAP.OneSync~~~~0.0.1.0                                  | Installed
OpenSSH.Client~~~~0.0.1.0                                      | Installed
Print.Management.Console~~~~0.0.1.0                            | Installed
Windows.Kernel.LA57~~~~0.0.1.0                                 | Installed
WMIC~~~~                                                       | Installed
```

#### Удаляем неиспользуемые компоненты
```PowerShell
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:App.StepsRecorder~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Browser.InternetExplorer~~~~0.0.11.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Hello.Face.20134~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Language.Handwriting~~~en-GB~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Language.OCR~~~en-GB~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Language.Speech~~~en-GB~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Language.TextToSpeech~~~en-GB~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:MathRecognizer~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Media.WindowsMediaPlayer~~~~0.0.12.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Wallpapers.Extended~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Ethernet.Client.Intel.E1i68x64~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Ethernet.Client.Intel.E2f68~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Ethernet.Client.Realtek.Rtcx21x64~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Ethernet.Client.Vmware.Vmxnet3~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Notepad.System~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.PowerShell.ISE~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Broadcom.Bcmpciedhd63~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Broadcom.Bcmwl63al~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Broadcom.Bcmwl63a~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Intel.Netwbw02~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Intel.Netwew00~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Intel.Netwew01~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Intel.Netwlv64~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Intel.Netwns64~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Intel.Netwsw00~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Intel.Netwtw02~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Intel.Netwtw04~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Intel.Netwtw06~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Intel.Netwtw08~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Intel.Netwtw10~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Marvel.Mrvlpcie8897~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Qualcomm.Athw8x~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Qualcomm.Athwnx~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Qualcomm.Qcamain10x64~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Ralink.Netr28x~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Realtek.Rtl8187se~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Realtek.Rtl8192se~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Realtek.Rtl819xp~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Realtek.Rtl85n64~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Realtek.Rtwlane01~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Realtek.Rtwlane13~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.Wifi.Client.Realtek.Rtwlane~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Microsoft.Windows.WordPad~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:OneCoreUAP.OneSync~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:OpenSSH.Client~~~~0.0.1.0
Dism /Image:"D:\image" /Remove-Capability /CapabilityName:Print.Management.Console~~~~0.0.1.0
```

#### Получаем список предустановленных пакетов
```PowerShell
Dism /Image:"D:\image" /Get-Packages | Select-String -pattern "Package Identity : .+"
Package Identity : Microsoft-OneCore-DirectX-Database-FOD-Package~31bf3856ad364e35~amd64~~10.0.22621.2428
Package Identity : Microsoft-Windows-Client-LanguagePack-Package~31bf3856ad364e35~amd64~en-GB~10.0.22621.2428
Package Identity : Microsoft-Windows-FodMetadata-Package~31bf3856ad364e35~amd64~~10.0.22621.1
Package Identity : Microsoft-Windows-Foundation-Package~31bf3856ad364e35~amd64~~10.0.22621.1
Package Identity : Microsoft-Windows-Kernel-LA57-FoD-Package~31bf3856ad364e35~amd64~~10.0.22621.2428
Package Identity : Microsoft-Windows-LanguageFeatures-Basic-en-gb-Package~31bf3856ad364e35~amd64~~10.0.22621.2428
Package Identity : Microsoft-Windows-WMIC-FoD-Package~31bf3856ad364e35~amd64~en-GB~10.0.22621.1
Package Identity : Microsoft-Windows-WMIC-FoD-Package~31bf3856ad364e35~amd64~~10.0.22621.2428
Package Identity : Microsoft-Windows-WMIC-FoD-Package~31bf3856ad364e35~wow64~en-GB~10.0.22621.1
Package Identity : Microsoft-Windows-WMIC-FoD-Package~31bf3856ad364e35~wow64~~10.0.22621.1
Package Identity : Package_for_DotNetRollup_481~31bf3856ad364e35~amd64~~10.0.9186.2
Package Identity : Package_for_KB5027397~31bf3856ad364e35~amd64~~22621.2355.1.1
Package Identity : Package_for_RollupFix~31bf3856ad364e35~amd64~~22621.2428.1.8
Package Identity : Package_for_ServicingStack_2423~31bf3856ad364e35~amd64~~22621.2423.1.1
```

#### Получаем список пакетов, которые устанавливаются автоматически для каждого нового пользователя
```PowerShell
Dism /Image:"D:\image" /Get-ProvisionedAppxPackages | Select-String -pattern "PackageName : .+"
PackageName : Clipchamp.Clipchamp_2.2.8.0_neutral_~_yxz26nhyzhsrt
PackageName : Microsoft.549981C3F5F10_3.2204.14815.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.BingNews_4.2.27001.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.BingWeather_4.53.33420.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.DesktopAppInstaller_2022.310.2333.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.GamingApp_2021.427.138.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.GetHelp_10.2201.421.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.Getstarted_2021.2204.1.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.HEIFImageExtension_1.0.43012.0_x64__8wekyb3d8bbwe
PackageName : Microsoft.HEVCVideoExtension_1.0.50361.0_x64__8wekyb3d8bbwe
PackageName : Microsoft.MicrosoftOfficeHub_18.2204.1141.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.MicrosoftSolitaireCollection_4.12.3171.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.MicrosoftStickyNotes_4.2.2.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.Paint_11.2201.22.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.People_2020.901.1724.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.PowerAutomateDesktop_10.0.3735.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.RawImageExtension_2.1.30391.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.ScreenSketch_2022.2201.12.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.SecHealthUI_1000.22621.1.0_x64__8wekyb3d8bbwe
PackageName : Microsoft.StorePurchaseApp_12008.1001.113.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.Todos_2.54.42772.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.VCLibs.140.00_14.0.30704.0_x64__8wekyb3d8bbwe
PackageName : Microsoft.VP9VideoExtensions_1.0.50901.0_x64__8wekyb3d8bbwe
PackageName : Microsoft.WebMediaExtensions_1.0.42192.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.WebpImageExtension_1.0.42351.0_x64__8wekyb3d8bbwe
PackageName : Microsoft.Windows.Photos_21.21030.25003.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.WindowsAlarms_2022.2202.24.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.WindowsCalculator_2020.2103.8.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.WindowsCamera_2022.2201.4.0_neutral_~_8wekyb3d8bbwe
PackageName : microsoft.windowscommunicationsapps_16005.14326.20544.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.WindowsFeedbackHub_2022.106.2230.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.WindowsMaps_2022.2202.6.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.WindowsNotepad_11.2112.32.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.WindowsSoundRecorder_2021.2103.28.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.WindowsStore_22204.1400.4.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.WindowsTerminal_3001.12.10983.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.Xbox.TCUI_1.23.28004.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.XboxGameOverlay_1.47.2385.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.XboxGamingOverlay_2.622.3232.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.XboxIdentityProvider_12.50.6001.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.XboxSpeechToTextOverlay_1.17.29001.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.YourPhone_1.22022.147.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.ZuneMusic_11.2202.46.0_neutral_~_8wekyb3d8bbwe
PackageName : Microsoft.ZuneVideo_2019.22020.10021.0_neutral_~_8wekyb3d8bbwe
PackageName : MicrosoftCorporationII.QuickAssist_2022.414.1758.0_neutral_~_8wekyb3d8bbwe
PackageName : MicrosoftWindows.Client.WebExperience_421.20070.195.0_neutral_~_cw5n1h2txyewy
```

#### Удаляем неиспользуемые пакеты
```PowerShell
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Clipchamp.Clipchamp_2.2.8.0_neutral_~_yxz26nhyzhsrt
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.549981C3F5F10_3.2204.14815.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.BingNews_4.2.27001.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.BingWeather_4.53.33420.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.GamingApp_2021.427.138.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.GetHelp_10.2201.421.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.Getstarted_2021.2204.1.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.MicrosoftOfficeHub_18.2204.1141.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.MicrosoftSolitaireCollection_4.12.3171.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.MicrosoftStickyNotes_4.2.2.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.Paint_11.2201.22.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.People_2020.901.1724.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.PowerAutomateDesktop_10.0.3735.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.ScreenSketch_2022.2201.12.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.StorePurchaseApp_12008.1001.113.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.Todos_2.54.42772.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WebMediaExtensions_1.0.42192.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:microsoft.windowscommunicationsapps_16005.14326.20544.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WindowsFeedbackHub_2022.106.2230.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WindowsMaps_2022.2202.6.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WindowsNotepad_11.2112.32.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.Xbox.TCUI_1.23.28004.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.XboxGameOverlay_1.47.2385.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.XboxGamingOverlay_2.622.3232.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.XboxIdentityProvider_12.50.6001.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.XboxSpeechToTextOverlay_1.17.29001.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.YourPhone_1.22022.147.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.ZuneMusic_11.2202.46.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:Microsoft.ZuneVideo_2019.22020.10021.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:MicrosoftCorporationII.QuickAssist_2022.414.1758.0_neutral_~_8wekyb3d8bbwe
Dism /Image:"D:\image" /Remove-ProvisionedAppxPackage /PackageName:MicrosoftWindows.Client.WebExperience_421.20070.195.0_neutral_~_cw5n1h2txyewy
```

#### Выполняем оптимизацию образа после удаления пакетов
```PowerShell
Dism /Image:"D:\image" /Optimize-ProvisionedAppxPackages
```

#### Обновляем пакеты с помощью Winget
> [!NOTE]
> TODO: Добавить инструкцию как скачать пакет из Windows Store и обновить его офлайн с помощью Winget (https://store.rg-adguard.net/)


#### Устанавливаем драйверы устройств
Для устройств Surface Microsoft публикует драйвера на своём сайте в виде MSI-файла. Например, [драйвера Surface Pro 7](https://www.microsoft.com/en-us/download/details.aspx?id=100419). Чтобы извлечь из MSI собственно драйверы устройств (inf-файлы) выполним следующую команду:

```PowerShell
cd E:\user_files\
msiexec /a SurfacePro7_Win11_22000_23.101.6978.0.msi TargetDir=E:\user_files\SurfacePro7_Win11_22000_23.101.6978.0 /qn
```

Наконец, добавляем распакованные драйвера в образ:
```PowerShell
Dism /Image:"D:\image" /Add-Driver /Driver:"E:\user_files\SurfacePro7_Win11_22000_23.101.6978.0\SurfaceUpdate" /Recurse
```

#### Настройка операционной системы
> [!NOTE]
> TODO: Оформить в виде таблицы с описанием изменений, которые вносит каждая команда

##### Правка HKLM\SOFTWARE

```PowerShell
reg load HKLM\OFFLINE D:\image\Windows\System32\config\SOFTWARE
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\OOBE                 /v BypassNRO                     /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\OOBE                       /v DisablePrivacyExperience      /t REG_DWORD /d 1 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Windows Chat"             /v ChatIcon                      /t REG_DWORD /d 3 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Windows Search"           /v AllowCloudSearch              /t REG_DWORD /d 0 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Windows Search"           /v AllowCortana                  /t REG_DWORD /d 0 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Windows Search"           /v ConnectedSearchUseWeb         /t REG_DWORD /d 0 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Windows Search"           /v DisableSearch                 /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\CloudContent     /v DisableWindowsConsumerFeatures     /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\CloudContent     /v DisableSoftLanding                 /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\CloudContent     /v DisableCloudOptimizedContent       /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\CloudContent     /v DisableConsumerAccountStateContent /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\AppCompat        /v AITEnable                          /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\AppCompat        /v VDMDisallowed                      /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\AppCompat        /v DisableEngine                      /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\AppCompat        /v DisablePCA                         /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\AppCompat        /v DisableUAR                         /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\DataCollection   /v AllowTelemetry                     /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\DataCollection   /v "Allow Telemetry"                  /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\DataCollection   /v AllowDeviceNameInTelemetry         /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\DataCollection   /v DisableOneSettingsDownloads        /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\DataCollection   /v DoNotShowFeedbackNotifications     /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\DataCollection   /v EnableOneSettingsAuditing          /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\DataCollection   /v LimitDiagnosticLogCollection       /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\DataCollection   /v LimitDumpCollection                /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Dsh                      /v AllowNewsAndInterests              /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\InputPersonalization     /v RestrictImplicitTextCollection     /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\InputPersonalization     /v RestrictImplicitInkCollection      /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Microsoft\PolicyManager\default\NewsAndInterests\AllowNewsAndInterests /v value   /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Policies\DataCollection /v AllowTelemetry         /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Policies\DataCollection /v "AllowTelemetry"       /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Policies\DataCollection /v MaxTelemetryAllowed    /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Policies\DataCollection /v MicrosoftEdgeDataOptIn /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Privacy                 /v TailoredExperiencesWithDiagnosticDataEnabled /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Policies\Explorer  /v NoRecentDocsHistory            /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Policies\Explorer  /v NoDesktop                      /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Policies\Explorer  /v AllowOnlineTips                /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Policies\System    /v DisableStartupSound            /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Explorer\Advanced  /v Start_TrackDocs                /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers /v DisableAutoplay         /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers /v DisableAutoplay  /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers\EventHandlersDefaultSelection\CameraAlternate\ShowPicturesOnArrival /v "(Default)" /t REG_SZ /d MSTakeNoAction /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers\EventHandlersDefaultSelection\StorageOnArrival                      /v "(Default)" /t REG_SZ /d MSTakeNoAction /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers\UserChosenExecuteHandlers\CameraAlternate\ShowPicturesOnArrival     /v "(Default)" /t REG_SZ /d MSTakeNoAction /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers\UserChosenExecuteHandlers\StorageOnArrival                          /v "(Default)" /t REG_SZ /d MSTakeNoAction /f
reg add  HKLM\OFFLINE\Microsoft\input\Settings                            /v IsVoiceTypingKeyEnabled        /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\input\Settings                            /v RestrictImplicitInkCollection  /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Microsoft\input\Settings                            /v RestrictImplicitTextCollection /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Microsoft\input\Settings                            /v DictationEnabled               /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\input\Settings                            /v DisablePersonalization         /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Microsoft\input\Settings                            /v HarvestContacts                /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\input\Settings                            /v IsVoiceTypingKeyEnabled        /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\input\Settings                            /v InsightsEnabled                /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\InputPersonalization                      /v InsightsEnabled                /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\InputPersonalization\TrainedDataStore     /v HarvestContacts                /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\InputPersonalization\TrainedDataStore     /v TouchKeyboard_EnableKeyAudioFeedback /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Personalization\Settings                  /v AcceptedPrivacyPolicy          /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Speech_OneCore\Settings\OnlineSpeechPrivacy /v HasAccepted /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CPSS\Store\AdvertisingInfo                                  /v Value /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CPSS\Store\InkingAndTypingPersonalization                   /v Value /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\activity                     /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\appDiagnostics               /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\appointments                 /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\bluetoothSync                /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\broadFileSystemAccess        /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\cellularData                 /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\chat                         /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\contacts                     /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\documentsLibrary             /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\downloadsFolder              /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\email                        /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\gazeInput                    /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\graphicsCaptureProgrammatic  /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\graphicsCaptureWithoutBorder /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\location                     /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\microphone                   /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\musicLibrary                 /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\phoneCall                    /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\phoneCallHistory             /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\picturesLibrary              /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\radios                       /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\userAccountInformation       /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\userDataTasks                /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\userNotificationListener     /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\videosLibrary                /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\webcam                       /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\wifiData                     /v Value /t REG_SZ /d Deny /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Camera                     /v AllowCamera                                      /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\AdvertisingInfo    /v DisabledByGroupPolicy                            /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\AppPrivacy         /v LetAppsAccessMicrophone                          /t REG_DWORD /d 2 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\AppPrivacy         /v LetAppsAccessMicrophone_ForceAllowTheseApps      /t REG_MULTI_SZ /d \0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\AppPrivacy         /v LetAppsAccessMicrophone_ForceDenyTheseApps       /t REG_MULTI_SZ /d \0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\AppPrivacy         /v LetAppsAccessMicrophone_UserInControlOfTheseApps /t REG_MULTI_SZ /d \0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\LocationAndSensors /v DisableLocation                                  /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\LocationAndSensors /v DisableWindowsLocationProvider                   /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\LocationAndSensors /v DisableLocationScripting                         /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\LocationAndSensors /v DisableSensors                                   /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\FileHistory        /v Disabled                                         /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\HomeGroup          /v DisableHomeGroup                                 /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\System             /v PublishUserActivities                            /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\System             /v UploadUserActivities                             /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\System             /v EnableActivityFeed                               /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\System             /v EnableCdp                                        /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\System             /v EnableMmx                                        /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\System             /v AllowClipboardHistory                            /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\System             /v AllowCrossDeviceClipboard                        /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\System             /v AllowCrossDeviceClipboard                        /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Messenger\Client           /v CEIP                                             /t REG_DWORD /d 2 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\SQMClient\Windows          /v CEIPEnable                                       /t REG_DWORD /d 0 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Registration Wizard Control" /v NoRegistration                        /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\PCHealth\HelpSvc           /v Headlines                                        /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\PCHealth\HelpSvc           /v MicrosoftKBSearch                                /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\PCHealth\ErrorReporting    /v DoReport                                         /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\PCHealth\ErrorReporting    /v AllOrNone                                        /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\PCHealth\ErrorReporting    /v IncludeMicrosoftApps                             /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\PCHealth\ErrorReporting    /v IncludeWindowsApps                               /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\PCHealth\ErrorReporting    /v IncludeKernelFaults                              /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\PCHealth\ErrorReporting    /v IncludeShutdownErrs                              /t REG_DWORD /d 0 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Control Panel\International" /v PreventGeoIdChange                            /t REG_DWORD /d 1 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Windows Error Reporting" /v Disabled                                  /t REG_DWORD /d 1 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Windows Error Reporting" /v LoggingDisabled                           /t REG_DWORD /d 1 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Windows Error Reporting" /v AutoApproveOSDumps                        /t REG_DWORD /d 0 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Windows Error Reporting" /v DontSendAdditionalData                    /t REG_DWORD /d 1 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Windows Error Reporting\Consent" /v DefaultConsent                    /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\SearchCompanion            /v DisableContentFileUpdates                        /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\TabletPC           /v PreventHandwritingDataSharing                    /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\HandwritingErrorReports /v PreventHandwritingErrorReports              /t REG_DWORD /d 1 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows\Windows Feeds"    /v EnableFeeds                                      /t REG_DWORD /d 0 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows NT\DNSClient"     /v DoHPolicy                                        /t REG_DWORD /d 3 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows NT\Printers"      /v DisableWebPrinting                               /t REG_DWORD /d 1 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows NT\Printers"      /v PublishPrinters                                  /t REG_DWORD /d 0 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows NT\Printers"      /v RegisterSpoolerRemoteRpcEndPoint                 /t REG_DWORD /d 2 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows NT\Printers"      /v DisableWebPnPDownload                            /t REG_DWORD /d 1 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows NT\Printers"      /v DisableHTTPPrinting                              /t REG_DWORD /d 1 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows NT\IIS"           /v PreventIISInstall                                /t REG_DWORD /d 1 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Windows NT\Terminal Services" /v fDenyTSConnections                           /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Peernet                    /v Disabled                                         /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\FindMyDevice               /v AllowFindMyDevice                                /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\MicrosoftAccount           /v DisableUserAuth                                  /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\MicrosoftEdge\Main         /v PreventLiveTileDataCollection                    /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\MicrosoftEdge\Main         /v PreventFirstRunPage                              /t REG_DWORD /d 1 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\MicrosoftEdge\Internet Settings" /v ProvisionedHomePages                      /t REG_SZ    /d "https://en.wikipedia.org/wiki/Main_Page" /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\MicrosoftEdge\Internet Settings" /v DisableLockdownOfStartPages               /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Conferencing               /v NoRDS                                            /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Biometrics                 /v Enabled                                          /t REG_DWORD /d 0 /f
reg add  "HKLM\OFFLINE\Policies\Microsoft\Biometrics\Credential Provider" /v Enabled                                      /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\WindowsFirewall\StandardProfile /v DoNotAllowExceptions                        /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\ScriptedDiagnosticsProvider\Policy /v DisableQueryRemoteServer         /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\Troubleshooting\AllowRecommendations /v TroubleshootingAllowRecommendations /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\ScriptedDiagnosticsProvider\Policy   /v EnableQueryRemoteServer        /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\ScriptedDiagnostics                  /v EnableDiagnostics              /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\CurrentVersion\AppModel\StateManager /v AllowSharedLocalAppData        /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\PreviewBuilds                        /v AllowBuildPreview              /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\SettingSync                          /v DisableSettingSync             /t REG_DWORD /d 2 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\PushToInstall                                /v DisablePushToInstall               /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Speech                                       /v AllowSpeechModelUpdate             /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\OneDrive                                     /v DisableFileSyncNGSC                /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\OneDrive                                     /v PreventNetworkTrafficPreUserSignIn /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\OneDrive                             /v DisableFileSyncNGSC                /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\OneDrive                             /v PreventNetworkTrafficPreUserSignIn /t REG_DWORD /d 1 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\GameDVR                              /v AllowGameDVR                       /t REG_DWORD /d 0 /f
reg add  HKLM\OFFLINE\Policies\Microsoft\Windows\WinRM\Service\WinRS                  /v AllowRemoteShellAccess             /t REG_DWORD /d 0 /f
reg unload HKLM\OFFLINE
```

##### Правка HKLM\SYSTEM
Отключаем гибернацию, файл подкачки и удалённое подключение

```PowerShell
reg load   HKLM\OFFLINE D:\image\Windows\System32\config\SYSTEM
reg add    HKLM\OFFLINE\ControlSet001\Control\Power                                   /v HibernateEnabled        /t REG_DWORD /d 0 /f
reg add    HKLM\OFFLINE\ControlSet001\Control\Power                                   /v HibernateEnabledDefault /t REG_DWORD /d 0 /f
reg add    "HKLM\OFFLINE\ControlSet001\Control\Session Manager\Memory Management"     /v SwapfileControl         /t REG_DWORD /d 0 /f
reg add    "HKLM\OFFLINE\ControlSet001\Control\Remote Assistance"                     /v fAllowFullControl       /t REG_DWORD /d 0 /f
reg add    "HKLM\OFFLINE\ControlSet001\Control\Remote Assistance"                     /v fAllowToGetHelp         /t REG_DWORD /d 0 /f
reg add    "HKLM\OFFLINE\ControlSet001\Control\Remote Assistance"                     /v fEnableChatControl      /t REG_DWORD /d 0 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\Dnscache\Parameters                    /v EnableMDNS              /t REG_DWORD /d 0 /f
reg unload HKLM\OFFLINE
```

##### Заблокируем все входящие соединения на уровне Microsoft Defender
> [!WARNING]
> Блокировка всех входящих подключений отключит в том числе возможность беспроводного подключения ко внешним дисплеям (miracast)

```PowerShell
reg load   HKLM\OFFLINE D:\image\Windows\System32\config\SYSTEM
reg add    HKLM\OFFLINE\ControlSet001\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules /v AllBlocked /t REG_SZ /d "v2.32|Action=Block|Active=TRUE|Dir=In|Name=AllBlocked|" /f
reg unload HKLM\OFFLINE
```

#### Редактируем параметры пользователя Default

В Windows по умолчанию присутствует пользователь `Default` и его настройки автоматически наследуются для каждого нового пользователя системы. Таким образом, если мы отредактируем файлы, регистр для пользователя `Default`, то эти изменения автоматически применятся для всех пользователей, которых мы создадим после установки системы.

##### Правка HKU

> [!NOTE]
> TODO: Оформить в виде таблицы с описанием изменений, которые вносит каждая команда

```PowerShell
reg load   HKU\OFFLINE D:\image\Users\Default\NTUSER.DAT
reg delete HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Run    /v OneDriveSetup /f
reg add    HKU\OFFLINE\Software\Microsoft\OneDrive                                 /v DisableFileSyncNGSC                             /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Policies\Microsoft\Windows\CloudContent            /v DisableSpotlightCollectionOnDesktop             /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Policies\Microsoft\Windows\CloudContent            /v DisableTailoredExperiencesWithDiagnosticData    /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Policies\Microsoft\Windows\CloudContent            /v DisableWindowsConsumerFeatures                  /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Policies\Microsoft\Windows\CloudContent            /v DisableWindowsSpotlightFeatures                 /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Policies\Microsoft\Windows\CloudContent            /v DisableThirdPartySuggestions                    /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Policies\Microsoft\Windows\CloudContent            /v DisableWindowsSpotlightOnActionCenter           /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Policies\Microsoft\Windows\CloudContent            /v DisableWindowsSpotlightOnSettings               /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Policies\Microsoft\Windows\CloudContent            /v DisableWindowsSpotlightWindowsWelcomeExperience /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Policies\Microsoft\Windows\Explorer                /v DisableSearchBoxSuggestions     /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\SOFTWARE\Policies\Microsoft\MicrosoftEdge\BooksLibrary      /v EnableExtendedBooksTelemetry    /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Policies\Microsoft\Windows\DataCollection          /v AllowTelemetry                  /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Policies\DataCollection /v MicrosoftEdgeDataOptIn    /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Privacy           /v TailoredExperiencesWithDiagnosticDataEnabled /t REG_DWORD /d 0 /f
reg add    "HKU\OFFLINE\AppEvents\Schemes\Apps\.Default\.Default\.None"            /ve /f
reg add    HKU\OFFLINE\AppEvents\Schemes                                           /ve /d ".None" /f
reg add    "HKU\OFFLINE\Control Panel\International\User Profile"                  /v HttpAcceptLanguageOptOut  /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Microsoft\GameBar                                  /v AutoGameModeEnabled       /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\GameBar                                  /v UseNexusForGameBarEnabled /t REG_DWORD /d 0 /f
reg add    "HKU\OFFLINE\Software\Microsoft\Windows NT\CurrentVersion\Windows"      /v LegacyDefaultPrinterMode  /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\AdvertisingInfo   /v Enabled                   /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CDP               /v CdpSessionUserAuthzPolicy /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CDP               /v EnableRemoteLaunchToast   /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CDP               /v NearShareChannelUserAuthzPolicy /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CDP               /v RomeSdkChannelUserAuthzPolicy   /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CDP\SettingsPage  /v BluetoothLastDisabledNearShare  /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer          /v ShowRecent               /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer          /v ShowFrequent             /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer          /v ShowCloudFilesInQuickAccess /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v TaskbarAl                /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v TaskbarMn                /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v Hidden                   /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v HideFileExt              /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v ShowTaskViewButton       /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v HideIcons                /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v LaunchTo                 /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v Start_TrackDocs          /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v Start_TrackProgs         /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v Start_IrisRecommendations /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v EnableSnapBar            /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v EnableTaskGroups         /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v MultiTaskingAltTabFilter /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers /v DisableAutoplay  /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers\EventHandlersDefaultSelection\CameraAlternate\ShowPicturesOnArrival /v "(Default)" /t REG_SZ /d MSTakeNoAction /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers\EventHandlersDefaultSelection\StorageOnArrival                      /v "(Default)" /t REG_SZ /d MSTakeNoAction /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers\UserChosenExecuteHandlers\CameraAlternate\ShowPicturesOnArrival     /v "(Default)" /t REG_SZ /d MSTakeNoAction /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers\UserChosenExecuteHandlers\StorageOnArrival                          /v "(Default)" /t REG_SZ /d MSTakeNoAction /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Search            /v SearchboxTaskbarMode     /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Search            /v SearchboxTaskbarModeCache /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Search            /v BingSearchEnabled        /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\SearchSettings    /v IsAADCloudSearchEnabled  /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\SearchSettings    /v IsDeviceSearchHistoryEnabled /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\SearchSettings    /v IsDynamicSearchBoxEnabled /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\SearchSettings    /v IsMSACloudSearchEnabled  /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\SearchSettings    /v SafeSearchMode           /t REG_DWORD /d 2 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer /v NoDesktop                /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Microsoft\TabletTip\1.7                            /v EnableKeyAudioFeedback   /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\TabletTip\1.7                            /v EnableAutocorrection     /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\TabletTip\1.7                            /v EnableSpellchecking      /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\TabletTip\1.7                            /v TipbandDesiredVisibility /t REG_DWORD /d 1 /f
reg add    "HKU\OFFLINE\Keyboard Layout\Preload"                                   /v 2                        /t REG_SZ    /d 00000419 /f
reg add    HKU\OFFLINE\Software\Microsoft\input\Settings                           /v IsVoiceTypingKeyEnabled  /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\input\Settings                           /v InsightsEnabled          /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\input\Settings                           /v HarvestContacts          /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\input\Settings                           /v RestrictImplicitInkCollection  /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Microsoft\input\Settings                           /v RestrictImplicitTextCollection /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Microsoft\InputPersonalization                     /v InsightsEnabled          /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\InputPersonalization\TrainedDataStore    /v HarvestContacts          /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\InputPersonalization                     /v RestrictImplicitInkCollection  /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Microsoft\InputPersonalization                     /v RestrictImplicitTextCollection /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Microsoft\Personalization\Settings                 /v AcceptedPrivacyPolicy          /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Siuf\Rules                               /v NumberOfSIUFInPeriod           /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Speech_OneCore\Settings\OnlineSpeechPrivacy /v HasAccepted /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CPSS\Store\InkingAndTypingPersonalization  /v Value /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\Diagnostics\DiagTrack  /v ShowedToastAtLevel                /t REG_DWORD /d 1 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\UserProfileEngagement  /v ScoobeSystemSettingEnabled        /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v ContentDeliveryAllowed            /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v DesktopSpotlightOemEnabled        /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v FeatureManagementEnabled          /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v OemPreInstalledAppsEnabled        /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled           /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEverEnabled       /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v SilentInstalledAppsEnabled        /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v SoftLandingEnabled                /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v SubscribedContent-310093Enabled   /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v SubscribedContent-338389Enabled   /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v SubscribedContent-338393Enabled   /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v SubscribedContent-353694Enabled   /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v SubscribedContent-353696Enabled   /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v SubscribedContent-88000326Enabled /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v SystemPaneSuggestionsEnabled      /t REG_DWORD /d 0 /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\activity                     /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\appDiagnostics               /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\appointments                 /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\bluetoothSync                /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\broadFileSystemAccess        /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\cellularData                 /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\chat                         /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\contacts                     /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\documentsLibrary             /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\downloadsFolder              /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\email                        /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\gazeInput                    /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\graphicsCaptureProgrammatic  /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\graphicsCaptureWithoutBorder /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\location                     /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\microphone                   /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\musicLibrary                 /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\phoneCall                    /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\phoneCallHistory             /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\picturesLibrary              /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\radios                       /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\userAccountInformation       /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\userDataTasks                /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\userNotificationListener     /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\videosLibrary                /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\webcam                       /v Value /t REG_SZ /d Deny /f
reg add    HKU\OFFLINE\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\wifiData                     /v Value /t REG_SZ /d Deny /f
reg unload HKU\OFFLINE
```

##### Apply-Unattend
Для изменения/отключения некоторых настроек системы можно воспользоваться так называемыми `Answer files` или `Unattend files`. Например, для каждого нового пользователя Windows по умолчанию устанавливает Microsoft Teams. Избавиться от этой "фичи" можно только с помощью `Answer files`.

```PowerShell
Dism /Image:"D:\image" /Apply-Unattend:E:\user_files\ApplyUnattend.xml
```
Содержимое файла `ApplyUnattend.xml`:
```XML
<!-- TODO: Add Encrypted DNS over HTTPS -->
<!-- 
    https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/components-b-unattend
-->
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <settings pass="offlineServicing">
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <!-- Disable MSTeams auto install -->
            <ConfigureChatAutoInstall>False</ConfigureChatAutoInstall>
            <OOBE>
                    <!-- Allow to create local profile -->
                    <HideOnlineAccountScreens>true</HideOnlineAccountScreens>
                    <!--
                        Personalize speech, typing, and inking input by sending contacts and calendar details, along with other associated input data to Microsoft
                        We will disable this "features" automatically.
                    -->
                    <ProtectYourPC>3</ProtectYourPC>
            </OOBE>
        </component>
        <component name="Microsoft-Windows-Authentication-AuthUI" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <DisableStartupSound>true</DisableStartupSound>
        </component>
        <component name="Microsoft-Windows-ErrorReportingCore" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <DisableWER>1</DisableWER>
        </component>
        <component name="Microsoft-Windows-International-Core" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <InputLocale>en-US;ru-RU</InputLocale>
        </component>
        <component name="Microsoft-Windows-International-Core-WinPE" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <InputLocale>en-US;ru-RU</InputLocale>
        </component>
        <component name="Microsoft-Windows-Printing-Spooler-Core" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <Start>0</Start>
        </component>
    </settings>
</unattend>
```

##### Задаём параметры MS Edge
Вы можете скопировать конфигурационный файл `Preferences` из каталога `%USERNAME%\AppData\Local\Microsoft\Edge\User Data\Default\` и поместить его папку пользователя `Default`. Таким образом, все новые пользователи системы будут иметь идентичные настройки браузера.
```PowerShell
New-Item -Path  "D:\image\Users\Default\AppData\Local\Microsoft\Edge\User Data\Default\" -ItemType directory
cp "E:\user_files\edge_Preferences.json" "D:\image\Users\Default\AppData\Local\Microsoft\Edge\User Data\Default\Preferences"
```

#### Оптимизация размера образа
Проверим размер образа:
```PowerShell
Dism /Image:"D:\image" /Cleanup-Image /AnalyzeComponentStore
Windows Explorer Reported Size of Component Store : 9.84 GB

Actual Size of Component Store : 9.57 GB

    Shared with Windows : 5.33 GB
    Backups and Disabled Features : 4.23 GB
    Cache and Temporary Data :  0 bytes

Date of Last Cleanup : 2023-05-05 15:03:53

Number of Reclaimable Packages : 0
Component Store Cleanup Recommended : Yes
```

Выполним очистку образа:
```PowerShell
Dism /Image:"D:\image" /Cleanup-Image /StartComponentCleanup
```

Итоговый размер образа:
```PowerShell
Dism /Image:"D:\image" /Cleanup-Image /AnalyzeComponentStore
Windows Explorer Reported Size of Component Store : 6.50 GB

Actual Size of Component Store : 6.48 GB

    Shared with Windows : 5.31 GB
    Backups and Disabled Features : 1.16 GB
    Cache and Temporary Data :  0 bytes

Date of Last Cleanup : 2023-08-29 19:23:14

Number of Reclaimable Packages : 0
Component Store Cleanup Recommended : No
```

Применим изменения и извлечём финальный образ:
```PowerShell
Dism /Unmount-Image /MountDir:"D:\image" /Commit
```

#### Установка системы на виртуальный диск VHDX
Windows поддерживает загрузку с виртуальных дисков. Я предпочитаю иметь на диске один VHDX вместо системного раздела с множеством системных файлов. К тому же, чтобы создать backup достаточно скопировать VHDX-файл.

Создадим виртуальный диск фиксированного размера 32 ГБ:
```PowerShell
diskpart
create vdisk file=D:\win11x64pro.vhdx maximum=32000 type=fixed
exit
```

Создадим на виртуальном диске раздел и смонтируем его:
```PowerShell
diskpart
select vdisk file=D:\win11x64pro.vhdx
attach vdisk
create partition primary
format quick label=VHDX
assign letter=v
exit
```

Развернём на виртуальном диске образ install.wim, который мы подготовили ранее. Добавим опцию '/compact', чтобы развернуть файлы операционной системы в сжатом виде и высвободить дополнительное дисковое пространство ([режим Compact OS](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/compact-os?view=windows-11)):
```PowerShell
Dism /Apply-Image /ImageFile:"E:\user_files\install.wim" /index:6 /ApplyDir:V:\ /compact
```

Смонтируем скрытый загрузочный раздел UEFI:
```PowerShell
diskpart
list disk
select disk 0
list partition
list volume
select volume 1
assign letter=S
exit
```

Добавим в UEFI раздел загрузку из D:\win11x64pro.vhdx:
```PowerShell
cd v:\windows\system32
bcdboot v:\windows /s S: /f UEFI
```

Теперь после включения компьютер будет по умолчанию загружаться с виртуального диска `D:\win11x64pro.vhdx`

Перезагружаем компьютер. Система попросит ввести имя пользователя, пароль и задать прочие настройки. Сразу после первого входа в систему необходимо установить обновления для [Microsoft Defender Antivirus](https://www.microsoft.com/en-us/wdsi/definitions/) и [Microsoft Edge](https://www.catalog.update.microsoft.com/Search.aspx?q=Microsoft+Edge-Extended+Stable+Channel+Version). Если открыть браузер до установки обновлений, то [параметры, которые мы задали ранее](https://github.com/ruslanbay/linuxoid-vs-windows/blob/main/README.md#%D0%B7%D0%B0%D0%B4%D0%B0%D1%91%D0%BC-%D0%BF%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D1%8B-ms-edge), могут не примениться.

#### Отключаем Reserved Storage

> [!NOTE]
> TODO: Добавить эту команду в [unnatended.xml
](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-firstlogoncommands)

Чтобы освободить примерно 6 GB дискового пространства [отключим Reserved Storage](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/dism-storage-reserve?view=windows-11):
```PowerShell
Dism /Online /Set-ReservedStorageState /State:disabled
```

Установка системы завершена!

## OPNsense firewall

### Установка OPNsense firewall
[OPNsense](https://opnsense.org/download/) является форком проекта pfSense и распространяется бесплатно. На [сайте pfSense](https://www.pfsense.org/download/) можно скачать Community Edition - бесплатную версию их firewall, но я выбрал OPNsense за более открытую модель разработки.

В документации pfSense есть [руководство по развёртыванию firewall в среде Hyper-V](https://docs.netgate.com/pfsense/en/latest/recipes/virtualize-hyper-v.html), но для наших целей эта инструкция нуждается в доработке. Сетевой адаптер `LAN` в нашем случае должен быть `Internal`, а не `Private`. Для `WAN` необходимо отключить совместное использование адаптера с операционной системы, то есть убрать галочку `Allow management operating system to share this network adapter`.

Далее нам необходимо создать виртуальную машину. Минимальные системные требования: 1ГБ RAM, 10ГБ Storage. Установку выполняем по инструкциям:
1. https://docs.netgate.com/pfsense/en/latest/recipes/virtualize-hyper-v.html
2. https://docs.opnsense.org/manual/install.html
3. https://docs.netgate.com/pfsense/en/latest/install/install-walkthrough.html

Дополнительно в настройках `Automatic Start Action` виртуальной машины необходимо разрешить автоматический запуск сразу после загрузки операционной системы.

### Настройка OPNsense firewall
Настройку firewall будем выполнять через веб-интерфейс, который по умолчанию открывается по адресу http://192.168.1.1. Предположим, мы хотим заблокировать доступ к сервисам Microsoft. Для примера рассмотрим microsoft.com. При обращении по адресу https://microsoft.com происходит перенаправление на страницу https://www.microsoft.com. При внимательном анализе можно заметить, что A и AAAA записи для доменов различаются: IP-адреса microsoft.com указывают на сервера Microsoft, www.microsoft.com - Akamai. В Windows я открываю ограниченный набор сайтов и ни один из них не использует сервера Akamai, поэтому в настройках Firewall я заблокирую доступ и к Microsoft, и к Akamai.

```PowerShell
nslookup microsoft.com

Non-authoritative answer:
Name:    microsoft.com
Addresses:  2603:1030:20e:3::23c
          2603:1030:c02:8::14
          2603:1010:3:3::5b
          2603:1020:201:10::10f
          2603:1030:b:3::152
          20.112.250.133
          20.231.239.246
          20.236.44.162
          20.70.246.20
          20.76.201.171
```

```PowerShell
nslookup www.microsoft.com

Name:    e13678.dscb.akamaiedge.net
Addresses:  2a02:26f0:f3:385::356e
          2a02:26f0:f3:38d::356e
          2a02:26f0:f3:38e::356e
          2a02:26f0:f3:395::356e
          2a02:26f0:f3:384::356e
          23.36.225.122
Aliases:  www.microsoft.com
          www.microsoft.com-c-3.edgekey.net
          www.microsoft.com-c-3.edgekey.net.globalredir.akadns.net
```

#### Firewall Aliases
В разделе `Firewall: Aliases` создадим алиас и назовём его Microsoft. В [OPNsense 22.7.2](https://opnsense.org/opnsense-22-7-2-released/) появился алиас типа BGP ASN:
> With this alias type you are able to select networks by their responsible parties. Using BGP parties announce the addresses they are responsible for to eachother. For example Cloudflare uses AS number `13335`, Microsoft is known to use `8075`. More background and how addresses are assigned is explained on [wikipedia](https://en.wikipedia.org/wiki/Autonomous_system_(Internet)).

Таким образом, чтобы выбрать все подсети, принадлежащие Microsoft, достаточно перечислить ASN. Самый простой способ найти ASN связанные с той или иной компанией - выполнить поиск на сайте [ipinfo.io](https://ipinfo.io/23.36.225.122) или [host.io](https://host.io/microsoft.com). Помимо AS8075, Microsoft принадлежат следующие ASN: 3598, 5761, 8068, 8069, 8070, 8075, 35106, 45139, 52985, 58593, 62540, 200517, 132348, 58862, 59067, 8812. Сохраняем алиас и не забываем нажать кнопку `Apply`, чтобы применить изменения. Создадим такой же алиас для Akamai:
![image](/imgs/alias-akamai-ms.png)

Чтобы убедиться, что алиасы работают перейдём в раздел `Firewall: Aliases`. Если значение в поле `Loaded#` отлично от нуля, значит IP-адреса для соответствующих ASN загружены успешно:
![image](/imgs/aliases.png)

Проверить наличие конкретной подсети можно в разделе `Firewall: Diagnostics: Aliases`. Например, IP-адрес [20.236.44.162](https://ipinfo.io/20.236.44.162) входит в подсеть 20.192.0.0/10, которая принадлежит Microsoft. Мы видим, что наш алиас включает эту подсеть:
![image](/imgs/aliases-diag.png)

Обратите внимание, что OPNsense получает данные об ASN из базы https://thyme.apnic.net/current/data-raw-table . Информация об ASN из других источников [может различаться](https://github.com/opnsense/core/issues/6953).

Если список IP-адресов не загружается, то необходимо проверить наличие ошибок в логе:
![image](/imgs/log.png)

Дополнительно я создам три алиаса с IP адресами из списка [WindowsSpyBlocker](https://github.com/crazy-max/WindowsSpyBlocker):
![image](/imgs/alias-windows-spy-blocker.png)

#### Firewall Rules

Чтобы заблокировать или разрешить доступ к адресу, который входит в алиас, создадим соответсвующее правило в `Firewall: Rules: LAN`. По умолчанию здесь присутствуют два правила:
`Default allow LAN to any rule`  и  `Default allow LAN IPv6 to any rule`. Нам необходимо отредактировать их, отключив опцию `Apply the action immediately on match`. Если вы запретили весь IPv6 трафик в разделе `Firewall: Settings: Advanced`, то последнее правило можно удалить.

Добавим правила для блокировки доступа к Akamai и Microsoft. В поле Destination выберем алиас, который мы создали на предыдущем шаге:
![image](/imgs/rule-akamai-ms.png)

Сохраняем и не забываем нажать кнопку `Apply`. Обратите внимание, что правила применяются сверху вниз, то есть их расположение в списке имеет значение. Пользовательские правила должны располагаться над `Default allow LAN to any rule`  и  `Default allow LAN IPv6 to any rule`. Пример как может выглядеть список правил:
![image](/imgs/rules.png)

Теперь при попытке перейти на страницу microsoft.com должна отображаться ошибка доступа.

Добавим аналогичные правила для блокировки IP адресов из списка [WindowsSpyBlocker](https://github.com/crazy-max/WindowsSpyBlocker):
![image](/imgs/rule-windows-spy-blocker.png)

## Туннелирование трафика через VPN
Настройку VPN необходимо производить на стороне firewall. Если установить VPN на устройстве, которое находится за firewall, то это осложнит фильтрацию трафика. Для настройки VPN воспользуемся [инструкцией](https://protonvpn.com/support/pfsense-2-6-x-vpn-setup/). Шаги 1-3 инструкции можно применить без изменений. На четвёртом шаге `Step Four: Setting up the firewall rules` задаётся маршрутизация трафика через VPN и здесь требуется уточнение.

Переходим в раздел `Firewall: NAT: Outbound`, выбираем режим `Manual outbound NAT rule generation
(no automatic rules are being generated)` и создадим два правила. Чтобы сохранить изменения не забудьте нажать кнопку `Save` и `Apply`.
![image](/imgs/firewall-nat-outbound.png)
![image](/imgs/rule-firewall-nat-outbound.png)

На шаге 7 указано:
> 7. Change Gateway to the previously created gateway (in our example, `ProtonVPNIS03UDP_VNV4`). Save and Apply the changes.

Видимо, инструкцию забыли обновить, так как на предыдущих шагах мы не создавали Gateway - в последних версиях pfSense и OPNsense он добавляется автоматически после добавления VPN клиента. Всё что нам нужно сделать это изменить правила `Default allow LAN to any rule`  и  `Default allow LAN IPv6 to any rule` в разделе `Firewall: Rules: LAN`:
![image](/imgs/rule-firewall-nat-outbound-2.png)

Наконец, переходим в раздел `VPN: OpenVPN: Connection Status` и перезапускаем VPN клиент. Далее открываем страницу https://ip.me. Если VPN подключение настроено верно, то вы увидите IP-адрес VPN провайдера.

## Предотвращение утечек DNS запросов

Изначально я настроил Unbound DNS по [инструкции Quad9](https://docs.quad9.net/Setup_Guides/Open-Source_Routers/OPNsense_%28Encrypted%29/). Тем не менее, я заметил что иногда часть DNS запросов уходят на сервера ISP. Чтобы избежать утечек DNS запросов отключим Unbound DNS в разделе `Services: Unbound DNS: General`, затем перейдём в `System: Settings: General` и зададим следующие настройки:
![image](/imgs/dns.png)

Готово! Проверить наличие утечек можно на странице https://ipleak.net/

<!--

### TODO
### Disable Recovery Mode
bcdedit /set {default} recoveryenabled no

`Get-AppxPackage | Where-Object{.NonRemovable -match 'False'} | SELECT PackageFullName`

`Resolve-DnsName -Type ALL -Name www.catalog.update.microsoft.com -Server 8.8.8.8`


## Отключаем неспользуемые сервисы
One computer with local windows account. No local network, no printers. Don't use Windows Hello (biometriccs)

| Service | Service name | Description |
|---|---|---|
| AxInstSV | ActiveX Installer | |
| CaptureService | CaptureService | |
| DiagTrack | Connected User Experiences and Telemetry | The Connected User Experiences and Telemetry service enables features that support in-application and connected user experiences. Additionally, this service manages the event driven collection and transmission of diagnostic and usage information (used to improve the experience and quality of the Windows Platform) when the diagnostics and usage privacy option settings are enabled under Feedback and Diagnostics.|
| diagsvc | Diagnostic Execution Service | Executes diagnostic actions for troubleshooting support |
| DPS | Diagnostic Policy Service | The Diagnostic Policy Service enables problem detection, troubleshooting and resolution for Windows components.  If this service is stopped, diagnostics will no longer function. |
| MapsBroker | Downloaded Maps Manager | Windows service for application access to downloaded maps. This service is started on-demand by applications accessing downloaded maps |
| fhsvc | File History Service | Protects user files from accidental loss by copying them to a backup location |
| fdPHost </b> FDResPub | Function Discovery Resource Publication | Publishes this computer and resources attached to this computer so they can be discovered over the network.  If this service is stopped, network resources will no longer be published and they will not be discovered by other computers on the network. |
# Geolocation Service
lfsvc
# Internet Connection Sharing (ICS) - Provides network address translation, addressing, name resolution and/or intrusion prevention services for a home or small office network.
SharedAccess
# Link-Layer Topology Discovery Mapper - Creates a Network Map, consisting of PC and device topology (connectivity) information, and metadata describing each PC and device.  If this service is disabled, the Network Map will not function properly.
lltdsvc
# McpManagementService - Universal Print Management Service. Office365 feature
McpManagementService
# MessagingService - SMS, MMS functionality
MessagingService
# Microsoft Account Sign-in Assistant - Enables user sign-in through Microsoft account identity services. If this service is stopped, users will not be able to logon to the computer with their Microsoft account.
wlidsvc
# Microsoft Windows SMS Router Service.
SmsRouter
# Natural Authentication - Signal aggregator service, that evaluates signals based on time, network, geolocation, bluetooth and cdf factors. Supported features are Device Unlock, Dynamic Lock and Dynamo MDM policies
NaturalAuthentication 
# Netlogon - Maintains a secure channel between this computer and the domain controller for authenticating users and services. If this service is stopped, the computer may not authenticate users and services and the domain controller cannot register DNS records. If this service is disabled, any services that explicitly depend on it will fail to start.
Netlogon
# Network Connected Devices Auto-Setup - Network Connected Devices Auto-Setup service monitors and installs qualified devices that connect to a qualified network. Stopping or disabling this service will prevent Windows from discovering and installing qualified network connected devices automatically. Users can still manually add network connected devices to a PC through the user interface.
NcdAutoSetup
# Network Location Awareness - Collects and stores configuration information for the network and notifies programmes when this information is modified. If this service is stopped, configuration information might be unavailable. If this service is disabled, any services that explicitly depend on it will fail to start.
NlaSvc
# NPSMSvc - Now Playing Session Manager
NPSMSvc
# OneSyncSvc - This service synchronizes mail, contacts, calendar and various other user data. Mail and other applications dependent on this functionality will not work properly when this service is not running.
OneSyncSvc
# OpenSSH Authentication Agent - Agent to hold private keys used for public key authentication.
ssh-agent
# Parental Controls - Enforces parental controls for child accounts in Windows. If this service is stopped or disabled, parental controls may not be enforced.
WpcMonSvc
# Payments and NFC/SE Manager
SEMgrSvc
# Peer Name Resolution Protocol - Enables serverless peer name resolution over the Internet using the Peer Name Resolution Protocol (PNRP). If disabled, some peer-to-peer and collaborative applications, such as Remote Assistance, may not function.
PNRPsvc
# Peer Networking Grouping - Enables multi-party communication using Peer-to-Peer Grouping.  If disabled, some applications, such as HomeGroup, may not function.
p2psvc
# Peer Networking Identity Manager - Provides identity services for the Peer Name Resolution Protocol (PNRP) and Peer-to-Peer Grouping services.  If disabled, the Peer Name Resolution Protocol (PNRP) and Peer-to-Peer Grouping services may not function, and some applications, such as HomeGroup and Remote Assistance, may not function correctly.
p2pimsvc
# Performance Logs & Alerts - Performance Logs and Alerts Collects performance data from local or remote computers based on preconfigured schedule parameters, then writes the data to a log or triggers an alert. If this service is stopped, performance information will not be collected. If this service is disabled, any services that explicitly depend on it will fail to start.
pla
# Phone Service - Manages the telephony state on the device
PhoneSvc
# PimIndexMaintenanceSvc - Indexes contact data for fast contact searching. If you stop or disable this service, contacts might be missing from your search results.
PimIndexMaintenanceSvc
# PNRP Machine Name Publication Service - This service publishes a machine name using the Peer Name Resolution Protocol.  Configuration is managed via the netsh context 'p2p pnrp peer' 
PNRPAutoReg
# Print Spooler
Spooler
# Printer Extensions and Notifications
PrintNotify
# PrintWorkflowUserSvc
PrintWorkflowUserSvc
# Problem Reports Control Panel Support - This service provides support for viewing, sending and deletion of system-level problem reports for the Problem Reports control panel.
wercplsupport
# Recommended Troubleshooting Service
TroubleshootingSvc
# Remote Desktop Configuration
SessionEnv
# Remote Desktop Services
TermService
UmRdpService
# Remote Registry -  Enables remote users to modify registry settings on this computer. If this service is stopped, the registry can be modified only by users on this computer. If this service is disabled, any services that explicitly depend on it will fail to start.
RemoteRegistry
# Retail Demo Service - The Retail Demo service controls device activity while the device is in retail demo mode.
RetailDemo
# Routing and Remote Access - Offers routing services to businesses in local area and wide area network environments.
RemoteAccess
# TCP/IP NetBIOS Helper
lmhosts
# Telephony
TapiSrv
# UnistoreSvc - Handles storage of structured user data, including contact info, calendars, messages and other content. If you stop or disable this service, apps that use this data might not work correctly.
UnistoreSvc
# UserDataSvc - Provides apps with access to structured user data, including contact info, calendars, messages and other content. If you stop or disable this service, apps that use this data might not work correctly.
UserDataSvc
# Web Account Manager - This service is used by Web Account Manager to provide single-sign-on to apps and services.
TokenBroker
# WebClient - Enables Windows-based programs to create, access, and modify Internet-based files. If this service is stopped, these functions will not be available. If this service is disabled, any services that explicitly depend on it will fail to start.
WebClient
# Windows Backup
SDRSVC
# Windows Biometric Service
WbioSrvc
# Windows Connect Now - Config Registrar    - WCNCSVC hosts the Windows Connect Now Configuration, which is Microsoft's Implementation of the Wireless Protected Setup (WPS) protocol. This is used to configure Wireless LAN settings for an Access Point (AP) or a Wireless Device
wcncsvc
# Windows Error Reporting Service
WerSvc
# Windows Insider Service
wisvc
# Windows Media Player Network Sharing Service
WMPNetworkSvc
# Windows Mixed Reality OpenXR Service
MixedRealityOpenXRSvc
# Windows PushToInstall Service - Provides infrastructure support for the Microsoft Store.  This service is started automatically and if disabled then remote installations will not function properly.
PushToInstall
# Windows Remote Management (WS-Management)
WinRM
# Windows Search - Provides content indexing, property caching, and search results for files, e-mail, and other content.
WSearch
# Work Folders - This service syncs files with the Work Folders server, enabling you to use the files on any of the PCs and devices on which you've set up Work Folders.
workfolderssvc
# Workstation - Creates and maintains client network connections to remote servers using the SMB protocol. If this service is stopped, these connections will be unavailable. If this service is disabled, any services that explicitly depend on it will fail to start.
LanmanWorkstation
# WWAN AutoConfig - This service manages mobile broadband (GSM & CDMA) data card/embedded module adapters and connections by auto-configuring the networks. It is strongly recommended that this service be kept running for best user experience of mobile broadband devices.
WwanSvc
# XBox staff
XboxGipSvc
XblAuthManager
XblGameSave
XboxNetApiSvc

```PowerShell
reg load   HKLM\OFFLINE D:\image\Windows\System32\config\SYSTEM
reg add    HKLM\OFFLINE\ControlSet001\Services\AxInstSV                     /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\CaptureService               /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\DiagTrack                    /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\diagsvc                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\DPS                          /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\MapsBroker                   /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\fhsvc                        /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\fdPHost                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\FDResPub                     /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\lfsvc                        /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\SharedAccess                 /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\lltdsvc                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\McpManagementService         /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\MessagingService             /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\wlidsvc                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\SmsRouter                    /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\NaturalAuthentication        /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\Netlogon                     /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\NcdAutoSetup                 /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\NlaSvc                       /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\NPSMSvc                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\OneSyncSvc                   /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\ssh-agent                    /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\WpcMonSvc                    /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\SEMgrSvc                     /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\PNRPsvc                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\p2psvc                       /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\p2pimsvc                     /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\pla                          /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\PhoneSvc                     /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\PimIndexMaintenanceSvc       /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\PNRPAutoReg                  /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\Spooler                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\PrintNotify                  /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\PrintWorkflowUserSvc         /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\wercplsupport                /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\TroubleshootingSvc           /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\SessionEnv                   /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\TermService                  /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\UmRdpService                 /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\RemoteRegistry               /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\RetailDemo                   /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\RemoteAccess                 /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\lmhosts                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\TapiSrv                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\UnistoreSvc                  /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\UserDataSvc                  /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\TokenBroker                  /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\WebClient                    /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\SDRSVC                       /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\WbioSrvc                     /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\wcncsvc                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\WerSvc                       /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\wisvc                        /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\WMPNetworkSvc                /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\MixedRealityOpenXRSvc        /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\PushToInstall                /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\WinRM                        /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\WSearch                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\workfolderssvc               /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\LanmanWorkstation            /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\WwanSvc                      /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\XboxGipSvc                   /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\XblAuthManager               /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\XblGameSave                  /v Start /t REG_DWORD /d 4 /f
reg add    HKLM\OFFLINE\ControlSet001\Services\XboxNetApiSvc                /v Start /t REG_DWORD /d 4 /f
reg unload HKLM\OFFLINE
```

-->
