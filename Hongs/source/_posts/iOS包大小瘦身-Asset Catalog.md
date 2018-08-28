---
title: iOS包大小瘦身-Asset Catalog
date: 2018-08-02 10:09:41
toc: true
categories: 包大小
tags: [iOS包大小,Asset Catalog]
---
通过Assert Catalog进行iOS包大小瘦身的原理及最优效果
<!-- more -->
## Asset Catalog
Asset Catalog是 iOS7 开始支持的图片资源管理方案，将分散在项目中大大小小的图片资源进行统一存放和集中管理。

## 瘦身原理
1. 通过Asset图片管理的1x、2x、3x图，iPhone X、各种plus机型的安装包只包含3x图，iPhone 8、7、6等机型的安装包只包含2x图，因此节省了图片资源大小（前提是开启BitCode，但是谁开了呢！！！）
2. 文件对齐，Asset方案将众多的文件打包到一个大文件中，减少文件数量，可减少每个文件的簇对齐导致的磁盘浪费（一个扇区512字节，一般一个簇单元有8个扇区，所以文件磁盘大小至少4K。比如，1字节的数据存在文件中，也是要占4K的磁盘大小）

## 实践方案
因公司App已存许久，且业务较多，让各个业务做Asset成本较高，所以做了个打包脚本统一处理。
脚本扫描各bundle中 size<2k 的png、jpg的文件，然后将找到的文件move到主工程Assets.xcassets中，存储路径还是xxx.bundle/xxxxx，并开启Provides Namespace。
最后这些assets只会生成一个assets.car文件。

## 实现效果
以一个76MB的APP（BitCode=NO）为例，实践以上Asset方案后，减小包大小3MB+，效果显著。

## 脱坑指南
在实践中，使用Asset时需要注意的坑s：

1. 只能使用[UIImage imageNamed:]加载图片，否则返回nil
2. 建议开启Provides Namespace（可用脚本开启）。该操作是为无缝支持[UIImage imageNamed:@"xxx.bundle/xxx"]调用方式，因为Asset的key恰好与相对路径访问的字符串一样，业务就不需要额外改动
3. imageNamed参数命名。接口中字符串不带绝对路径的前缀：/xxx/AlipayWallet.app/，接口中字符串不带后缀：@2x.png、@3x.jpg
4. 线程确认。必须确认在主线程调用[UIImage imageNamed:@"xxx.bundle/xxx"]。在iOS8以下[UIImage imageNamed:]方法不保证线程安全，iOS 8、iOS 10各有一个小版本也不保证线程安全，所以……线程确认吧
5. 图片变大。这是一个推测。一般framework中的图片是压缩过的，但当图片移入Asset时会被反压缩[（在头条也有类似推论）](https://techblog.toutiao.com/2018/06/04/gan-huo-jin-ri-tou-tiao-iosduan-an-zhuang-bao-da-xiao-you-hua-si-lu-yu-shi-jian/)。解析ipa包中assets.car文件，对比framework原图片，每个图片都变大了，但无法排除assets.car解析工具原因（PS.试过3个工具都一样）。所以假设图片会变大，那么只将 size<2k (3k、4k都可以，设个让文件对齐功能效果最佳的阈值)的图片进行asset，在Appstore验证后果然安装包更小，所以初步验证推论成立

## 后续
BitCode=NO的前提下，最大化Asset减包效果的方案是size向4k的倍数对齐。<br/><br/>
比如4.1K的文件Asset后，可能变大为4.9K，按8K对齐算，4.1K文件实际-3.1K（8-4.9）；<br/>
又比如7.6K的文件Asset后，可能变大为8.7K，超过下个对齐阈8K，7.6K文件实际+1.1K（8.7-7.6）。<br/>
`下个文件对齐的阈值 - 当前文件的大小 = 优化size`<br/>
`Asset后文件反压缩的大小 - 当前文件的大小 = 膨胀size`<br/>
`膨胀size - 优化size < 0` 时才有Asset价值。<br/>
但`Asset后文件反压缩的大小`在build阶段无法得到，或许假设一个magic number， `magic number = Asset后文件反压缩的大小 / 当前文件的大小` 是个估算方案吧。