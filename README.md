
# 通用插件

* **插件未开源**

## 作者信息
Copyright FirePlume, All Rights Reserved. Email: fireplume@126.com
 
作者网址：
[Bilibili](https://space.bilibili.com/395084718)、
[YouTube](https://www.youtube.com/@FirePlume126)、
[GitHub](https://www.github.com/FirePlume126)

**[返回目录](https://www.github.com/FirePlume126/FP_Readme#Directory)**

<a name="fpcommon"></a>
## FPCommon

* 此类插件包含通用基础功能
	- [FPCommon](#fpcommon)：包含通用基础功能的插件
		- [FPInput](#fpcommon-fpinput)：管理输入(仅用于C++)
		- [FPUI](#fpcommon-fpui)：管理用户界面
	- [FPLoadingScreen](#fploadingscreen)：游戏启动或地图转换期间的加载界面

<a name="fpcommon-fpInput"></a>
### FPInput

管理输入(仅用于C++)，绑定输入、更改绑定按键。
`UFPInputComponent`是该模块的主要类，它继承于`UEnhancedInputComponent`，在`UFPInputComponent`中管理`FGameplayTag`和输入委托，重复绑定时(`UInputAction`和`ETriggerEvent`和绑定函数的对象相同)，会覆盖旧的绑定(旧的绑定会移除)。

* **使用指南**

1、在项目设置中把`DefaultInputComponentClass`的默认值设置为`UFPInputComponent`

2、**FPInput**项目设置

|属性|数据类型|描述|
|:-:|:-:|:-:|
|DefaultInputConfig|`FFPInputConfig`|在`APlayerController`中绑定输入时，会自动调用`DefaultInputConfig`，生命周期和`APlayerController`相同|
|AbilityInputConfig|`FFPInputConfig`|在`APawn`中绑定输入时，会自动调用`AbilityInputConfig`|
|Context|`UInputMappingContext`|输入映射上下文|
|Priority|`int32`|优先级|
|InputActions|`TMap<FGameplayTag, UInputAction*>`|`UInputAction`要和`Context`绑定|

![FPInputSettings](https://github.com/FirePlume126/FP_Common/blob/main/Images/FPInputSettings.png)

3、调用`UFPInputFunctionLibrary`的函数绑定输入、更改按键绑定，按键绑定保存在“GameUserSettings.ini”

```c++
// 绑定输入映射上下文，重复绑定时，会忽略新绑定
// @param InActor 输入APlayerController调用DefaultInputConfig，输入其他AActor调用AbilityInputConfig
static void AddInputMappings(AActor* InActor);

// 解除输入映射上下文
// @param InActor 输入APlayerController调用DefaultInputConfig，输入其他AActor调用AbilityInputConfig
static void RemoveInputMappings(AActor* InActor);

// 绑定输入动作，重复绑定时(InputTag、TriggerEvent和绑定函数的对象相同)，会覆盖旧的绑定(旧的绑定会移除)
// @param InInputComponent 输入组件，APlayerController的输入组件会调用DefaultInputConfig，否则调用AbilityInputConfig
// @param InputTag 输入标签需要在项目设置DefaultInputConfig或AbilityInputConfig中设置
// @param TriggerEvent 触发状态
// @param Object 绑定函数的类
// @param Func 绑定的函数
template<class UserClass, typename FuncType>
static uint32 BindInputAction(UInputComponent* InInputComponent, const FGameplayTag& InputTag, ETriggerEvent TriggerEvent, UserClass* Object, FuncType Func);

// 绑定GAS输入，重复绑定时(InputTag、TriggerEvent和绑定函数的对象相同)，会覆盖旧的绑定(旧的绑定会移除)
// @param InInputComponent 输入组件，APlayerController的输入组件会调用DefaultInputConfig，否则调用AbilityInputConfig
// @param InputTag 输入标签需要在项目设置AbilityInputConfig中设置
// @param TriggerEvent 触发器事件
// @param Object 绑定函数的类，建议在ACS中调用
// @param Func 绑定的函数
// @param AbilitySpecHandle 能力规格句柄
// @param bActivate 为true是激活能力，为false时关闭能力
template<class UserClass, typename FuncType>
static uint32 BindGASInput(UInputComponent* InInputComponent, const FGameplayTag& InputTag, ETriggerEvent TriggerEvent, UserClass* Object, FuncType Func, const FGameplayAbilitySpecHandle AbilitySpecHandle, bool bActivate);

// 解除能力绑定
static void RemoveAbilityBind(UInputComponent* InInputComponent, uint32 InBindHandle);

// 通过绑定数据解除能力绑定
static void RemoveAbilityInputForBindingInfo(UInputComponent* InInputComponent, const FFPInputBindingInfo& InAction);

// 解除所以能力绑定
static void RemoveAllAbilityBinds(UInputComponent* InInputComponent);

// 查找默认输入动作
static const UInputAction* FindDefaultInputActionForTag(const FGameplayTag& InInputTag);

// 查找能力输入动作
static const UInputAction* FindAbilityInputActionForTag(const FGameplayTag& InInputTag);

// 更改多个绑定，并应用保存
// @param InMapping FName是映射名称，FKey是对应的按键
UFUNCTION(BlueprintCallable, Category = "FPInput")
static void ChangeBindings(TMap<FName, FKey> InMappings);

// 通过映射名称获取按键
UFUNCTION(BlueprintPure, Category = "FPInput")
static FKey GetKeyForName(const FName& InMappingName);

// 通过按键获取映射名称，可以用来检测按键有没有被使用
static TArray<FName> GetMappingNamesForKey(const FKey& InKey);
```

<a name="fpcommon-fpui"></a>
### FPUI

管理用户界面，此模块参考[Lyra Starter Game](https://www.fab.com/listings/93faede1-4434-47c0-85f1-bf27c0820ad0)，用四个堆栈管理控件，聚焦控件时会设置输入模式和显示鼠标光标

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPUIManagerSubsystem|UI管理子系统，管理控件，把`W_UILayoutManager`添加到视口，用来添加移除控件|
|FPUILayoutManager|UI布局管理：用来管理UCommonActivatableWidgetContainerBase堆栈控件|
|W_UILayoutManager|继承`FPUILayoutManager`的控件，添加了四个堆栈控件|
|FPUIUserWidget|继承于`UCommonActivatableWidget`，聚焦`FPUIUserWidget`控件时，会设置输入模式和显示鼠标光标，默认游戏输入模式且不显示鼠标。可以通过`InputMode`设置控件的输入模式 ![FPUI_InputMode](https://github.com/FirePlume126/FP_Common/blob/main/Images/FPUI_InputMode.png)|
|FPUIMenuWidget|继承于`FPUIUserWidget`，默认UI输入模式且显示鼠标，可以被Escape按键关闭|

* **使用指南**

1、跟据[CommonUI官方文档](https://dev.epicgames.com/documentation/unreal-engine/common-ui-quickstart-guide-for-unreal-engine)设置，已在插件中创建了`BP_CommonInputData`和`BP_CommonInput_KeyboardMouse`

`Engine`>`GeneralSettings`>`DefaultClasses`>`GameViewportClientClass`设置为`CommonGameViewportClient`；
`Game`>`CommonInputSettings`>`InputData`设置为`BP_CommonInputData`；
`Game`>`CommonInputSettings`>`PlatformInput`>`Windows`>`Default`>`ControllerData`设置为`BP_CommonInput_KeyboardMouse`

2、创建控件，建议继承`FPUIUserWidget`类。聚焦`FPUIUserWidget`控件时，会设置输入模式和显示鼠标光标

3、在**FPUI**项目设置中添加控件

|UI层堆栈|描述|
|:-:|:-:|
|GameLayerWidgetClasss|游戏层控件类：HUD控件、背景控件|
|GameMenuLayerWidgetClasss|游戏菜单层控件类：与游戏菜单相关控件，如库存控件|
|MenuLayerWidgetClasss|菜单层控件类：设置控件|
|ModalLayerWidgetClasss|模态层控件类：对话框|

![FPUISettings](https://github.com/FirePlume126/FP_Common/blob/main/Images/FPUISettings.png)

4、添加移除控件
![FPUI_Widget](https://github.com/FirePlume126/FP_Common/blob/main/Images/FPUI_Widget.png)

```c++
// 添加控件
// @param InSlotTag 控件对应的标签，在项目设置FPUI中设置
// @return 如果控件已添加，会返回旧控件指针
UFUNCTION(BlueprintCallable, BlueprintCosmetic, meta = (WorldContext = "WorldContextObject"), Category = "FPUI")
static UCommonActivatableWidget* AddWidget(const UObject* WorldContextObject, UPARAM(meta = (Categories = "FPUI.Slot")) FGameplayTag InSlotTag);

// 移除控件
// @param InSlotTag 控件对应的标签，在项目设置FPUI中设置
UFUNCTION(BlueprintCallable, BlueprintCosmetic, meta = (WorldContext = "WorldContextObject"), Category = "FPUI")
static void RemoveWidget(const UObject* WorldContextObject, UPARAM(meta = (Categories = "FPUI.Slot")) FGameplayTag InSlotTag);

// 移除所有控件
UFUNCTION(BlueprintCallable, BlueprintCosmetic, meta = (WorldContext = "WorldContextObject"), Category = "FPUI")
static void RemoveAllWidget(const UObject* WorldContextObject);

// 查找已添加到屏幕的控件
UFUNCTION(BlueprintPure, BlueprintCosmetic, meta = (WorldContext = "WorldContextObject"), Category = "FPUI")
static UCommonActivatableWidget* FindWidget(const UObject* WorldContextObject, UPARAM(meta = (Categories = "FPUI.Slot")) FGameplayTag InSlotTag);
```

<a name="fploadingscreen"></a>
## FPLoadingScreen

游戏启动或转换地图期间的加载界面，该模块基于`MoviePlayer`。
在`MoviePlayer`基础上添加了Slate，层级由低到高分别是：影片，背景控件，加载控件，提示框控件，渐变控件

* **使用指南**

1、开启`bStartupLoadingScree`仅在首次打开游戏时显示一次，对`StartupLoadingScreen`进行设置；开启`bSwitchMapLoadingScreen`，每次切换地图时都会显示，对`SwitchMapLoadingScreen`进行设置。				
![FPLoadingScreen](https://github.com/FirePlume126/FP_Common/blob/main/Images/FPLoadingScreen.png)

2、设置影片：将影片文件复制到“Content/Movies/...”文件夹中，然后将影片路径填入`MoviePaths`。
![FPLoadingScreen_Movie](https://github.com/FirePlume126/FP_Common/blob/main/Images/FPLoadingScreen_Movie.png)

3、设置背景控件：开启`bShowBackgroundWidget`启动背景控件，背景控件主要是由`SBorder`和`SImage`组成。`SBorder`用来设置背景颜色，`SImage`可以设置图片的位置和大小，`Images`如果添加多张图片，会随机获取图片。
![FPLoadingScreen_Background](https://github.com/FirePlume126/FP_Common/blob/main/Images/FPLoadingScreen_Background.png)

4、设置加载控件：开启`bShowLoadingWidget`启动加载控件，加载控件提供了三种样式，分别是`SCircularThrobber`、`SThrobber`和图像序列，将图像序列的图像添加到`Images`中，由`SImage`播放。
![FPLoadingScreen_Loading](https://github.com/FirePlume126/FP_Common/blob/main/Images/FPLoadingScreen_Loading.png)

5、设置提示控件：开启`bShowTipWidget`启动提示控件，提示控件主要是由`SBorder`和`STextBlock`组成。
![FPLoadingScreen_Tip](https://github.com/FirePlume126/FP_Common/blob/main/Images/FPLoadingScreen_Tip.png)

6、设置渐变控件：开启`bShowGradientWidget`启动渐变控件，渐变控件会在加载结束后启动，先让屏幕变黑，然后缓慢让屏幕变亮。`bAutoExecuteGradient`为自动执行渐变，为true时会自动渐变并销毁控件，为false时需要调用`UFPLoadingScreenFunctionLibrary::ExecuteLoadingScreenGradient()`才会执行渐变并销毁控件；`BlackScreenTime`为黑屏时间，`GradientTime`为渐变时间。
![FPLoadingScreen_Gradient](https://github.com/FirePlume126/FP_Common/blob/main/Images/FPLoadingScreen_Gradient.png)
