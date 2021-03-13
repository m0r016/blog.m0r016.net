---
title: Fallout4のCTDの原因を突き止める
date: 2021-03-12 11:54:00
categories: [Game, PC]
tags: Fallout4
description: "Fallout4のCTDの原因を突き止める"
#thumbnail: "/2021/03/06/install-ammo-ex/ammo-ui-logo.png"
#cover: "/2021/03/06/install-ammo-ex/ammo-ui.jpg"
---

### はじめに
Fallout4でmodを入れているとCTDが起きることがあるため、CTD対策Mod [Buffout4](https://fallout4.2game.info/detail.php?id=47359)を入れ、原因を突き止める。

### 目次
<!-- toc -->

### 1.必要なもの
必須項目に
・本体(1.10.162.0以上)
・[F4SE](https://f4se.silverlock.org/)
・[Address Library for F4SE Plugins](https://fallout4.2game.info/detail.php?id=47327)
・[xSE PluginPreloader F4](https://fallout4.2game.info/detail.php?id=33946)
・[TBB Redistributables](https://www.nexusmods.com/fallout4/mods/47359)
・[Microsoft Visual C++ Redistributable for Visual Studio 2019](https://fallout4.2game.info/jump.php?https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads)
とあるので導入していく。
導入されているmodによっては動かないこともあるので確認してほしい
<!-- more -->

### 導入成功
無事導入に成功した、普通に遊んでいると`/Document/My Games/Fallout4/F4SE下にcrash-log`と生成されるので中身を読んでみる。
```log
Fallout 4 v1.10.163
Buffout 4 v1.20.3

Unhandled exception "EXCEPTION_ACCESS_VIOLATION" at 0x7FF710CC74CD Fallout4.exe+1CD74CD

SETTINGS:
	[Compatibility]
		F4EE: false
	[Fixes]
		ActorIsHostileToActor: true
		CellInit: true
		EncounterZoneReset: true
		GreyMovies: true
		MovementPlanner: true
		PackageAllocateLocation: true
		SafeExit: true
		UnalignedLoad: true
		UtilityShader: true
	[Patches]
		Achievements: true
		BSTextureStreamerLocalHeap: true
		HavokMemorySystem: true
		MaxStdIO: -1
		MemoryManager: true
		MemoryManagerDebug: false
		ScaleformAllocator: true
		SmallBlockAllocator: true
		WorkshopMenu: true
	[Warnings]
		CreateTexture2D: true
		ImageSpaceAdapter: true

SYSTEM SPECS:
	OS: Microsoft Windows 10 Pro v10.0.19041
	CPU: GenuineIntel Intel(R) Core(TM) i7-3770K CPU @ 3.50GHz
	GPU #1: Nvidia GP107 [GeForce GTX 1050 Ti]
	GPU #2: Intel Xeon E3-1200 v2/3rd Gen Core processor Graphics Controller
	GPU #3: Microsoft Microsoft Basic Render Driver
	PHYSICAL MEMORY: 12.07 GB/14.95 GB

PROBABLE CALL STACK:
	[0] 0x7FF710CC74CD Fallout4.exe+1CD74CD -> 83066+0x2D
	[1] 0x7FF710CC60E5 Fallout4.exe+1CD60E5 -> 503292+0x45
	[2] 0x7FF70F3B7BCA Fallout4.exe+03C7BCA -> 310686+0x3A
	[3] 0x7FF710B40F26 Fallout4.exe+1B50F26 -> 329005+0xC6
	[4] 0x7FF710B41432 Fallout4.exe+1B51432 -> 194800+0x202
	[5] 0x7FF710B43F67 Fallout4.exe+1B53F67 -> 1492866+0x67
	[6] 0x7FF710B0CFED Fallout4.exe+1B1CFED -> 1079791+0x3D
	[7] 0x7FFF3E587034 KERNEL32.DLL+0017034
	[8] 0x7FFF3FF3D241    ntdll.dll+004D241
…
```
とあるが、4行目の`Unhandled exception "EXCEPTION_ACCESS_VIOLATION" at 0x7FF710CC74CD Fallout4.exe+1CD74CD`や41行目の`PROBABLE CALL STACK:`以下が原因となっているらしい。
この場合、1CD74CDが真っ先に引っかかるのでロードオーダーを見てみる。すると以外にも装備modが出てきた。
抜いてみると、確かにCTDが減っていて、突然ゲームが落ちてしまうなどというストレスが減った。

## おわりに
このほかにもCTDについては様々な原因があるが、modの依存によってCTDすることは少なくとも減るため、参考にしてほしい。
他にも何か情報があれば、コメント欄にて是非。