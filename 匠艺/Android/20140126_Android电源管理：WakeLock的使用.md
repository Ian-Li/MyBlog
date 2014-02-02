Android电源管理,WakeLock的使用
----------
Android是针对移动端设计的操作系统，电源管理非常重要。所以当手机休眠时，屏幕和键盘背光会关闭，而后CPU会挂起，以便省电。手机休眠可通过按手机电源键手动休眠，或根据设置的休眠时间，当手机一段时间未操作自动休眠。

在现实场景中，这种机制会造成某些困惑。譬如：当用户使用手机阅读时，因为一段时间未操作，导致在阅读时手机自动休眠。还有一些应用希望手机休眠后，CPU依然运转，保证应用后台服务持续运行。

为了解决上述问题，Android提出了**锁**的概念，使用**WakeLock**这种电源管理机制灵活控制CPU，屏幕，物理键盘背光在手机休眠时的行为。

使用WakeLock前需要在Manifest文件中申请权限：
	
	<uses-permission android:name="android.permission.WAKE_LOCK" />

示例代码：

	PowerManager pm = (PowerManager) getSystemService(Context.POWER_SERVICE);
	PowerManager.WakeLock wl = pm.newWakeLock(PowerManager.FULL_WAKE_LOCK
                | PowerManager.ON_AFTER_RELEASE, "");

	wl.acquire();

	// ...do someting...
	
	wl.release();


**注意：WakeLock的获取acquire()和释放release()必须成对出现，并且当任务完成后需尽快释放锁以降低电能损耗。**

PowerManager.newWakeLock(int levelAndFlags, String tag)的参数levelAndFlags表示锁的等级和锁的标记，通过不同锁等级和锁标记的组合(**OR运算**)使用，灵活控制手机休眠行为。

**四种锁等级：**

ARTIAL_WAKE_LOCK：保持CPU运转，但屏幕和键盘背光灯可关闭。

SCREEN_DIM_WAKE_LOCK：保持CPU运转，屏幕会变暗，键盘背光灯可关闭。当用户通过电源按钮手动触发设备休眠时，系统会自动释放此锁，使CPU和屏幕关闭。

SCREEN_BRIGHT_WAKE_LOCK：保持CPU运转，屏幕全亮，键盘背光灯可关闭。当用户通过电源按钮手动触发设备休眠时，系统会自动释放此锁，使CPU和屏幕关闭。

FULL_WAKE_LOCK：保持CPU运转且屏幕和键盘背光灯全亮。当用户通过电源按钮手动触发设备休眠时，系统会自动释放此锁，使CPU和屏幕关闭。

<table>
<tbody>
<tr><td><em>Level Value</em></td><td><em>CPU</em></td><td><em>Screen</em></td><td><em>Keyboard</em></td>
</tr>
<tr><td>PARTIAL_WAKE_LOCK</td><td>On</td><td>Off</td><td>Off</td></tr>
<tr><td>SCREEN_DIM_WAKE_LOCK</td><td>On</td><td>Dim</td><td>Off</td></tr>
<tr><td>SCREEN_BRIGHT_WAKE_LOCK</td><td>On</td><td>Bright</td><td>Off</td></tr>
<tr><td>FULL_WAKE_LOCK</td><td>On</td><td>Bright</td><td>Bright</td></tr>
</tbody>
</table>

**两种锁标记：**

ACQUIRE_CAUSES_WAKEUP：当获得WakeLock时，如果屏幕已亮则维持。

ON_AFTER_RELEASE：当释放WakeLock后，如果屏幕已亮，则维持屏幕一小段时间。

**注意：这两种锁标记是用来控制屏幕的，不能和PARTIAL_WAKE_LOCK一起使用。**