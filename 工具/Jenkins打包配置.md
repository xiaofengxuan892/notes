[TOC]



#### 热更项目配置：

```
set pg=dctripeaksb
rd /s /q UGameAndroid\unityLibrary\src\main\assets
rd /s /q UGameAndroid\unityLibrary\src\main\jniLibs
rd /s /q UGameAndroid\unityLibrary\src\main\Il2CppOutputProject
rd /s /q UGameAndroid\unityLibrary\src\main\jniStaticLibs

if %APK_debug% == true (
"C:\Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnitySyncAndroidStudio_Debug -quit
)
if %APK_debugFormal% == true (
"C:\Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnitySyncAndroidStudio_DebugFormal -quit
)
if %APK_release_aab% == true (
"C:\Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnitySyncAndroidStudio_Release -quit
)
if %Assembly_debug% == true (
"C:\Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnitySyncAssemblyDll_Debug -quit
)
if %Assembly_debugFormal% == true (
"C:\Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnitySyncAssemblyDll_DebugFormal -quit
)
if %Assembly_release% == true (
"C:\Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnitySyncAssemblyDll_Release -quit
)
if %AOT_Assembly_debug% == true (
"C:\Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnitySyncAllDll_Debug -quit
)
if %AOT_Assembly_debugFormal% == true (
"C:\Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnitySyncAllDll_DebugFormal -quit
)
if %AOT_Assembly_release% == true (
"C:\Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnitySyncAllDll_Release -quit
)


@ECHO OFF
echo Exit Code is %errorlevel%
if %errorlevel% neq 0 (
   echo unity编译错误： %errorlevel%
   type unity_build.log
   echo 请检查以上unity错误！
   exit /b %errorlevel%
)

echo 开始android编译

cd %WORKSPACE%\UGameAndroid
rd /s /q outputs
mkdir outputs

if %APK_debug% == true (
   call _build_%pg%_debug.bat
   echo d | XCOPY app\build\outputs outputs\outputs_debug /s /e
)

if %APK_debugFormal% == true (
   call _build_%pg%_debugFormal.bat
   echo d | XCOPY app\build\outputs outputs\outputs_debugFormal /s /e
)

if %APK_release_aab% == true (
   call _build_%pg%_aab.bat
   echo d | XCOPY app\build\outputs outputs\outputs_aab /s /e
)


echo 飞书通知
curl --ssl-no-revoke -X POST -H "Content-Type: application/json;charset=utf-8" -d "{\"msg_type\": \"text\",\"content\": {\"text\":\"%pg% Success\nhttp://hq.aoemo.com:28088/job/%pg%_hot/%BUILD_NUMBER%/\"}}" "https://open.feishu.cn/open-apis/bot/v2/hook/f2e92143-1d80-46db-91d9-a66c77ca2739"

```



#### 打白包配置：

```
set pg=dcquizb
rd /s /q UGameAndroid\unityLibrary\src\main\assets
rd /s /q UGameAndroid\unityLibrary\src\main\jniLibs
rd /s /q UGameAndroid\unityLibrary\src\main\Il2CppOutputProject
rd /s /q UGameAndroid\unityLibrary\src\main\jniStaticLibs


if %debug% == true (
"C:Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnityWhitePg_Debug -quit
)

if %release_aab% == true (
"C:Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnityWhitePg_Release -quit
)

@ECHO OFF
echo Exit Code is %errorlevel%
if %errorlevel% neq 0 (
   echo unity编译错误： %errorlevel%
   type unity_build.log
   echo 请检查以上unity错误！
   exit /b %errorlevel%
)

echo 开始android编译

echo 飞书通知
curl -X POST -H "Content-Type: application/json;charset=utf-8" -d "{\"msg_type\": \"text\",\"content\": {\"text\":\"%pg% Success\nhttp://hq.aoemo.com:28088/job/%pg%_whitePg/%BUILD_NUMBER%/\"}}" "https://open.feishu.cn/open-apis/bot/v2/hook/f2e92143-1d80-46db-91d9-a66c77ca2739"
```



#### 变现包配置：

```
set pg=dcquizb
rd /s /q UGameAndroid\unityLibrary\src\main\assets
rd /s /q UGameAndroid\unityLibrary\src\main\jniLibs
rd /s /q UGameAndroid\unityLibrary\src\main\Il2CppOutputProject
rd /s /q UGameAndroid\unityLibrary\src\main\jniStaticLibs

"C:Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Unity.exe" -logFile unity_build.log -batchmode -buildTarget Android -projectPath UGameFrame\%pg%_UClient -executeMethod BuildProjectApkClientTool.BuildUnitySyncAndroidStudio -quit

@ECHO OFF
echo Exit Code is %errorlevel%
if %errorlevel% neq 0 (
   echo unity编译错误： %errorlevel%
   type unity_build.log
   echo 请检查以上unity错误！
   exit /b %errorlevel%
)

echo 开始android编译

cd %WORKSPACE%\UGameAndroid
rd /s /q outputs
mkdir outputs

if %debug% == true (
   call _build_%pg%_debug.bat
   echo d | XCOPY app\build\outputs outputs\outputs_debug /s /e
)

if %debugFormal% == true (
   call _build_%pg%_debugFormal.bat
   echo d | XCOPY app\build\outputs outputs\outputs_debugFormal /s /e
)

if %release_aab% == true (
   call _build_%pg%_aab.bat
   echo d | XCOPY app\build\outputs outputs\outputs_aab /s /e
)

echo 飞书通知
curl --ssl-no-revoke -X POST -H "Content-Type: application/json;charset=utf-8" -d "{\"msg_type\": \"text\",\"content\": {\"text\":\"%pg% Success\nhttp://hq.aoemo.com:28088/job/%pg%/%BUILD_NUMBER%/\"}}" "https://open.feishu.cn/open-apis/bot/v2/hook/f2e92143-1d80-46db-91d9-a66c77ca2739"
```

