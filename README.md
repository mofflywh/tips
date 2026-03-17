# ahk Shift IME
```
#IfWinNotActive ahk_exe VirtualBoxVM.exe
; 左Shiftが単体で押されたら、直接入力 → LAlt
~LShift Up::
if (A_PriorKey = "LShift") {
    Send, {vk1A}
}
Return

; 右Shiftが単体で押されたら、ひらがな入力 → RAlt
~RShift Up::
if (A_PriorKey = "RShift") {
    Send, {vk16}
}
Return
```

# KQL
- 特定の文字列をパスに含むプロセス起動
```
DeviceEvents
| where FolderPath contains "PortableApps"

DeviceEvents
| where InitiatingProcessFolderPath contains "PoratableApp"
```
- 2025/10/21 UACで管理者権限のもの
```
DeviceEvents
| where ProcessTokenElevation == "TokenElevationTypeFull"
```
  - https://learn.microsoft.com/ja-jp/defender-xdr/advanced-hunting-deviceevents-table
  - ProcessTokenElevation
```
新しく作成されたプロセスに適用されるトークン昇格の種類を示します。 
使用可能な値: TokenElevationTypeLimited (restricted)、
TokenElevationTypeDefault (standard)、
TokenElevationTypeFull (管理者特権)
```
- 2025/4/11 Defender for CloudAppsの検知からのリンク
```
// Find all the SharePoint files uploaded, downloaded, or modified by the cloud app in the last 30 days
CloudAppEvents
| where RawEventData.ApplicationId == "66035fdf-7b75-4675-b48b-b0dfb359a836"    // postman
| where RawEventData.Workload == "SharePoint"
| where ActionType == "FileUploaded" or ActionType == "FileDownloaded"
| where Timestamp between (datetime("2025-03-12 00:00:00Z")..30d)    // ここで調節
| extend tostring(RawEventData.Id)
| summarize arg_max(Timestamp, *) by RawEventData_Id
| sort by Timestamp desc
| project Timestamp, OAuthApplicationId = tostring(RawEventData.ApplicationId), ReportId, AccountId, AccountObjectId, AccountDisplayName, IPAddress, UserAgent, ResourceType = tostring(RawEventData.ItemType), Workload = tostring(RawEventData.Workload), ActionType, FileId = tostring(RawEventData.ListItemUniqueId), FileName = tostring(RawEventData.SourceFileName), FileExtension = tostring(RawEventData.SourceFileExtension), FileUrl = tostring(RawEventData.ObjectId), FileSize = strcat(round(RawEventData.FileSizeBytes/1048576, 2), "MB"), SensitivityLabel = tostring(RawEventData.SensitivityLabelId), tostring(RawEventData)
| limit 1000
```
- 2025/3/24 スキャン完了ログ Sentinel
```
DeviceEvents
| where ActionType contains "scan"
| where DeviceName startswith "nki-"
| where AdditionalFields.ScanTypeIndex == "Full"
| project TimeGenerated, ActionType, AdditionalFields.ScanTypeIndex, DeviceName

// 2025/4/13 改良
DeviceEvents
| where ActionType contains "scan"
| project TimeGenerated, ActionType, AdditionalFields.ScanTypeIndex, DeviceName
```

- 2025/1/22 SMS認証ユーザピックアップ
```
SigninLogs
| extend deviceDisplayName = tostring(DeviceDetail.displayName)
| extend authStatusadditionalDetails = tostring(Status.additionalDetails)
| extend authDatetime = tostring(parse_json(AuthenticationDetails)[0].authenticationStepDateTime)
| extend authMethod = tostring(parse_json(AuthenticationDetails)[0].authenticationMethod)
| extend authMethodDetail = tostring(parse_json(AuthenticationDetails)[0].authenticationMethodDetail)
| where authMethodDetail startswith "+X" or authMethodDetail startswith "XX"
| project UserDisplayName, DeviceDetail.displayName, SourceSystem, AppDisplayName, AuthenticationRequirement, Status.additionalDetails, parse_json(AuthenticationDetails)[0].authenticationStepDateTime, parse_json(AuthenticationDetails)[0].authenticationMethod, parse_json(AuthenticationDetails)[0].authenticationMethodDetail
```
- 2025/1/27 アカウント作成or変更 ノイズ多そう
```
2025/1/27 アカウント作成or変更
```


# PowerShell
- インストールされたソフトの一覧を名称でフィルタ
```
$paths = @("HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall","HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall","HKCU:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall","HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall");foreach ($path in $paths) {Get-ItemProperty -Path "$path\*" | Where-Object { $_.DisplayName -like "*Netskope*" } | Select-Object DisplayName, DisplayVersion, Publisher, PSPath }
```
- Netskopeエージェントのサービス有無  ※トンネルしてるかではない。
- `Get-Service |? "Name" -eq "stAgentSvc"`



# CloudWatch json query
- signature_idが数字2桁以上
  - `{ $.event.alert.signature_id = %[0-9]{2,}% }`
- signature_idが1から始まりかつIPアドレスのパターンを指定
  - `{ $.event.alert.signature_id = %1.+% && $.event.src_ip = %^187\.62\.+% }`
- authorizationを持つもの
  - `{ $.event.http.request_headers[*].name = "authorization" }`
- 他
  - `{$.event.http.request_headers[*].value = %ikedc% && $.event.http.request_headers[*].value = "POST"}`
  - `{ $.event.http.http_user_agent = %PowerShell% }`
  - `{ $.event.alert.signature_id = %^72[0-9]{4}% }`
  - `{$.event.http.url = %/objects/% }`
  - `{$.event.alert.signature_id = 740000 && $.event.http.request_headers[*].name = "authorization" && ($.event.http.request_headers[*].value = %daiwaliving% || $.event.http.request_headers[*].value = %master%)}` めちゃ遅いのでひと月とかむり
- url中のUUID抽出
  - `{ $.event.http.url = %[0-9a-f]{8}\-[0-9a-f]{4}\-[0-9a-f]{4}\-[0-9a-f]{4}\-[0-9a-f]{12}% }`
- ファイル直指定
  - `{ $.event.http.url = %^/canopy/|[^/]+% }`

# Windows検索
```
更新日時：2023/1/20 .. 2023/1/31
```

# 日本語でよく使われるencoding
- utf_8 UTF-8 (別名: utf-8 U8 utf8 cp65001 )
- shift_jis シフトJIS (別名: csshiftjis shiftjis sjis s_jis )
- cp932 シフトJIS(拡張シフトJIS) (別名: 932 ms932 mskanji mks-kanji )
- euc_jp EUC-JP (別名: eucjp ujis u-jis )
- iso2022_jp JIS(ISO-2022-JP) (別名: csiso2022jp iso2022jp iso-2022-jp )

# Word
http://office-qa.com/Word/wd211.htm

