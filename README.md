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

