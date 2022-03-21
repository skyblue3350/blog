+++
author = "すかい"
title = "PTC Creo 3.0でデフォルトの単位系を変更する"
date = "2016-10-19"
description = "PTC Creo 3.0でデフォルトの単位系を変更する"
tags = [
    "PTC Creo",
]
+++

PTC Creo 3.0はデフォルトの単位系だとインチポンド秒になっててわかりにくいのでミリメートルキログラム秒に変更する
プロジェクト単位だとファイル→準備→モデル特性から変更出来る
ただこれだと毎回操作が必要なのでconfig.proを編集してデフォルト値を変更する

インストールパス\PTC\Creo 3.0\M070\Common Files\text\config.pro
にコンフィグファイルがあるのでこれを編集する
当該箇所は249行目付近

```
***** Options for MMKS Unit System *****
```

の方がコメントついてると思うので外して

```
***** Options for INLBS Unit System *****
```

の方にコメントをつける（頭に!を付与する）

上記操作をすると多分こんな感じになる

```
! ***** Options for MMKS Unit System *****
!
drawing_setup_file $pro_directory\creo_standards\draw_standards\standard_mm.dtl
pro_unit_length unit_mm
pro_unit_mass unit_kilogram
pro_unit_sys mmks
template_designasm $pro_directory\creo_standards\templates\start_assembly_mmks.asm
template_drawing $pro_directory\creo_standards\templates\a3_drawing.drw
template_mfgcast $pro_directory\creo_standards\templates\mmks_mfg_cast.asm
template_mfgemo $pro_directory\creo_standards\templates\mmks_mfg_emo.asm
template_mfgmold $pro_directory\creo_standards\templates\mmks_mfg_mold.asm
template_mfgnc $pro_directory\creo_standards\templates\mmks_mfg_nc.asm
template_sheetmetalpart $pro_directory\creo_standards\templates\sheetmetal_start_part_mmks.prt
template_solidpart $pro_directory\creo_standards\templates\solid_start_part_mmks.prt
weld_ui_standard iso
!
! ***** Options for INLBS Unit System *****
!
!drawing_setup_file $pro_directory\creo_standards\draw_standards\standard_in.dtl
!pro_unit_length unit_inch
!pro_unit_mass unit_pound
!pro_unit_sys proe_def
!template_designasm $pro_directory\creo_standards\templates\start_assembly_inlbs.asm
!template_drawing $pro_directory\creo_standards\templates\c_drawing.drw
!template_mfgcast $pro_directory\creo_standards\templates\inlbs_mfg_cast.asm
!template_mfgemo $pro_directory\creo_standards\templates\inlbs_mfg_emo.asm
!template_mfgmold $pro_directory\creo_standards\templates\inlbs_mfg_mold.asm
!template_mfgnc $pro_directory\creo_standards\templates\inlbs_mfg_nc.asm
!template_sheetmetalpart $pro_directory\creo_standards\templates\sheetmetal_start_part_inlbs.prt
!template_solidpart $pro_directory\creo_standards\templates\solid_start_part_inlbs.prt
!weld_ui_standard ansi
!default_dec_places 3
```

起動してデフォルト値が変わってるか確認する

## 参考記事

- [THERE ARE 3 LOCATIONS PRO/ENGINEER ‘LOOKS’ FOR ITS CONFIGURATIONS](http://www.codexcreo.com/config_pro.html)
- [Creo（Pro/E） Know-how and FAQs](http://www.page.sannet.ne.jp/gah01300/proe/manual/knowhow/config_pro.html)
