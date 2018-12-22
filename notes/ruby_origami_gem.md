## Origami gemを触ってみて

### ライセンス
LGPL 3.0+

### PDFを読み込こんで各Origami::Pageを配列で取得

```
Origami::PDF.read(pdf_path, verbosity: Origami::Parser::VERBOSE_QUIET).pages
```

### Origami::Pageについて

```
Origami::Page < Origami::Dictionary
Origami::Dictionary < Hash
```

### Origami::Dictionaryについて

- keyはOrigami::Name
- valueはOrigami::NameだったりOrigami::Rectangleだったり、Origami::〇〇

イメージはハッシュ
```
Origami::Dictionary = {
  Origami::Name => Origami::〇〇,
  Origami::Name => Origami::〇〇,
  Origami::Name => Origami::〇〇,
  Origami::Name => Origami::〇〇,...
}
```

### Origami::Nameについて

Origami::Name.valueでタイプっぽいものがとれた

```
# 例
Origami::PDF.read(pdf_path, verbosity: Origami::Parser::VERBOSE_QUIET)
  .pages.first.map(&:value)
# 出力結果
[:Type, :Parent, :Resources, :MediaBox, :Contents, :Group, :Tabs]
```
### PDFの中身をとってみる

```
Origami::PDF.read(pdf_path, verbosity: Origami::Parser::VERBOSE_QUIET).pages.each do |page|
  p page.keys.map(&:value)
end
# 出力結果
...
[:Type, :Parent, :Resources, :Annots, :MediaBox, :Contents, :Group, :Tabs]
[:Type, :Parent, :Resources, :MediaBox, :Contents, :Group, :Tabs]
[:Type, :Parent, :Resources, :Annots, :MediaBox, :Contents, :Group, :Tabs]
[:Type, :Parent, :Resources, :MediaBox, :Contents, :Group, :Tabs]
[:Type, :Parent, :Resources, :Annots, :MediaBox, :Contents, :Group, :Tabs]
[:Type, :Parent, :Resources, :MediaBox, :Contents, :Group, :Tabs]
[:Type, :Parent, :Resources, :MediaBox, :Contents, :Group, :Tabs]
[:Type, :Parent, :Resources, :MediaBox, :Contents, :Group, :Tabs]
[:Type, :Parent, :Resources, :MediaBox, :Contents, :Group, :Tabs]
[:Type, :Parent, :Resources, :MediaBox, :Contents, :Group, :Tabs]
...
```

### PDFをエディタで開いたみたい見る

```
# keysでは無理だった
Origami::PDF.read(pdf_path, verbosity: Origami::Parser::VERBOSE_QUIET)
  .pages.each { |k,v| puts k }
# 出力結果
# 1ページ目
/Page
2 0 R
<<
        /Font <<
                /F1 5 0 R
                /F2 12 0 R
                /F3 14 0 R
        >>
        /ExtGState <<
                /GS10 10 0 R
                /GS11 11 0 R
        >>
        /XObject <<
                /Image16 16 0 R
                /Meta18 18 0 R
        >>
        /ProcSet [ /PDF /Text /ImageB /ImageC /ImageI ]
>>
0
4 0 R
<<
        /Type /Group
        /S /Transparency
        /CS /DeviceRGB
>>
/S
# 2ページ目
...
```
