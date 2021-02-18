## How to add the blur effects on MiuiSystemUI.apk?

If you have a low-end device, you probably noticed the gray background color on your control center and notification shade. Let's bring back the glorious blur effect.

Pre-requisites:

+ `adb`
+ `smali/baksmali`
+ `zip`
+ `A tool to decompile an apk file`
+ `Common knowledge. I'm not responsible if you bricked your device or caused a thermonuclear war. ;)`

### Prepare the tools needed.

Install the tools required for this operation. Use your favorite distro's package manager to install it or clone it from its github repository.

### Pull MiuiSystemUI.apk from your device

```
$ adb pull /system/priv-app/MiuiSystemUI/MiuiSystemUI.apk .
```

The command above will copy the apk file to $PWD. It's recommended to create a backup. We will be overwriting it later.


### Decompile or Extract the APK

Decompile the APK with your favorite tool. I will use `MT Manager`, an android application, for this. The extracted files and folders should be:

	+ assets/
	+ kotlin/
	+ META-INF/
	+ res/
	+ AndroidManifest.xml 
	+ classes.dex
	+ resources.arsc

Go to the extracted folder.

### Disassemble classes.dex


Disassemble it by:

```
$ baksmali d classes.dex
```

This command should create the `out/` folder. It will contain all the `.smali` files extracted from `classes.dex`.

Go inside the `out/` folder.

### Editing .smali files to enable blur

This is the important part. There are two files we need to edit. The `ControlPanelWindowManager.smali` that will enable the blur on the control center and `StatusBarWindowManager.smali` that will enable the blur on the notification shade.

1. Open `com/android/systemui/miui/statusbar/phone/ControlPanelWindowManager.smali` with your favorite text editor.
2. Find the `applyBlurRatio()` method. It should be on line 101 to 148 (could be different on your device). Then change the whole function to:

```smali
.method private applyBlurRatio(F)V
    .registers 5

    .line 180
    invoke-virtual {p0}, Lcom/android/systemui/miui/statusbar/phone/ControlPanelWindowManager;->hasAdded()Z

    move-result v0

    if-eqz v0, :cond_2b

    .line 181
    new-instance v0, Ljava/lang/StringBuilder;

    invoke-direct {v0}, Ljava/lang/StringBuilder;-><init>()V

    const-string v1, "setBlurRatio: "

    invoke-virtual {v0, v1}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;

    invoke-virtual {v0, p1}, Ljava/lang/StringBuilder;->append(F)Ljava/lang/StringBuilder;

    invoke-virtual {v0}, Ljava/lang/StringBuilder;->toString()Ljava/lang/String;

    move-result-object v0

    const-string v1, "ControlPanelWindowManager"

    invoke-static {v1, v0}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I

    .line 182
    iget-object v0, p0, Lcom/android/systemui/miui/statusbar/phone/ControlPanelWindowManager;->mLpChanged:Landroid/view/WindowManager$LayoutParams;

    iget-object v1, p0, Lcom/android/systemui/miui/statusbar/phone/ControlPanelWindowManager;->mControlPanel:Lcom/android/systemui/miui/statusbar/phone/ControlPanelWindowView;

    invoke-virtual {v1}, Landroid/widget/FrameLayout;->getViewRootImpl()Landroid/view/ViewRootImpl;

    move-result-object v1

    const/4 v2, 0x0

    invoke-static {v0, v1, p1, v2}, Landroid/view/SurfaceControlCompat;->setBlur(Landroid/view/WindowManager$LayoutParams;Landroid/view/ViewRootImpl;FI)V

    .line 183
    invoke-direct {p0}, Lcom/android/systemui/miui/statusbar/phone/ControlPanelWindowManager;->apply()V

    :cond_2b
    return-void
.end method
```

3. Save it.

4. Open `com/android/systemui/statusbar/phone/StatusBarWindowManager.smali` with your favorite text editor.

5. Find the `applyBlurRatio()` method. It should be on line 294 to 341. Then change the whole function to:

```smali
.method private applyBlurRatio(Lcom/android/systemui/statusbar/phone/StatusBarWindowManager$State;)V
    .registers 6

    .line 320
    iget-object v0, p0, Lcom/android/systemui/statusbar/phone/StatusBarWindowManager;->mLpChanged:Landroid/view/WindowManager$LayoutParams;

    iget-object v1, p0, Lcom/android/systemui/statusbar/phone/StatusBarWindowManager;->mStatusBarView:Landroid/view/ViewGroup;

    invoke-virtual {v1}, Landroid/view/ViewGroup;->getViewRootImpl()Landroid/view/ViewRootImpl;

    move-result-object v1

    iget v2, p1, Lcom/android/systemui/statusbar/phone/StatusBarWindowManager$State;->blurRatio:F

    const/4 v3, 0x0

    invoke-static {v0, v1, v2, v3}, Landroid/view/SurfaceControlCompat;->setBlur(Landroid/view/WindowManager$LayoutParams;Landroid/view/ViewRootImpl;FI)V

    .line 323
    iget-object p0, p0, Lcom/android/systemui/statusbar/phone/StatusBarWindowManager;->mBlurRatioListeners:Ljava/util/List;

    invoke-interface {p0}, Ljava/util/List;->iterator()Ljava/util/Iterator;

    move-result-object p0

    :goto_14
    invoke-interface {p0}, Ljava/util/Iterator;->hasNext()Z

    move-result v0

    if-eqz v0, :cond_26

    invoke-interface {p0}, Ljava/util/Iterator;->next()Ljava/lang/Object;

    move-result-object v0

    check-cast v0, Lcom/android/systemui/statusbar/phone/StatusBarWindowManager$BlurRatioChangedListener;

    .line 324
    iget v1, p1, Lcom/android/systemui/statusbar/phone/StatusBarWindowManager$State;->blurRatio:F

    invoke-interface {v0, v1}, Lcom/android/systemui/statusbar/phone/StatusBarWindowManager$BlurRatioChangedListener;->onBlurRatioChanged(F)V

    goto :goto_14

    :cond_26
    return-void
.end method
```

6. Save it.

### Assemble .smali files back to classes.dex

Inside `out/` folder. Execute the command below to assemble all smali files back to classes.dex

```
$ smali a . -o classes.dex
```

This should create a new `classes.dex` on the current directory.

### Build a new MiuiSystemUI.apk

Move or copy the compiled `classes.dex` to the extracted `MiuiSystemUI.apk`. This should replace the old one. Then delete the `out/` folder. We don't need it anymore. Create a new MiuiSystemUI.apk by:

```
$ zip -r MiuiSystemUI.apk assets/ kotlin/ META-INF/ res/ AndroidManifest.xml classes.dex resources.arsc
```

### Push the new MiuiSystemUI.apk

Go to your custom recovery, mount `/system`, then push the modified MiuiSystemUI.apk.

```
$ adb push MiuiSystemUI.apk /system/priv-app/MiuiSystemUI/
```

### Finish!

Reboot.

### Extras

You can also make a magisk module with this! Tried and tested it myself. Sadly, I only use China-rom and didn't want to maintain a module for the other variants. So yeah.