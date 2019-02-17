# Intellij IDEA 修改项目名称



## 1.在终端或者直接给文件夹重命名

## 2.重新导入项目

## 3.重命名项目根目录下原项目名.iml为新项目名.xml

![img](https://ask.qcloudimg.com/http-save/yehe-3090887/4rlc1nfsdn.png?imageView2/2/w/1620)

1.1

## 4.打开.idea文件夹，修改.name的文件内容为新项目名

## 5.修改compiler.xml中module标签的name属性为新项目名

![img](https://ask.qcloudimg.com/http-save/yehe-3090887/5nv1a74q1s.png?imageView2/2/w/1620)

1.2

## 5.修改modules.xml中的module标签fileurl属性的链接地址末尾为新项目名.iml

![img](https://ask.qcloudimg.com/http-save/yehe-3090887/7iiegvrs1e.png?imageView2/2/w/1620)

1.3

## 6.修改workspace.xml中的所有原项目名为新项目名（查找替换）

## 7.修改pom.xml中name为新项目名