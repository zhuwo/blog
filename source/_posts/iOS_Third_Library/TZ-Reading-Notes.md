---
title: TZ_Reading_Notes
date: 2023-08-30 14:37:01
categories:
- 技术
- iOS
tags:
- ImagePicker
---

  比较知名的图片选择器源码阅读和理解。TZImagePciker是一款github star超过8k的图片选择库。地址：https://github.com/banchichen/TZImagePickerController


  ## LxGridViewFlowLayout

  使用了这个Layout，主要是实现CollectionView Cell可以拖拽的效果，添加了长按的手势识别，不断更新浮动的位置，一旦拖拽的位置到达新的位置，就用`performBatchUpdates`更新cell位置，已实现拖拽换位置的效果。

  ## Demo入口

  拍照和拍视频，使用系统提供的UIImagePickerController;相册使用TZImagePickerController。

  ## TZImagePickerController

  是一个UINavigationController，根VC是TZAlbumPickerController。第一次进入，push到TZPhotoPickerController，主要照片选择器。

 ## TZPhotoPickerController

  照片选择主体页面。来源，getCameraRollAlbumWithFetchAssets, 获取图片getPhotoWithAsset， 点击图片预览TZPhotoPreviewController。
  拍照cell，点击跳到系统拍照的ImagePickerController。
  Done按钮: 按operation获取选择的照片内容并返回，跳出pop presentedViewController。
  FullImage按钮：showPhotoBytes -> getPhotosBytesWithArray,会获取当前所选的照片的原图大小

  ## TZAlbumPickerController

  相册选择列表页面

 ## TZImageManager

    1. 权限获取和权限状态管理
    2. 照片处理-- Get Album 获得相册/相册数组方法： getCameraRollAlbumWithFetchAssets
    3. 获取对应照片：getPhotoWithAsset

 ## TZPhotoPreviewController

   大页面CollectionView, 每个Cell占满一个屏幕，滚动翻页系统自带collectionView.pagingEnabled。
