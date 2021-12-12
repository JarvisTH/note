## clion undefined reference to `__imp_WSAStartup'

在CMakeLists.txt文件中添加如下代码：

```
link_libraries(ws2_32)
//*添加位置要注意*
```

