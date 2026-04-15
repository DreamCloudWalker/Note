确认机制：在其它应用前台时，QuickStep 由 `TouchInteractionService` 创建 `OtherActivityInputConsumer`；`disableHorizontalSwipe` 目前仅在 `isInExclusionRegion` 为 true 时启用，因此在导航条区域默认允许横向“快速切换”。将 `disableHorizontalSwipe` 设为始终为 `true` 可屏蔽在其它应用内从底部横滑切换应用。

TouchInteractionService.java

+3-1

```Java
 || gestureState.getActivityInterface().deferStartingActivity(mDeviceState, event);
  final boolean disableHorizontalSwipe = mDeviceState.isInExclusionRegion(event);
  // When false, nav-bar horizontal drags start "quick switch" between recent tasks. Set true
  // to disable that gesture while another app is in the foreground (swipe-up still works).
  final boolean disableHorizontalSwipe = true;
```

TouchInteractionService.java

+1-1

```
   "overview shouldn't have focus, using OverviewWithoutFocusInputConsumer");
  final boolean disableHorizontalSwipe = mDeviceState.isInExclusionRegion(event);
  final boolean disableHorizontalSwipe = true;
  return new OverviewWithoutFocusInputConsumer(activity, mDeviceState, gestureState,
```

## 原因说明

你说的「在 App A 最底部左右滑切换到 App B」是 QuickStep / Launcher 作为 Recents 提供者 时的 底部导航条横向快速切换（quick switch）：

- 前台是其它应用时，触摸由 `TouchInteractionService` 交给 `OtherActivityInputConsumer` 处理。
- 参数 `disableHorizontalSwipe` 为 false 时，一旦识别成「横向为主」的手势，就会走切到上一个/下一个任务的逻辑。
- 原逻辑里只有 `mDeviceState.isInExclusionRegion(event)` 为 true 时才把 `disableHorizontalSwipe` 设为 true，所以在普通底部区域横向滑仍然会切应用。

`OtherActivityInputConsumer` 里在超过 touch slop 且横向位移大于纵向时会直接取消手势，从而阻断 quick switch：

```java
                if (!mPassedPilferInputSlop) {
                    if (passedSlop) {
                        if (mDisableHorizontalSwipe
                                && Math.abs(displacementX) > Math.abs(displacementY)) {
                            // Horizontal gesture is not allowed in this region
                            forceCancelGesture(ev);
                            break;
```



## 已做修改

在 `TouchInteractionService.java` 里：

1. `createOtherActivityInputConsumer`：把 `disableHorizontalSwipe` 固定为 `true`，这样在其它 App 内从底部横滑不再触发应用切换；上滑回桌面 / 进概览仍以纵向为主，不受影响（横向占优才会被取消）。
2. `createOverviewInputConsumer` 里 `OverviewWithoutFocusInputConsumer` 分支：同样改为 `true`，避免在类似「概览无焦点」场景下仍允许横向行为不一致。

若你还希望 在桌面上也禁止从导航条横滑切任务，需要再改 `QuickstepLauncher.createTouchControllers()`：对 NO_BUTTON 去掉 `NoButtonQuickSwitchTouchController`，对 TWO_BUTTONS 去掉 `QuickSwitchTouchController` / `TransposedQuickSwitchTouchController`。需要的话可以说一下要关到哪几种导航模式。
