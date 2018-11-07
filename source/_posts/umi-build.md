---
title: Umi项目按需构建
catalog: true
date: 2018-11-07 11:10:17
subtitle: 构建教程
header-img: "/img/article_header/article_header.png"
tags:
- React
---

## umi项目按需构建

使用 UmiJS 创建 react 项目，其约定式路由和插件化的项目结构可以大大的提高编码效率。但是实际项目开发中有时候我们会遇到这样的需求。同一套代码里包含 n 个管理系统的模块儿，但是对于不同的项目我只需要部署其中的某几个模块儿，那么如果我把整个项目都构建出来，显然是代码冗余，没有必要的，所以此时我们就可以利用UmiJS的约定式路由的优势加上shell脚本实现按需构建。


### 解决方案
编写shell构建脚本 build.sh,假设项目名称叫做smart-portal。

```
# 创建临时文件夹
echo "==========  Create temp dir  =========="
mkdir temp

# 拷贝pages文件夹下所有内容作为备份
echo "==========  Backups pages file to temp dir  =========="
cp -r ./smart-portal/src/pages/* ./temp

# 定义基础文件
baseFiles="document.ejs,global.js"

# 拼接基础文件与传入参数
buildServices=${baseFiles},$1
echo $buildServices

# 循环读取pages文件夹
for dir in $(ls ./smart-portal/src/pages/)
do
  # 定义是否删除当前文件夹标志
  delFlag="false"

  # IFS是内部的域分隔符默认值为空白
  oldIFS=$IFS
  IFS=,

  # 遍历要构建的模块判断当前文件夹要不要删除
  for value in $buildServices
  do
    if [ $dir == $value ]
    then
      delFlag="true"
    fi
  done

  if [ $delFlag != "true" ]
  then
     echo "==========  delete "[$dir]"  =========="
     rm -r ./smart-portal/src/pages/$dir
  fi

  # 还原域分隔符默认值为空白
  IFS=$oldIFS

done

cd ./smart-portal

echo "==========  Start building the application  =========="
npm run build
echo "==========  End building the application  =========="

# 压缩构建出来的产物
# cd ./dist
# echo "==========  begin to zip smart-portal =========="
# tar -zcvf ./smart-portal.tar.gz *
# cd ..

cd ..

echo "========== Revert pages file form temp =========="
cp -r  ./temp/* ./smart-portal/src/pages

echo "========== Delete old temp file =========="
rm -r ./temp


echo "==========  All the builds have been completed  =========="

```
