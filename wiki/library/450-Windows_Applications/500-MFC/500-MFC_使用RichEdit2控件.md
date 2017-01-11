

# MFC 使用RichEdit2控件

在资源编辑窗口中添加好了RichEdit2控件的时候会提示需要riched20支持，这绝对不是无意义的提示，这据话说明你需要在项目中的某个地方loadrichedit20库。

我一般是在项目的启动代码里面加，例如：

项目：KeyScriptTest

启动源代码：KeyScriptTest.cpp

在打开主窗口前添加以下代码：

```cpp
    HINSTANCE hLib;
	hLib = LoadLibrary("riched20.dll");

    // 打开主窗口
    CKeyScriptTestDlg dlg;
	m_pMainWnd = &dlg;
	INT_PTR nResponse = dlg.DoModal();
```
