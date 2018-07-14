---
layout:     post
title:      "iOS-Fastlane自动化打包脚本"
subtitle:   "在本文中记录iOS-Fastlane自动化打包脚本。"
date:       2018-07-14
author:     "minsol"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - 备忘录
---


```bash
#!/bin/bash
# 使用 fastlane gym 自动打包 ipa

# echo util
COLOR_Cyan='\033[0;36m'
COLOR_Red='\033[41;37m'
COLOR_Default='\033[0;m'

function echo_log() {
    echo "${COLOR_Cyan}$1${COLOR_Default}"
}

function echo_error() {
    echo "${COLOR_Red}$1${COLOR_Default}"
}
# todo: 打包前先执行 pod install

# 计时
SECONDS=0
NOW=$(date +"%Y_%m_%d_%H_%M_%S")

#<-----------------------------脚本配置项目（根据实际情况修改）------------------------------------>
# 配置工程路径（必须）
readonly targetProject_Path="/Users/guoguangtao/Desktop/A8/A8TV_IPhone"
# 配置项目名称（必须）
readonly targetProject_Name="A8TV_IPhone"
readonly target_Workspace_path="${targetProject_Path}/${targetProject_Name}.xcworkspace"

# 配置输出IPA路径（必须）
readonly saveAndUpload_Path="/Users/guoguangtao/Desktop/test"
readonly output_Archive_Path="${saveAndUpload_Path}/${targetProject_Name}${NOW}.xcarchive"
readonly output_IPA_Name="${targetProject_Name}${NOW}.ipa"
readonly output_IPA_Path="${saveAndUpload_Path}/${output_IPA_Name}"

#<--------------------------------------上传账号信息配置---------------------------------------------->
# 配置蒲公英（必须）
#pgy_userKey="e0f747d911678c1946a255a3c474ff33"
#pgy_apiKey="0b09934f2cbbaa28bb3c3cc28ba09f40"

# 公司账号
pgy_userKey="69cf59912c409cdf0b1c71d606abcf9c"
pgy_apiKey="c82f0298d6616c3dc3a4ac02c7919399"

# 配置Apple_ID（必须）
#apple_ID="lvrvi27@163.com"
#apple_Password="U5KNJa3r68pb"
#<--------------------------------------------------------------------------------------------->


#<-------------------------------------用户输入打包方式----------------------------------------->
echo_log "=================选择打包方式(输入序号)=============="
echo_log "  1 Release"
echo_log "  2 Debug"
read -t 30 -p "选择打包方式:" build_Configuration_count
echo "\n"
echo_log "打包方式:$build_Configuration_count"

# 目前支持["app-store", "ad-hoc", "package", "enterprise", "development", "developer-id"]即xcodebuild的method参数
if test $build_Configuration_count = "1"
then
    build_Configuration="Release"
    export_Method="app-store"
    # export_Method="ad-hoc"
elif test $build_Configuration_count = "2"
then 
    build_Configuration="Debug"
    export_Method="development"
else
    echo "参数无效...."
    exit 1
fi


#输出设定的变量值
echo_log "============ config list =============="
echo_log "1 build configuration = ${build_Configuration}"
echo_log "2 export method = ${export_Method}"
echo_log "3 workspace path = ${target_Workspace_path}"
echo_log "4 scheme name = ${targetProject_Name}"
echo_log "5 archive path = ${output_Archive_Path}"
echo_log "6 ipa path = ${output_IPA_Path}"
echo_log "7 ipa name = ${output_IPA_Name}"
echo_log "======================================="

# 强制更新证书 (非必须)
# fastlane sigh download_all -u ${apple_ID} --adhoc --force


#说明：
#用\来进行换行分隔，一条shell命令过长时可以进行分割显示.
#$变量名是引用变量，拿来使用
#|| exit 指明如果这一条命令执行失败，则退出当前shell.

# gym和build_app都是build_ios_app的别名
fastlane gym --workspace ${target_Workspace_path} \
            --scheme ${targetProject_Name} \
            --clean \
            --configuration ${build_Configuration} \
            --archive_path ${output_Archive_Path} \
            --export_method ${export_Method} \
            --output_directory ${saveAndUpload_Path} \
            --output_name ${output_IPA_Name} \
            --export_xcargs "-allowProvisioningUpdates"

# $?    显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。
# 不等于则为真
if [ "$?" -ne "0" ]; then
    echo_error "build error 停止自动构建"
    exit 1
fi

echo_log "===== 打包耗时：${SECONDS}s ====="
echo_log "===== 包路径：${output_IPA_Path} ====="

open ${saveAndUpload_Path}

# CURL 上传到蒲公英，并接收 Response
PGY_RESPONSE=`curl -F "file=@${output_IPA_Path}" -F "uKey=${pgy_userKey}" -F "_api_key=${pgy_apiKey}" https://www.pgyer.com/apiv1/app/upload`

if [ "$?" -ne "0" ]; then
    echo_error "upload error"
    exit 1
fi


# 上传到APP Store
#altoolPath="/Applications/Xcode.app/Contents/Applications/Application Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Versions/A/Support/altool"
#"${altoolPath}" --validate-app -f ${output_IPA_Path} -u ${apple_ID} -p ${apple_Password} -t ios --output-format xml
#"${altoolPath}" --upload-app -f ${output_IPA_Path} -u ${apple_ID} -p ${apple_Password} -t ios --output-format xml





```

