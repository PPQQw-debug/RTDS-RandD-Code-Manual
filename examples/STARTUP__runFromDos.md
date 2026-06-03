# 示例：启动脚本说明（runFromDos）

## 1. 场景

该脚本是应用启动入口之一，负责在固定目录下拉起 Java 进程并设置 JavaFX 与本地库参数。

## 2. 代码示例（工作目录固定）

```bat
@echo off
rem %~dp0 expands to the drive letter and directory path of this batch file,
rem so the script always runs from its own directory regardless of install location.
pushd "%~dp0"
```

来源文件：runFromDos.bat

说明：

- pushd "%~dp0" 确保脚本从自身目录执行，避免安装路径变化导致依赖找不到。  
- 这是 Windows 启动脚本稳定性的关键点。

## 3. 代码示例（JavaFX 启动参数）

```bat
"openjdk21\bin\java"^
 --module-path="javafx-sdk-21\lib"^
 --add-modules="javafx.controls,javafx.swing,javafx.web"^
 -cp ".;../;../SECURITY;../DOC;rscad_fx.jar"^
 rscad.RSCAD_FX
```

来源文件：runFromDos.bat

说明：

- 模块路径与 --add-modules 决定 JavaFX 运行时可用组件。  
- -cp 指定类路径并最终启动主类 rscad.RSCAD_FX。

风险：

- 调整模块参数时需与 Java 版本、JavaFX 版本同步验证。  
- 路径改动可能导致启动失败或功能模块缺失。

## 变更记录

- 2026-06-03：初版建立。
