---
layout: post
title: 查找对应目录进行压缩
---
`$ tar zcvf 20170608.tar.gz $(find ./post_data/png/ -name "20170608" -type d)`