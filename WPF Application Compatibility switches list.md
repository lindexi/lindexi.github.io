# WPF Application Compatibility switches list

This is a list of all System.AppContext base application compatibility switches present in WPF today, along with their defaults and related documentation. 

<!--more-->
<!-- CreateTime:2022/2/25 19:50:37 -->

<!-- 发布 -->

From: [https://github.com/vatsan-madhavan/WpfAppCompatQuirks](https://github.com/vatsan-madhavan/WpfAppCompatQuirks)

Thank you [Vatsan Madhavan](https://github.com/vatsan-madhavan)

<div id="toc"></div>

## Usage

The compatibility switches can be set by the `runtimeconfig.json` file in dotnet 5 or greater and it can be set by the `App.config` file in .NET Framework, and it can also be set programmatically by calling the AppContext.SetSwitch method. See [.NET Runtime config options - .NET Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/core/runtime-config/ )

For example, we can trun on the WM_Pointer support by add the content in the `runtimeconfig.template.json` file.

```json
{
    "configProperties": 
    {
        "Switch.System.Windows.Input.Stylus.EnablePointerSupport": true
    }
}
```

You can find all the code in github: [https://github.com/lindexi/lindexi_gd/tree/b3c6a4299c3d517a357cc05f045dd3a16ec261e2/WhajecichalLaykeljalha](https://github.com/lindexi/lindexi_gd/tree/b3c6a4299c3d517a357cc05f045dd3a16ec261e2/WhajecichalLaykeljalha)

## Compatibility switches

### Glossary

tfm: target framework moniker, see [https://docs.microsoft.com/en-us/dotnet/standard/frameworks](https://docs.microsoft.com/en-us/dotnet/standard/frameworks?WT.mc_id=WD-MVP-5003260)

### Switch.MS.Internal.DoNotUseCulturePreservingDispatcherOperations

Full Name: MS.Internal.BaseAppContextSwitches.DoNotUseCulturePreservingDispatcherOperations

feature is on when tfm > net452

#### Default Value

true if tfm <= net452 

false otherwise

#### Comments

Starting .NET 4.6, ExecutionContext tracks Thread.CurrentCulture and Thread.CurrentUICulture, which would be restored
to their respective previous values after a call to ExecutionContext.Run. This behavior is undesirable within the
Dispatcher - various dispatcher operations can run user code that can in turn set Thread.CurrentCulture or
Thread.CurrentUICulture, and we do not want those values to be overwritten with their respective previous values.
To work around the new ExecutionContext behavior, we introduce CulturePreservingExecutionContext for use within
Dispatcher and DispatcherOperation. WPF in .NET 4.6 & 4.6.1 shipped with buggy behavior - each DispatcherOperation
ends with all modificaitons to culture infos being reverted.Though unlikely, if some applications targeting 4.6 or
above might have taken a dependence on this bug, we provide this compatiblity switch that can be enabled by the application.


### Switch.MS.Internal.UseSha1AsDefaultHashAlgorithmForDigitalSignatures

Full Name: MS.Internal.BaseAppContextSwitches.UseSha1AsDefaultHashAlgorithmForDigitalSignatures

feature is enabled when tfm > net47

#### Default Value

true if tfm <= net47

false otherwise

#### Comments

PacakageDigitalSignatureManager.DefaultHashAlgorithm is now SHA256. Setting this flag will make it SHA1 as it is in legacy scenarios prior to .NET 4.7.1.



### Switch.MS.Internal.DoNotInvokeInWeakEventTableShutdownListener

Full Name: MS.Internal.BaseAppContextSwitches.DoNotInvokeInWeakEventTableShutdownListener

feature is always enabled by default

#### Default Value

false

#### Comments

Allowing developers to turn off the Invoke added by DDVSO:543980 as there are compat issues with timing during shutdown for some applications.


### Switch.MS.Internal.EnableWeakEventMemoryImprovements

Full Name: MS.Internal.BaseAppContextSwitches.EnableWeakEventMemoryImprovements

feature is always enabled by default

#### Default Value

false

#### Comments

Enable/disable various perf and memory improvements related to WeakEvents


### Switch.MS.Internal.EnableCleanupSchedulingImprovements

Full Name: MS.Internal.BaseAppContextSwitches.EnableCleanupSchedulingImprovements

feature is always enabled by default

#### Default Value

false

#### Comments

Enable/disable heuristic for scheduling cleanup of



### Switch.System.Windows.Markup.DoNotUseSha256ForMarkupCompilerChecksumAlgorithm 	

Full Name: MS.Internal.BuildTasksAppContextSwitches.DoNotUseSha256ForMarkupCompilerChecksumAlgorithm

feature enabled when tfm > net471

#### Default Value

true if tfm <= net471

false otherwise

#### Comments

The WPF MarkupCompiler provides compilation services for XAML markup files. In the .NET Framework 4.7.1 and earlier versions, the default hash algorithm used for checksums was SHA1. Due to recent security concerns with SHA1, this default has been changed to SHA256 starting with the .NET Framework 4.7.2. This change affects all checksum generation for markup files during compilation.



### Switch.System.Windows.DoNotScaleForDpiChanges

Full Name: MS.Internal.CoreAppContextSwitches.DoNotScaleForDpiChanges

feature is enabled when tfm > net461

#### Default Value

true if tfm <= net461

false otherwise

#### Comments

Determines whether DPI changes occur on a per-system (a value of false) or per-monitor basis (a value of true).



### Switch.System.Windows.Input.Stylus.DisableStylusAndTouchSupport

#### Comments

Switch to enable WPF support for the WM_POINTER based stylus/touch stack


### Switch.System.Windows.Media.ImageSourceConverter.OverrideExceptionWithNullReferenceException

feature is enabled when tfm > net462

#### Default Value

true if tfm <= net462

false otherwise

#### Comments

Switch to enable the correct exception being thrown in ImageSourceConverter.ConvertFrom instead of NullReferenceException




### Switch.System.Windows.Diagnostics.DisableDiagnostics

feature is always enabled by default

#### Default Value

false

#### Comments

Switch to disable diagnostic features


### Switch.System.Windows.Diagnostics.AllowChangesDuringVisualTreeChanged

feature is always enabled by default

#### Default Value
false

#### Comments

Switch to allow changes during a VisualTreeChanged event


### Switch.System.Windows.Input.Stylus.DisableImplicitTouchKeyboardInvocation

feature is always enabled by default

#### Default Value

false

#### Comments

Switch to disable automatic touch keyboard invocation on focus of a control


### Switch.UseLegacyAccessibilityFeatures

Full Name: MS.Internal.CoreAppContextSwitches.UseNetFx47CompatibleAccessibilityFeatures

features are enabled when tfm > net47

#### Default Value

true if tfm <= net47; false otherwise

#### Comments

See <https://github.com/microsoft/dotnet/blob/master/Documentation/compatibility/wpf-accessibility-improvements.MD>



### Switch.UseLegacyAccessibilityFeatures.2

Full Name: MS.Internal.CoreAppContextSwitches.UseNetFx471CompatibleAccessibilityFeatures

features are enabled when tfm > net471

#### Default Value

true if tfm <= net471; false otherwise

#### Comments

See <https://github.com/microsoft/dotnet/blob/master/Documentation/compatibility/wpf-accessibility-improvements-48.md>

### Switch.UseLegacyAccessibilityFeatures.3

Full Name: MS.Internal.CoreAppContextSwitches.UseNetFx472CompatibleAccessibilityFeatures

features are enabled when tfm > net472

#### Default Value

true if tfm <= net472; false otherwise

#### Comments

See <https://github.com/microsoft/dotnet/blob/master/Documentation/compatibility/wpf-accessibility-improvements-48.md>

### Switch.UseLegacyToolTipDisplay

Full Name: System.Windows.AccessibilitySwitches.UseLegacyToolTipDisplay

features are enabled when tfm > net472

#### Default Value

true if tfm <= net472; false otherwise

#### Comments

See <https://github.com/microsoft/dotnet/blob/master/Documentation/compatibility/wpf-accessibility-improvements-48.md>

### Switch.System.Windows.Controls.ItemsControlDoesNotSupportAutomation

Full Name: System.Windows.AccessibilitySwitches.ItemsControlDoesNotSupportAutomation

features are enabled when tfm > net472

#### Default Value

true if tfm <= net472; false otherwise

#### Comments

See <https://github.com/microsoft/dotnet/blob/master/releases/net472/dotnet472-changes.md>


### Switch.System.Windows.Media.ShouldRenderEvenWhenNoDisplayDevicesAreAvailable

Full Name: MS.Internal.CoreAppContextSwitches.ShouldRenderEvenWhenNoDisplayDevicesAreAvailable

feature is always enabled by default

#### Default Value

false

#### Comments

Desktop/Interactive Window Stations:

Rendering will be throttled back/stopped when no display devices are available. For e.g., when a TS
session is in WTSDisconnected state, the OS may not provide any display devices in response to our enumeration.
If an application would like to continue rendering in the absence of display devices (accepting that
it can lead to a CPU spike), it can set to true.
Service/Non-interactive Window Stations
Rendering will continue by default, irrespective of the presence of display devices.Unless the WPF
API's being used are shortlived (like rendering to a bitmap), it can lead to a CPU spike.
If an application running inside a service would like to receive the 'default' WPF behavior,
i.e., no rendering in the absence of display devices, then it should set
to true.

In pseudocode,

```
IsNonInteractiveWindowStation = !Environment.UserInteractive
IF DisplayDevicesNotFound() THEN
IF IsNonInteractiveWindowStation THEN
// We are inside a SCM service
// Default = True, AppContext switch can override it to False
ShouldRender = !CoreAppContextSwitches.ShouldNotRenderInNonInteractiveWindowStation
ELSE
// Desktop/interactive mode, including WTSDisconnected scenarios
// Default = False, AppContext switch can override it to True
ShouldRender = CoreAppContextSwitches.ShouldRenderEvenWhenNoDisplayDevicesAreAvailable
END IF
END IF
```

### Switch.System.Windows.Media.ShouldNotRenderInNonInteractiveWindowStation

Full Name: MS.Internal.CoreAppContextSwitches.ShouldNotRenderInNonInteractiveWindowStation

feature is always enabled by default

#### Default Value

false

#### Comments


Desktop/Interactive Window Stations:

Rendering will be throttled back/stopped when no display devices are available. For e.g., when a TS
session is in WTSDisconnected state, the OS may not provide any display devices in response to our enumeration.
If an application would like to continue rendering in the absence of display devices (accepting that
it can lead to a CPU spike), it can set
to true.
Service/Non-interactive Window Stations
Rendering will continue by default, irrespective of the presence of display devices.Unless the WPF
API's being used are shortlived (like rendering to a bitmap), it can lead to a CPU spike.
If an application running inside a service would like to receive the 'default' WPF behavior,
i.e., no rendering in the absence of display devices, then it should set
to true.

In pseudocode,

```
IsNonInteractiveWindowStation = !Environment.UserInteractive
IF DisplayDevicesNotFound() THEN
IF IsNonInteractiveWindowStation THEN
// We are inside a SCM service
// Default = True, AppContext switch can override it to False
ShouldRender = !CoreAppContextSwitches.ShouldNotRenderInNonInteractiveWindowStation
ELSE
// Desktop/interactive mode, including WTSDisconnected scenarios
// Default = False, AppContext switch can override it to True
ShouldRender = CoreAppContextSwitches.ShouldRenderEvenWhenNoDisplayDevicesAreAvailable
END IF
END IF
```

### Switch.System.Windows.DoNotUsePresentationDpiCapabilityTier2OrGreater

Full Name: MS.Internal.CoreAppContextSwitches.DoNotUsePresentationDpiCapabilityTier2OrGreater

feature is enabled when tfm > net472

#### Default Value

true if tfm <= net472

false otherwise

#### Comments

When set to true, WPF will not enable the compatibility breaking bug fixes associated with
features advertised by "Switch.System.Windows.PresentationDpiCapabilityTier2"

The following behavior would be turned off when this flag is set by the application:

- Improvements to how HwndHost sizes child windows in response to DPI changes
- Improvements to window placement during startup

The following fixes would remain unaffected:

- High-DPI related accessibility fixes.


### Switch.System.Windows.DoNotUsePresentationDpiCapabilityTier3OrGreater

Full Name: MS.Internal.CoreAppContextSwitches.DoNotUsePresentationDpiCapabilityTier3OrGreater

feature is always enabled by default

#### Default Value

false

#### Comments

Reserved for future use


### Switch.System.Windows.AllowExternalProcessToBlockAccessToTemporaryFiles 

Full Name: MS.Internal.CoreAppContextSwitches.AllowExternalProcessToBlockAccessToTemporaryFiles

feature is always enabled by default

#### Default Value

false



### Switch.System.Windows.EnableLegacyDangerousClipboardDeserializationMode

Full Name: MS.Internal.CoreAppContextSwitches.EnableLegacyDangerousClipboardDeserializationMode

feature is always enabled by default

#### Default Value

false

#### Comments

Malicious managed objects could be placed in the clipboard lying about its format,
to fix this OleConverter now restricts object deserialization in some cases.
When this switch is enabled behavior falls back to deserializing without restriction.


### Switch.MS.Internal.DoNotApplyLayoutRoundingToMarginsAndBorderThickness

Full Name: MS.Internal.FrameworkAppContextSwitches.DoNotApplyLayoutRoundingToMarginsAndBorderThickness

feature is enabled when tfm > net452

#### Default Value

true if tfm <= net452

false otherwise

#### Comments

See <https://github.com/microsoft/dotnet/blob/master/Documentation/compatibility/wpf-layout-rounding-of-margins-has-changed.md>



### Switch.System.Windows.Controls.Grid.StarDefinitionsCanExceedAvailableSpace

Full Name: MS.Internal.FrameworkAppContextSwitches.GridStarDefinitionsCanExceedAvailableSpace

feature is enabled when tfm > net462

#### Default Value

true if tfm <= net462

false otherwise

#### Comments

See <https://github.com/microsoft/dotnet/blob/master/Documentation/compatibility/wpf-grid-allocation-of-space-to-star-columns.md>


### Switch.System.Windows.Controls.TabControl.SelectionPropertiesCanLagBehindSelectionChangedEvent

Full Name: MS.Internal.FrameworkAppContextSwitches.SelectionPropertiesCanLagBehindSelectionChangedEvent

feature is enabled when tfm > net47

#### Default Value

true if tfm <= net47

false otherwise


#### Comments

See <https://github.com/microsoft/dotnet/blob/master/Documentation/compatibility/wpf-tabcontrol-selectionchanged-and-selectedcontent.md>


### Switch.System.Windows.Data.DoNotUseFollowParentWhenBindingToADODataRelation

Full Name: MS.Internal.FrameworkAppContextSwitches.DoNotUseFollowParentWhenBindingToADODataRelation

feature is enabled when tfm > net471

#### Default Value

true if tfm <= net471

false otherwise


#### Comments

See <https://github.com/microsoft/dotnet/blob/master/Documentation/compatibility/wpf-MasterDetail-ADOdata-PrimaryKey.md>


### Switch.System.Windows.Controls.Text.UseAdornerForTextboxSelectionRendering

Full Name: MS.Internal.FrameworkAppContextSwitches.UseAdornerForTextboxSelectionRendering

feature is always off by default; this is an opt-in only feature

#### Default Value

always true

#### Comments

Switch to enable non-adorner based rendering of TextSelection in TextBox and PasswordBox




### Switch.System.Windows.Baml2006.AppendLocalAssemblyVersionForSourceUri

Full Name: MS.Internal.FrameworkAppContextSwitches.AppendLocalAssemblyVersionForSourceUri

#### Default Value

false


#### Comments

Switch to enable appending the local assembly version to the Uri being set for ResourceDictionary.Source via Baml2006ReaderInternal.


### Switch.System.Windows.Data.Binding.IListIndexerHidesCustomIndexer

Full Name: MS.Internal.FrameworkAppContextSwitches.IListIndexerHidesCustomIndexer

feature is enabled when net4 > tfm > net472

#### Default Value

true if net4 >= tfm <= net472

false otherwise

#### Comments

Switch to enable IList indexer hiding a custom indexer in a binding path



### Switch.System.Windows.Controls.KeyboardNavigationFromHyperlinkInItemsControlIsNotRelativeToFocusedElement

Full Name: MS.Internal.FrameworkAppContextSwitches.KeyboardNavigationFromHyperlinkInItemsControlIsNotRelativeToFocusedElement



#### Default Value

false


#### Comments

Switch to enable keyboard navigation from a hyperlink to go to the wrong place




### Switch.System.Windows.Automation.Peers.ItemAutomationPeerKeepsItsItemAlive

Full Name: MS.Internal.FrameworkAppContextSwitches.ItemAutomationPeerKeepsItsItemAlive 

#### Default Value

false

#### Comments

Switch to opt-out of the ItemAutomationPeer weak-reference.
Setting this to true can avoid NRE crashes, but re-introduces some memory leaks


### Switch.System.Windows.Media.MediaContext.DisableDirtyRectangles

The problem is due to a limitation of D3D - the content it presents via dirty-rectangles can be out of sync with the rest of the content, if the timing of the presents and fills is unfortunate. WPF cannot fix this, but we can mitigate it by giving the app a way to disable the dirty-rectangle optimization. This impacts the graphics performance; by opting-in the app accepts the tradeoff of performance vs. fidelity.

See <https://github.com/dotnet/wpf/pull/5837> <https://github.com/dotnet/wpf/issues/5441>

### Switch.System.Windows.Media.MediaContext.EnableDynamicDirtyRectangles

See Switch.System.Windows.Media.MediaContext.DisableDirtyRectangles

See <https://github.com/dotnet/wpf/pull/5837> <https://github.com/dotnet/wpf/issues/5441>


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
