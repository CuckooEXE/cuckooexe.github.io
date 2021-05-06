---
layout: post
title: Icon Shuffle
excerpt_separator: <!--more-->
categories:
- Capability Development
author:
- Axel Persinger
---

My friend and I were discussing how nerfarious it would be to shuffle someone's icons on their desktop...

<!--more-->

So I wrote some code to do exactly that! I haven't written anything using the Win32 API recently, so I wanted a small side project. 
Definitely don't compile this binary and add it to `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp`, and **definitely** don't do the following:

```powershell
# https://docs.microsoft.com/en-us/powershell/module/scheduledtasks/new-scheduledtask?view=windowsserver2019-ps
# https://stackoverflow.com/a/57051967/10280970
$A = New-ScheduledTaskAction -Execute "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\IconShuffle.exe"
$T = New-ScheduledTaskTrigger -Daily -At 01:00
$t2 = New-ScheduledTaskTrigger -Once -RepetitionInterval (New-TimeSpan -Minutes 15) -RepetitionDuration (New-TimeSpan -Hours 23 -Minutes 55) -At 01:00
$T.Repetition = $t2.Repetition
$P = New-ScheduledTaskPrincipal $(whoami)
$S = New-ScheduledTaskSettingsSet
$D = New-ScheduledTask -Action $A -Principal $P -Trigger $T -Settings $S
Register-ScheduledTask T1 -InputObject $D
```

:^)

Anyway, here's the [code](https://github.com/CuckooEXE/Peeve/tree/master/IconShuffle), you can compile it yourself with MSVC. Just be sure to use the x64 target, because this won't work as x86 (unless you're running a 32-bit version of Windows).


**EDIT**: I decided to add a few more capabilities to my [Peeve](https://github.com/CuckooEXE/Peeve) project.

## Resources

Made an extensive use of the following resources:

 - [CodeProject - Save and Restore Icon Positions on Desktop](https://www.codeproject.com/Articles/639486/Save-and-Restore-Icon-Positions-on-Desktop)
 - [CodeProject - Stealing Program's Memory](https://www.codeproject.com/Articles/5570/Stealing-Program-s-Memory)
 - [StackOverflow - How do I get the window handle of the desktop?](https://stackoverflow.com/a/5691808/10280970)
