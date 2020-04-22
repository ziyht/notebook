	1. 需要添加一个 merge driver:

git config --global merge.ours.driver true

2. 在这个.gitattributes(前缀有个点)

 在文件里添加需要被忽略的文件:

 config.xml merge=ours

